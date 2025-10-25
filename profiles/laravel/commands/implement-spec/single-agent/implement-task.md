# Implement Single Task (Single-Agent Mode)

This command implements ONE task from a spec using the sequential git branching workflow.

**Use this command** when you're ready to implement the next task in your spec. Each task will be done in its own branch with its own PR targeting the spec branch.

## Before You Begin

Verify:
- Spec has been created and has a draft PR
- Spec branch exists (named without date prefix)
- Previous task PR (if any) has been merged into spec branch

## Task Implementation Process

### Step 1: Verify Spec Branch and Identify Next Task

1. **Determine spec name** from spec folder:
   - Find spec folder: `agent-os/specs/YYYY-MM-DD-spec-name/`
   - Extract branch name: Remove date prefix → `spec-name`

2. **Check current git branch**:
   ```bash
   git branch --show-current
   ```

   **If on spec branch**: Continue to next step

   **If on main/master**:
   ```bash
   git checkout [spec-name]
   ```

   **If on task branch**: **STOP** and display:
   ```
   ⚠️  Currently on task branch: [branch-name]

   You need to:
   1. Finish the current task
   2. Create and merge its PR
   3. Switch back to the spec branch

   Then run this command again.
   ```

3. **Pull latest from spec branch**:
   ```bash
   git pull origin [spec-name]
   ```

4. **Identify next task**:
   - Open `agent-os/specs/[spec-folder]/tasks.md`
   - Find first task group without `[x]` checkbox
   - Note task number and title

5. **Confirm with user**:
   ```
   Ready to implement: **Task [N]: [Task Title]**

   This task includes these subtasks:
   - [ ] [subtask 1]
   - [ ] [subtask 2]
   - [ ] [subtask 3]

   Shall I proceed with implementing this task? (yes/no)
   ```

   **WAIT for explicit user confirmation**. Do not proceed without "yes".

### Step 2: Create Task Branch

1. **Create branch name**:
   - Format: `[task-number]-[task-name-slug]`
   - Example: Task "1: Database Schema Updates" → `1-database-schema-updates`

2. **Create and checkout task branch**:
   ```bash
   git checkout -b [task-number]-[task-name-slug]
   ```

   You are now on the task branch, ready to implement.

### Step 3: Implement the Task

Follow the implementation workflow:

```
{{workflows/implementation/implement-task}}
```

**Key steps**:
- Write tests first (TDD)
- Implement functionality
- Ensure all tests pass
- Run `composer ready` and fix any issues
- Check off subtasks in `tasks.md` as you complete them
- Document your work

### Step 4: Document Implementation

Follow the documentation workflow:

```
{{workflows/implementation/document-implementation}}
```

Create implementation report at:
`agent-os/specs/[spec-folder]/implementation/[task-number]-[task-name].md`

### Step 5: Update Tasks List

Follow the update workflow:

```
{{workflows/implementation/update-tasks-list}}
```

Mark the parent task and all subtasks as complete in `tasks.md`.

### Step 6: Commit Changes

1. **Stage all changes**:
   ```bash
   git add .
   ```

2. **Commit with descriptive message**:
   ```bash
   git commit -m "Complete Task [N]: [Task Title]

   [Brief 1-2 sentence description of what was implemented]

   Subtasks completed:
   - [subtask 1]
   - [subtask 2]
   - [subtask 3]

   Tests: All passing
   Code quality: composer ready passes

   🤖 Generated with Claude Code (https://claude.com/claude-code)

   Co-Authored-By: Claude <noreply@anthropic.com>"
   ```

### Step 7: Push Task Branch

```bash
git push -u origin [task-number]-[task-name-slug]
```

### Step 8: Create PR Targeting Spec Branch

**Important**: The PR must target the **spec branch**, NOT main.

1. **Get spec PR number** (for reference in task PR):
   ```bash
   gh pr list --head [spec-name] --json number --jq '.[0].number'
   ```

2. **Create PR**:
   ```bash
   gh pr create \
     --base [spec-name] \
     --title "Task [N]: [Task Title]" \
     --body "$(cat <<'EOF'
   ## Task Overview

   Implements Task [N] from the [spec-name] specification.

   ### What Was Implemented

   [Brief description of changes]

   ### Key Changes

   **Files Created:**
   - [list new files]

   **Files Modified:**
   - [list modified files]

   **Database Changes:**
   - [list migrations/schema changes, or "None"]

   ### Testing

   - [x] All new tests pass
   - [x] All existing tests pass
   - [x] Manual testing completed
   - [x] `composer ready` passes (no lint/type errors)

   ### Documentation

   - [x] Implementation documented in `implementation/[task-number]-[task-name].md`
   - [x] Tasks list updated with completion checkmarks

   ---

   **Part of spec PR:** #[spec-pr-number]

   🤖 Generated with Claude Code (https://claude.com/claude-code)
   EOF
   )"
   ```

   This creates a **regular PR** (not draft) targeting the spec branch.

### Step 9: Return to Spec Branch

After creating the PR, switch back to spec branch:

```bash
git checkout [spec-name]
```

This keeps you positioned correctly for the next task.

### Step 10: Notify User

Display this message:

```
✅ Task [N] Complete: [Task Title]

**Task Branch:** [task-number]-[task-name-slug]
**Pull Request:** [PR-URL]
**Targets:** [spec-name] branch (NOT main)

### Summary

[Brief summary from implementation report]

### Files Changed

**Created:**
- [list]

**Modified:**
- [list]

### Testing Status

✓ All tests passing
✓ Code quality checks passed
✓ Implementation documented

### Next Steps

1. **Review the PR** at [PR-URL]
   - This PR shows only the changes for this specific task
   - Easy to review in isolation

2. **Merge the PR** into the [spec-name] branch when satisfied

3. **Ready for next task?** Let me know when you've merged and I'll implement the next task

### Progress

**Completed:** [N] of [Total] tasks
**Remaining:** [X] tasks
```

**STOP HERE**. Wait for user to review, merge, and request the next task.

## User Standards Compliance

Ensure all implementation follows:

{{standards/*}}

## Repeat for Next Task

When user indicates they've merged the PR and are ready for the next task, run this command again. It will:
1. Pull the latest from spec branch (including the just-merged task)
2. Identify the next uncompleted task
3. Create a new task branch
4. Implement the next task
5. Create another PR targeting spec branch

Continue this cycle until all tasks in `tasks.md` are complete.
