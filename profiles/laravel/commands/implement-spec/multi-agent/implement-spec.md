# Spec Implementation Process

Now that we have a spec and tasks list ready for implementation, we will proceed with implementation of this spec by following this sequential, task-by-task process.

**Important Workflow Notes:**
- Implementation is **sequential**, not parallel - one task at a time
- Each task gets its own branch that targets the spec branch (not main)
- Each task gets its own PR (not draft) for isolated review
- User reviews and merges each task PR before moving to the next task

Process overview:

PHASE 1: Verify we're on the spec branch and identify next task
PHASE 2: Create task branch and implement the task
PHASE 3: Create task PR targeting spec branch
PHASE 4: Wait for user instruction before continuing

Follow each of these phases and their individual workflows IN SEQUENCE:

## Sequential Task Process

### PHASE 1: Verify Spec Branch and Identify Next Task

1. **Determine spec name** from the spec folder path (e.g., `agent-os/specs/2025-03-15-password-reset/` → spec branch: `password-reset`)

2. **Check current git branch**:
   ```bash
   git branch --show-current
   ```

   - If on spec branch: Continue to next step
   - If on main/master: Checkout spec branch (`git checkout [spec-name]`)
   - If on task branch: Error - you should merge the task PR before starting a new task

3. **Pull latest from spec branch**:
   ```bash
   git pull origin [spec-name]
   ```

4. **Read tasks.md** and identify the next uncompleted task (first task without [x])

5. **Confirm with user**:
   "Ready to implement: **Task X: [Task Title]**

   This task includes:
   - [List subtasks]

   Shall I proceed? (yes/no)"

   Wait for user confirmation before proceeding.

### PHASE 2: Create Task Branch and Implement

1. **Create task branch** from spec branch:
   - Branch name format: `[task-number]-[task-name-slug]`
   - Example: For "Task 1: Database Schema" → branch: `1-database-schema`

   ```bash
   git checkout -b [task-number]-[task-name-slug]
   ```

2. **Determine the appropriate implementer subagent**:
   - Read `agent-os/roles/implementers.yml`
   - Match task type to implementer role
   - If uncertain, use `full-stack-implementer` as default

3. **Delegate to implementer subagent**:

   Provide the subagent with:
   - The specific task group (parent task + all subtasks)
   - The spec file: `agent-os/specs/[this-spec]/spec.md`
   - The full tasks list: `agent-os/specs/[this-spec]/tasks.md`

   Instruct subagent to:
   1. Implement the task according to spec requirements
   2. Write tests following TDD approach
   3. Ensure all tests pass
   4. Update `tasks.md` to check off completed subtasks
   5. Document work in `agent-os/specs/[this-spec]/implementation/[task-number]-[task-name].md`

4. **After implementation completes**:
   - Run `composer ready` to verify code quality
   - Ensure all tests pass
   - Verify task is checked off in tasks.md

### PHASE 3: Create Task PR Targeting Spec Branch

1. **Commit all changes**:
   ```bash
   git add .
   git commit -m "Complete Task [N]: [Task Title]

   [Brief description of what was implemented]

   Subtasks completed:
   - [List checked-off subtasks]

   🤖 Generated with Claude Code (https://claude.com/claude-code)

   Co-Authored-By: Claude <noreply@anthropic.com>"
   ```

2. **Push task branch**:
   ```bash
   git push -u origin [task-number]-[task-name-slug]
   ```

3. **Create PR targeting spec branch** (NOT draft, NOT targeting main):
   ```bash
   gh pr create \
     --base [spec-name] \
     --title "Task [N]: [Task Title]" \
     --body "$(cat <<'EOF'
   ## Task Completed

   Implements Task [N] from the [spec-name] spec.

   ### What Changed

   [Summarize the changes made]

   ### Files Modified/Created

   - [List key files]

   ### Testing

   - [ ] All new tests pass
   - [ ] Existing tests still pass
   - [ ] `composer ready` passes

   ### Verification

   Implementation documented in: `agent-os/specs/[spec-path]/implementation/[task-number]-[task-name].md`

   ---

   Part of spec: #[spec-pr-number]

   🤖 Generated with Claude Code (https://claude.com/claude-code)
   EOF
   )"
   ```

4. **Switch back to spec branch**:
   ```bash
   git checkout [spec-name]
   ```

### PHASE 4: Wait for User Review

Display to user:

```
✅ Task [N] Implementation Complete

**Branch:** [task-number]-[task-name-slug]
**PR:** [PR URL] (targets spec branch: [spec-name])

### What Was Implemented

[Summary from implementation report]

### Next Steps

1. Review the PR at [PR URL]
2. Merge the PR into the spec branch when satisfied
3. Let me know when you're ready for the next task

**Remaining tasks:** [X] of [Y] tasks remaining
```

**STOP HERE** - Do not proceed to the next task until user instructs you to continue.

When user says they're ready for the next task, return to PHASE 1.

## When All Tasks Complete

After the final task is merged:

1. **Verify spec branch has all tasks**:
   - All items in tasks.md should be checked off
   - All implementation reports should exist
   - All tests should pass

2. **Prompt user about final verification**:

   "All tasks have been completed and merged into the spec branch.

   Would you like me to:
   1. Run final verification checks?
   2. Mark the spec PR as ready for review?

   Or would you prefer to do this manually?"

3. **If user requests final verification**:
   - Use **implementation-verifier** subagent
   - Provide the spec path
   - Generate final verification report
   - Display results to user

4. **If user requests marking PR ready**:
   ```bash
   git checkout [spec-name]
   gh pr ready [spec-pr-number]
   ```

## Important Notes

- **Never work on multiple tasks simultaneously** - one task branch at a time
- **Always target the spec branch**, not main, for task PRs
- **Wait for user to merge** before creating the next task branch
- **Pull latest from spec branch** before each new task to get merged changes
- User will handle merging the spec branch into main after all tasks are complete
