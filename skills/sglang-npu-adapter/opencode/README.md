# Opencode管理自定义skill

本文面向一个常见场景：同事已经开发好业务 Skill，其他人需要在自己的 OpenCode 项目中安装、更新和卸载。

示例使用 [`sglang-npu-adapter`](https://github.com/Talantan1102/Ascend-SGLang-Skills/tree/main/skills/sglang-npu-adapter)。它不是只有一个 `SKILL.md`，还带有四个 OpenCode 子 Agent，因此两部分都要管理。

## 1. 目录约定

推荐安装到业务项目内，只影响当前项目：

```text
your-project/
└── .opencode/
    ├── skills/
    │   └── sglang-npu-adapter/
    │       ├── SKILL.md
    │       ├── prompts/
    │       ├── references/
    │       ├── scripts/
    │       └── templates/
    └── agents/
        ├── architecture-analyst.md
        ├── debug-engineer.md
        ├── precision-rca.md
        └── test-validator.md
```

注意：仓库中的源目录是 `opencode/agent`，目标目录是 `.opencode/agents`，一个是单数，一个是复数。

## 2. 安装

先准备 Skill 仓库地址和业务项目路径。下面只保留核心命令。

### Windows PowerShell

```powershell
$SkillsRepo = "D:\tools\Ascend-SGLang-Skills"
$Project = "D:\work\your-project"
git clone https://github.com/Talantan1102/Ascend-SGLang-Skills.git $SkillsRepo
New-Item -ItemType Directory -Force "$Project\.opencode\skills","$Project\.opencode\agents" | Out-Null
Copy-Item -Recurse "$SkillsRepo\skills\sglang-npu-adapter\opencode\skills\sglang-npu-adapter" "$Project\.opencode\skills\"
Copy-Item "$SkillsRepo\skills\sglang-npu-adapter\opencode\agent\*.md" "$Project\.opencode\agents\"
```

### Linux / macOS

```bash
SKILLS_REPO="$HOME/tools/Ascend-SGLang-Skills"
PROJECT="/path/to/your-project"
git clone https://github.com/Talantan1102/Ascend-SGLang-Skills.git "$SKILLS_REPO"
mkdir -p "$PROJECT/.opencode/skills" "$PROJECT/.opencode/agents"
cp -R "$SKILLS_REPO/skills/sglang-npu-adapter/opencode/skills/sglang-npu-adapter" "$PROJECT/.opencode/skills/"
cp "$SKILLS_REPO"/skills/sglang-npu-adapter/opencode/agent/*.md "$PROJECT/.opencode/agents/"
```

安装后重启 OpenCode，并从业务项目根目录打开它。

![安装结果：Skill 文件完整、四个 Agent 已安装并被 OpenCode 发现](https://raw.githubusercontent.com/Talantan1102/Ascend-SGLang-Skills/main/skills/sglang-npu-adapter/opencode/assets/01-install-result.png)

本机实测结果：Skill 源目录和目标目录各有 60 个文件，SHA-256 目录校验一致；四个 Agent 均被识别为项目级 subagent。

## 3. 更新

更新时不要直接覆盖旧目录。先把当前版本移到备份目录，再拉取仓库并重新执行安装部分的两条复制命令。这样新版本有问题时可以回退。

### Windows PowerShell

```powershell
$Backup = "$Project\.opencode-backups\sglang-npu-adapter-$(Get-Date -Format yyyyMMdd-HHmmss)"
New-Item -ItemType Directory -Force "$Backup\skill","$Backup\agents" | Out-Null
Move-Item "$Project\.opencode\skills\sglang-npu-adapter" "$Backup\skill\"
@("architecture-analyst.md","debug-engineer.md","precision-rca.md","test-validator.md") | ForEach-Object { Move-Item "$Project\.opencode\agents\$_" "$Backup\agents\" }
git -C $SkillsRepo pull --ff-only
# 重新执行“安装”中的两条 Copy-Item
```

### Linux / macOS

```bash
BACKUP="$PROJECT/.opencode-backups/sglang-npu-adapter-$(date +%Y%m%d-%H%M%S)"
mkdir -p "$BACKUP/skill" "$BACKUP/agents"
mv "$PROJECT/.opencode/skills/sglang-npu-adapter" "$BACKUP/skill/"
mv "$PROJECT/.opencode/agents/"{architecture-analyst,debug-engineer,precision-rca,test-validator}.md "$BACKUP/agents/"
git -C "$SKILLS_REPO" pull --ff-only
# 重新执行“安装”中的两条 cp
```

更新后重启 OpenCode，再做一次调用验证。

## 4. 卸载

推荐先做可恢复卸载：把文件移出 `.opencode/skills` 和 `.opencode/agents`，不要立刻删除。

### Windows PowerShell

```powershell
$Disabled = "$Project\.opencode-disabled\sglang-npu-adapter"
New-Item -ItemType Directory -Force "$Disabled\skill","$Disabled\agents" | Out-Null
Move-Item "$Project\.opencode\skills\sglang-npu-adapter" "$Disabled\skill\"
@("architecture-analyst.md","debug-engineer.md","precision-rca.md","test-validator.md") | ForEach-Object { Move-Item "$Project\.opencode\agents\$_" "$Disabled\agents\" }
```

### Linux / macOS

```bash
DISABLED="$PROJECT/.opencode-disabled/sglang-npu-adapter"
mkdir -p "$DISABLED/skill" "$DISABLED/agents"
mv "$PROJECT/.opencode/skills/sglang-npu-adapter" "$DISABLED/skill/"
mv "$PROJECT/.opencode/agents/"{architecture-analyst,debug-engineer,precision-rca,test-validator}.md "$DISABLED/agents/"
```

重启 OpenCode 后，`sglang-npu-adapter` 和四个 Agent 应不再出现。确认不需要恢复后，再删除 `.opencode-disabled/sglang-npu-adapter`。

![更新与卸载结果：备份完整，移出扫描目录后 OpenCode 不再发现该 Skill](https://raw.githubusercontent.com/Talantan1102/Ascend-SGLang-Skills/main/skills/sglang-npu-adapter/opencode/assets/02-lifecycle-result.png)

## 5. 成功调用 Skill

在业务项目中打开 OpenCode，输入：

```text
请调用 skill 工具加载 sglang-npu-adapter。加载成功后，告诉我：
1. Skill 名称；
2. Step 0 是什么；
3. 开始执行前需要哪些输入。
不要运行脚本，不要调用子 Agent，不要修改文件。
```

判断成功不能只看回答中是否出现 Skill 名称，还要看 OpenCode 是否真的产生了 `skill` 工具调用，并且状态为 `completed`。

![成功调用结果：skill 工具加载完成，OpenCode 正确回答 Step 0 和所需输入](https://raw.githubusercontent.com/Talantan1102/Ascend-SGLang-Skills/main/skills/sglang-npu-adapter/opencode/assets/03-skill-call-success.png)

本机实测中，OpenCode 实际加载了 `sglang-npu-adapter`，正确读出 Step 0 为“初始化”，所需输入为 `WORKSPACE_DIR`、`ModelName` 和 `ModelPath`。本次只验证 Skill 加载，没有运行模型适配脚本，也没有连接 NPU 环境。

## 6. 团队如何做到一键管理

OpenCode 当前主要通过目录发现 Skill，本身没有完整的 `skill install / update / remove` 生命周期命令。对于只有 Skill 目录的简单包，可以使用第三方 [`npx skills`](https://github.com/vercel-labs/skills)。

本例还包含四个独立 Agent，单独运行 `npx skills` 只会发现 `sglang-npu-adapter` Skill，不会自动管理这些 Agent。因此团队内最实用的做法，是由 Skill 仓库同时提供 `install.ps1` 和 `install.sh`，把本文的复制、备份和卸载动作封装起来。使用者只需传入业务项目路径。

## 注意事项

- 必须复制整个 Skill 目录，不能只复制 `SKILL.md`。
- 安装、更新、卸载后都应重启 OpenCode。
- 同名 Skill 或 Agent 同时存在于项目级和用户级目录时，容易误判实际加载来源；团队复用优先采用项目级安装。
- `sglang-npu-adapter` 内包含 Shell 脚本。Windows 上真正执行适配流程时，需要 WSL 或 Linux 环境；本 Wiki 的本地证据只覆盖安装、发现和加载。

## 参考资料

- [OpenCode Skills 官方文档](https://opencode.ai/docs/skills/)
- [OpenCode Agents 官方文档](https://opencode.ai/docs/agents/)
- [sglang-npu-adapter 示例仓库](https://github.com/Talantan1102/Ascend-SGLang-Skills/tree/main/skills/sglang-npu-adapter)
