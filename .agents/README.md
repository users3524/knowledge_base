# 项目 AI 配置说明

这个目录存放当前知识库专用的 AI 约束和 Skill。下面的配置以 Codex 为准；其他 AI
客户端通常有各自的约定文件名，不能保证自动读取这里的内容。

## 当前结构

```text
AGENTS.md                                  # 仓库级自动加载入口
.agents/
|-- constraints.md                        # 详细维护约束
|-- README.md                             # 本教程
`-- skills/
    |-- knowledge-base-maintainer/        # 本知识库的整理与索引维护
    |-- llm-intern-skill/                 # LLM 实习简历与面试准备
    `-- circuit-analyze/                  # 硬件原理图分析
```

## 已安装的第三方 Skill

- `llm-intern-skill`：来源于 [couragec/LLMInternSkill](https://github.com/couragec/LLMInternSkill)，
  安装版本 `11db44f57b3d78ae3f83072b3959cd7a5d85df0b`。
- `circuit-analyze`：来源于 [q1109/Hardware_analyze](https://github.com/q1109/Hardware_analyze) 的
  `skills/circuit-analyze` 子目录，安装版本 `d6dd11c93641186c918fd07fb5e070e0543ca05f`。

这里安装的是仓库内容快照，不包含嵌套 `.git` 目录，也不会随上游仓库自动更新。更新前应先比较
本地改动与上游版本，避免直接覆盖已经定制过的 Skill。

## Codex 如何找到这些文件

### 1. 项目约束

把 `AGENTS.md` 放在仓库根目录。Codex 从当前工作目录向上查找 `AGENTS.md`，并把它作为
该目录树的持久约束。子目录可以再放一个更具体的 `AGENTS.md`，用于覆盖或补充该子树规则。

任意命名的 `.md` 文件不会自动成为约束，所以根目录的 `AGENTS.md` 明确要求 AI 在相关任务
中读取 `.agents/constraints.md`。

### 2. 项目级 Skill

项目 Skill 使用以下固定结构：

```text
.agents/skills/<skill-name>/SKILL.md
```

重新打开工作区或新建 Codex 会话后，可直接在提示词中调用：

```text
使用 $knowledge-base-maintainer 整理这篇笔记并更新目录索引。
```

Skill 的 `description` 决定自动触发场景；显式写 `$skill-name` 最可靠。修改或新增 Skill 后，
应新建会话，让客户端重新扫描技能元数据。

### 3. 用户级 Skill

需要在所有仓库复用的 Skill，放在：

```text
$CODEX_HOME/skills/<skill-name>/SKILL.md
```

未设置 `CODEX_HOME` 时，通常是：

```text
~/.codex/skills/<skill-name>/SKILL.md
```

这台 Windows 机器当前实际使用 `C:\Users\71085\.codex\skills`。

## 让旧版本也能找到项目 Skill

如果某个 Codex 客户端没有扫描 `.agents/skills`，可以在用户级目录创建 Windows 目录联接，
让 Skill 的真实文件仍保存在本仓库中。在仓库根目录运行：

```powershell
$source = (Resolve-Path '.agents\skills\knowledge-base-maintainer').Path
$target = Join-Path $HOME '.codex\skills\knowledge-base-maintainer'
New-Item -ItemType Directory -Force (Split-Path $target) | Out-Null
New-Item -ItemType Junction -Path $target -Target $source
```

若目标已存在，先确认它是否是需要保留的真实目录，不要直接覆盖。创建后重启 Codex 或新建会话。

## 验证方法

1. 在这个仓库中新建 Codex 会话。
2. 询问“当前仓库有哪些约束”，确认回答提到 `.agents/constraints.md`。
3. 输入 `使用 $knowledge-base-maintainer 检查知识库索引`，确认 Skill 能被识别。
4. 若无法识别，检查目录名与 `SKILL.md` frontmatter 中的 `name` 是否一致，再使用上面的目录联接方案。

## 不建议的做法

- 不要只创建一个随意命名的 `.ai` 目录并期待 Codex 自动递归读取。
- 不要把 `CODEX_HOME` 直接改成本仓库的 `.agents`。`CODEX_HOME` 还包含登录、全局配置、会话等
  用户级状态；这样做会同时切换整套 Codex 环境，而不只是 Skill 搜索目录。
- 不要把大量领域资料直接塞入 `SKILL.md`。核心工作流留在 `SKILL.md`，长资料按需放到
  `references/`，脚本放到 `scripts/`，模板或素材放到 `assets/`。
