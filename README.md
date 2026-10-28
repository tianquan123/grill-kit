# grill-kit

一组给 AI 编码代理用的中文技能包：定档、拷问、成文、实施、验证、评审，全流程固化成可复用的 `SKILL.md`。

软件代理不缺写代码的能力，缺的是**动手之前把话说清楚**的纪律：这次改动值不值得做、接缝（可替换的边界）划在哪、验收标准是什么、凭什么说做完了。grill-kit 把这套纪律拆成 15 个技能，每个技能一个目录、一份 `SKILL.md`，任何支持 skills 的代理都能直接读。

主链是一条签字流水线，每一环的产出物是下一环的输入，每一道关卡都要人点头：

```text
use-grill ──► grilling ──► to-spec ──► to-plan ──► implement ──► verification ──► code-review
   定档          拷问        设计文档      实施计划      逐任务实施        证据           三条线
```

局部档与试验档不跑完整主链，链长见 [use-grill](use-grill/SKILL.md) 的「编排」表。

`discuss-mode` 与 `constraints` 不在这条链上，它们是贯穿全线的**横切两层**：前者是**输出协议**，链上每一轮按它的格式说话，`use-grill` 强制依赖它——开工时必须先加载；后者是**质量底线**，即这个仓库「绿」的定义，链上每个「声称通过」的点都以它为准，`use-grill` 开工时会先读它（存在的话）。

贯穿全套技能的一条底线：**随任务简单而缩水的是产出物，永远不是批准。**

仓库名取自这条链上最有个性的那一环 `grilling`——在你写下第一行代码之前，先被无情拷问一遍。

## 目录

- [安装](#安装)
- [使用](#使用)
- [技能一览](#技能一览)
- [范围外（刻意不做）](#范围外刻意不做)
- [目录结构](#目录结构)
- [维护者](#维护者)
- [贡献](#贡献)
- [许可](#许可)

## 安装

```bash
git clone https://github.com/tianquan123/grill-kit.git

# 最小入口：use-grill 硬依赖 discuss-mode，grilling 是它的下一步；写代码的局部档还需要 ponytail
cp -r grill-kit/use-grill grill-kit/discuss-mode grill-kit/grilling grill-kit/ponytail ~/.claude/skills/   # Claude Code
cp -r grill-kit/use-grill grill-kit/discuss-mode grill-kit/grilling grill-kit/ponytail "$DSH_HOME/skills/" # DSH

# 完整主链：15 个技能目录一起复制
cp -r grill-kit/{use-grill,grilling,discuss-mode,to-spec,to-plan,implement,ponytail,verification,code-review,constraints,tdd,debugging,domain-modeling,codebase-design,spring-boot-conventions} ~/.claude/skills/
```

无构建依赖：技能是纯 Markdown 加 YAML front matter，不需要构建，也不需要运行时。但技能之间有硬依赖——`use-grill` 依赖 `discuss-mode`，架构档还会依次加载 `to-spec`、`to-plan`、`implement`；写代码前，`use-grill` 会加载配套技能 `ponytail`，后端任务还会加载 `spring-boot-conventions`，`implement` 依赖这些预载。

## 使用

大多数代理会读 `description` 自行判断该不该触发，也可以指名调用（各代理语法不同，Claude Code 是 `/技能名`）：

```text
> 帮我给订单模块加个导出 CSV 的功能
【命中 use-grill】先定档：这是局部档，三句话设计 + 等你点头，再动手。

> 先别动手，帮我 grill 一下：我打算用事件溯源重写订单模块
【命中 grilling】把这棵决策树一批一批地问完当前可问的问题，一轮内问完当前可问的全部问题，每题附推荐答案。

> 从 main 起评审我这个分支
【命中 code-review】按深度跑 Standards、Spec、质审中的两或三条线，各配一个并行子代理，最后并排汇报。

> 测试挂了，你看看怎么修
【命中 debugging】先找根因，根因没落地之前不给修复方案。

> 用最少的代码把这个功能实现了
【命中 ponytail】先按顺序判断：已有实现？标准库？平台原生？最后才写能跑通的最少代码。

> /discuss-mode
【打开讨论模式】本次会话按结论先行、取舍讲透、不写安抚语输出；说 normal mode 关闭。
```

技能既可以被模型按 `description` 自动选中，也可以由别的技能显式加载——`use-grill` 会按关卡依次加载 `discuss-mode`、`to-spec`、`to-plan`、`implement`，写代码前再加载 `ponytail`，后端任务还会加载 `spring-boot-conventions`。哪一步必须停下来等人，由 `use-grill` 的关卡决定，而不是靠技能本身拒绝加载。

## 技能一览

| 技能 | 作用 | 什么时候用 |
|---|---|---|
| [use-grill](use-grill/SKILL.md) | 开工先定档（试验档 / 局部 / 架构），再决定这条流程走多长，并取得开工批准 | 要新增 / 修改 / 重构代码；说「开工」「开始做」时（报错、纯评审、纯讨论走对应技能） |
| [grilling](grilling/SKILL.md) | 把方案当决策树，一轮问完当前可问的问题，无情拷问直到达成共识 | 想压测自己的思路，或说出任何 "grill" 类触发词 |
| [to-spec](to-spec/SKILL.md) | 把谈定的内容固化成设计文档，写进 `docs/specs/`：意图、取舍、验收清单、范围外 | 访谈结束、准备冻结意图时 |
| [to-plan](to-plan/SKILL.md) | 把设计文档变成带状态字段的实施计划，写进 `docs/plans/` | 设计已签字、准备排任务时 |
| [implement](implement/SKILL.md) | 按实施计划逐任务实现：当前分支、当前会话、不提交，进度就地更新在计划文件里 | 计划已签字，开始写代码时 |
| [verification](verification/SKILL.md) | 证据先于断言：没有本次实际跑出来的证据，不许说完成 / 修好 / 通过 | 声称完成之前；验证条件不允许时走显式降级路径 |
| [constraints](constraints/SKILL.md) | 把这个仓库的「绿」写成 `CONSTRAINTS.md`（命令 + 底线），并在声称通过前核对这次 diff 有没有放松它 | 仓库没有成文质量门时；要跑构建 / 测试 / 静态检查时；想改测试或阈值让它变绿时 |
| [code-review](code-review/SKILL.md) | 对一次改动按固定点做评审：Standards（是否合规范）、Spec（是否符合意图，简单改动可省）与质审（代码质量与风险）三条线按深度取舍，各配一个并行子代理，发现按 CRITICAL→INFO 分档 | 评审分支、PR、进行中的改动，或「从 X 起评审」 |
| [tdd](tdd/SKILL.md) | 红 → 绿循环的参考：什么算值得留下的测试、放哪里、有哪些反模式；重构另走独立任务；与 `ponytail` 的分工：它定验证规模、`tdd` 定红→绿节奏 | 想测试先行构建功能或修 bug，提到 "red-green" 时 |
| [debugging](debugging/SKILL.md) | 先找根因，后谈修复；症状消失不等于问题解决 | 遇到报错、测试失败、行为不符预期，在提出任何修复之前 |
| [domain-modeling](domain-modeling/SKILL.md) | 主动构建并打磨项目的领域模型，维护 `CONTEXT.md` 与 ADR | 讨论代码库术语、记录或修改 ADR 时 |
| [codebase-design](codebase-design/SKILL.md) | 深模块（接口小、内部实现多）的设计词汇：找准接缝（可替换的边界），小接口藏大行为 | 设计或改进模块接口、寻找深化机会时 |
| [spring-boot-conventions](spring-boot-conventions/SKILL.md) | Spring Boot 后端的职责归属、分层与拆分：先读邻居，再判断 Controller / Service / 持久层 / Client 的放置位置；写路径默认分步编排、读路径由 QueryService 取数并组装 VO | 新增或修改 Spring Boot 后端代码时；`use-grill` 在写设计前按项目情况加载 |
| [ponytail](ponytail/SKILL.md) | 极简实现：按「需不需要做 → 已有 → 标准库 → 平台原生 → 已装依赖 → 一行」的顺序逐级判断，只写能跑通的最少代码 | 要写代码时；`use-grill` 在进入实现前加载（试验档除外） |
| [discuss-mode](discuss-mode/SKILL.md) | 讨论与汇报的输出协议：结论先行、事实/推断分清、方案给取舍、风险直说、只留有用的信息 | 想砍掉填充、客套和跑题时；说 `/discuss-mode` 开启（模式开关） |

`tdd` / `debugging` / `domain-modeling` / `codebase-design` 是配套技能，在对应环节被主链调用，也可以单独使用；`spring-boot-conventions` 同理，由 `use-grill` 在写设计前按项目情况加载。`ponytail` 也是配套技能，由 `use-grill` 在进入实现前加载（试验档除外）。

## 范围外（刻意不做）

这条链只走到「验证过、评审过」，然后停下报告。下面这些是**刻意**不做的，列在这里是为了不再有人（包括 AI）往里加：

| 不做 | 为什么 |
|---|---|
| 提交、PR、发版、CI 流水线 | 链刻意停在「报告并停」：`implement` 不提交，提交与集成由你自己决定 |
| 可观测性、上线灰度、回滚预案 | 同上——没有部署环节，就没有承载它们的上下文 |
| 前端工程、浏览器验证、性能预算 | 面向后端 / 服务端。需要时再加，而不是先加着占位 |
| 独立的「该用哪个技能」路由技能 | `use-grill` 的定档与分流就是路由，多一个只会多一处不一致 |
| 独立的 git 技能 | 不提交，就没有 git 纪律可讲 |
| 独立的访谈技能 | `grilling` 已经覆盖，而且它的「没有可问的问题」比信心百分比更可核对 |

判断标准只有一条：**它有没有一条现有技能表达不了的机制。** 没有，就折进现有技能，不要新增目录。

## 目录结构

每个技能一个目录，技能名即目录名：

```text
skill-name/
├── SKILL.md            # 技能正文；front matter 里的 name / description 决定何时被触发
├── agents/
│   └── openai.yaml     # 界面信息（display_name / short_description；可选 default_prompt）
└── ...                 # 按需拆分的参考文档，如 tdd/tests.md、codebase-design/DEEPENING.md
```

## 维护者

- [@tianquan123](https://github.com/tianquan123)

## 贡献

- 提问、报错、提需求都走 [Issues](https://github.com/tianquan123/grill-kit/issues)。
- 接受 PR。加新技能时，一个目录一个技能，目录名与 front matter 里的 `name` 保持一致。
- 技能正文保持中文；`description` 要写清「什么时候用」，这是它被触发的唯一依据。
- 动了技能就顺手更新本文件的「技能一览」，仓库里不留破链接。

## 许可

`UNLICENSED`：本仓库尚未声明开源许可，在加入 `LICENSE` 文件之前保留所有权利。

所有者：[tianquan123](https://github.com/tianquan123)。
