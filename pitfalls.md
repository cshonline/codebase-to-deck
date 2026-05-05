# 踩坑记录

## PIT-001: Inline SVG 导致 PPTX 导出崩溃
- **日期**：2026-05-04
- **场景**：Developer 角色 cover slide 中使用了 inline `<svg>` 元素展示 logo
- **现象**：export_deck_pptx.mjs 报错 `TypeError: el.className.includes is not a function`，该 slide 转换失败
- **根因**：html2pptx.js 遍历 DOM 元素时调用 `el.className.includes()`，但 SVG 元素的 `className` 是 `SVGAnimatedString`（不是普通 string），没有 `includes` 方法
- **修复**：在 pptx-constraints.md 新增 Rule 6（禁止 inline SVG，用 `<img>` 引用 SVG 文件替代），同时在 SKILL.md Phase 5 约束列表中注明。将 SVG 文件保存到 `images/` 目录，通过 `<img src="../images/favicon.svg">` 引用
- **代价**：迭代2 Developer 覆盖导出少1张 slide，手动修复后重新导出

## PIT-002: Subagent Bash 权限受限
- **日期**：2026-05-04
- **场景**：所有 8 个测试 subagent 在尝试执行 `node`、`npm install` 等 bash 命令时被权限系统拒绝
- **现象**：Agent 在生成 HTML slides 后 stall 600 秒超时，无法完成 PPTX 导出
- **根因**：Subagent 不继承主 session 的 bash 权限，导致 node/npm 命令全部被拒
- **修复**：主 session 预装依赖后，由主 session 执行 PPTX 导出。Skill 中增加说明：如遇 bash 权限限制，先生成 HTML slides，导出由主 session 完成
- **代价**：每轮测试需要人工补导出步骤

## PIT-003: 并发 8 agent 触发 API 限流
- **日期**：2026-05-04
- **场景**：Iteration-1 同时启动 8 个 subagent（4 with_skill + 4 without_skill）
- **现象**：1 个 agent 收到 429 速率限制，其余因权限问题 stall
- **修复**：Iteration-2 改为分两批（每批 2 个 with_skill），避免并发过多
- **代价**：总测试时间延长（串行代替并行）

## PIT-004: SVG filter effects 导致 PPTX 中 favicon 出现可见边框
- **日期**：2026-05-04
- **场景**：cover slide 使用 `<img src="../images/favicon.svg">` 引用包含复杂 filter effects 的 favicon
- **现象**：PPTX 中 favicon 周围出现可见色块/边框，原始 SVG 在浏览器中显示正常
- **根因**：SVG 包含 15 个 `feGaussianBlur`/`feFlood`/`feBlend` 滤镜，效果延伸到 viewBox 边界外。PptxGenJS 光栅化 SVG 时无法正确处理这些 filter，产生溢出边框。透明背景也可能丢失
- **修复**：在 SKILL.md Phase 2 Step 7 增加 SVG 预处理规则——含 filter effects 的 SVG 需提取主路径（跳过 mask/filter/defs），用 sharp 转为透明背景 PNG 后引用。简单 SVG（仅 path/circle/rect/polygon）可原样使用
- **验证**：简化版 PNG（51% 透明像素）在 PPTX 深色/浅色背景上均显示正确
- **代价**：需要额外 SVG 分析 + sharp 转换步骤，但保证了所有 slide 中 logo 的正确渲染

## PIT-005: 中文标题在 PPTX 中换行（Rule 10 对中文不够保守）
- **日期**：2026-05-04
- **场景**：Iteration-3 Partner cover "Tobee TTRP System v1.0 | 2026" 和 Investor 标题 "企业协作 SaaS 赛道 — 市场趋势与机会窗口" 中 "口" 字被折行
- **现象**：HTML 中中文标题一行显示正常，PPTX 中最后一个字折到第二行
- **根因**：Rule 10 原定 85-90% 容器宽度对英文够用，但中文字符比拉丁字符宽很多，特别是包含 em-dash（—）和竖线（|）的标题。85-90% 宽度对中文标题不够
- **修复**：在 pptx-constraints.md Rule 10 和 SKILL.md Phase 5 中将中文标题的容器宽度要求加严到 75-80%。同时建议缩短中文标题，去掉 em-dash，拆为标题+副标题
- **代价**：中文标题需更短，但避免了 PPTX 中的折行问题

## PIT-006: Developer 角色缺少行业背景和 doc/code 差异分析
- **日期**：2026-05-04
- **场景**：Iteration-3 Developer deck 未包含行业背景 slide，也未做 README vs code 差异说明
- **现象**：用户反馈"缺行业背景介绍"和"代码层和 README 的差异未做说明"
- **根因**：role-templates.md 中 Developer 的 content areas 列表中没有显式包含行业背景和 doc/code 差异，虽然 Universal Content Areas 提到所有 role 都应有行业背景，但 Developer 的具体列表中遗漏了
- **修复**：在 role-templates.md Developer 内容列表中新增 #2 行业背景（技术视角）和 #12 文档与代码差异两个 content area。架构与模块设计从 #3 改为 #4，新增可视化图表指引（用 HTML div 构建架构图/泳道图/流程图）
- **代价**：Developer deck 可能增加 2-3 张 slide，但内容完整性大幅提升

## PIT-007: Sales 角色"异议与应对"区块内容重叠
- **日期**：2026-05-04
- **场景**：Sales deck 中 "常见客户异议与应对依据" 的 "依据来源" 和 "诚实告知的已知限制" 区块有部分重叠
- **现象**：用户反馈两个区块内容边界不清
- **修复**：在 role-templates.md Sales #6 中明确区分：objections 是客户可能提出的质疑及应对策略；limitations 是我们主动坦诚的已知局限。两个区块不应重叠
- **代价**：无

## PIT-008: Audience Advocate 角色定位模糊，改为自定义角色更合理
- **日期**：2026-05-04
- **场景**：Audience Advocate 作为第 8 个 preset role，但其本质是"以特定受众视角组织内容"的方法论，而非一个独立的角色
- **现象**：用户要求从 preset 中去掉 Audience Advocate。如果用户需要"让不懂技术的人也能看懂"，应该走 Custom Role 流程，从 End User 或其他模板推断
- **修复**：从 SKILL.md 和 role-templates.md 中移除 Audience Advocate preset role（7→7 presets）。通用内容质量循环保留，所有角色都有。触发词如"让XX角色能看懂"走 Custom Role 推断
- **代价**：无

## PIT-009: 非终端用户角色的行业背景和痛点应前置
- **日期**：2026-05-04
- **场景**：原来所有角色的 slide 顺序是 Cover → Product Positioning → Industry Background → Value。但非终端用户角色（Sales, BD, Partner, Developer, PM, Operations）需要先理解行业和痛点才能理解产品定位
- **现象**：用户反馈"目标对象如果不是终端用户/客户，需要有行业背景和用户痛点，并且需要前置到产品定位和价值之前"
- **修复**：将 slide structure 改为双轨制：非 End-User 角色先 Cover → Industry Background & Pain Points → Product Positioning → Value；End User 角色保持 Cover → Positioning → Value（因为用户自己就是痛点的承受者）
- **代价**：无

## PIT-010: huashu-design 硬编码绝对路径导致可移植性差
- **日期**：2026-05-04
- **场景**：Phase 7 PPTX 导出依赖 huashu-design 的 export_deck_pptx.mjs 脚本，但 SKILL.md 中硬编码了绝对路径 `/<path_to_user_folder>/.agents/skills/huashu-design/scripts/export_deck_pptx.mjs`
- **现象**：用户指出三个问题：(1) 绝对路径不可移植到其他机器 (2) 没有 huashu-design 时整个 skill 不可用 (3) 缺少 HTML fallback 输出
- **根因**：初期设计时将 huashu-design 作为硬依赖，未考虑安装场景差异
- **修复**：改为 optional dependency 模式——Phase 7 分两路径：Path A（huashu-design 已安装 → `find` 动态发现脚本路径 → PPTX 导出），Path B（未安装 → 纯 HTML 输出 + 安装建议 + 用户同意后自动安装）。新增 `assets/deck_index.html` 作为 bundled HTML viewer（基于 huashu-design 的 deck_index.html 模板，canvas 尺寸适配 960×540）。Bundle Reference 表移除硬编码路径，改为运行时发现说明
- **代价**：无 PPTX 导出能力的用户只能得到 HTML 输出，但核心功能（slide 生成、内容质量循环）不受影响

## PIT-011: position: absolute 导致中文文本换行后元素重叠
- **日期**：2026-05-04
- **场景**：iteration-5 的 5 个 eval 中，3 个出现 slide 内容重叠（html_no_overflow 和 html_bottom_margin_safe 失败）
- **现象**：Developer deck 多张 slide 表格/列表与下方区块重叠；Sales deck 功能-利益转化表底部引用与内容重叠；Partner/Investor deck 同样存在重叠
- **根因**：所有 slide 使用 `position: absolute` + 硬编码 `top` 值定位各内容块。中文文本换行后实际高度超出预期，但后续块的 `top` 值不会自动调整，导致重叠。例如 Developer 的 db-schema slide：左列表 4 项从 top:100pt 开始，下一行从 top:310pt 开始，但 4 条长描述换行后已超过 210pt 高度
- **修复**：在 `references/pptx-constraints.md` 新增 Rule 11——内容块使用 flow layout（一个 absolute wrapper 包裹，内部用 flex/margin/padding 排版），禁止对可换行内容使用 absolute + hardcoded top。同时在 SKILL.md Phase 6 约束列表中增加布局规则引用。将模板示例从多个 absolute 块改为单 wrapper + flow layout 模式
- **代价**：布局灵活性略降（所有内容必须线性排列或用 flex 分栏），但彻底消除重叠问题

## PIT-012: `<h3>` + `display: inline` 在 `<li>` 内导致 PPTX 双重渲染
- **日期**：2026-05-04
- **场景**：Developer deck 的 product-positioning slide 和 glossary slide 使用 `<h3 style="display: inline;">` / `<h2 style="display: inline;">` 实现彩色内联文本
- **现象**：html2pptx.js 将 `<h3>` 同时作为独立文本元素处理（check 4b 对 H1-H6 生效）和作为 `<li>` 子元素处理，导致内容重复或丢失。`parseInlineFormatting` 只识别 `<b>/<strong>/<i>/<em>/<u>/<span>`，不识别 `<h3>`
- **根因**：Heading 标签在 html2pptx.js 中是"文本元素"，会被独立处理，不是合法的内联格式化元素
- **修复**：在 `references/pptx-constraints.md` 新增 Rule 13——内联彩色文本必须用 `<span>` + `font-weight: 700`，禁止用 `<h*>` + `display: inline`。Glossary 中 `<h2>` + `<p>` 兄弟 inline 改为单个 `<p>` + `<span>`
- **代价**：失去 heading 标签的语义，但 PPTX 不使用 HTML 语义

## PIT-013: `flex-wrap: wrap` 导致 call chain 箭头在 PPTX 中折行错位
- **日期**：2026-05-04
- **场景**：Developer deck 的 call-chain slide 使用 `display: flex; flex-wrap: wrap; gap: 6pt` 展示 API 调用链，pill 形状 `<div>` 之间用 `<p>-></p>` 作为箭头
- **现象**：HTML 中箭头和 pill 自然流动，PPTX 中箭头全部错位折行
- **根因**：PPTX 没有 flex 布局。html2pptx 将每个 pill `<div>` 和箭头 `<p>` 转为独立的 PPTX 元素，用浏览器计算的 `getBoundingClientRect()` 定位。但 PPTX 中文本比 HTML 宽 5-10%，pill 内部文本变宽后撑大 pill，导致后续元素溢出到下一行。每个独立元素的位置是基于 HTML 布局计算的，不适应 PPTX 的文本宽度变化
- **修复**：在 `references/pptx-constraints.md` 新增 Rule 12——调用链/流程图使用单个 `<p>` + `<span>` 内联格式化，不用 flex-wrap。每个调用链变为一个 PPTX 文本元素，彻底消除 flex 折行问题
- **代价**：失去 pill 形状的彩色背景，但彩色文本足以区分各步骤

## PIT-014: `system-ui` 字体导致 PPTX 使用完全不同的字体渲染
- **日期**：2026-05-04
- **场景**：所有 slide 的 body 使用 `font-family: system-ui, -apple-system, "PingFang SC", sans-serif`
- **现象**：HTML 预览中文字宽度正常，PPTX 中中文字符明显更宽，大量文本意外换行。Rule 10 的 85-90%/75-80% 宽度补偿无法覆盖此差异
- **根因**：html2pptx.js 用 `computed.fontFamily.split(',')[0]` 提取字体名。`system-ui` 不是合法的 PowerPoint 字体名，PPTX 回退到 Calibri/SimSun。PingFang SC（HTML）和 Calibri（PPTX）的 CJK 字符宽度差异可达 30-50%，远超 Rule 10 假设的 5-10%
- **修复**：在 `references/pptx-constraints.md` 新增 Rule 14——font-family 必须使用显式字体名 `"PingFang SC", "Microsoft YaHei", Arial, sans-serif`，禁止使用泛型值 `system-ui`、`-apple-system`。更新模板示例和 SKILL.md Phase 6 约束列表
- **代价**：无。PingFang SC 在 macOS 上显示效果与 system-ui（实际也是 PingFang SC）一致，但 PPTX 现在能正确使用 PingFang SC 而非 Calibri

## PIT-015: README 中的产品名覆盖了 UI 中提取的真实产品名
- **日期**：2026-05-05
- **场景**：TTRP-System 项目中，UI 页面显示"拓蜂工作台"，但生成的 deck 全部使用"TTRP System"（来自 README 标题）
- **现象**：所有 slide 的 Cover 和标题页都显示"TTRP System"，而非用户在 UI 中看到的"拓蜂工作台"
- **根因**：SKILL.md 的 Fact Priority Rules（UI > Code > Doc）定义正确，但 Phase 2 的 10 个步骤中没有显式的"从 UI 提取产品名"步骤。Step 10 读 README 时拿到 "TTRP System"，因无更早锁定的 UI-extracted name，直接采用
- **修复**：在 Phase 2 Step 1 之后新增 Step 1.5（Product name extraction），明确要求按 UI templates > Config/manifest > README/docs 的优先级提取产品名并锁定为 `productName`。Step 10 加注"不得用 README 名称覆盖已提取的 productName"
- **代价**：用户反馈后手动修复，影响所有已生成的 deck