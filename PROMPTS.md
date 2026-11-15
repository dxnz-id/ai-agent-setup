# Agent prompts

Standard prompts for agents that install, sync, verify, or extend this setup. Copy one block into your agent. Every prompt reads `README.md`, which stays the source of truth for sources and commands.

- Repo: <https://github.com/dxnz-id/ai-agent-setup>
- Canonical skill folder: `~/.agents/skills/`
- Sync state: `~/.agents/skills/.sync-state`

## Install

```text
Install the AI agent setup from https://github.com/dxnz-id/ai-agent-setup on this machine.

Read README.md and run sections 1 to 4 in order. Use the state format from the Sync section of PROMPTS.md.

Rules:
- Install skills only in ~/.agents/skills/. Never copy skill files into a project.
- Install and symlink one skill folder at a time. Never symlink the whole skills directory.
- Ask before installing plugins or editing agent config.
- Skip what is already installed and say so.
- Do not use npm or npx on Termux.

When finished:
1. Run the checks in README.md section 6.
2. Create ~/.agents/skills/.sync-state using the format in the Sync prompt below.
3. Report installed, skipped, failed, and the state file path.
```

## Sync

```text
Sync this machine's AI agent setup with https://github.com/dxnz-id/ai-agent-setup.

Ask this before doing anything else:
"Which change report do you want?
1. Repo changes since the last sync
2. Upstream commits for each skill and tool
3. Both
I recommend both."
Wait for the answer.

Then:
1. Read ~/.agents/skills/.sync-state. If it does not exist, say this is the first sync and treat the state as empty.
2. Use the current checkout if you are inside the repo. Otherwise clone it to a temporary directory. Fetch the latest main. A shallow clone may not contain the history needed for the repo diff; run git fetch --unshallow or clone again without --depth.
3. Read README.md. It is the source of truth for the skill and tool list, sources, and install commands.
4. Build the report:
   - Repo changes: git log --oneline <repo_commit>..HEAD -- README.md PROMPTS.md, then summarize added, removed, or changed skills and tools.
   - Upstream changes: compare each state entry with its upstream source. For git entries, compare the recorded commit with git ls-remote <source> HEAD and list the commits that touch the skill or tool path, not every commit in the repo. For npm entries, compare the recorded version with the latest published version. For file entries, download the source and compare sha256.
   - Local drift: skills listed in README.md but missing from ~/.agents/skills, folders in ~/.agents/skills that README.md does not list, and symlinks that do not point into ~/.agents/skills.
5. Show the report before changing anything. Group it as: new, removed, source changed, upstream updated, local drift.
6. Ask before deleting a skill folder or symlink, overwriting a real folder that is not a symlink, or updating a plugin or tool.
7. Apply only approved changes:
   - Missing skill: run its command from README.md section 1.
   - Copied skill: remove the old folder after approval, then re-run its clone and copy step.
   - Cloned skill: git -C ~/.agents/skills/<name> pull --ff-only.
   - Curl skill: download the file again.
   - Plugin or tool: show the per-agent command from README.md section 3 and wait for confirmation.
   Never vendor skill files into the repo.
8. Verify every installed skill has SKILL.md, including nested skills such as humanizer-id/jurnal, and every agent symlink points into ~/.agents/skills.
9. Write ~/.agents/skills/.sync-state:
   last_sync=<ISO 8601 timestamp with timezone>
   repo_commit=<full commit hash of the repo checkout>
   skill:<name>=git:<commit>
   skill:<name>=file:<sha256>
   tool:<name>=git:<commit>
   tool:<name>=npm:<version>
   tool:<name>=file:<sha256>
   Use the form that matches how the item is installed. Keep one line per entry.
10. Report the last sync time, what changed, what you applied, what you skipped, and what failed.
```

## Verify

```text
Verify the AI agent setup on this machine without changing anything.

Read README.md sections 1, 2, and 6. Then check:
- every skill listed in README.md, including nested skills such as humanizer-id/jurnal, exists and contains SKILL.md
- no extra skill folders exist in ~/.agents/skills
- each Claude Code and Antigravity symlink points into ~/.agents/skills, one folder per skill
- ~/.config/opencode/opencode.jsonc contains the ponytail and superpowers plugins
- ~/.agents/skills/.sync-state exists and parses as key=value lines

Report every gap with the exact command that fixes it. Do not run the fixes.
```

## Add skill

```text
Add a skill to https://github.com/dxnz-id/ai-agent-setup.

Read README.md and PROMPTS.md first. Input: <name>, <source URL>, <install method>.

Steps:
1. Install it in ~/.agents/skills/<name> and verify SKILL.md exists.
2. Add a row to the skill table in README.md and, if needed, a command block in section 1.
3. Update section 5 if the skill needs a special update command.
4. Run the Verify prompt checks.
5. Add skill:<name>=<git commit or file sha256> to ~/.agents/skills/.sync-state.
6. Show the diff. Do not commit or push unless asked.
```
