---
name: update-skill
description: Update any pipeline skill and ship the change as a git branch + pull request in one command. Use when the user says "update the <name> skill: <change>", "/update-skill <name> <change>", or asks to improve/fix how a pipeline command behaves.
---

# Update a skill — local edit + branch + PR in one command

Input: a skill name (one of the folders in `.claude/skills/`) and a
description of the change. The user runs one command; everything below is
automatic.

## Steps

1. **Read the target SKILL.md** and apply the requested change. Keep the
   skill's structure; never remove the pipeline guardrails (writer no-web
   rule, allowlist-only citations, gate order, Gate 4 blocking) unless the
   user explicitly and specifically asks — and restate the risk if they do.
2. **Show the user a short before/after summary** of what changed in the
   skill (not the whole file — just the changed behavior).
3. **Ship it on a branch** (never commit to main directly):
   ```
   git checkout -b skill/<skill-name>-<short-change-slug>
   git add .claude/skills/<skill-name>/
   git commit  (message: "Update <skill-name> skill: <change summary>")
   git push -u origin <branch>
   gh pr create  (title + body summarizing the behavior change)
   git checkout main
   ```
   Commit messages are plain — no AI co-author attribution or generated-with
   footers.
4. **Report** the PR link. The change goes live for everyone when the PR is
   merged and they pull (or when the plugin version is bumped, once the
   skills are packaged as a plugin).

## Fallbacks (handle gracefully, never fail silently)

- **No remote configured**: commit on the branch locally, tell the user the
  branch name and that a remote is needed (`git remote add origin <url>`)
  before it can be shared.
- **`gh` not authenticated**: push the branch, give the user the compare URL
  to open the PR manually.
- **Dirty working tree with unrelated changes**: only stage the skill folder;
  leave everything else untouched.
- The local session keeps using the updated skill immediately — the PR is for
  distributing the change to the rest of the org.
