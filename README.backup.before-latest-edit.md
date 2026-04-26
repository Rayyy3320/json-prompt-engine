# json-prompt-engine

把一次性 prompt，变成可审查、可治理、可复用的 prompt package。

`json-prompt-engine` 是一个面向 AI Agent / Skill 工作流的 prompt 治理引擎：它帮你把复杂、容易失控的自然语言需求，先整理成可审查、可复用、可交付的内部 JSON prompt，再生成最终答案或下游 prompt package。

它不是一个独立 CLI，也不是单纯的 prompt 模板合集。它更像一层治理流程：在真正生成内容之前，先做任务分类、治理强度判断、first principles 分解、prompt review 和 answer review，再决定应该交付什么。

![json-prompt-engine workflow](./1.0/mermaid-diagram.png)

## 快速导航

- [简介](#简介)
- [为什么需要它](#为什么需要它)
- [适合谁](#适合谁)
- [它能产出什么](#它能产出什么)
- [效果示例](#效果示例)
- [核心能力](#核心能力)
- [和普通 prompt 模板有什么不同](#和普通-prompt-模板有什么不同)
- [安装方法](#安装方法)
- [工作原理](#工作原理)
- [为什么可信](#为什么可信)
- [仓库结构](#仓库结构)
- [边界说明](#边界说明)
- [许可证](#许可证)

## 简介

这个仓库当前提供的是一套说明驱动型 skill，而不是一个独立可执行程序。

它的核心不是直接回答问题，而是先把用户请求转换成一个内部任务控制对象，也就是 internal JSON prompt。你可以把它理解成：**模型内部使用的结构化任务说明书**。这个对象会承载目标、约束、假设、角色、审查规则和交付要求，然后再决定最终产出：

- `final_answer`：直接面向用户的最终答案
- `json_prompt`：面向下游模型的结构化 prompt package
- `hybrid_prompt_package`：同时保留结构化字段和编译后的指令字符串

如果你的任务只是一次性简单问答，直接写 prompt 往往更轻；但如果你的 prompt 需要被复用、审计、交给下游模型或沉淀成 skill，这种治理层才开始真正有价值。

## 为什么需要它

当 prompt 只是一次性提问时，写清楚通常就够了。

但当 prompt 变成可复用工作流时，问题往往不在“文案不够长”，而在：

- 任务边界容易越写越散
- 约束散落在自然语言里，难以检查
- 修改 prompt 时缺少治理规则
- 生成前没有 `prompt_review`
- 生成后没有 `answer_review`
- 下游模型拿到的信息过多，既冗余又不稳定

`json-prompt-engine` 的目标，不是把 prompt 写得更华丽，而是把这些隐性控制逻辑显式化：先结构化，再生成；先审查，再交付。

## 适合谁

这个 skill 更适合：

- 经常需要审计、升级、复用 prompt 的用户
- 正在维护 AI Agent / Skill 工作流的人
- 需要把自然语言需求交付给下游模型或团队的人
- 希望 prompt 生成过程可检查、可追踪、可约束的人

它不太适合：

- 只想要一次性聊天回复的简单任务
- 不需要结构化交付的轻量问答
- 只想要一个现成成品 prompt、不关心治理过程的人

## 它能产出什么

当前仓库明确支持三类交付物：

| 交付物 | 适合场景 | 说明 |
|---|---|---|
| `final_answer` | 直接回复用户 | 保留 review 后的自然语言交付 |
| `json_prompt` | 交给下游模型继续执行 | 返回 reviewed downstream JSON prompt package |
| `hybrid_prompt_package` | 同时需要结构化字段与兼容指令 | 返回 delivery payload，并在需要时附带 `compiled_instruction` |

## 效果示例

下面这个示例只使用仓库中真实存在的流程和字段，不引入未实现能力。

用户输入：

> 帮我优化这个 prompt，让它更适合生成产品立项文档，并输出可复用的 hybrid prompt package。

`json-prompt-engine` 不会直接开始润色，而是会先判断：

- 这不是普通写作，而是现有 prompt 的升级任务
- 应优先路由到 `prompt-upgrade` 这一类正式 slot
- 这类任务通常要求更严格的治理强度
- 对复杂 prompt 升级，应启用 first principles 分解
- 最终交付不只是修改建议，还应包含可复用的 prompt package

因此它更可能先做这些事：

1. 检查目标、格式、受众、约束是否缺失
2. 解析本次任务需要多严格的控制方式，也就是 `control_mode`
3. 起草 `role_spec`、`task_spec`、`structure_constraints`
4. 运行 `prompt_review`
5. 生成 `hybrid_prompt_package`
6. 再运行 `answer_review`

你最终拿到的，不只是“改写后的 prompt”，而是：

- 面向用户的修改建议
- 面向下游模型的结构化交付字段
- 必要时附带的编译指令字符串

## 核心能力

`json-prompt-engine` 的核心不是“把 prompt 写得更像样”，而是让 prompt 生成过程更稳定。

- **任务分类**：先识别任务类型，再决定进入哪个 slot
- **治理强度解析**：用 `control_mode` 判断这次任务需要多严格的控制方式
- **first principles 分解**：对复杂任务先拆问题，再写 prompt
- **结构化控制层**：显式起草 `role_spec`、`task_spec`、`structure_constraints`
- **生成前审查**：`prompt_review` 先检查 prompt 本身是否够清楚、够稳定
- **生成后审查**：`answer_review` 再检查结果是否跑偏、遗漏或虚假自信
- **精简交付**：默认只向下游交付必要字段，也就是 slim payload，而不是整包暴露内部治理对象

## 和普通 prompt 模板有什么不同

普通 prompt 模板通常关注“怎么把话写好”。

`json-prompt-engine` 更关注“怎么让 prompt 生成过程可治理”：

| 对比项 | 普通 prompt 模板 | json-prompt-engine |
|---|---|---|
| 核心目标 | 写出一段更好的 prompt | 生成可审查、可复用的 prompt package |
| 控制方式 | 靠自然语言约束 | 用结构化字段承载约束 |
| 审查机制 | 通常没有 | `prompt_review` + `answer_review` |
| 交付方式 | 一段文本 | `final_answer` / `json_prompt` / `hybrid_prompt_package` |
| 适用场景 | 一次性任务 | prompt audit、prompt upgrade、Agent skill 工作流 |

## 安装方法

本仓库目前还不是独立 CLI 工具，也尚未提供包管理器安装入口或 Marketplace 发布说明。

因此，当前阶段的“安装”，应理解为：**按宿主类型使用对应的 skill 目录。**

### 1. 克隆仓库

```bash
git clone <your-repo-url>
cd <your-repo-folder>
```

`（信息不足：请将 <your-repo-url> 和仓库名替换为实际 GitHub 地址）`

### 2. 按宿主使用对应目录

#### Codex / Cursor

使用：

```text
.agents/skills/json-prompt-engine/
```

这个目录包含：

- `SKILL.md`
- `references/`
- `agents/openai.yaml`

#### Claude Code

使用：

```text
.claude/skills/json-prompt-engine/
```

这个目录包含：

- `SKILL.md`
- `references/`

### 3. 最小验证方式

接入后，可以先用下面这类请求验证宿主是否识别到 skill：

> 请把这个自然语言需求升级成受治理的 JSON prompt，并输出 hybrid_prompt_package。

如果宿主能够识别到 `json-prompt-engine`，它不应该立刻只给你一句成品 prompt，而应先表现出：

- 任务分类或 slot 识别
- 治理强度判断
- first principles 或控制层起草
- review 后再交付结果

### 4. 当前阶段的已知边界

- 自动触发是否发生，取决于宿主自己的 skill 发现与匹配机制
- 手动调用语法因宿主而异，应以宿主支持的 skill invocation 方式为准
- `agents/openai.yaml` 是 Codex / OpenAI 专属 companion metadata，不应假设 Claude Code 或 Cursor 会读取它

## 工作原理

`json-prompt-engine` 的核心思想是：**先治理，再生成；先结构化，再交付。**

你可以把几个关键术语先这样理解：

- `control_mode`：本次任务需要多严格的控制方式
- `prompt_review`：生成前检查 prompt 本身是否合格
- `answer_review`：生成后检查答案是否满足目标
- `slim payload`：只给下游模型必要字段，减少冗余和泄露

### 主流程

1. 识别任务类型与 slot
2. 检查是否缺少会实质改变结果的关键信息
3. 只进行一次澄清；若用户要求继续，则记录最小合理假设
4. 解析 `control_mode`
5. 对非简单任务启用 first principles
6. 起草 `role_spec`、`task_spec`、`structure_constraints`、`anti_sycophancy_harness`
7. 构造 internal JSON prompt
8. 运行 `prompt_review`
9. 生成当前目标产物
10. 运行 `answer_review`
11. 修订并终检
12. 按 `summary_visibility` 决定用户可见内容

### 为什么要先有 internal JSON prompt

因为这个 skill 的中枢不是最终答案，而是内部治理对象。

这个内部对象的价值主要有三点：

- **可审查**：review 可以围绕结构化对象进行，而不是只盯着最终成文
- **可复用**：prompt 不再只是一段临时文本，而是可复查的工件
- **可安全交付**：下游模型默认只拿到必要字段，而不是完整治理元数据

## 为什么可信

这个 skill 的价值不来自包装语，而来自仓库里已经显式公开的治理结构。

### 1. 流程可信

根据 `SKILL.md` 和 `1.0` 中的工作流分析文档，它有固定主链路，而不是零散技巧拼装：

`任务分类 -> slot 选择 -> 一次性澄清检查 -> control_mode 解析 -> first principles -> 控制层起草 -> internal JSON prompt -> prompt_review -> answer generation -> answer_review -> 终检 -> 展示`

### 2. 结构可信

仓库中已经把几层关键控制机制写成显式结构，而不是口头约定：

- `control_mode`
- `control_floor`
- `change_governance`
- `prompt_review`
- `answer_review`
- `delivery_spec`

`1.0` 分析材料还明确指出了一些关键隐性逻辑，例如：

- schema 默认值不等于实际执行默认值
- `control_floor` 是硬下限，不是软建议
- `correction_policy` 负责诊断，`change_governance` 负责审批

### 3. 资产可信

仓库 `1.0/` 目录里已包含：

- 工作流分析 PDF
- 工作流分析 Word 文档
- Mermaid 流程图 PNG

这些资产至少说明：这个项目已经具备公开说明、内部评审和进一步发布包装的基础材料，而不只是零散文件。

## 仓库结构

当前仓库中真实存在的核心内容如下：

```text
.
├── 1.0/
│   ├── json-prompt-engine_workflow_analysis.docx
│   ├── json-prompt-engine_workflow_analysis.pdf
│   └── mermaid-diagram.png
├── .agents/
│   └── skills/
│       └── json-prompt-engine/
│           ├── SKILL.md
│           ├── PORTABILITY.md
│           ├── agents/
│           │   └── openai.yaml
│           └── references/
│               ├── json-schema.md
│               ├── template-registry.md
│               └── delivery-schema.md
└── .claude/
    └── skills/
        └── json-prompt-engine/
            ├── SKILL.md
            └── references/
                ├── json-schema.md
                ├── template-registry.md
                └── delivery-schema.md
```

关键文件说明：

- `SKILL.md`：主 workflow、控制层与 review / revision 规则
- `references/json-schema.md`：内部治理 JSON schema
- `references/template-registry.md`：task type、slot 与 overlay 规则
- `references/delivery-schema.md`：内部对象到交付 payload 的映射规则
- `PORTABILITY.md`：Codex / Cursor / Claude Code 的适配边界
- `1.0/`：面向公开发布准备的工作流分析材料与流程图

## 边界说明

`json-prompt-engine` 不是一个独立 CLI，也不是通用 prompt 模板合集。

它更适合作为 AI Agent / Skill 宿主环境中的治理层：当你需要把用户请求升级成可审查、可复用、可交付的 prompt package 时，它才真正有价值。

当前仓库也有几条需要直说的边界：

- 仓库中未发现运行时代码或 orchestration 脚本
- 当前可见主体是 skill 文档、schema、delivery mapping 和适配层
- 自动触发、审批暂停恢复、宿主级执行细节仍依赖外部宿主实现

如果你的任务只是一次性轻量问答，直接写 prompt 可能更轻。

## 许可证

`（信息不足：当前仓库未见 License 文件，README 暂无法写明正式开源许可证。）`

如果你准备把这个仓库直接公开到 GitHub，建议在发布前补充标准 `LICENSE` 文件；否则访客虽然能阅读内容，但无法明确判断复用、分发和修改权限。
