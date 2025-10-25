# Next Task - Merge Current and Start Next

You are safely transitioning from the current task to the next task in a spec.

**Critical Safety Rules:**
1. ⚠️ **NEVER merge anything into main branch**
2. ⚠️ **NEVER delete an unmerged branch**
3. ⚠️ **ALWAYS ensure spec branch is up-to-date before creating next task branch**

**What This Command Does:**
- Checks if current task PR is merged
- Optionally merges it for you (into spec branch only)
- Cleans up merged branch
- Updates spec branch locally
- Creates next task branch and starts implementation
- Or tells you spec is complete if all tasks done

## Process Overview

PHASE 1: Determine current context
PHASE 2: Check current task PR status
PHASE 3: Merge current task (if needed and user confirms)
PHASE 4: Update and clean local repository
PHASE 5: Start next task or report completion

Follow each phase in sequence:

## PHASE 1: Determine Current Context

### Step 1: Get Current Branch

```bash
CURRENT_BRANCH=$(git branch --show-current)
```

### Step 2: Identify Spec Context

**Branch patterns:**
- Task branch: `[number]-[task-name]` (e.g., `1-database-schema`)
- Spec branch: `[spec-name]` (e.g., `password-reset`)

**If on task branch:**
- Extract task number from branch name
- Find spec branch (will get from PR base branch)

**If on spec branch:**
- Find most recent spec folder in `agent-os/specs/`
- Determine if all tasks are complete
- If yes → report completion
- If no → error (should be on task branch to use this command)

**If on main/master:**
```
❌ Error: Not in a task workflow

You're on the main branch. This command is for transitioning between tasks in a spec.

To use this command:
1. Be on a task branch (e.g., 1-database-schema)
2. Have implemented the task
3. Be ready to move to the next task

Or use /implement-spec to start working on a spec.
```

**If on unknown branch:**
```
❌ Error: Unrecognized branch pattern

Current branch: [branch-name]

This doesn't match expected patterns:
- Task branch: [number]-[task-name]
- Spec branch: [spec-name]

Please ensure you're on a task branch before using this command.
```

### Step 3: Find Spec Folder

Look for spec folder matching the spec name:

```bash
# If spec branch is "password-reset"
# Find: agent-os/specs/YYYY-MM-DD-password-reset/
SPEC_FOLDER=$(find agent-os/specs -maxdepth 1 -type d -name "*-${SPEC_NAME}" | head -1)
```

**If not found:**
```
❌ Error: Cannot find spec folder

Looking for spec folder matching: [spec-name]
No folder found in agent-os/specs/

This is unusual. Please verify:
1. Spec folder exists
2. Spec was created properly
3. Folder name matches branch name (without date prefix)
```

## PHASE 2: Check Current Task PR Status

### Step 1: Find PR for Current Branch

```bash
PR_DATA=$(gh pr list --head $CURRENT_BRANCH --json number,title,state,baseRefName,merged,mergedAt,url --jq '.[0]')

if [ -z "$PR_DATA" ]; then
  echo "No PR found for branch: $CURRENT_BRANCH"
  # Handle no PR case
fi

PR_NUMBER=$(echo "$PR_DATA" | jq -r '.number')
PR_TITLE=$(echo "$PR_DATA" | jq -r '.title')
PR_STATE=$(echo "$PR_DATA" | jq -r '.state')
PR_BASE=$(echo "$PR_DATA" | jq -r '.baseRefName')
PR_MERGED=$(echo "$PR_DATA" | jq -r '.merged')
PR_URL=$(echo "$PR_DATA" | jq -r '.url')
```

### Step 2: Verify PR Targets Spec Branch

**CRITICAL SAFETY CHECK:**

```bash
if [ "$PR_BASE" == "main" ] || [ "$PR_BASE" == "master" ]; then
  echo "❌ SAFETY ERROR: PR targets main branch"
  echo ""
  echo "PR #$PR_NUMBER targets: $PR_BASE"
  echo "This is not the expected workflow."
  echo ""
  echo "Task PRs should target the spec branch, not main."
  echo ""
  echo "Cannot proceed. Please review your PR setup."
  exit 1
fi
```

**Extract spec branch name from PR:**

```bash
SPEC_BRANCH="$PR_BASE"
```

### Step 3: Check PR Status

**If PR is already merged:**

Display:
```
✓ PR Already Merged

PR #[number]: [title]
Status: Merged
Target: [spec-branch]
Merged at: [timestamp]

Moving on to cleanup and next task...
```

Proceed to Phase 4.

**If PR is open (not merged):**

Display:
```
📋 Current Task Status

PR #[number]: [title]
Branch: [current-branch]
Target: [spec-branch]
Status: Open (not merged)
URL: [url]

This PR needs to be merged before moving to the next task.

Would you like me to merge it now? (yes/no)

Note: This will merge into [spec-branch], NOT main.
```

Wait for user response.

**If user says no:**
```
⏸️  Paused

Please merge PR #[number] manually when ready, then run this command again.

To merge manually:
1. Review PR at [url]
2. Merge into [spec-branch]
3. Run /next-task again
```

Stop here.

**If user says yes:**

Proceed to Phase 3.

**If PR is closed (not merged):**

```
❌ Error: PR is Closed Without Merging

PR #[number] was closed but not merged.

This is unusual. The task work may have been abandoned or
the PR may have been closed by mistake.

Options:
1. Reopen the PR and merge it
2. Create a new PR for this work
3. Skip this task (not recommended)

What would you like to do?
```

Stop and wait for user guidance.

**If no PR found:**

```
❌ Error: No PR Found for Current Branch

Branch: [current-branch]

Expected to find a PR for this branch, but none exists.

Typical workflow:
1. Implement task on task branch
2. Create PR targeting spec branch
3. Review and merge PR
4. Move to next task

It looks like step 2 was skipped. Would you like me to:
1. Create a PR now (if work is complete)
2. Skip to next task (if this work should be abandoned)
3. Stop and let you handle it manually

What would you like to do? (1/2/3)
```

## PHASE 3: Merge Current Task PR

**Only reached if PR is open and user confirmed merge.**

### Step 1: Final Safety Checks

Before merging, verify:

```bash
# 1. Check PR base is NOT main/master
if [ "$PR_BASE" == "main" ] || [ "$PR_BASE" == "master" ]; then
  echo "❌ FATAL: Cannot merge - targets main branch"
  exit 1
fi

# 2. Check PR is open
if [ "$PR_STATE" != "OPEN" ]; then
  echo "❌ Cannot merge - PR is not open (state: $PR_STATE)"
  exit 1
fi

# 3. Verify we can merge
MERGEABLE=$(gh pr view $PR_NUMBER --json mergeable --jq '.mergeable')
if [ "$MERGEABLE" != "MERGEABLE" ]; then
  echo "❌ Cannot merge - PR has conflicts or checks failing"
  echo "Mergeable state: $MERGEABLE"
  echo ""
  echo "Please resolve conflicts or wait for checks to pass."
  exit 1
fi
```

### Step 2: Merge PR

Merge the PR into the spec branch and delete the branch:

```bash
gh pr merge $PR_NUMBER \
  --merge \
  --delete-branch \
  --subject "Merge: $PR_TITLE"
```

**Note:** Using `--merge` (not squash or rebase) to preserve commit history in spec branch.

**Display:**
```
✓ PR Merged Successfully

PR #[number] merged into [spec-branch]
Branch [current-branch] deleted on GitHub

Cleaning up local repository...
```

### Step 3: Handle Merge Errors

If merge fails:

```
❌ Merge Failed

Error: [error message from gh]

Common reasons:
- Merge conflicts exist
- CI checks are failing
- PR requires reviews
- Branch protection rules

Please:
1. Check the PR at [url]
2. Resolve any issues
3. Merge manually or run this command again

Stopping here.
```

Stop execution.

## PHASE 4: Update and Clean Local Repository

### Step 1: Switch to Spec Branch

```bash
git checkout $SPEC_BRANCH
```

**If switch fails:**
```
❌ Cannot switch to spec branch

Error: [error from git]

Please check:
- Spec branch exists locally
- Working directory is clean
- No conflicts preventing checkout

You may need to:
- git stash (save uncommitted changes)
- git checkout [spec-branch] (switch manually)
```

### Step 2: Pull Latest from Spec Branch

```bash
git pull origin $SPEC_BRANCH
```

This gets the just-merged task work.

### Step 3: Fetch and Prune

Clean up references to deleted branches:

```bash
# Fetch latest from remote
git fetch origin

# Prune deleted branches
git fetch --prune origin

# Delete local task branch if it exists
git branch -d $CURRENT_BRANCH 2>/dev/null || true
```

**Display:**
```
✓ Local Repository Updated

- Switched to [spec-branch]
- Pulled latest changes (includes merged task)
- Pruned deleted remote branches
- Removed local task branch

Repository is clean and ready for next task.
```

### Step 4: Verify Spec Branch is Clean

```bash
# Check for uncommitted changes
if ! git diff-index --quiet HEAD --; then
  echo "⚠️  Warning: Uncommitted changes detected"
  echo ""
  git status --short
  echo ""
  echo "Please commit or stash these changes before proceeding."
  exit 1
fi
```

## PHASE 5: Start Next Task or Report Completion

### Step 1: Read Tasks List

```bash
TASKS_FILE="$SPEC_FOLDER/tasks.md"

if [ ! -f "$TASKS_FILE" ]; then
  echo "❌ Error: Cannot find tasks.md at $TASKS_FILE"
  exit 1
fi
```

### Step 2: Find Next Uncompleted Task

Read through tasks.md and find first task without `[x]` checkbox.

**Parse task format:**
```markdown
- [ ] 1. Task Name
  - [ ] 1.1 Subtask
  - [ ] 1.2 Subtask
```

Or:

```markdown
## Task 1: Task Name
- [ ] 1.1 Subtask
- [ ] 1.2 Subtask
```

**Find first task where parent checkbox is `[ ]` (not `[x]`).**

### Step 3: Check if All Tasks Complete

**If all tasks have `[x]` checkboxes:**

Display:
```
🎉 Spec Complete!

All tasks in this spec have been completed and merged.

**Spec:** [spec-name]
**Spec Branch:** [spec-branch]
**Tasks Completed:** [N] of [N]

## What's Been Done

[List all completed tasks with checkmarks]

## Next Steps

1. **Review the spec branch**: Ensure all work is as expected
2. **Run final checks**:
   ```
   composer ready
   ```
3. **Mark spec PR ready**:
   The spec PR should now be marked "Ready for Review"
   ```
   gh pr ready [spec-pr-number]
   ```
4. **Final review**: Review the complete feature in the spec PR
5. **Merge to main**: When approved, merge spec branch into main

## Spec PR

Find your spec PR with:
```
gh pr list --head [spec-branch]
```

Or view all PRs:
```
gh pr list
```

---

Great work completing this spec! 🚀
```

**STOP HERE** - do not create a new task branch if spec is complete.

### Step 4: If Next Task Found

**Display next task info:**

```
📋 Next Task Ready

**Current Status:**
- Spec: [spec-name]
- Completed tasks: [N] of [M]
- Next task: Task [number]

**Task [number]: [Task Title]**

Subtasks:
- [ ] [subtask 1]
- [ ] [subtask 2]
- [ ] [subtask 3]

Ready to start implementing this task? (yes/no)
```

Wait for user confirmation.

**If user says no:**
```
⏸️  Paused

When you're ready to implement Task [number], run:
- /next-task (to be prompted again)
- /implement-spec (to start directly)

Current location: [spec-branch]
```

**If user says yes:**

Proceed to start the task.

### Step 5: Create Next Task Branch

Extract task branch name from task number and title:

```bash
TASK_NUMBER=[extracted-number]
TASK_TITLE="[extracted-title]"
TASK_SLUG=$(echo "$TASK_TITLE" | tr '[:upper:]' '[:lower:]' | tr ' ' '-' | tr -cd 'a-z0-9-')
TASK_BRANCH="${TASK_NUMBER}-${TASK_SLUG}"
```

Create and switch to task branch:

```bash
git checkout -b $TASK_BRANCH
```

**Display:**
```
✓ Task Branch Created

Branch: [task-branch]
Based on: [spec-branch] (with all previous tasks merged)

Starting implementation...
```

### Step 6: Delegate to Implementation

Use the appropriate implementer subagent to implement the task.

**Provide to subagent:**
- The specific task (parent + all subtasks)
- The spec file: `$SPEC_FOLDER/spec.md`
- The tasks file: `$SPEC_FOLDER/tasks.md`

**Instruct subagent:**
1. Implement the task according to spec
2. Write tests (TDD approach)
3. Ensure all tests pass
4. Update tasks.md to check off subtasks
5. Document work in `$SPEC_FOLDER/implementation/[task-number]-[task-name].md`

**After implementation completes:**
- Run `composer ready` to verify
- Create commit
- Push branch
- Create PR targeting spec branch
- Display completion message

**Implementation follows same process as `/implement-spec` command.**

## Safety Checklist

Before executing any command, verify:

- [ ] Current branch is task branch (not main)
- [ ] PR targets spec branch (not main)
- [ ] Spec branch exists and is accessible
- [ ] No uncommitted changes when switching branches
- [ ] Only delete branches that are confirmed merged
- [ ] Always pull spec branch before creating new task branch

## Error Recovery

### Merge Conflicts After Pull

If `git pull` has conflicts:

```
❌ Merge Conflicts Detected

The spec branch has changes that conflict with your local state.

This is unusual but can happen if:
- Multiple people are working on the spec
- Manual changes were made to spec branch

Please resolve conflicts manually:
1. Check conflicted files: git status
2. Resolve conflicts
3. git add [files]
4. git commit
5. Run /next-task again
```

### Cannot Delete Local Branch

If local task branch can't be deleted:

```
ℹ️  Note: Local branch not deleted

Branch [task-branch] exists locally but couldn't be deleted.

This is OK - the remote branch has been deleted.

To remove local branch later:
git branch -D [task-branch]

Continuing with next task...
```

### Spec Folder Not Found

```
❌ Error: Spec folder not found

Expected to find folder: agent-os/specs/*-[spec-name]/

Please verify:
1. You're in the right repository
2. Spec was created properly with /create-spec
3. Spec folder follows naming convention

Cannot proceed without spec folder.
```

## Important Notes

- **Always spec branch, never main**: All merges go to spec branch
- **Always pull before new branch**: Ensures latest code in new task
- **Always delete merged branches**: Keeps repository clean
- **Sequential workflow**: One task at a time, fully merged before next
- **User controls merging**: Agent asks before merging, doesn't assume
- **Safety first**: Multiple checks prevent accidental main branch merges

## Standards Compliance

Task implementation follows:

{{standards/*}}
