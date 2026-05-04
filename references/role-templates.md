# Role Content Templates

Detailed content areas for each of the 7 preset roles. Read the relevant section during Phase 1 (role confirmation) and Phase 4 (content generation).

All content areas listed here should be presented to the user for confirmation/addition before codebase analysis begins (Checkpoint 1).

---

## Universal Content Areas (All Roles)

These two sections appear in every deck regardless of role. Their depth and framing are adapted per audience.

### 行业背景 (Industry Background)

Every deck includes 1-2 slides of industry context so the audience can situate the product. Depth and angle vary by role:

| Role | Industry background focus |
|------|--------------------------|
| Developer | Technical standards, protocols, compliance requirements (e.g., PCI-DSS for fintech, HIPAA for healthcare). Industry-specific tooling and conventions the codebase follows or deviates from. |
| PM | Market landscape — key players, market size/trends, regulatory shifts. Where this product sits in the value chain. Unmet needs the product addresses. |
| Operations | User behavior trends in this domain. Industry benchmarks for engagement, retention, conversion. Seasonal or cyclical patterns visible in the product's data model. |
| Sales | Industry pain points that resonate with buyers. Market trends that create urgency. Regulatory or compliance selling angles. |
| End User | Why this industry has the problem the product solves. What alternatives exist in the market. How the product improves on traditional approaches. |
| Partner/Reseller | Industry growth indicators. Vertical-specific demand signals. Regulatory requirements that create partner opportunities (e.g., compliance consulting, integration services). |
| BD | Market ecosystem map — who are the players, what roles they fill. Partnership landscape in this vertical. Industry consolidation or fragmentation trends. |

**Source**: Primarily from codebase analysis (Phase 2 Step 8). Augmented with 1-2 rounds of web search if the industry context is not fully clear from code alone.

### 术语表 (Glossary)

Every deck ends with a glossary reference slide (before Summary). Terms are extracted from code (Phase 2 Step 9) and explained at the audience's level:

| Role | Glossary style |
|------|---------------|
| Developer | Technical terms: API names, protocol acronyms, design pattern names, internal module abbreviations, error codes. Include code references (file:line) where the term originates. |
| PM | Product-domain terms: feature names, user-facing concepts, business metrics, workflow state names. Bridge between code terminology and user-facing language. |
| Operations | Metric names, event names, funnel stages, configuration parameters. Include what each metric measures and where to find it in the admin/monitoring tools. |
| Sales | Product feature names (as customers hear them), competitive feature names, industry acronyms. Translate internal code names to customer-facing language. |
| End User | Product feature names, UI element names, status labels. Plain-language definitions with no jargon. Include "where to find it" guidance. |
| Partner/Reseller | Technical integration terms, deployment concepts, API terminology, and customer-facing feature names. Dual-purpose: technical enough for integration teams, clear enough for partner sales teams. |
| BD | Market/industry terms, partnership model vocabulary, competitive positioning terms. Standardized industry language that facilitates partner conversations. |

---

## Developer (研发)

**Perspective**: Technical peer who needs to understand the system deeply enough to work on it.

**Content areas**:

1. **产品定位与价值** — What product does this codebase represent? Who is it for? What problem does it solve?
2. **行业背景（技术视角）** — Technical standards, protocols, compliance requirements relevant to this domain. Industry-specific tooling and conventions the codebase follows or deviates from. Always included, not optional.
3. **核心功能清单（技术视角）** — Each feature mapped to its code module: which directory, which files, which entry points
4. **架构与模块设计** — High-level architecture visualization. Use structured HTML diagrams (nested divs with borders/colors) to create: architecture block diagrams showing module relationships, swimlane diagrams showing request flow across layers, or sequence-style flow diagrams showing data flow. Pure HTML/CSS shapes (no SVG, no images) — use colored div blocks with labels and connecting lines to illustrate component relationships. Avoid text-only architecture descriptions when a visual diagram can be built.
5. **技术栈与设计模式** — Languages, frameworks, key libraries. Design patterns observed in the codebase (MVC, CQRS, event-driven, etc.)
6. **入口到服务层的调用链** — For each major feature: route/command/event → controller → service → repository/external API. Show the actual call chain with file paths
7. **特殊业务规则实现** — Complex or non-obvious business logic. Edge cases handled in code. Validation rules that go beyond simple CRUD
8. **编码规范与质量** — Code style observed from actual files. Testing patterns and coverage approach. Linting/formatting configuration
9. **非功能需求实现** — Performance (caching, indexing, pagination), security (auth, encryption, CORS), observability (logging, monitoring, tracing)
10. **技术债务与演进** — TODO/FIXME/HACK comments inventory with severity assessment. Deprecated patterns still in use. Areas where implementation diverges from best practices. Group into categories: "should fix soon" vs "acceptable for now" vs "architectural concern".
11. **系统当前非功能层可优化点** — Concrete optimization opportunities with evidence from code: N+1 queries, missing indexes, synchronous operations that could be async, lack of circuit breakers, etc. For each: what the issue is, where in the code, what improvement would look like.
12. **文档与代码差异** — Compare README/docs claims against actual implementation. For each discrepancy: the doc claim, the actual code behavior, and the source file. Example: "README claims PostgreSQL support, but code only has MySQL adapters (see `src/db/mysql.ts`). Current implementation: MySQL only." If no docs exist, note "无 README/docs，无法对比".

**Conflict rule**: When code and docs disagree, flag both — show the doc claim AND the actual code implementation with file references.

---

## Product Manager (产品经理)

**Perspective**: Product owner who needs the complete picture to make roadmap decisions.

**Content areas**:

1. **产品核心价值与受众** — What fundamental problem does this product solve? For whom? What's the core value proposition?
2. **完整功能点清单与状态** — Every feature identified from code, with implementation status (fully implemented / partial / stub / planned). Include version/edition gating if present
3. **功能演进历史** — From CHANGELOG, git log, or version files: how features evolved over time. Major milestones
4. **用户动线（多角色路径）** — If the product supports multiple user roles, map the journey for each: entry → key actions → outcomes. Extract from route definitions and role/permission logic
5. **端到端业务流程图** — The full business flow from user action to data persistence to response. Extract from controller → service → repository chains
6. **适用/不适用场景** — Based on feature analysis: where this product excels and where it's not the right fit
7. **特殊业务规则** — Non-obvious rules, constraints, or behaviors that a PM should know about. Edge cases from validation logic
8. **需求落地成本预估** — For each major feature area: code complexity indicators (number of files touched, cross-module dependencies) that hint at development cost for changes
9. **竞品功能差异点** — Based on competitive research: feature-by-feature comparison with key competitors
10. **数据埋点与分析线索** — Analytics/tracking events found in code. What user behavior is already instrumented
11. **产品发展方向建议** — End the deck with 3-4 feasible product directions, each with 2-3 concrete feature proposals. Directions should differ in strategy: e.g., "deepen core workflow" vs. "expand to new vertical" vs. "platform/ecosystem play" vs. "enterprise features"

**Conflict rule**: Flag code/doc discrepancies with specific evidence.

---

## Product Operations (产品运营)

**Perspective**: Growth-oriented operator who needs to understand user behavior drivers and leveragable touchpoints.

**Content areas**:

1. **用户行为触点地图** — From routes, events, and UI components: every place where users interact with the product. Map the full touchpoint journey
2. **运营可干预环节** — Features controllable via config, feature flags, or admin panels. A/B test capabilities in the codebase
3. **A/B测试与灰度能力** — Any experimentation framework, feature flag system, or gradual rollout mechanism found in code
4. **用户留存的"啊哈时刻"** — Core value-delivery features identified from the product's primary workflows. What makes users stick
5. **数据埋点与效果监控** — Complete inventory of analytics events, tracking calls, monitoring endpoints. What's measured and what's missing
6. **运营后台与配置项** — Admin panel features, configurable parameters, content management capabilities. What ops can control without developer help
7. **内容运营素材自动化线索** — Any content generation, templating, or personalization features that could feed into content operations
8. **竞品运营策略对比** — From competitive research: how competitors approach growth, retention, and user engagement

---

## Sales (销售)

**Perspective**: Revenue-focused professional who needs to articulate value, handle objections, and close deals.

**Content areas**:

1. **产品核心价值与独特卖点** — What makes this product different and better? Distilled from actual feature implementation
2. **目标客户画像及痛点匹配** — Who is this product for? What pain points does it address? Extracted from the product's target domain and feature set
3. **功能-利益转化表** — Each core feature translated into customer-facing value. Format: "Feature X → Benefit: [specific outcome for customer]"
4. **竞品对比胜出点** — Where this product wins against competitors. Based on code-level evidence (e.g., "supports real-time collaboration via WebSocket" vs competitor's polling approach)
5. **成功案例/场景故事线索** — Typical usage scenarios derived from the product's workflow and user journey code. Not fabricated case studies — realistic scenarios the sales team can develop
6. **常见客户异议与应对依据** — Known limitations with honest framing: performance constraints, feature gaps, integration limitations. Backed by actual code analysis. Structure clearly: separate "objection → response" from "known limitations we disclose proactively". Do NOT overlap these two sections — objections are customer pushback we address; limitations are ours to volunteer honestly.
7. **部署与试用门槛** — How easy is it to get started? Extracted from setup scripts, Docker configs, deployment docs, quickstart guides found in code
8. **报价与套餐线索** — Multi-tier signals from code: feature gates, edition checks, usage limits, enterprise feature flags. Not pricing amounts — structural indicators of how the product could be packaged

---

## End User (终端用户)

**Perspective**: The person who actually uses the product. Needs practical, jargon-free guidance.

**Content areas**:

1. **功能如何解决我的痛点** — Each core feature explained as a solution to a real user problem. No technical jargon
2. **功能使用地图** — Overview of all available features organized by task/user goal. "When you want to do X, use feature Y"
3. **典型使用路径** — Step-by-step walkthrough of the most common tasks. Derived from the primary user workflow in the code
4. **产品边界与"做不了"清单** — Honest list of things the product doesn't do, based on actual feature analysis. Better to set expectations than disappoint
5. **错误处理与自愈指南** — Common error states identified from error handling code. What they mean and what the user should do. Self-service recovery steps
6. **与其他工具的协同方式** — Integration points found in code: APIs, webhooks, export formats, import capabilities
7. **能力边界认知** — Performance characteristics, data limits, concurrent usage constraints extracted from code (rate limits, pagination, caching behavior)

---

## Partner/Reseller (代理商/合作伙伴, includes 渠道商)

**Perspective**: Business partner who needs to understand the product's market potential and sell/resell it to their own clients.

**Content areas**:

1. **产品解决的客户普遍痛点** — What universal customer problems does this product address? Synthesized from the product's feature set and target domain
2. **典型客户画像与行业适用性** — Who benefits most? Which industries? Extracted from the product's domain logic and feature specialization
3. **核心卖点与价值主张** — Key selling points with evidence. Not marketing fluff — backed by actual implementation capabilities
4. **市场前景与机会** — Market size, growth trends, industry tailwinds. Where this product category is heading. Why partners should invest now. Based on web research for market data
5. **竞品功能对比（代码级实锤）** — Detailed feature comparison with competitors, where this product's capabilities are verified against actual code. E.g., "Supports SSO via SAML 2.0 and OIDC (see `auth/saml.ts`, `auth/oidc.ts`)" vs competitor's claimed support
6. **合作模式与收益** — How partners can engage (resell, refer, co-sell, integrate). Revenue sharing models from competitive research. What makes this product's partnership attractive vs. competitors'
7. **销售赋能素材线索** — Information partners can use in their own sales materials: feature highlights, differentiation points, technical specifications

---

## Business Development (商务)

**Perspective**: Deal-maker focused on partnership structures, go-to-market strategies, and competitive positioning in the ecosystem.

**Content areas**:

1. **产品核心价值主张** — What unique value does this product bring to potential partnerships? What's the hook for a BD conversation
2. **目标市场与客户画像** — Market segment analysis from the product's feature set and domain
3. **核心卖点与差异化** — What sets this product apart in a partnership context
4. **竞品合作方式对比** — From web research: how do competitors structure their partnerships? API partner programs, reseller agreements, integration partnerships, co-marketing. Must be based on actual search results — if not found, state honestly
5. **竞品功能对比（正向负向口碑）** — Competitive feature comparison including user sentiment: what users praise and complain about for each competitor
6. **合作模式建议** — Based on the product's architecture and competitive landscape: suggested partnership models (API integration, white-label, referral, co-selling, marketplace listing)
7. **集成与对接能力** — API surface, webhook support, SDK availability, integration endpoints — the technical foundation for partnerships
8. **市场定位与机会** — Where does this product fit in the market ecosystem? What partnership opportunities exist based on competitive gaps

---

## Custom Role Inference Guidelines

When the user names a role not in the 7 presets, follow this process:

1. **Classify the role** into one of these categories:
   - **Internal technical** (architect, SRE, DevOps, QA) → lean toward Developer template, adjust depth
   - **Internal business** (executive, finance, legal, HR) → lean toward PM/BD template, emphasize business impact
   - **External technical** (auditor, consultant, integrator) → lean toward Developer template, add compliance/integration focus
   - **External business** (investor, analyst, media) → lean toward Sales/BD template, emphasize market positioning
   - **End consumer variant** (specific industry, demographic) → lean toward End User template, add domain-specific context

2. **Generate focus areas**: Pick 5-8 from the relevant template above, then adjust for the role's specific concerns.

3. **Present and confirm**: Show the inferred focus areas and ask the user to confirm, add, or remove items.
