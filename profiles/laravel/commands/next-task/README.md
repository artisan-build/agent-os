# next-task Command

## Purpose

Safely transitions from a completed task to the next task in a spec by merging the current task PR and starting implementation of the next task.

## When to Use

Use this command when:
- ✅ You've completed a task and created a PR
- ✅ The task PR has been reviewed (or you're ready to merge)
- ✅ You want to move on to the next task
- ✅ You're on the task branch for the completed task

## What It Does

1. **Checks current task PR** - finds PR for current branch
2. **Verifies safety** - ensures PR targets spec branch (NOT main)
3. **Optionally merges** - asks if you want to merge, then does it safely
4. **Deletes merged branch** - removes branch from GitHub after merge
5. **Updates local repo** - switches to spec branch, pulls, fetches, prunes
6. **Finds next task** - reads tasks.md to identify next uncompleted task
7. **Creates task branch** - creates new branch for next task
8. **Starts implementation** - begins implementing the next task
9. **Or reports completion** - if all tasks are done

## Critical Safety Rules

The command enforces these safety rules:

1. ⚠️ **NEVER merges into main branch**
2. ⚠️ **NEVER deletes an unmerged branch**
3. ⚠️ **ALWAYS ensures spec branch is up-to-date before new task branch**

Multiple checks throughout prevent accidents.

## Usage

### Multi-Agent Mode (Claude Code)

```bash
/next-task
```

### Single-Agent Mode

Run the command file:
```bash
agent-os/profiles/laravel/commands/next-task/single-agent/next-task.md
```

## Typical Workflow

### Scenario: Completing Task 1, Moving to Task 2

```bash
# You're on branch: 1-database-schema
# You've completed the task, created PR, had it reviewed

/next-task

# Agent checks PR status:
# "PR #124: Task 1 - Database Schema
#  Target: password-reset ✓
#  Status: Open
#
#  Would you like me to merge it? (yes/no)"

# You: "yes"

# Agent safely merges:
# "✓ Merged into password-reset
#  ✓ Deleted branch on GitHub
#  ✓ Updated local repository
#
#  Next: Task 2 - Authentication Logic
#  Ready to start? (yes/no)"

# You: "yes"

# Agent creates branch 2-authentication-logic and starts implementing
```

### Scenario: PR Already Merged

```bash
# PR was already merged (manually or by teammate)

/next-task

# Agent detects:
# "✓ PR already merged
#  Moving to next task...
#
#  Next: Task 2 - Authentication Logic
#  Ready to start? (yes/no)"

# Seamlessly continues to next task
```

### Scenario: All Tasks Complete

```bash
# On the last task branch

/next-task

# Agent checks tasks:
# "🎉 Spec Complete!
#
#  All 5 tasks have been completed and merged.
#
#  Next steps:
#  1. Run composer ready
#  2. Mark spec PR ready for review
#  3. Final review and merge to main
#
#  Great work!"

# Does NOT create a new task branch
```

## What Gets Checked

### PR Status Checks

- **PR exists** for current branch
- **PR targets spec branch** (not main) ✅ CRITICAL
- **PR is open or already merged** (not closed without merge)
- **PR is mergeable** (no conflicts, checks passing)

### Safety Checks

Before merging:
- Confirms PR base is spec branch (not main)
- Confirms PR is in mergeable state
- Asks user for confirmation

After merging:
- Deletes branch only after confirmed merge
- Pulls latest to get merged changes
- Verifies clean working directory

### Repository Checks

- Spec folder exists and is accessible
- Tasks file exists and is readable
- No uncommitted changes before switching branches
- Spec branch can be checked out and pulled

## Decision Points

The command stops and asks for user input at these points:

### 1. Should Merge PR?

```
Would you like me to merge it now? (yes/no)

⚠️  This will:
- Merge into [spec-branch] (NOT main)
- Delete branch on GitHub
- Start next task
```

**If "no"**: Stops, asks you to merge manually

**If "yes"**: Proceeds with safe merge

### 2. Start Next Task?

```
Next: Task [N] - [title]

Ready to start implementing? (yes/no)
```

**If "no"**: Stops on spec branch, you can run command later

**If "yes"**: Creates task branch and starts implementation

## Safety Features

### Multiple Main Branch Checks

The command checks PR base in **three places**:

1. **After getting PR data** - first check
2. **Before merging** - pre-merge verification
3. **In merge command** - final safety check

If ANY check detects main branch target:
```
❌ SAFETY ERROR: PR targets main branch

Task PRs should target spec branch, not main.

STOPPING - Cannot proceed.
```

### Only Deletes Merged Branches

Branch deletion happens as part of `gh pr merge --delete-branch`, which:
- Only executes if merge succeeds
- Only deletes after confirmed merge
- Never deletes unmerged branches

### Always Updates Spec Branch

Before creating new task branch:
```bash
git checkout [spec-branch]
git pull origin [spec-branch]
git fetch --prune origin
```

This ensures new task branch includes all previously merged work.

### Verifies Clean State

Before switching branches:
```bash
if ! git diff-index --quiet HEAD --; then
  echo "⚠️  Uncommitted changes detected"
  echo "Please commit or stash"
  exit 1
fi
```

## Error Handling

### No PR Found

```
❌ No PR found for branch: 1-database-schema

Expected to find PR for this branch.

Did you forget to create the PR?

Options:
1. Create PR now (if work is complete)
2. Skip to next task (abandon this work)
3. Handle manually
```

### PR Targets Main

```
❌ SAFETY ERROR: PR targets main branch

PR #124 targets: main

Task PRs should target the spec branch.

Cannot proceed. Please review PR setup.
```

### PR Has Conflicts

```
❌ Cannot merge - PR has conflicts

Mergeable state: CONFLICTING

Please resolve conflicts at: [PR URL]

Then run this command again.
```

### Merge Fails

```
❌ Merge failed

[Error from gh command]

You may need to:
- Resolve conflicts
- Wait for CI to pass
- Get required approvals
- Merge manually
```

### Spec Folder Not Found

```
❌ Cannot find spec folder

Looking for: agent-os/specs/*-password-reset/

Please verify spec folder exists.
```

### All Tasks Complete

```
🎉 Spec Complete!

All tasks merged into spec branch.

Next steps:
1. composer ready
2. Mark spec PR ready
3. Final review
4. Merge to main
```

## What Happens Step by Step

### 1. Verify Context

- Get current branch
- Confirm it's a task branch
- Find PR for branch

### 2. Check PR Status

- Get PR details via `gh pr`
- Verify PR targets spec branch (SAFETY CHECK)
- Check if already merged or needs merging

### 3. Merge if Needed

- Ask user for confirmation
- Verify mergeable state
- Execute merge: `gh pr merge --merge --delete-branch`
- Confirm success

### 4. Update Local Repository

- Switch to spec branch: `git checkout [spec]`
- Pull merged changes: `git pull origin [spec]`
- Fetch and prune: `git fetch --prune`
- Delete local task branch: `git branch -d [task]`

### 5. Find Next Task

- Locate spec folder
- Read tasks.md
- Find first uncompleted task
- Or detect all tasks complete

### 6. Start Next Task

- Ask user to confirm
- Create task branch: `git checkout -b [number]-[name]`
- Implement task (delegate to implementer)
- Create PR when done
- Switch back to spec branch

## Integration with Workflow

This command is the glue that connects completed tasks to new tasks:

```
Task 1 Branch → PR created → /next-task
                              ↓
                         Merge PR #124
                              ↓
                    Update spec branch
                              ↓
                  Create Task 2 branch → Implement
                                            ↓
                                      PR created → /next-task
                                                      ↓
                                                   (repeat)
```

## Example Session

```bash
# Scenario: Just finished Task 1

$ /next-task

📋 Current Task PR Status

PR #124: Task 1 - Database Schema
Branch: 1-database-schema
Target: password-reset ✓ (not main)
Status: Open - needs to be merged
URL: https://github.com/user/repo/pull/124

Would you like me to merge it now? (yes/no)

$ yes

✓ PR Merged Successfully
- PR #124 merged into password-reset
- Branch 1-database-schema deleted on GitHub

✓ Local Repository Updated
- Switched to password-reset
- Pulled latest changes
- Pruned deleted branches

📋 Next Task Ready

Progress: 1 of 5 tasks complete

Next: Task 2 - Authentication Logic

Subtasks:
- [ ] 2.1 Create authentication service
- [ ] 2.2 Add login/logout endpoints
- [ ] 2.3 Write authentication tests

Ready to start implementing? (yes/no)

$ yes

✓ Task Branch Created
Branch: 2-authentication-logic
Based on: password-reset (includes Task 1)

Starting implementation...

[Agent implements Task 2...]

✅ Task 2 Implementation Complete

Branch: 2-authentication-logic
PR: https://github.com/user/repo/pull/125

Next Steps:
1. Review PR
2. Merge when satisfied
3. Run /next-task for Task 3
```

## Command Options

None - the command is interactive and asks questions as needed.

## Related Commands

- `/create-spec` - Creates spec with draft PR (starts the workflow)
- `/implement-spec` - Implements tasks (manual version, one at a time)
- `/get-pr-feedback` - Address review feedback on task PRs
- `/composer-ready` - Fix quality issues before merging

## Tips

1. **Use after each task**: Makes workflow smooth and automatic
2. **Review before merging**: Even though command merges for you, review first
3. **Trust the safety checks**: Won't let you merge to main
4. **Let it clean up**: Handles branch deletion and repo updates
5. **Stop if unsure**: Can always say "no" and handle manually

## Benefits

- **Streamlined workflow**: One command to transition tasks
- **Safety built-in**: Multiple checks prevent main branch accidents
- **Clean repository**: Auto-deletes merged branches, keeps things tidy
- **Always up-to-date**: Ensures spec branch is current before new work
- **Detects completion**: Knows when spec is done, doesn't create extra branches
- **User in control**: Asks permission at key decision points

This command makes the sequential task workflow effortless while maintaining safety.
