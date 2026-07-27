# Asterflow 融合架构调研

调研日期：2026-07-26

本报告只使用项目 README、官方文档、ADR、Skill 定义和源码入口等一手资料。
除本报告外，调研没有修改任何上游项目，也没有读取 MyOutBrain 私人实例的
Vault、SQLite、对象存储或生成视图。

## 1. 调研边界

### 已确认融合对象

| 对象 | 调研版本 | 在 Asterflow 中的预期作用 |
|---|---|---|
| oh-my-opencode-slim（OMO-Slim） | `2.2.8`，归档源码来自 `1c0e1f4` | OpenCode 插件基座、Agent 工厂、后台任务控制面、权限与 Hook |
| Matt Pocock Skills | `mattpocock/skills@ed37663` | 可组合、可检测的工程工作流与共享工程词汇 |
| MyOutBrain（MOB） | V2 / 协议 V2.3 / Schema 11，源码 `0734b45` | 跨客户端长期记忆、召回、学习信号、反思与人工审阅 |

OMO-Slim 的版本、仓库地址与 MIT 许可证来自
[`package.json`](../../upstreams/oh-my-opencode-slim/package.json)；其 README
将产品定义为 OpenCode 上的专业 Agent 编排插件
([README L25-L54](https://github.com/alvinunreal/oh-my-opencode-slim/blob/1c0e1f4abe217b6965997201c37ff1de6720c13d/README.md#L25-L54))。
Matt Skills 的调研提交通过官方 GitHub API 固定为 `ed37663`。MyOutBrain 的
版本边界来自其一手
[README L3-L12](https://github.com/Devwallem/MyOutBrain/blob/0734b45acab58c90f0347055fafce1d0ba119d4d/README.md#L3-L12)。

### 尚待用户确认或并非独立融合对象

- **Dream**：目前是 Asterflow 想要的知识整理机制名称，不是已发现的独立本地
  上游项目。它应建立在 MOB 已有的学习信号、Reflector、统一审阅和计划反思
  协议上，而不是再建一套记忆事实源。
- **OpenClaw**：仅用于比较 Dreaming 和人格边界，不属于融合对象。其产品使命
  是本地、常驻、多渠道个人助手，Gateway 只是控制面
  ([官方 README L267-L273](https://github.com/openclaw/openclaw#openclaw--personal-ai-assistant))，
  与面向工程任务的 Asterflow 不同。
- **Asterflow 的 Direct / Grill / 只读导航器、Builder、Verifier、Curator**：
  是目标架构角色，不是可直接 vendoring 的第四个上游。
- **MyOutBrain 的客户端适配器**：是连接同一核心的薄入口，不应被当作独立
  记忆项目。MyOutBrain 明确要求入口重装不能复制或删除规范数据
  ([README L122-L146](https://github.com/Devwallem/MyOutBrain/blob/0734b45acab58c90f0347055fafce1d0ba119d4d/README.md#L122-L146))。

## 2. OMO-Slim

### 2.1 使命与顶层控制流

OMO-Slim 的核心目标是让一个 Orchestrator 按质量、速度和成本，将代码侦察、
外部研究、架构判断、UI 与有界实现分配给不同专业 Agent；Orchestrator 规划
工作图、后台派发并汇总结果
([README L25-L40](https://github.com/alvinunreal/oh-my-opencode-slim/blob/1c0e1f4abe217b6965997201c37ff1de6720c13d/README.md#L25-L40))。

真实运行 Prompt 已经弱化了 README 的“Pantheon”人格叙事。它把 Orchestrator
定义为工作流管理器，而不是默认实现者；非平凡工作依次经过理解、路径选择、
委派检查、短工作图、后台任务纪律、会话复用和窄范围验证
([`orchestrator.ts` L145-L246](https://github.com/alvinunreal/oh-my-opencode-slim/blob/1c0e1f4abe217b6965997201c37ff1de6720c13d/src/agents/orchestrator.ts#L145-L246))。

后台编排文档给出的闭环是：

```text
understand → dependency graph → background specialists
→ terminal results → reconcile → follow-up → verify → respond
```

其中 Orchestrator 仍拥有最终验证责任，专业 Agent 的输出只被视为输入，不是
最终事实
([background-orchestration.md](../../upstreams/oh-my-opencode-slim/docs/background-orchestration.md))。

`src/index.ts` 是插件组合根；它装配配置、Agent、工具、MCP、Hooks、后台 job
board、task session、multiplexer、interview 和 preset 管理。归档源码的
[`codemap.md`](../../upstreams/oh-my-opencode-slim/codemap.md) 与
[`src/index.ts`](../../upstreams/oh-my-opencode-slim/src/index.ts) 是后续拆分
控制面的首要入口。

### 2.2 Agent 与 Skill 机制

OMO-Slim 当前提供以下主要职责：

- Explorer：只读代码库侦察；
- Librarian：外部文档与版本化 API 研究；
- Oracle：架构、高风险调试与按需审查；
- Designer：UI/UX 设计和实现；
- Fixer：输入明确、范围有界的实现；
- Council：多模型判断与综合；
- Observer：隔离主上下文的视觉读取。

这些职责来自 README 的 Agent 清单
([Explorer L275-L310](https://github.com/alvinunreal/oh-my-opencode-slim/blob/1c0e1f4abe217b6965997201c37ff1de6720c13d/README.md#L275-L310)，
[Oracle L316-L350](https://github.com/alvinunreal/oh-my-opencode-slim/blob/1c0e1f4abe217b6965997201c37ff1de6720c13d/README.md#L316-L350)，
[Librarian/Designer/Fixer L406-L523](https://github.com/alvinunreal/oh-my-opencode-slim/blob/1c0e1f4abe217b6965997201c37ff1de6720c13d/README.md#L406-L523))，
而真正可执行的路由契约位于
[`src/agents/orchestrator.ts`](../../upstreams/oh-my-opencode-slim/src/agents/orchestrator.ts)。

OMO 将 Skill 定义为注入 Agent system prompt 的工作流说明，而不是运行进程；
Skill assignment 同时就是权限授予，支持显式列表、`*` 和否定规则
([README L578-L605](https://github.com/alvinunreal/oh-my-opencode-slim/blob/1c0e1f4abe217b6965997201c37ff1de6720c13d/README.md#L578-L605))。
源码通过 `filter-available-skills` Hook 在运行时过滤可见 Skill，因此这里可以
直接承载 Asterflow 的 Agent 能力白名单，而不必另造第二套权限系统
([`filter-available-skills`](../../upstreams/oh-my-opencode-slim/src/hooks/filter-available-skills/index.ts))。

### 2.3 记忆与反思现状

OMO 的 background job board、task session alias 和 checkpoint snapshot 主要
解决**当前会话任务状态与上下文复用**，并不是长期知识事实源
([background-orchestration.md](../../upstreams/oh-my-opencode-slim/docs/background-orchestration.md))。

其 `reflect` Skill 会分析历史 session，并把摘要缓存到 OMO 自己的配置目录；
session mode 甚至直接查询 OpenCode SQLite
([`reflect/SKILL.md` L30-L103](../../upstreams/oh-my-opencode-slim/src/skills/reflect/SKILL.md))。
这与 MOB 的“入口不得直接读取数据库、不得形成客户端专属记忆孤岛”边界冲突，
因此不能原样成为 Dream 的持久层。

### 2.4 可复用组件

- `src/index.ts` 的 OpenCode 插件组合根；
- Agent factory、模型 preset、自定义 Agent 与 per-Agent MCP/Skill 权限；
- native background task、job board、可复用 task session 和取消/回流机制；
- Hook 聚合层和 cache-safe 尾部注入；
- writer scope/文件所有权纪律；
- 最窄有效验证策略；
- multiplexer 与可选 Companion 作为观察面，而不是控制面。

### 2.5 主要融合风险

1. **控制权过宽**：现 Orchestrator 同时计划、路由、汇总和验证；Asterflow 若要
   独立 Verifier 和唯一完成屏障，需要把“验证结论”从 Prompt 习惯提升为显式状态。
2. **默认权限过宽**：README 示例给 Orchestrator `skills: ["*"]`
   ([README L169-L180](https://github.com/alvinunreal/oh-my-opencode-slim/blob/1c0e1f4abe217b6965997201c37ff1de6720c13d/README.md#L169-L180))，
   与严格能力白名单相反。
3. **人格噪声**：Pantheon 叙事可以从用户文档和展示层移除；Agent factory 与
   运行 Prompt 已经主要是职责契约，无需因去人格化而推倒运行架构。
4. **双重记忆**：OMO `reflect --sessions` 的直接数据库读取和私有摘要缓存不能
   与 MOB 并行成为长期记忆。
5. **顶层宏冲突**：`deepwork` 等 Orchestrator 宏可以保留为显式模式，但不能
   自动覆盖 Direct / Grill / Wayfinder 的唯一顶层路由；Scout 只是 Direct
   内部的只读侦察阶段。

## 3. Matt Pocock Skills

### 3.1 使命与调用边界

Matt Skills 明确反对由大型框架“拥有整个过程”，主张 Skill 小型、可组合、
易修改、模型无关
([README L13-L19](https://github.com/mattpocock/skills/blob/ed37663cc5fbef691ddfecd080dff42f7e7e350d/README.md#L13-L19))。

它最值得 Asterflow 继承的不是另一套 Orchestrator，而是调用边界：

- user-invoked Skill 负责显式编排；
- model-invoked Skill 是模型可以按任务自动调用的复用纪律；
- user-invoked Skill 可以调用 model-invoked Skill，但不能嵌套另一个
  user-invoked Skill
  ([README L171-L205](https://github.com/mattpocock/skills/blob/ed37663cc5fbef691ddfecd080dff42f7e7e350d/README.md#L171-L205))。

### 3.2 官方主流程

`ask-matt` 的主流程是：

```text
grill-with-docs
→ 必要时 prototype
→ 单 session implement
  或 multi-session to-spec → to-tickets → fresh implement per issue
→ implement 内部 TDD → 双轴 code-review → commit
```

一手来源是
[`ask-matt/SKILL.md` L13-L31](https://github.com/mattpocock/skills/blob/ed37663cc5fbef691ddfecd080dff42f7e7e350d/skills/engineering/ask-matt/SKILL.md#L13-L31)
与
[`implement/SKILL.md` L7-L15](https://github.com/mattpocock/skills/blob/ed37663cc5fbef691ddfecd080dff42f7e7e350d/skills/engineering/implement/SKILL.md#L7-L15)。

Grilling primitive 要求一次只问一个问题；环境中可查的事实由 Agent 自己查，
真正的决定交还用户，在用户确认共同理解前不得行动
([`grilling/SKILL.md` L6-L12](https://github.com/mattpocock/skills/blob/ed37663cc5fbef691ddfecd080dff42f7e7e350d/skills/productivity/grilling/SKILL.md#L6-L12))。
这适合成为 Asterflow 的 Grill 能力，但顶层路由需要限制它只询问**阻塞当前
可观察结果的最少决定**。

### 3.3 Wayfinder 的重大命名冲突

Matt `/wayfinder` 不是普通的只读代码导航器。它专门处理“超过一个 Agent
session、到目标的道路仍在雾中”的巨大工作，在 issue tracker 上建立
`wayfinder:map` 与 decision tickets；默认产出决定，不产出交付物
([`wayfinder/SKILL.md` L7-L16](https://github.com/mattpocock/skills/blob/ed37663cc5fbef691ddfecd080dff42f7e7e350d/skills/engineering/wayfinder/SKILL.md#L7-L16))。

它会创建、认领和关闭 issue，每个 session 通常只解决一个 decision ticket，
并可为 research ticket 并行启动研究 Agent
([`wayfinder/SKILL.md` L105-L128](https://github.com/mattpocock/skills/blob/ed37663cc5fbef691ddfecd080dff42f7e7e350d/skills/engineering/wayfinder/SKILL.md#L105-L128))。
`ask-matt` 也明确要求只把它用于 huge/foggy effort，地图清晰后交给
`to-spec`，而不是直接实现
([`ask-matt/SKILL.md` L44-L49](https://github.com/mattpocock/skills/blob/ed37663cc5fbef691ddfecd080dff42f7e7e350d/skills/engineering/ask-matt/SKILL.md#L44-L49))。

因此建议：

- 把 Asterflow 日常复杂任务的只读结构探索者命名为 `Scout` 或 `Mapper`；
- 保留 `Wayfinder` 作为用户显式调用、跨 session 的 Matt 决策地图模式；
- 若必须沿用名字，则明确区分 `Scout` 与 `Wayfinder Macro`，不要使用
  `Wayfinder-lite` 这种仍易混淆的命名。

### 3.4 可复用能力与风险

可复用能力：

- Grilling 的“事实自己查、决定问用户、一次一问”；
- TDD 的纵向 red-green-refactor；
- `domain-modeling` 的 ubiquitous language 与 ADR 边界；
- `codebase-design` 的 deep module/seam 词汇；
- spec/ticket 的 tracer-bullet 与 blocking edges；
- code review 的 Standards / Spec 双轴分离；
- research 的一手来源与后台调研产物。

融合风险：

1. Matt 官方流程倾向从 Grill 开始，而 Asterflow 需要让明确、局部、可验证任务
   直接进入 Direct。
2. `implement` 默认继续到完整 code review 和 commit；不能被 Builder 自动宏调用，
   否则会扩大授权和耗时。
3. `code-review` 固定启动两个并行子 Agent
   ([`code-review/SKILL.md` L7-L13](https://github.com/mattpocock/skills/blob/ed37663cc5fbef691ddfecd080dff42f7e7e350d/skills/engineering/code-review/SKILL.md#L7-L13))，
   不应等同于每次任务的最窄 Verifier。
4. Matt Skills 应以固定 commit vendoring 或明确更新策略接入，不能假定某台机器
   的安装副本就是最新事实源。

## 4. MyOutBrain（MOB）

### 4.1 使命与事实源

MyOutBrain 是面向单一创作者、本地优先、跨 Codex/OpenCode/Claude Code 的
长期知识与认知记忆核心。它明确不是当前阶段的完整桌面应用、自主同伴或人格系统
([README L3-L12](https://github.com/Devwallem/MyOutBrain/blob/0734b45acab58c90f0347055fafce1d0ba119d4d/README.md#L3-L12))。

其关键原则是一个私人实例、本地优先、人工拥有语义权、来源可追溯、入口可替换、
紧凑召回与冲突不投票
([README L14-L22](https://github.com/Devwallem/MyOutBrain/blob/0734b45acab58c90f0347055fafce1d0ba119d4d/README.md#L14-L22))。

SQLite 规范状态与内容寻址对象是事实源；Vault 只是可以重建的人类/Obsidian
视图。源码仓库和私人实例必须分离
([README L24-L60](https://github.com/Devwallem/MyOutBrain/blob/0734b45acab58c90f0347055fafce1d0ba119d4d/README.md#L24-L60))。

### 4.2 前台召回与学习闭环

MOB 已经提供 Asterflow 所需的“像 cache 一样”接入 seam：

1. 任务前通过 MCP 获取任务相关的紧凑召回包；
2. 需要核验时才按同一次召回展开证据；
3. 任务结束只识别用户纠正、已确认决定、可复用步骤、重复失败及解决办法、
   值得研究的问题五类明确学习信号；
4. Reflector 把有界输入形成统一审阅提案；
5. 创作者修改、批准、拒绝或延期后，正确意图才物化；
6. 后续入口共享同一 memory ID、版本和审阅状态。

这些行为来自
[README L153-L187](https://github.com/Devwallem/MyOutBrain/blob/0734b45acab58c90f0347055fafce1d0ba119d4d/README.md#L153-L187)
和
[L224-L234](https://github.com/Devwallem/MyOutBrain/blob/0734b45acab58c90f0347055fafce1d0ba119d4d/README.md#L224-L234)。

源码中的 `MemoryGateway` 明确返回 task-scoped memory 而不暴露持久化细节，并
提供 `submit_learning_signal`、有预算的 `reflection_inputs` 与 `reflect_now`
([`memory_gateway.py` L187-L258](https://github.com/Devwallem/MyOutBrain/blob/0734b45acab58c90f0347055fafce1d0ba119d4d/src/myoutbrain/memory_gateway.py#L187-L258))。
`DomainProtocol` 统一协商和分派 recall、learning signal、counterevidence、
review 与 maintenance
([`domain_protocol.py` L186-L245](https://github.com/Devwallem/MyOutBrain/blob/0734b45acab58c90f0347055fafce1d0ba119d4d/src/myoutbrain/domain_protocol.py#L186-L245))。
MCP 只暴露 `myoutbrain_gateway`，并显式禁止入口直访私有存储
([`mcp_server.py` L56-L128](https://github.com/Devwallem/MyOutBrain/blob/0734b45acab58c90f0347055fafce1d0ba119d4d/src/myoutbrain/mcp_server.py#L56-L128))。

### 4.3 Dream 的正确接入位置

MOB 已有 Companion/Reflector 分工与 scheduled reflection 的
enqueue/claim/lease/complete/return/abandon 状态机。Asterflow 的 Dream 不应再
维护一套私有调度和知识表，而应：

- 让前台 Companion 只做紧凑召回和学习信号提交；
- 让 Curator/Dream 领取有界 reflection run；
- 做去重、冲突识别、候选排序和证据校验；
- 只提交 unified-review proposals；
- 永不代表用户批准或直接改写 canonical memory。

MyOutBrain 明确把自动知识批准、自动反证裁决、人格和无人值守模型调用列为非目标
([README L239-L251](https://github.com/Devwallem/MyOutBrain/blob/0734b45acab58c90f0347055fafce1d0ba119d4d/README.md#L239-L251))，
并禁止绕过 MCP/CLI 直接读取私有存储
([README L269-L275](https://github.com/Devwallem/MyOutBrain/blob/0734b45acab58c90f0347055fafce1d0ba119d4d/README.md#L269-L275))。

### 4.4 许可证风险

MyOutBrain 当前仓库明确写明尚未包含开源许可证，不能假定其代码可公开复制、
修改或再分发
([README L277-L280](https://github.com/Devwallem/MyOutBrain/blob/0734b45acab58c90f0347055fafce1d0ba119d4d/README.md#L277-L280))。
因此现阶段推荐通过 MCP/CLI 领域协议集成；若 Asterflow 未来需要复制
MyOutBrain 实现代码，必须先由仓库所有者明确授权或添加许可证。

## 5. OpenClaw：仅用于 Dreaming 对比

OpenClaw 是常驻、多渠道个人助手，拥有本地 Gateway、每 Agent workspace、
注入式 `AGENTS.md`/`SOUL.md`/`TOOLS.md` 与 workspace skills
([官方 README L339-L347](https://github.com/openclaw/openclaw#highlights)，
[L422-L426](https://github.com/openclaw/openclaw#agent-workspace--skills))。
这些人格、渠道和 always-on 产品能力不进入 Asterflow 的初期边界。

可借鉴的 Dreaming 机制：

- Light 阶段去重和暂存候选，不写长期记忆；
- REM 阶段形成主题与反思，不写长期记忆；
- Deep 阶段按相关度、频率、query diversity、recency 等信号排序；
- 写入前从当前活跃来源 rehydrate，跳过已删除或过期片段；
- `DREAMS.md` 只作为人类审阅日记，明确排除自身回灌；
- shadow trial 只生成 report，不自动晋升。

这些机制来自 OpenClaw 官方
[Dreaming 文档 L43-L88](https://docs.openclaw.ai/concepts/dreaming#phase-model)
与
[L98-L115](https://docs.openclaw.ai/concepts/dreaming#deep-ranking-signals)。

不能照搬的是 Deep 阶段自动追加 `MEMORY.md`
([Dreaming L75-L79](https://docs.openclaw.ai/concepts/dreaming#phase-model))。
这会违反 MOB 的人工语义批准边界。Asterflow 可以复用其**候选排序与可解释性
思想**，但最终输出必须是 MOB 审阅提案，而不是 canonical write。

OpenClaw 的 `memory_search` → `memory_get` 两阶段接口也验证了“先小结果、再按需
展开”的 token 经济性
([Memory overview L106-L121](https://docs.openclaw.ai/concepts/memory#memory-tools))；
但 Asterflow 应使用 MOB 的 task-scoped recall/evidence expansion，而不是再建
一套 Markdown 记忆。

## 6. 建议的 Asterflow 目标架构

### 6.1 唯一控制平面

```text
user task
  ↓
MOB compact recall（仅 substantive task）
  ↓
Router
  ├─ Direct：结果与到达路径足够清楚
  │    └─ 入口/调用链未知时，先运行只读 Scout 子阶段
  ├─ Grill：当前 session 内可走完的阻塞性用户决策
  └─ Wayfinder：有明确 Destination，但路径存在跨 session 决策迷雾
  ↓
Builder / Designer / bounded specialists
  ↓
Verifier completion barrier
  ├─ fail → bounded remediation
  └─ pass → deliver
  ↓
explicit learning-signal check
  ↓
MOB proposal pipeline / scheduled Dream
```

顶层仍只有 Direct / Grill / Wayfinder 三个入口。Router 可以推荐 Wayfinder，
但首次创建持久地图需要用户确认；用户显式运行 `/wayfinder` 或引用已有地图时
视为授权。`Scout` 是 Direct 路线中的
只读侦察阶段，不拥有独立控制流：结构事实查清后必须回到同一个 Direct 任务，
再决定实现或因发现真正的跨 session 迷雾而提议升级到 Wayfinder。这样既保留
三路确定性控制面，也不会把 Matt Wayfinder 错用成普通 Explorer。

只有 Asterflow Orchestrator 拥有流程状态转换权。Matt Skill、OMO specialist 与
MOB Reflector 都是受调用的能力或子流程，不能再各自启动一条顶层生命周期。

### 6.2 Agent 最小集合

| 角色 | 读写边界 | 固定输出 |
|---|---|---|
| Orchestrator | 控制流程；只直接完成微小动作 | 路由、任务图、所有权、验收结果 |
| Scout | 默认只读 | 入口、调用链、必改/勿改位置、最窄验证、剩余用户决定 |
| Builder | 限定写入范围 | 一个可运行纵向切片及验证证据 |
| Designer | 限定 UI 写入范围 | UI/UX 纵向切片及视觉验证 |
| Verifier | 默认只读；不替 Builder 修复 | pass/fail、直接证据、最小失败原因 |
| Curator/Dream | 只经 MOB Gateway 读写提案 | 候选、来源、冲突、盲区、审阅 ID |

OMO 的 Librarian、Oracle、Council、Observer 可保留为按需专业 lane，不必全部
成为顶层阶段。

### 6.3 Skill 策略

- Orchestrator：Router、Grill primitive、显式 Wayfinder Macro；
- Scout：research、domain-modeling 的只读部分、代码探索；
- Builder：TDD、diagnosing-bugs、局部 codebase-design；
- Verifier：项目特定 evidence path；完整双轴 code-review 只在风险合理或用户
  显式要求时启用；
- Curator/Dream：只使用 MOB Companion/Reflector 领域协议。

默认采用 allowlist，禁止 OMO 示例中的 `skills: ["*"]`。user-invoked Skill
不能由模型自动升级调用；任何会创建 issue、commit、PR 或长期知识变更的宏，
都必须保留其原有授权边界。

## 7. 最值得直接复用与明确淘汰的部分

### 直接复用或小幅改造

1. OMO 的插件组合根、Agent factory、配置 schema、后台 task/job board、
   session reuse、Hook 聚合、skill filtering 和 cache-safe injection。
2. Matt 的 Grilling、TDD、diagnosis、domain-modeling、codebase-design、
   tracer-bullet ticket 与双轴 review 纪律，但由 Asterflow 控制何时调用。
3. MOB 的 `myoutbrain_gateway`、版本/能力协商、task-scoped recall、学习信号、
   unified review、scheduled reflection lease 与人工批准。
4. OpenClaw Dreaming 的候选暂存、去重、query diversity、rehydration、
   解释报告和“Dream 输出不得回灌”原则。

### 不应原样融合

1. OMO Pantheon 人格叙事和默认全 Skill 权限；
2. OMO `reflect --sessions` 直接读取 OpenCode SQLite 与客户端专属摘要缓存；
3. Matt 主流程作为每个任务的自动强制链；
4. 把 Matt Wayfinder 改写成日常只读探索；
5. OpenClaw `SOUL.md`、always-on、多渠道个人助手与自动 deep promotion；
6. 任何绕过 MOB Gateway 的 Vault/SQLite/object-store 访问；
7. 任何把“任务完成”“消息很多”或“时间到了”直接当作长期学习信号的机制。

## 8. 下一阶段需要回答的问题

1. Asterflow 是保留 OMO 包结构并逐步替换控制流，还是创建新的 package surface，
   仅从 `upstreams/oh-my-opencode-slim` 搬运已选模块？
2. Router 的确定性输入、阈值与可观察日志格式是什么？
3. Verifier 的 pass/fail 是否进入显式状态机，如何防止 Builder 自证完成？
4. 如何在不破坏 Provider prompt cache byte-prefix 的前提下，把 MOB recall 放到
   payload 尾部？
5. Scout、Wayfinder Macro、Oracle 与 Council 的升级条件如何避免重叠？
6. MOB Gateway 当前各入口需要声明哪些最小 capability，哪些写操作必须携带
   `expected_version` 与 idempotency key？
7. Dream 采用何种候选排序信号，但仍保证没有用户批准就绝不 materialize？
8. MyOutBrain 未授权复制代码的情况下，Asterflow 的发布、安装与测试如何只依赖
   协议契约？

## 9. 调研结论

三套融合对象已经覆盖大多数机械能力：

- OMO-Slim 提供 OpenCode 控制面、Agent、任务状态、权限和 Hook；
- Matt Skills 提供可组合的工程纪律与词汇；
- MOB 提供受治理、跨入口、可追溯的长期记忆。

真正需要新设计的不是更多 Agent，而是**唯一控制权、术语和完成屏障**。最关键的
三个约束是：

1. 日常只读探索不要冒用 Matt `Wayfinder` 的名字和契约；
2. Matt/OMO 的工作流只能作为能力，不能嵌套夺取顶层控制流；
3. Dream 只能生成和排序 MOB 审阅提案，不能绕过人工批准写入规范记忆。
