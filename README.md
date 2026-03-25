# AI / CLI Ignore Setup for Mobile Projects

A shell script that generates ignore and instruction files for common AI coding tools used in Android, iOS, and Kotlin Multiplatform projects. Run it from the root of any mobile project.

## Supported tools

| Tool | File(s) generated | Aliases |
|------|-------------------|---------|
| `gemini` | `.geminiignore` | — |
| `qwen` | `.qwenignore` | — |
| `windsurf` | `.codeiumignore` | `codeium` |
| `jetbrains` | `.aiignore` | `aiassistant`, `idea` |
| `noai` | `.noai` | `jetbrains-noai` |
| `claude` | `.claude/settings.json` | `anthropic` |
| `codex` | `AGENTS.md` | `gpt`, `openai` |
| `copilot` | `.github/copilot-instructions.md`, `.copilotignore` | `github` |
| `all` | all of the above | — |

**Default set:** `gemini,qwen,windsurf,jetbrains,claude,codex,copilot`

The `noai` tool is opt-in. It creates a `.noai` file that disables JetBrains AI Assistant for the project entirely.

## File types

The script handles two kinds of output differently:

- **Ignore files** (`.geminiignore`, `.qwenignore`, `.codeiumignore`, `.aiignore`, `.copilotignore`) — always merged. Effective rules from `.gitignore` and any existing content in the target file are preserved in the output; duplicate lines are removed automatically.
- **Instruction / config files** (`.claude/settings.json`, `AGENTS.md`, `.github/copilot-instructions.md`, `.noai`) — skipped if they already exist. Pass `--force` to overwrite, `--backup` to save a `.bak` copy before overwriting.

## Quick start

Run from the root of your project:

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/frizeiro/mobile-ai-ignores/main/setup_ai_ignores.sh)"
```

## Options

```
--profile balanced|strict   Rule intensity. Default: balanced
--tools <csv>               Comma-separated list of tools. Default: gemini,qwen,windsurf,jetbrains,claude,codex,copilot
--force                     Overwrite existing instruction/config files
--backup                    Save a .bak copy before overwriting or merging any file
--dry-run                   Preview what would be created or updated without writing anything
-h, --help                  Show usage
```

## Interactive prompts

Both prompts are skipped automatically when stdin or stdout is not a terminal (CI, piped input).

### 1. Custom patterns

Before generating files, the script asks for extra patterns to append to every ignore file. Enter one pattern per line; an empty line finishes input. Any `.gitignore`-style pattern is accepted.

```
Add custom files/folders to ignore? (one per line, empty line to finish)
secrets/
internal-docs/
*.local.swift
             ← empty line to finish
```

Custom patterns are appended as a dedicated `# Custom patterns` section at the end of each ignore file, after the generated rules.

### 2. Git tracking

After writing files, the script lists what was written and asks how to handle them:

```
Generated files:
  .geminiignore
  .qwenignore
  ...

What would you like to do with these files?
  1) Track in git (force-add even if globally ignored)
  2) Add paths to local .gitignore
  3) Add paths to global gitignore
  0) Skip
Choice [0]:
```

- **1 — Track in git:** runs `git add --force` on each file, staging them for commit even if a global gitignore would otherwise exclude them.
- **2 — Local .gitignore:** appends each path to the project's `.gitignore`, keeping the files untracked for everyone on the team.
- **3 — Global gitignore:** appends each path to your user-level gitignore (`core.excludesfile`), keeping them untracked on your machine only without touching project files. If no global gitignore is configured, it creates `~/.gitignore_global` and sets it via `git config --global core.excludesfile`.
- **0 / Enter — Skip:** does nothing.

## Examples

Generate the default set:

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/frizeiro/mobile-ai-ignores/main/setup_ai_ignores.sh)"
```

Generate only Gemini, Qwen, Windsurf and Codex files with the strict profile:

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/frizeiro/mobile-ai-ignores/main/setup_ai_ignores.sh)" -- \
  --profile strict --tools gemini,qwen,windsurf,codex
```

Preview all changes without writing files:

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/frizeiro/mobile-ai-ignores/main/setup_ai_ignores.sh)" -- \
  --tools all --profile strict --dry-run
```

Overwrite existing instruction files and keep backups:

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/frizeiro/mobile-ai-ignores/main/setup_ai_ignores.sh)" -- \
  --tools all --force --backup
```

## Profiles

Both profiles always ignore:

- **Security and secrets:** `*.jks`, `*.keystore`, `*.pem`, `*.key`, `*.p12`, `*.pfx`, `.env`, `.env.*`, `local.properties`, `google-services.json`, `GoogleService-Info.plist`, `service-account-key.json`, `signing-key.json`, `Secrets.apple`
- **Android / Gradle:** `.gradle/`, `**/build/`, `captures/`, `.externalNativeBuild/`, `.cxx/`, `*.apk`, `*.aab`
- **Apple platforms:** `DerivedData/`, `*.xcuserstate`, `*.moved-aside`, `*.dSYM/`, `*.xcarchive`, `*.ipa`, `.swiftpm/`, `.build/`
- **Kotlin Multiplatform:** `.kotlin/`, `.konan/`
- **Coverage, logs, caches:** `coverage/`, `*.hprof`, `*.class`, `*.pyc`, `*.log`
- **Generated dependency trees:** `vendor/bundle/`

**`balanced`** additionally hides only the noisy parts of `.idea/` (`workspace.xml`, `tasks.xml`, `caches/`, `shelf/`, `httpRequests/`), `.DS_Store`, `Thumbs.db`, and Fastlane output (`fastlane/screenshots/`, `fastlane/test_output/`). Project source, assets, and most editor config remain visible.

**`strict`** additionally hides the entire `.idea/` and `.vscode/` directories, `Pods/`, `node_modules/`, `*.framework/`, `*.xcframework/`, and all image (`*.png`, `*.jpg`, `*.jpeg`, `*.gif`, `*.webp`), video (`*.mov`, `*.mp4`), audio (`*.wav`, `*.mp3`), and document (`*.pdf`) assets. Better for privacy and token efficiency, but more aggressive.

## Sources

- Anthropic Claude Code settings and permissions: `docs.anthropic.com`
- GitHub Copilot CLI instructions and agents: `docs.github.com`
- Windsurf ignore rules: `docs.windsurf.com`
- JetBrains AI Assistant project restrictions: `jetbrains.com/help`
- Qwen Code ignore rules: `qwenlm.github.io`
- OpenAI Codex model guidance: `platform.openai.com/docs`
