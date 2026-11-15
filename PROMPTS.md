# Agent prompts

Standard prompts for agents that install, sync, or extend this setup. Copy one block into your agent. Every prompt reads `README.md`, which stays the source of truth for sources and commands.

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

Do not change anything until the user picks an action.

1. Read ~/.agents/skills/.sync-state. If it does not exist, say this is the first sync and treat the state as empty.
2. Use the current checkout if you are inside the repo. Otherwise clone it to a temporary directory. Fetch the latest main.
3. Read README.md. It is the source of truth for the skill and tool list, sources, and install commands.
4. Build the status report before changing anything:
   - List: every skill in README.md, including nested skills such as humanizer-id/jurnal, whether it exists in ~/.agents/skills, and every folder in ~/.agents/skills that README.md does not list.
   - Upstream: for each skill and tool, compare the state entry with its upstream source. For git entries, fetch the upstream repo and compare the recorded commit with its HEAD. List the commits that touch the skill or tool path, not every commit in the repo. For npm entries, compare the recorded version with the latest published version. For file entries, download the source and compare sha256. Mark each item as up-to-date, outdated (with the number of commits or the version gap), or unknown.
   - Drift: symlinks that do not point into ~/.agents/skills, and missing OpenCode plugin entries.
   Start the report with the last sync time and repo commit, then group it as: missing, extra, outdated, up-to-date, drift.
5. Ask:
   "Which sync do you want?
   1. Sync the list only (install missing skills, report extras)
   2. Sync upstream only (update installed skills and tools)
   3. Both
   I recommend both."
   Wait for the answer.
6. Apply only the chosen scope:
   - List: install missing skills with the commands in README.md section 1. Report extra folders. Delete an extra folder only after explicit confirmation.
   - Upstream: update installed skills and tools. Copied skill: remove the old folder after approval, then re-run its clone and copy step. Cloned skill: git -C ~/.agents/skills/<name> pull --ff-only. Curl skill: download the file again. Plugin or tool: show the per-agent command from README.md section 3 and wait for confirmation.
   - Both: run list, then upstream.
   Never vendor skill files into the repo.
7. Verify every installed skill has SKILL.md, including nested skills such as humanizer-id/jurnal, and every agent symlink points into ~/.agents/skills.
8. Write ~/.agents/skills/.sync-state:
   last_sync=<ISO 8601 timestamp with timezone>
   repo_commit=<full commit hash of the repo checkout>
   skill:<name>=git:<commit>
   skill:<name>=file:<sha256>
   tool:<name>=git:<commit>
   tool:<name>=npm:<version>
   tool:<name>=file:<sha256>
   Use the form that matches how the item is installed. Keep one line per entry.
9. Report the last sync time, what changed, what you applied, what you skipped, and what failed.
```

## Add skill

```text
Add a skill to https://github.com/dxnz-id/ai-agent-setup.

Read README.md and PROMPTS.md first. Input: <name>, <source URL>, <install method>.

Steps:
1. Install it in ~/.agents/skills/<name> and verify SKILL.md exists.
2. Add a row to the skill table in README.md and, if needed, a command block in section 1.
3. Update section 5 if the skill needs a special update command.
4. Run the read-only checks from step 4 of the Sync prompt.
5. Add skill:<name>=<git commit or file sha256> to ~/.agents/skills/.sync-state.
6. Show the diff. Do not commit or push unless asked.
```
