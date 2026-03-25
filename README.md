# AI / CLI Ignore Setup for Mobile Projects

This repository provides a shell script that can be executed directly from GitHub to generate ignore and instruction files for common AI coding tools used in Android, iOS, and Kotlin Multiplatform projects.

It now generates files per tool instead of forcing the same file for every CLI.

## Supported tools

- `Gemini` -> `.geminiignore`
- `Qwen Code` -> `.qwenignore`
- `Windsurf / Codeium` -> `.codeiumignore`
- `JetBrains AI Assistant` -> `.aiignore`
- `JetBrains AI Assistant disable switch` -> `.noai` when you opt into `noai`
- `Claude Code` -> `.claude/settings.json`
- `Codex / GPT-based coding workflows` -> `AGENTS.md`
- `GitHub Copilot CLI` -> `.github/copilot-instructions.md`, `AGENTS.md` compatible guidance, and `.copilotignore`

## Why this changed

Different tools no longer share a single universal ignore-file convention:

- Claude Code uses project settings in `.claude/settings.json`
- Codex and other agent-style workflows commonly use `AGENTS.md`
- GitHub Copilot CLI supports `AGENTS.md` and `.github/copilot-instructions.md`
- JetBrains AI Assistant supports `.aiignore` and `.noai`

Because of that, the script generates the right file type for each tool.

## Quick start

Run this from the root of your project:

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/frizeiro/mobile-ai-ignores/main/setup_ai_ignores.sh)"
```

## Options

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/frizeiro/mobile-ai-ignores/main/setup_ai_ignores.sh)" -- --help
```

Main options:

- `--profile balanced|strict`
- `--tools gemini,qwen,windsurf,jetbrains,noai,claude,codex,copilot`
- `--force` for non-ignore files that already exist
- `--backup`
- `--dry-run`

During an interactive run, the script first asks for custom files or folders to add to every ignore file. Enter one pattern per line and press Enter on an empty line to finish. Any valid `.gitignore`-style pattern is accepted:

```
Add custom files/folders to ignore? (one per line, empty line to finish)
secrets/
internal-docs/
*.local.swift
             ← empty line to finish
```

At the end of an interactive run, the script lists the written files and asks what to do with them:

```
What would you like to do with these files?
  1) Track in git (force-add even if globally ignored)
  2) Add paths to local .gitignore
  3) Add paths to global gitignore
  0) Skip
Choice [0]:
```

- **1 — Track in git:** runs `git add --force` on each file so they are staged for commit even if a global gitignore would normally exclude them. Useful when you want these config files committed to the repository.
- **2 — Local .gitignore:** appends each file path to the project's `.gitignore`, keeping them out of version control for everyone on the team.
- **3 — Global gitignore:** appends each path to your user-level gitignore (`core.excludesfile`), keeping them untracked on your machine only without affecting other contributors.
- **0 / Enter — Skip:** does nothing.

## Examples

Generate the default set:

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/frizeiro/mobile-ai-ignores/main/setup_ai_ignores.sh)"
```

Generate only Gemini, Qwen, Windsurf and Codex files:

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/frizeiro/mobile-ai-ignores/main/setup_ai_ignores.sh)" -- \
  --tools gemini,qwen,windsurf,codex
```

Preview changes without writing files:

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/frizeiro/mobile-ai-ignores/main/setup_ai_ignores.sh)" -- \
  --tools all --profile strict --dry-run
```

Overwrite existing non-ignore files and keep backups:

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/frizeiro/mobile-ai-ignores/main/setup_ai_ignores.sh)" -- \
  --tools all --force --backup
```

## Profiles

`balanced`

- Hides secrets and generated build output
- Keeps potentially useful project assets and most editor config visible

`strict`

- Also hides heavy media, framework bundles, dependency trees, and more editor files
- Better for privacy and token efficiency, but more aggressive

## Notes

- Ignore files are merged in sections: effective rules from `.gitignore`, preserved local target rules, the generated rules from the script, and finally any custom patterns you entered
- Existing non-ignore files are not overwritten unless you pass `--force`
- If you pass `--backup`, replaced or merged files are copied to `<name>.bak`
- Interactive runs prompt to track, locally ignore, or globally ignore the written files; the default is skip
- `gpt` and `openai` are accepted as aliases of `codex`
- `codeium` is accepted as an alias of `windsurf`
- `noai` or `jetbrains-noai` creates a project-level `.noai` file that disables JetBrains AI Assistant entirely

## Sources

- Anthropic Claude Code settings and permissions: `docs.anthropic.com`
- GitHub Copilot CLI instructions and agents: `docs.github.com`
- Windsurf ignore rules: `docs.windsurf.com`
- JetBrains AI Assistant project restrictions: `jetbrains.com/help`
- Qwen Code ignore rules: `qwenlm.github.io`
- OpenAI Codex model guidance: `platform.openai.com/docs`
