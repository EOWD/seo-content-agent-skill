---
name: update-skill
description: Update any pipeline skill and ship the change as a git branch + pull request in one command. Use when the user says "update the <name> skill: <change>", "/update-skill <name> <change>", or asks to improve/fix how a pipeline command behaves.
---

# Update a skill — local edit + branch + PR in one command

Input: a skill name (one of the folders in `.claude/skills/`) and a
description of the change. The user runs one command; everything below is
automatic.

## Steps

0. **Critical-change soft check.** If the requested change touches a
   non-negotiable guardrail (writer no-web rule, allowlist-only citations,
   product-fact sourcing, gate order, Gate 4 blocking — the list in
   HANDOVER.md §5) or otherwise changes what the pipeline is allowed to
   publish without human review, treat it as critical: ask the user to
   confirm with the passphrase in `.env`'s `SKILL_UPDATE_PASSPHRASE` before
   applying anything. This is a deliberate-confirmation step, not a real
   access control — it exists so a critical guardrail change is never applied
   from an offhand or ambiguous instruction. If `SKILL_UPDATE_PASSPHRASE` is
   unset in `.env`, there's nothing to confirm against — skip the check and
   proceed. Non-critical changes (wording, thresholds, formatting, new
   non-guardrail behavior) never need this — don't ask for it reflexively.
1. **Read the target SKILL.md** (in `skills/<skill-name>/`) and apply the
   requested change. Keep the skill's structure; never remove the pipeline
   guardrails (writer no-web rule, allowlist-only citations, gate order,
   Gate 4 blocking) unless the user explicitly and specifically asks — and
   restate the risk if they do.
2. **Bump the plugin version** in `.claude-plugin/plugin.json` (patch bump
   for tweaks, minor for new behavior) so installed copies pick the change
   up via `/plugin update` after the PR merges.
3. **Show the user a short before/after summary** of what changed in the
   skill (not the whole file — just the changed behavior).
4. **Ship it on a branch** (never commit to main directly):
   ```
   git checkout -b skill/<skill-name>-<short-change-slug>
   git add skills/<skill-name>/ .claude-plugin/plugin.json
   git commit  (message: "Update <skill-name> skill: <change summary>")
   git push -u origin <branch>
   gh pr create  (title + body summarizing the behavior change)
   git checkout main
   ```
   Commit messages are plain — no AI co-author attribution or generated-with
   footers.
5. **Report** the PR link. The change goes live for everyone when the PR
   merges and they run `/plugin update seo-content-engine` (or re-pull, if
   they use the repo directly).

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
