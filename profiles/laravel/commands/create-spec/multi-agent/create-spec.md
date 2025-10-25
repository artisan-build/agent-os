# Create Spec Process

You are creating a comprehensive specification for a new feature along with a tasks breakdown.  This process will follow 3 main phases, each with their own workflows:

Process overview (details to follow)

PHASE 1. Write the spec document
PHASE 2. Create the tasks list
PHASE 3. Verify the spec & tasks list
PHASE 4. Display the results to user

Follow each of these phases and their individual workflows IN SEQUENCE:

## Process:

### PHASE 1: Delegate to Spec Writer

Use the **spec-writer** subagent to create the specification document for this spec:

Provide the spec-writer with:
- The spec folder path (find the current one or the most recent in `agent-os/specs/*/`)
- The requirements from `planning/requirements.md`
- Any visual assets in `planning/visuals/`

The spec-writer will create `spec.md` inside the spec folder.

Wait until the spec-writer has created `spec.md` before proceeding with PHASE 2 (delegating to task-list-creator).

### PHASE 2: Delegate to Tasks List Creator

Once `spec.md` has been created, use the **tasks-list-creator** subagent to break down the spec into an actionable tasks list with strategic grouping and ordering.

Provide the tasks-list-creator:
- The spec folder path (find the current one or the most recent in `agent-os/specs/*/`)
- The `spec.md` file that was just created.
- The original requirement from `planning/requirements.md`
- Any visual assets in `planning/visuals/`

The tasks-list-creator will create `tasks.md` inside the spec folder.

### PHASE 3: Verify Specifications

Use the **spec-verifier** subagent to verify accuracy:

Provide the spec-verifier with:
- ALL of the questions that were asked to the user during requirements gathering (from earlier in this conversation)
- ALL of the user's raw responses to those questions (from earlier in this conversation)
- The spec folder path

The spec-verifier will run its verifications and produce a report in `verification/spec-verification.md`

### PHASE 4: Create Spec Branch and Draft PR

After the spec and tasks are created and verified:

1. **Extract spec name** from the spec folder path (e.g., `agent-os/specs/2025-03-15-password-reset/` → branch name: `password-reset`)

2. **Check current branch**:
   - If on `main` or `master`: Create new branch with spec name
   - If on a different branch: Ask user if you should create the spec branch

3. **Create spec branch**:
   ```bash
   git checkout -b [spec-name]
   ```

4. **Commit spec files**:
   ```bash
   git add agent-os/specs/[date-spec-name]/
   git commit -m "Initialize spec: [spec-name]

   - Created spec.md with feature requirements
   - Created tasks.md with implementation breakdown
   - Completed initial verification

   🤖 Generated with Claude Code (https://claude.com/claude-code)

   Co-Authored-By: Claude <noreply@anthropic.com>"
   ```

5. **Push branch and create draft PR**:
   ```bash
   git push -u origin [spec-name]
   gh pr create --draft --title "[spec-name]" --body "$(cat <<'EOF'
   ## Spec Overview

   [Brief description from spec.md overview]

   ## Implementation Plan

   This spec will be implemented incrementally with each top-level task in its own PR targeting this spec branch:

   [List top-level tasks from tasks.md]

   ## Status

   - [ ] Spec reviewed and approved
   - [ ] Implementation tasks (will be completed in separate PRs)
   - [ ] Final verification

   ---

   🤖 Generated with Claude Code (https://claude.com/claude-code)
   EOF
   )"
   ```

### PHASE 5: Display Results

DISPLAY to the user:
- The spec creation summary from spec-writer
- The tasks list creation summary from tasks-list-creator
- The verification summary from spec-verifier
- The spec branch name and draft PR URL

If verification found issues, highlight them for the user's attention.

**Next Steps for User:**
1. Review the draft PR at [PR URL]
2. Request any changes or iterations needed
3. When satisfied, mark the PR as "Ready for Review" or let me know to begin implementation
4. Each task will be implemented in its own branch/PR targeting this spec branch

## Expected Output

After completion, you should have:

```
agent-os/specs/[date-spec-name]/
├── planning/
│   ├── initialization.md
│   ├── requirements.md
│   ├── workflow.yml
│   └── visuals/
├── implementation/
│   ├── 1_planning/
│   │   ├── recap.md
│   │   └── verification.md
│   ├── 2_[phase]/
│   │   └── spec.md
│   ├── 3_[phase]/
│   │   └── spec.md
│   └── [etc]/
├── spec.md
└── tasks.md
```
