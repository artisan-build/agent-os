# Next Task - Merge Current and Start Next (Single-Agent)

This command safely transitions from the current completed task to the next task in a spec.

**Critical Safety Rules:**
1. ⚠️ **NEVER merge anything into main branch**
2. ⚠️ **NEVER delete an unmerged branch**
3. ⚠️ **ALWAYS ensure spec branch is up-to-date before creating next task branch**

## What This Does

- Checks current task PR status
- Optionally merges it for you (into spec branch ONLY)
- Deletes merged branch on GitHub
- Updates spec branch locally
- Creates next task branch
- Starts implementing next task
- Or reports spec completion if all tasks done

## Process

### Step 1: Verify Current Branch

Get current branch:
```bash
CURRENT_BRANCH=$(git branch --show-current)
echo "Current branch: $CURRENT_BRANCH"
```

**Expected patterns:**
- Task branch: `[number]-[task-name]` (e.g., `1-database-schema`)
- Spec branch: `[spec-name]` (e.g., `password-reset`)

**If on main/master:**
```
❌ Error: Not in Task Workflow

You're on the main branch.

This command is for transitioning between tasks in a spec.

To use:
1. Be on a task branch (e.g., 1-database-schema)
2. Have completed the task
3. Be ready to move to next task
```
Stop here.

**If on spec branch:**
Check if all tasks are complete. If yes, report completion. If no, error (should be on task branch).

**If on unrecognized branch:**
```
❌ Error: Unrecognized Branch Pattern

Current: [branch]

Expected:
- Task branch: [number]-[task-name]
- Spec branch: [spec-name]

Cannot proceed.
```
Stop here.

### Step 2: Find PR for Current Branch

```bash
PR_DATA=$(gh pr list --head $CURRENT_BRANCH --json number,title,state,baseRefName,merged,mergedAt,url --jq '.[0]')

if [ -z "$PR_DATA" ]; then
  echo "❌ Error: No PR found for branch $CURRENT_BRANCH"
  exit 1
fi

# Extract PR details
PR_NUMBER=$(echo "$PR_DATA" | jq -r '.number')
PR_TITLE=$(echo "$PR_DATA" | jq -r '.title')
PR_STATE=$(echo "$PR_DATA" | jq -r '.state')
PR_BASE=$(echo "$PR_DATA" | jq -r '.baseRefName')
PR_MERGED=$(echo "$PR_DATA" | jq -r '.merged')
PR_URL=$(echo "$PR_DATA" | jq -r '.url')
```

### Step 3: CRITICAL SAFETY CHECK - Verify PR Base

**This is the most important check:**

```bash
if [ "$PR_BASE" == "main" ] || [ "$PR_BASE" == "master" ]; then
  echo "❌ SAFETY ERROR: PR Targets Main Branch"
  echo ""
  echo "PR #$PR_NUMBER targets: $PR_BASE"
  echo "URL: $PR_URL"
  echo ""
  echo "Task PRs should target the spec branch, NOT main."
  echo "This is incorrect workflow setup."
  echo ""
  echo "STOPPING - Cannot proceed with merge."
  exit 1
fi
```

**Extract spec branch from PR base:**
```bash
SPEC_BRANCH="$PR_BASE"
echo "Spec branch: $SPEC_BRANCH"
```

### Step 4: Check PR Status

**If PR is already merged ($PR_MERGED == "true"):**

Display:
```
✓ PR Already Merged

PR #[number]: [title]
Merged into: [spec-branch]
Merged at: [timestamp]

Moving to cleanup and next task...
```

Skip to Step 6.

**If PR is open ($PR_STATE == "OPEN" and $PR_MERGED == "false"):**

Display:
```
📋 Current Task PR Status

PR #[number]: [title]
Branch: [current-branch]
Target: [spec-branch] ✓ (not main)
Status: Open - needs to be merged
URL: [url]

This PR must be merged before moving to the next task.

Would you like me to merge it now? (yes/no)

⚠️  This will:
- Merge into [spec-branch] (NOT main)
- Delete branch [current-branch] on GitHub
- Update local repository
- Start next task
```

**Wait for user response.**

**If user says "no":**
```
⏸️  Paused

Please merge PR #[number] when ready:
[url]

After merging, run this command again to continue to next task.
```
Stop here.

**If user says "yes":**
Continue to Step 5.

**If PR is closed but not merged:**
```
❌ Error: PR Closed Without Merge

PR #[number] was closed but not merged.

Cannot automatically proceed.

Options:
1. Reopen and merge the PR
2. Create new PR for this work
3. Abandon this task (not recommended)

Please handle manually.
```
Stop here.

### Step 5: Merge PR (User Confirmed)

**Final pre-merge checks:**

```bash
# Verify PR is mergeable
MERGEABLE=$(gh pr view $PR_NUMBER --json mergeable --jq '.mergeable')

if [ "$MERGEABLE" != "MERGEABLE" ]; then
  echo "❌ Cannot Merge"
  echo ""
  echo "PR state: $MERGEABLE"
  echo ""
  echo "Reasons this might fail:"
  echo "- Merge conflicts exist"
  echo "- CI checks are failing"
  echo "- Required reviews not approved"
  echo ""
  echo "Please resolve issues at: $PR_URL"
  echo "Then run this command again."
  exit 1
fi

# Double-check not merging to main
if [ "$PR_BASE" == "main" ] || [ "$PR_BASE" == "master" ]; then
  echo "❌ FATAL: Merge blocked - would merge to main"
  exit 1
fi
```

**Execute merge:**

```bash
echo "Merging PR #$PR_NUMBER into $SPEC_BRANCH..."

gh pr merge $PR_NUMBER \
  --merge \
  --delete-branch \
  --subject "Merge: $PR_TITLE"

if [ $? -ne 0 ]; then
  echo "❌ Merge failed"
  echo "Check error above and resolve issues."
  echo "You may need to merge manually: $PR_URL"
  exit 1
fi
```

**Display success:**
```
✓ PR Merged Successfully

- PR #[number] merged into [spec-branch]
- Branch [current-branch] deleted on GitHub

Updating local repository...
```

### Step 6: Update Local Repository

**Switch to spec branch:**

```bash
echo "Switching to spec branch: $SPEC_BRANCH"
git checkout $SPEC_BRANCH

if [ $? -ne 0 ]; then
  echo "❌ Cannot switch to spec branch"
  echo "Please check for uncommitted changes or conflicts"
  exit 1
fi
```

**Pull latest (includes merged task):**

```bash
echo "Pulling latest from $SPEC_BRANCH..."
git pull origin $SPEC_BRANCH

if [ $? -ne 0 ]; then
  echo "❌ Pull failed - may have conflicts"
  echo "Please resolve manually and run command again"
  exit 1
fi
```

**Fetch and prune:**

```bash
echo "Fetching and pruning..."
git fetch origin
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

Repository is clean and ready.
```

**Verify clean state:**

```bash
if ! git diff-index --quiet HEAD --; then
  echo "⚠️  Warning: Uncommitted changes detected"
  echo ""
  git status --short
  echo ""
  echo "Please commit or stash before proceeding."
  exit 1
fi
```

### Step 7: Find Next Task

**Locate spec folder:**

```bash
# Spec branch "password-reset" → folder "*-password-reset"
SPEC_FOLDER=$(find agent-os/specs -maxdepth 1 -type d -name "*-${SPEC_BRANCH}" 2>/dev/null | head -1)

if [ -z "$SPEC_FOLDER" ]; then
  echo "❌ Error: Cannot find spec folder"
  echo "Looking for: agent-os/specs/*-${SPEC_BRANCH}/"
  echo "Please verify spec folder exists"
  exit 1
fi

TASKS_FILE="$SPEC_FOLDER/tasks.md"

if [ ! -f "$TASKS_FILE" ]; then
  echo "❌ Error: Cannot find $TASKS_FILE"
  exit 1
fi
```

**Read tasks.md and find next uncompleted task:**

Look for first task where parent checkbox is `[ ]` (not `[x]`).

Task formats to recognize:
```markdown
- [ ] 1. Task Name
  - [ ] 1.1 Subtask
```

Or:
```markdown
## Task 1: Task Name
- [ ] 1.1 Subtask
```

**Parse to extract:**
- Task number
- Task title
- All subtasks

### Step 8: Check Completion Status

**If all tasks have [x] checkboxes:**

Display:
```
🎉 Spec Complete!

All tasks have been completed and merged into the spec branch.

**Spec:** [spec-name]
**Spec Branch:** [spec-branch]
**Completed Tasks:** [N] of [N]

## Tasks Completed

[List all tasks with ✓]

## Next Steps

The spec is ready for final review:

1. **Run final quality check:**
   ```
   composer ready
   ```

2. **Find spec PR:**
   ```
   gh pr list --head [spec-branch]
   ```

3. **Mark PR ready for review:**
   ```
   gh pr ready [pr-number]
   ```

4. **Review complete feature** in spec PR

5. **Merge into main** when approved

---

Congratulations on completing this spec! 🚀
```

**STOP HERE** - do not create new branch.

### Step 9: Start Next Task

**If next task found, display:**

```
📋 Next Task Ready

**Progress:** [N] of [M] tasks complete

**Next: Task [number] - [title]**

Subtasks:
- [ ] [subtask 1]
- [ ] [subtask 2]
- [ ] [subtask 3]

Ready to start implementing? (yes/no)
```

**Wait for user confirmation.**

**If user says "no":**
```
⏸️  Paused

When ready to implement Task [number]:
- Run /next-task again
- Or run /implement-spec

Currently on: [spec-branch]
```
Stop here.

**If user says "yes":**
Continue to create task branch and implement.

### Step 10: Create Task Branch

**Generate branch name:**

```bash
TASK_NUMBER=[extracted-number]
TASK_TITLE="[extracted-title]"

# Convert to kebab-case
TASK_SLUG=$(echo "$TASK_TITLE" | tr '[:upper:]' '[:lower:]' | tr ' ' '-' | tr -cd 'a-z0-9-')

TASK_BRANCH="${TASK_NUMBER}-${TASK_SLUG}"

echo "Creating task branch: $TASK_BRANCH"
```

**Create and switch to branch:**

```bash
git checkout -b $TASK_BRANCH

if [ $? -ne 0 ]; then
  echo "❌ Cannot create branch"
  echo "Branch may already exist or other git issue"
  exit 1
fi
```

**Display:**
```
✓ Task Branch Created

Branch: [task-branch]
Based on: [spec-branch] (includes all previous merged tasks)

Ready to implement...
```

### Step 11: Implement Task

Follow the task implementation workflow:

```
{{workflows/implementation/implement-task}}
```

**Implementation steps:**
1. Write tests first (TDD)
2. Implement functionality
3. Ensure all tests pass
4. Run `composer ready` until clean
5. Update tasks.md with completions
6. Document in implementation/[task-number]-[task-name].md

**After implementation:**
1. Commit changes
2. Push branch
3. Create PR targeting spec branch:

```bash
# Get spec PR number for reference
SPEC_PR=$(gh pr list --head $SPEC_BRANCH --json number --jq '.[0].number')

gh pr create \
  --base $SPEC_BRANCH \
  --title "Task $TASK_NUMBER: $TASK_TITLE" \
  --body "Implements Task $TASK_NUMBER from spec

Part of spec PR: #$SPEC_PR

[Implementation details]

✅ Tests passing
✅ composer ready clean

🤖 Generated with Claude Code"
```

4. Switch back to spec branch
5. Display completion message

**Completion message:**

```
✅ Task [number] Implementation Complete

**Branch:** [task-branch]
**PR:** [url] → targets [spec-branch]

### Next Steps

1. Review PR at [url]
2. Merge PR when satisfied
3. Run /next-task to continue to next task

**Progress:** Task [N] of [M] complete
```

## Safety Checks Summary

Throughout this process, the following safety checks are enforced:

1. ✅ **Never merge to main**: Multiple checks verify PR base is spec branch
2. ✅ **Only delete merged branches**: Branch deleted only after confirmed merge
3. ✅ **Spec branch always updated**: Pull before creating new task branch
4. ✅ **Clean working directory**: Verify no uncommitted changes before switching
5. ✅ **User confirmation**: Ask before merging, ask before starting next task

## Error Recovery

### Merge Conflicts

If `git pull` has conflicts:
```
❌ Conflicts Detected

Resolve conflicts manually:
1. git status (see conflicted files)
2. Edit and resolve conflicts
3. git add [files]
4. git commit
5. Run /next-task again
```

### Cannot Delete Branch

If local branch won't delete:
```
ℹ️  Local branch remains

Remote branch deleted, but local branch exists.

To remove later: git branch -D [branch]

Continuing...
```

### Missing Spec Folder

```
❌ Cannot find spec folder

Expected: agent-os/specs/*-[spec-name]/

Verify:
1. Spec created with /create-spec
2. Folder exists and follows naming
3. You're in correct repository
```

## Important Notes

- **Sequential workflow**: Complete one task, merge, then next
- **Always ask first**: Never assumes user wants to merge or continue
- **Safety paramount**: Multiple checks prevent main branch merges
- **Clean repository**: Always update before new work
- **User in control**: Stops and waits at key decision points

## Standards Compliance

{{standards/*}}
