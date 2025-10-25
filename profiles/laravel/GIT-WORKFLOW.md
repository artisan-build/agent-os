# Git Workflow for Laravel Profile

## Overview

The Laravel profile implements a deliberate, review-focused git workflow that creates clear checkpoints for review at both the specification and implementation levels.

## Workflow Summary

```
main
 └─ spec-branch (draft PR)
     ├─ 1-task-name (PR → spec-branch)
     ├─ 2-task-name (PR → spec-branch)
     ├─ 3-task-name (PR → spec-branch)
     └─ ... (all tasks merged) → mark ready for review → merge to main
```

## Phase 1: Spec Creation (`/new-spec` + `/create-spec`)

### 1.1 Initialize Spec (`/new-spec`)
- Creates spec folder: `agent-os/specs/YYYY-MM-DD-spec-name/`
- Gathers requirements from user
- No git operations yet

### 1.2 Create Spec Documents (`/create-spec`)

**Actions:**
1. Creates `spec.md` and `tasks.md`
2. Runs verification
3. **Creates spec branch** (without date prefix)
   - Example: `agent-os/specs/2025-03-15-password-reset/` → branch: `password-reset`
4. **Commits spec files** to spec branch
5. **Pushes spec branch** to origin
6. **Creates draft PR** targeting `main`

**Draft PR Purpose:**
- Review the spec and task breakdown before any implementation
- Iterate on requirements, approach, or task organization
- PR description includes all tasks that will be implemented

**User Actions:**
- Review the spec in the draft PR
- Request changes/iterations as needed
- When satisfied: Either mark as "Ready for Review" or instruct agent to begin implementation

## Phase 2: Sequential Task Implementation (`/implement-spec`)

### 2.1 Task-by-Task Workflow

For **each task** in the spec (processed sequentially):

1. **Verify on spec branch**
   - Ensure current branch is the spec branch
   - Pull latest changes: `git pull origin [spec-name]`

2. **Identify next task**
   - Read `tasks.md`
   - Find first uncompleted task (no `[x]` checkbox)
   - Confirm with user before proceeding

3. **Create task branch**
   - Branch name: `[task-number]-[task-name-slug]`
   - Example: `1-database-schema`, `2-user-authentication`, etc.
   - Created from spec branch: `git checkout -b 1-database-schema`

4. **Implement task**
   - Delegate to appropriate implementer subagent
   - Write tests (TDD approach)
   - Ensure all tests pass
   - Update `tasks.md` to check off completed subtasks
   - Document in `implementation/` folder

5. **Create task PR**
   - Commit changes
   - Push task branch
   - **Create PR targeting spec branch** (NOT main, NOT draft)
   - Switch back to spec branch
   - **STOP and wait for user**

6. **User reviews and merges**
   - User reviews the isolated changes for this task
   - User merges PR into spec branch
   - User instructs agent to continue with next task

7. **Repeat for next task**
   - Pull latest from spec branch (includes merged work)
   - Create new task branch
   - Implement next task
   - Create PR targeting spec branch
   - Wait for merge
   - Continue...

### 2.2 Why This Approach?

**Benefits:**
- **Isolated review**: Each PR contains only the changes for one task
- **Incremental progress**: Can stop at any point, each merged task is done
- **Clear history**: Each task is a separate commit/PR in the spec branch
- **Easier debugging**: If something breaks, easy to identify which task caused it
- **Flexibility**: Can adjust approach between tasks based on learnings

**Trade-offs:**
- Slower than parallel implementation (by design)
- More user involvement required (merge each PR)
- More deliberate and careful

## Phase 3: Spec Completion

After all tasks are implemented and merged into spec branch:

1. **Final verification** (optional)
   - Run final verification checks
   - Generate verification report
   - Ensure all tests pass

2. **Mark spec PR ready**
   - Change draft PR to "Ready for Review"
   - Final review of complete feature

3. **Merge to main**
   - User merges spec branch PR into main
   - Feature is now in production codebase

## Branch Naming Conventions

### Spec Branch
- Format: `[spec-name]` (without date prefix)
- Example: `password-reset`, `user-dashboard`, `api-rate-limiting`
- Extracted from spec folder name

### Task Branch
- Format: `[task-number]-[task-name-slug]`
- Example: `1-database-schema`, `2-authentication-logic`, `3-api-endpoints`
- Task number matches the task number in `tasks.md`

## Pull Request Strategy

### Spec PR (Draft)
- **Created by**: `/create-spec` command
- **Targets**: `main` branch
- **Type**: Draft initially
- **Purpose**: Review spec before implementation starts
- **Contains**: Spec documents, task breakdown, verification reports
- **Marked ready**: After all tasks merged OR user marks it ready manually

### Task PRs (Regular)
- **Created by**: `/implement-spec` command (after each task)
- **Targets**: Spec branch (NOT main)
- **Type**: Regular PR (not draft)
- **Purpose**: Review individual task implementation
- **Contains**: Code changes for one specific task only
- **Merged by**: User, after reviewing changes

## Example Workflow

### Starting a New Feature

```bash
# User: "/new-spec for password reset functionality"
# Agent: Gathers requirements, creates spec folder

# User: "/create-spec"
# Agent: Creates spec.md, tasks.md, creates branch 'password-reset',
#        pushes, creates draft PR #123 targeting main

# User reviews draft PR #123, requests some changes to task breakdown

# Agent updates tasks.md in response

# User: "Looks good, let's start implementation"
```

### Implementing Tasks

```bash
# Agent: "Ready to implement Task 1: Database Schema. Shall I proceed?"
# User: "yes"

# Agent:
#   - Creates branch '1-database-schema' from 'password-reset'
#   - Implements migration, model updates
#   - Runs tests, updates tasks.md
#   - Commits, pushes, creates PR #124 targeting 'password-reset'
#   - Switches back to 'password-reset' branch
#   - "Task 1 complete. PR: [URL]. Let me know when merged and ready for Task 2."

# User reviews PR #124, merges into 'password-reset' branch

# User: "next task"

# Agent:
#   - Pulls latest from 'password-reset' (includes merged Task 1)
#   - "Ready to implement Task 2: Authentication Logic. Shall I proceed?"
#
# ... continues for all tasks ...
```

### Completing the Spec

```bash
# After final task is merged:

# Agent: "All tasks complete. Would you like me to mark PR #123 ready for review?"
# User: "yes"

# Agent: Runs final verification, marks PR #123 as ready

# User reviews complete feature in PR #123, merges into main
```

## Commands and Git Operations Summary

| Command | Git Operations |
|---------|---------------|
| `/new-spec` | None - just creates spec folder |
| `/create-spec` | Creates spec branch, commits spec files, pushes, creates draft PR targeting main |
| `/implement-spec` | For each task: creates task branch from spec branch, implements, commits, pushes, creates PR targeting spec branch, switches back to spec branch |

## Important Notes

- **GitHub CLI** (`gh`) is used for all GitHub operations
- **Sequential only** - never work on multiple tasks in parallel
- **Always pull** before creating new task branch (gets merged changes from previous tasks)
- **User controls merge** - agent never merges PRs, only creates them
- **Spec branch is base** - all task branches and PRs target the spec branch, not main
- **Draft only for spec** - spec PR starts as draft, task PRs are regular PRs
