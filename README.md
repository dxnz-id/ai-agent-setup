# ai-agent-setup

A reusable AI coding agent setup for new devices. This repo is intentionally just a README: the list of plugins, skills, sources, and the commands to install them. No vendored skill files, no installer script.

Principles:

- **Skills are universal.** Every skill lives once in `~/.agents/skills/`. Agents that support the `SKILL.md` format can read them, some directly, some through symlinks.
- **Plugins are not universal.** Each agent has its own plugin mechanism, so the plugin section is always per-agent.
- **Universal setup first.** Add exceptions only when the agent on your device does not read `~/.agents/skills/`.

Prerequisites: `git`, `curl`, and an agent that supports `SKILL.md` skills. OpenCode must be 2.0.4 or later for the Superpowers plugin.

---

## 1. Universal setup: skills

Every skill lives in one place:

```sh
mkdir -p ~/.agents/skills
```

These agents read it directly with no extra configuration:

| Agent | Global locations read |
| --- | --- |
| OpenCode | `~/.agents/skills/`, `~/.config/opencode/skills/` |
| Codex | `~/.agents/skills/` |
| Cursor | `~/.agents/skills/`, `~/.cursor/skills/` |
| Gemini CLI | `~/.agents/skills/`, `~/.gemini/skills/` |

If your agent is in this table, skip section 2.

### Skill list

| Skill | Source | Command |
| --- | --- | --- |
| bahasa-inti, bahasa-marketing, bahasa-website, bahasa-seo, bahasa-peneliti | [alvinindra/bahasa-skills](https://github.com/alvinindra/bahasa-skills) | see block below |
| humanizer-id (+ `jurnal` skill) | [apissaj/humanizer-id](https://github.com/apissaj/humanizer-id) | `git clone --depth 1 https://github.com/apissaj/humanizer-id.git ~/.agents/skills/humanizer-id` |
| material-3 | [hamen/material-3-skill](https://github.com/hamen/material-3-skill) | see block below |
| create-verification-skill | [cursor/plugins](https://github.com/cursor/plugins) (`pstack/skills/create-verification-skill`) | see block below |
| code-simplification | [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills) (`skills/code-simplification`) | see block below |
| officecli (skill only) | [officecli.ai/SKILL.md](https://officecli.ai/SKILL.md) | see block below |
| find-skills | [vercel-labs/skills](https://github.com/vercel-labs/skills) | see block below |
| stop-slop | [hardikpandya/stop-slop](https://github.com/hardikpandya/stop-slop) | `git clone --depth 1 https://github.com/hardikpandya/stop-slop.git ~/.agents/skills/stop-slop` |
| tui-design | [hyperb1iss/hyperskills](https://github.com/hyperb1iss/hyperskills) | see block below |

Skills that need extra steps:

```sh
# bahasa-skills: the repo holds 5 skills under skills/
git clone --depth 1 https://github.com/alvinindra/bahasa-skills.git /tmp/bahasa-skills
cp -r /tmp/bahasa-skills/skills/bahasa-* ~/.agents/skills/

# material-3: the skill lives in skills/material-3
git clone --depth 1 https://github.com/hamen/material-3-skill.git /tmp/material-3-skill
cp -r /tmp/material-3-skill/skills/material-3 ~/.agents/skills/

# create-verification-skill: pull one folder from a large repo
git clone --depth 1 --filter=blob:none --sparse https://github.com/cursor/plugins.git /tmp/cursor-plugins
git -C /tmp/cursor-plugins sparse-checkout set pstack/skills/create-verification-skill
mkdir -p ~/.agents/skills/create-verification-skill
cp -r /tmp/cursor-plugins/pstack/skills/create-verification-skill/. ~/.agents/skills/create-verification-skill/

# code-simplification
git clone --depth 1 https://github.com/addyosmani/agent-skills.git /tmp/agent-skills
cp -r /tmp/agent-skills/skills/code-simplification ~/.agents/skills/

# officecli: skill file only, the binary is installed separately (see section 4)
mkdir -p ~/.agents/skills/officecli
curl -fsSL https://officecli.ai/SKILL.md -o ~/.agents/skills/officecli/SKILL.md

# find-skills
git clone --depth 1 https://github.com/vercel-labs/skills.git /tmp/vercel-skills
cp -r /tmp/vercel-skills/skills/find-skills ~/.agents/skills/

# tui-design
git clone --depth 1 https://github.com/hyperb1iss/hyperskills.git /tmp/hyperskills
cp -r /tmp/hyperskills/skills/tui-design ~/.agents/skills/
```

Verify:

```sh
ls ~/.agents/skills/*/SKILL.md
```

Then in your agent, ask **"list your available skills"** or open `/skills` (OpenCode, Antigravity CLI).

---

## 2. When an agent is not compatible

Some agents do not read `~/.agents/skills/`. Do not change the universal setup; just add a symlink **per skill**.

| Agent | Folder it reads | Action |
| --- | --- | --- |
| Claude Code | `~/.claude/skills/` | symlink (block below) |
| Antigravity app / IDE | `~/.gemini/config/skills/` | symlink |
| Antigravity CLI | `~/.gemini/antigravity-cli/skills/` or `~/.gemini/skills/` | symlink |
| Other SKILL.md agents | their own skill folder | symlink or copy |

```sh
# Claude Code
mkdir -p ~/.claude/skills
for s in ~/.agents/skills/*/; do
  d=~/.claude/skills/"$(basename "$s")"
  [ -d "$d" ] && [ ! -L "$d" ] && rm -rf "$d"
  ln -sfn "$s" "$d"
done

# Antigravity app / IDE
mkdir -p ~/.gemini/config/skills
for s in ~/.agents/skills/*/; do
  d=~/.gemini/config/skills/"$(basename "$s")"
  [ -d "$d" ] && [ ! -L "$d" ] && rm -rf "$d"
  ln -sfn "$s" "$d"
done

# Antigravity CLI
mkdir -p ~/.gemini/antigravity-cli/skills
for s in ~/.agents/skills/*/; do
  d=~/.gemini/antigravity-cli/skills/"$(basename "$s")"
  [ -d "$d" ] && [ ! -L "$d" ] && rm -rf "$d"
  ln -sfn "$s" "$d"
done
```

The `rm -rf` line replaces a real folder left by the copy fallback below. It never removes a symlink; `ln -sfn` replaces those directly.

Important: symlink **one skill folder at a time**, never the whole `skills/` directory. Claude Code and Codex write internal files into their skill folders, and symlinking the whole directory can break that.

If the agent does not support symlinks (for example Windows without symlink permission), copy the folder:

```sh
cp -r ~/.agents/skills/humanizer-id ~/.claude/skills/
```

The trade-off: updates must be applied in both places.

---

## 3. Plugins per agent

Plugins change agent behavior (not just add instructions), so there is no single command for all agents. The table covers the two plugins used in this setup.

### Ponytail

Source: [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail) — the "lazy senior dev" ruleset, `/ponytail lite|full|ultra|off`.

| Agent | Command |
| --- | --- |
| OpenCode | `opencode plugin add @dietrichgebert/ponytail` |
| Claude Code | `/plugin marketplace add DietrichGebert/ponytail` then `/plugin install ponytail@ponytail` |
| Codex | `codex plugin marketplace add DietrichGebert/ponytail` then `codex plugin add ponytail@ponytail` |
| Gemini CLI / Antigravity | follow the repo's `INSTALL.md` (Gemini/Antigravity section) |

### Superpowers

Source: [obra/superpowers](https://github.com/obra/superpowers) — brainstorming, TDD, systematic debugging, writing plans, subagent-driven development.

| Agent | Command |
| --- | --- |
| OpenCode | add `"superpowers@git+https://github.com/obra/superpowers.git"` to `plugins` in `opencode.jsonc` |
| Claude Code | `/plugin install superpowers@claude-plugins-official` |
| Codex | `/plugins`, search `superpowers`, choose Install |
| Antigravity | `agy plugin install https://github.com/obra/superpowers` |
| Cursor | `/add-plugin superpowers` |

If an agent has no plugin system, skip this section. The core Superpowers skills still work if their skill folders are copied into `~/.agents/skills/` (see the repo for locations).

### Example config: OpenCode

`~/.config/opencode/opencode.jsonc`:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "plugins": [
    "@dietrichgebert/ponytail",
    "superpowers@git+https://github.com/obra/superpowers.git"
  ]
}
```

---

## 4. Platform notes

### Termux (Android)

Termux's bundled `npm` and `npx` may fail with `bad interpreter: /usr/bin/env`. That is why every skill here installs with `git` or `curl`; no npm step is required.

OfficeCLI (the `.docx/.xlsx/.pptx` binary) needs glibc and **does not run** on plain Termux. The `officecli` skill still installs, but the `officecli` command is only useful on Linux/macOS, or in Termux through `glibc-runner` / `proot-distro`. The glibc route is worth a try but is not guaranteed to work with this .NET binary; if it fails, use a Debian container:

```sh
dpkg --configure -a          # only if dpkg was interrupted before
pkg install -y glibc-runner
grun ~/.local/bin/officecli --version

# fallback if the command above fails
pkg install proot-distro
proot-distro install debian
```

### Linux / macOS

Every step in this README works as written. For OfficeCLI:

```sh
curl -fsSL https://raw.githubusercontent.com/iOfficeAI/OfficeCLI/main/install.sh | bash
```

---

## 5. Updates

Skills installed by clone:

```sh
git -C ~/.agents/skills/humanizer-id pull --ff-only
git -C ~/.agents/skills/stop-slop pull --ff-only
```

Skills copied from a temp clone have no git metadata; re-run their clone and `cp` steps from section 1 to update them.

OpenCode plugins:

```sh
opencode plugin update
```

With symlinks, updating once in `~/.agents/skills/` is enough; every agent sees the new version.

---

## 6. Verification

```sh
# every skill exists in the canonical location
ls ~/.agents/skills/*/SKILL.md

# other agents' symlinks point to the canonical location
ls -la ~/.claude/skills/
```

In the agent:

- OpenCode: `/skills`
- Antigravity CLI: `/skills` (existing session: `/skills reload`)
- Codex: type `$`
- Claude Code: `/skills` or ask "list your available skills"

If a skill does not appear: make sure the folder contains `SKILL.md` (not `skill.md`), the folder name matches `name` in the frontmatter, and the agent has been restarted.

---

## What this repo deliberately leaves out

- Installer script: every command is run manually from the README.
- Version pins: repos always track the latest version.
- Vendored skill files: each upstream repo remains the source of truth.

The trade-off: a new device needs a few copy-paste blocks, and this README can go stale if agent paths or commands change. If that becomes too tedious, add an `install.sh` to this repo.
