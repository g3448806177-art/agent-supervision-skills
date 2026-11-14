# Agent Supervision Skills · Codex 版

**让多代理协作有明确分工、可追溯证据和独立验收。**

Two instruction-based skills for task planning, independent supervision, and evidence-based acceptance in Codex. Chinese instructions; MIT licensed.

本仓库包含两个配套 skill：

| Skill | 职责 |
| --- | --- |
| [project-master-control](skills/project-master-control/SKILL.md) | 判断任务大小，写任务卡，分配文件与依赖，协调执行并跟到交付 |
| [task-supervisor](skills/task-supervisor/SKILL.md) | 开工前核对目标，独立检查证据，提出问题并在修复后复验 |

小任务直接完成。多个独立工作包、跨模块依赖或复杂验收才进入完整协作流程。

## 为什么做这个

并行执行能加快工作，但我还希望明确：谁维护计划、谁改哪些文件、谁独立验收、问题怎样关闭，以及什么证据才能支持“完成”。这两个 skill 将这些约定写进可复用的工作流程。

灵感之一是 Claude Code 的 **Ultracode**：它会自动编排动态工作流，工作流本身也可以包含验证阶段。本项目把任务卡、职责隔离、唯一监督登记和问题复验作为显式约定。关于 Ultracode 的行为，以 [Claude Code 官方文档](https://code.claude.com/docs/en/workflows#let-claude-decide-with-ultracode) 为准。

这是独立的社区项目，与 Anthropic、OpenAI 均无隶属关系。它不修改 Ultracode，也没有经过与 Ultracode 的效果对照实验。当前版本按 **Codex 普通独立对话** 的协作方式编写，尚未验证 Claude Code 端到端兼容性。

## 工作流程

```text
识别角色和任务大小
  ├─ 小任务：当前对话完成
  └─ 大任务：读取规则 → 写齐任务卡与共享约定
             → 建立或复用唯一监督 → 开工目标对齐
             → 按依赖分派执行 → 阶段监督
             → 原负责人修复 → 同一监督复验 → 最终交付
```

- 主控维护 `MASTER_PLAN.md`，记录目标、任务卡、真实对话 ID、模型配置证据和进度。
- 监督维护 `SUPERVISION.md`，记录问题 ID、证据、责任任务、修复要求和复验结果。
- 执行者负责实现和自测；监督者不修改实现后再自称独立验收。
- 请求模型成功与实际运行配置核实分开记录；关键问题未关闭就不宣布全部完成。
- 同目标的继续、追问和修复复用原监督；执行者发现范围超出原卡时，可在用户授权与原根主控具体批准后创建新增执行对话，仍由原主控统一登记和原监督验收。

## 安装

两个目录互相引用，请一起安装并保留同级关系。它们都是文本指令，不需要运行服务或提供 API key。

在 Codex 中可以请求安装：

```text
使用 $skill-installer 从 https://github.com/g3448806177-art/agent-supervision-skills
同时安装 skills/project-master-control 和 skills/task-supervisor。
```

也可以在 macOS / Linux 手动安装到当前官方文档列出的用户级目录 `~/.agents/skills`：

```sh
git clone https://github.com/g3448806177-art/agent-supervision-skills.git
cd agent-supervision-skills
(
  set -eu
  skills_dir="$HOME/.agents/skills"
  mkdir -p "$skills_dir"
  for name in project-master-control task-supervisor; do
    if [ -e "$skills_dir/$name" ] || [ -L "$skills_dir/$name" ]; then
      echo "Already exists: $skills_dir/$name — back it up or choose another install scope."
      exit 1
    fi
  done
  cp -R skills/project-master-control "$skills_dir/"
  cp -R skills/task-supervisor "$skills_dir/"
)
```

如果宿主已从其他位置加载同名 skill，先备份并选择保留哪份，避免重复安装。安装后检查 skill 列表；未显示时重启 Codex。项目级安装也可将这两个目录放入项目的 `.agents/skills/`。加载位置和调用方式见 [OpenAI 官方说明](https://learn.chatgpt.com/docs/build-skills#where-codex-loads-local-skills)。

本仓库提供源文件安装方式，尚未上架插件市场。

## 使用

调用主控，并按实际项目明确授权范围，例如：

```text
请用 $project-master-control 完成这个项目：<填写目标、输入与验收条件>。
如果任务需要拆分，我授权你在本目标范围内创建普通独立执行和监督对话，
按任务难度选择宿主支持的模型与推理强度，读取进度，并发送协调与修复消息。
我也允许执行者发现任务超出原卡后，经原根主控具体批准，创建本批新增普通执行对话。
新对话沿用原项目、主控和监督；再次扩展仍需原根主控批准。开串不自动增加跨对话发信权限。
超出范围或预算的操作仍需确认。
```

仅核查现有成果：

```text
请用 $task-supervisor 核查 <交付物路径> 是否满足 <原始要求>。
读取原始证据和最终版本，列出尚未满足的要求，不修改被验收实现。
```

`allow_implicit_invocation: true` 允许宿主按描述选择 skill，但不保证每次自动触发。希望稳定进入指定流程时，在请求里显式点名。安装文件本身不授予创建对话、对外写入或费用使用权限。

## 宿主能力与边界

完整流程需要宿主能够创建、读取、发送消息到普通独立对话，并有界等待；按卡选模型还需要模型选择与运行配置核实能力。技能文件本身不提供这些工具。

- 本版要求的普通独立对话与内部子代理是不同对象，不能相互冒称。
- 缺少能力时，先完成不依赖该能力的盘点、计划和交接，并报告实际缺口；仅做自查时不能称独立验收通过。
- 文中的具体模型名称是可配置示例，须以宿主实际支持和当前用户授权为准。
- 更多对话通常增加时间和 token 消耗。任务卡应约定预算和验收范围。
- 这些是指令级约定，不能保证模型永不偏离，也不提供对话结束后的常驻监控。
- 如需适配 Claude Code，须按其实际协作工具调整对话登记、模型选择和协调方式，并重新验证。

## 版本与验证范围

当前版本：`2026-11-14.6`。新增简短回报、按任务卡与契约版本核对、重复/冲突处理、依赖精确放行及主控待决负担调度；扩展提案缺项交原执行者补齐。保留拆分、完整上下文预算、原根主控批准扩展和独立验收。

大任务必读两个技能入口及完整协作规则。仅输出任务提示词时读取 `task-prompts.md`；涉及数据、费用或软件发布时按需读取 `evidence-checks.md`。

技能规则不包含作者的私人称呼或历史项目约定；安装者须在当前会话提供适用授权。首个公开版本为 `2026-11-14.1`。

发布检查覆盖技能结构、文件引用、内容检查、规则审稿及上传一致性。本版四类回报实测正在进行；不据此声称真实视频交付、大规模并发或速度收益。

## 贡献

欢迎提交 Issue 或 Pull Request。报告问题时请提供宿主及版本、最小任务示例、期望行为、实际结果和可公开的证据；请勿上传密钥、聊天隐私或业务数据。修改规则应保留小任务直办、职责隔离、授权边界和基于证据的验收。

## 许可证

[MIT License](LICENSE) · Copyright (c) 2026 g3448806177-art
