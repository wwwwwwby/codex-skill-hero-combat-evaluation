# Codex Skill: Hero Combat Evaluation

私有 Codex skill，用于审阅 FlamonMoba 英雄战力评估文档，并在作者确认 Markdown 后生成新版敌方 1v1 收益/强敌战力评估代码。

## Install

在 Codex 中让助手执行：

```text
请使用 skill-installer 安装这个私有 skill：
https://github.com/wwwwwwby/codex-skill-hero-combat-evaluation/tree/main/skills/hero-combat-evaluation
```

如果手动运行安装器，使用自己机器上的 Codex skill-installer 脚本路径：

```bash
python <CODEX_HOME>/skills/.system/skill-installer/scripts/install-skill-from-github.py --url https://github.com/wwwwwwby/codex-skill-hero-combat-evaluation/tree/main/skills/hero-combat-evaluation
```

安装后重启 Codex。

## Usage

在 Codex 中直接说：

```text
使用 $hero-combat-evaluation 审阅这份英雄战力评估文档，先整理并确认 Markdown，再生成代码。
```

## Contents

- `skills/hero-combat-evaluation/SKILL.md`
- `skills/hero-combat-evaluation/references/flamonmoba-1v1-checklist.md`
- `skills/hero-combat-evaluation/agents/openai.yaml`

## Access

这是私有仓库。使用者需要先获得该 GitHub 仓库访问权限，并确保本机 Codex/GitHub CLI 使用的 GitHub 凭据可访问私有仓库。
