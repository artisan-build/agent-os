---
description: Process issue based on classification
step: 2
total_steps: 2
version: 1.0
encoding: UTF-8
---

# Step 2: Process Issue Based on Classification

<ai_meta>
  <parsing_rules>
    - Process XML blocks first for structured data
    - Execute instructions in sequential order
    - Route to appropriate workflow based on issue_type from Step 1
  </parsing_rules>
  <step_context>
    - current_step: 2
    - total_steps: 2
    - previous_step: 1-retrieve-and-classify.md
  </step_context>
</ai_meta>

## Step Overview

<purpose>
  - Process issue according to its classification
  - Execute appropriate workflow (bug, feature, patch, or question)
  - Create branches, tests, PRs, or specs as needed
</purpose>

<required_data_from_step_1>
  - issue_number
  - issue_title
  - issue_body
  - issue_author
  - issue_url
  - issue_type
</required_data_from_step_1>

<process_flow>

<workflow name="bug">

## Bug Workflow

<bug_instructions>

### Bug Processing Steps

**Step 2.1: Check for Sufficient Information**

Review the issue body for:
- [ ] Steps to reproduce the bug
- [ ] Expected behavior description
- [ ] Actual behavior description
- [ ] Environment details (Laravel version, PHP version, etc. - if relevant)

IF any critical information is missing:
  1. Determine what specific information is needed
  2. Create a helpful comment requesting it:
     ```bash
     gh issue comment {issue_number} --body "$(cat <<'EOF'
     Hi @{issue_author}, thank you for reporting this issue.

     To help us diagnose and fix this bug, could you please provide:

     - [Specific information needed]
     - [Additional context needed]
     - [Steps to reproduce if missing]

     This information will help us reproduce and resolve the issue more quickly.
     EOF
     )"
     ```
  3. STOP workflow - no further action until more information provided
  4. Report to user: "I've requested more information from the issue author. We'll need to wait for their response before proceeding."

IF sufficient information exists: Continue to Step 2.2

**Step 2.2: Assess Bug Complexity**

Analyze the bug to determine complexity:

**SIMPLE BUG Indicators:**
- Single component or file affected
- Clear reproduction steps
- Obvious cause or solution
- Can be tested with a single regression test
- Likely 1-3 file changes needed

**COMPLEX BUG Indicators:**
- Multiple components involved
- Unclear root cause
- Requires deep investigation
- Needs multiple regression tests
- Likely 5+ files need changes
- Architectural implications

Make determination: Is this a SIMPLE or COMPLEX bug?

IF SIMPLE: Continue to Step 2.3
IF COMPLEX: Skip to Step 2.4

**Step 2.3: Fix Simple Bug**

Execute the following workflow:

1. **Create Branch:**
   ```bash
   git checkout main
   git pull origin main
   git checkout -b {issue_number}-{kebab-case-of-issue-title}
   ```

2. **Write Regression Test:**
   - Create a test that demonstrates the bug
   - Run the test to verify it FAILS (proving the bug exists)
   - Test file should be in appropriate location (tests/Feature/ or tests/Unit/)
   - Run: `composer test -- --filter={TestClassName}`

3. **Fix the Bug:**
   - Make minimal changes to fix the issue
   - Follow code style standards in @.agent-os/standards/
   - Keep changes focused and small

4. **Verify Fix:**
   - Run regression test: `composer test -- --filter={TestClassName}`
   - Verify test now PASSES
   - Run full suite: `composer ready`

5. **Evaluate Result:**

   IF all tests pass:
     - Continue to Step 2.5 (Create PR)

   IF fix unsuccessful or `composer ready` reveals bigger issues:
     - Reclassify as COMPLEX bug
     - Continue to Step 2.4

**Step 2.4: Handle Complex Bug**

For complex bugs, delegate to spec creation workflow:

1. Tell user: "This bug is complex and requires detailed planning. I'll create a spec to document the fix requirements."

2. Invoke spec creation:
   ```
   I need to create a spec for this bug fix. Here's the context:

   **Bug Issue:** #{issue_number}
   **Title:** {issue_title}
   **Description:** {issue_body}

   This is a complex bug that requires:
   - Multiple file changes
   - Comprehensive test coverage
   - Careful investigation of root cause

   Please follow @.agent-os/instructions/create-spec.md to create a specification for fixing this bug.
   ```

3. STOP workflow - delegate to create-spec command
4. The spec will capture full requirements for the bug fix

**Step 2.5: Create Pull Request (Simple Bugs Only)**

After successful fix:

1. **Commit Changes:**
   ```bash
   git add .
   git commit -m "Fix: {brief description of fix}

   Resolves #{issue_number}

   - Added regression test demonstrating bug
   - Fixed issue in {file names}
   - All tests passing

   🤖 Generated with [Claude Code](https://claude.com/claude-code)

   Co-Authored-By: Claude <noreply@anthropic.com>"
   ```

2. **Push Branch:**
   ```bash
   git push origin {issue_number}-{kebab-case-title}
   ```

3. **Create Pull Request:**
   ```bash
   gh pr create --title "Fix: {issue_title}" --body "$(cat <<'EOF'
   ## Summary

   Fixes #{issue_number}

   ## Changes Made

   - Added regression test demonstrating the bug
   - Fixed issue by [brief explanation]
   - Verified all tests pass

   ## Testing

   - Regression test: [test file path]
   - All tests passing via `composer ready`

   ## Related Issue

   Closes #{issue_number}
   EOF
   )" --base main
   ```

4. **Link PR to Issue:**
   The "Closes #{issue_number}" in the PR body will auto-link and close the issue when merged.

5. Report to user:
   ```
   ✅ Bug Fixed and PR Created

   - Branch: {branch_name}
   - Regression test: [test file]
   - All tests passing
   - PR: [PR URL from gh output]
   - Issue will auto-close when PR is merged
   ```

</bug_instructions>

</workflow>

<workflow name="feature">

## Feature Request Workflow

<feature_instructions>

### Feature Request Processing Steps

**Step 2.1: Check for Sufficient Context**

Review the issue body for:
- [ ] Clear description of desired functionality
- [ ] Use cases or user stories
- [ ] Acceptance criteria (if provided)
- [ ] Any technical considerations

IF critical context is missing:
  1. Determine what information is needed to create a good spec
  2. Create a comment requesting it:
     ```bash
     gh issue comment {issue_number} --body "$(cat <<'EOF'
     Hi @{issue_author}, thank you for this feature request.

     To help us create a detailed specification, could you please provide:

     - [Specific use case details needed]
     - [User story clarifications needed]
     - [Technical requirements if known]

     This will help us plan and implement this feature effectively.
     EOF
     )"
     ```
  3. STOP workflow
  4. Report to user: "I've requested more details from the issue author before creating a spec."

IF sufficient context exists: Continue to Step 2.2

**Step 2.2: Create Feature Spec**

Delegate to spec creation workflow:

1. Tell user: "I have enough context to create a spec for this feature request."

2. Invoke spec creation:
   ```
   I need to create a spec for this feature request. Here's the context:

   **Feature Request:** #{issue_number}
   **Title:** {issue_title}
   **Description:** {issue_body}

   Please follow @.agent-os/instructions/create-spec.md to create a specification for implementing this feature.
   ```

3. The spec creation workflow will:
   - Create spec folder with date and name
   - Create spec.md with requirements
   - Create technical specifications
   - Create task breakdown
   - Link back to issue #{issue_number}

4. After spec is created, add comment to issue:
   ```bash
   gh issue comment {issue_number} --body "$(cat <<'EOF'
   Spec created for this feature request: [spec path]

   The spec documents requirements and implementation plan. Work will proceed according to the Agent OS workflow.
   EOF
   )"
   ```

5. Report to user:
   ```
   ✅ Feature Spec Created

   - Spec: [spec folder path]
   - Linked to issue #{issue_number}
   - Ready for implementation via execute-tasks workflow
   ```

</feature_instructions>

</workflow>

<workflow name="patch">

## Patch Workflow

<patch_instructions>

### Patch Application Steps

**Step 2.1: Extract Patch Code**

1. Identify code blocks in issue body
2. Determine which files need changes
3. Extract the patch code

**Step 2.2: Create Branch and Apply Patch**

1. **Create Branch:**
   ```bash
   git checkout main
   git pull origin main
   git checkout -b {issue_number}-{kebab-case-of-issue-title}
   ```

2. **Apply Patch:**
   - Modify the files as specified in the patch
   - Follow the exact changes provided
   - Maintain code style consistency

3. **Verify Quality:**
   ```bash
   composer ready
   ```

4. **Evaluate Result:**

   IF `composer ready` passes:
     - Continue to Step 2.3 (Create PR)

   IF `composer ready` fails:
     - Review failures
     - IF easily fixable: fix and retry
     - IF complex issues: report to user
       ```
       ⚠️ Patch Issues Detected

       The patch from issue #{issue_number} causes the following problems:
       [list failures from composer ready]

       This may need manual review or reclassification as a bug/feature request.
       ```
     - STOP workflow

**Step 2.3: Create Pull Request**

After successful patch application:

1. **Commit Changes:**
   ```bash
   git add .
   git commit -m "Apply patch from issue #{issue_number}

   {brief description of what patch does}

   - Applied code changes from #{issue_number}
   - All quality checks passing

   🤖 Generated with [Claude Code](https://claude.com/claude-code)

   Co-Authored-By: Claude <noreply@anthropic.com>"
   ```

2. **Push Branch:**
   ```bash
   git push origin {issue_number}-{kebab-case-title}
   ```

3. **Create Pull Request:**
   ```bash
   gh pr create --title "Patch: {issue_title}" --body "$(cat <<'EOF'
   ## Summary

   Applies patch from #{issue_number}

   ## Changes Made

   - Applied code changes as specified in issue
   - Verified with `composer ready`

   ## Related Issue

   Closes #{issue_number}
   EOF
   )" --base main
   ```

4. Report to user:
   ```
   ✅ Patch Applied and PR Created

   - Branch: {branch_name}
   - All quality checks passing
   - PR: [PR URL]
   - Issue will auto-close when PR is merged
   ```

</patch_instructions>

</workflow>

<workflow name="question">

## Question Workflow

<question_instructions>

### Question Response Steps

**Step 2.1: Analyze Question**

1. Read question carefully
2. Determine what user is asking
3. Check if question reveals an actual bug or feature request

**Step 2.2: Research Answer**

Search for answer in:
- Project documentation (README, docs folder)
- Code comments and docblocks
- Related code examples in codebase
- Common patterns in the project

**Step 2.3: Provide Response**

1. **If question is answerable:**
   ```bash
   gh issue comment {issue_number} --body "$(cat <<'EOF'
   Hi @{issue_author},

   [Clear explanation of the answer]

   [Code examples if relevant - with proper formatting]

   [Links to documentation if available]

   [Suggestions for further help if needed]

   I hope this helps! If this answers your question, feel free to close this issue.
   EOF
   )"
   ```

2. **If question reveals a bug:**
   - Reclassify as BUG
   - Tell user: "This question actually reveals a bug. Reclassifying and processing as a bug."
   - Return to Bug Workflow

3. **If question reveals a feature need:**
   - Reclassify as FEATURE REQUEST
   - Tell user: "This question suggests we need a new feature. Reclassifying as a feature request."
   - Return to Feature Request Workflow

4. **If unable to answer:**
   ```bash
   gh issue comment {issue_number} --body "$(cat <<'EOF'
   Hi @{issue_author},

   This requires input from the project maintainer or someone with deeper context. I've tagged this for human review.

   [Explain what additional context is needed]
   EOF
   )"
   ```
   - Add label if possible: `gh issue edit {issue_number} --add-label "needs-human-review"`

5. Report to user:
   ```
   ✅ Question Addressed

   - Added response comment to issue #{issue_number}
   - [Suggested closing / Tagged for review / Reclassified]
   ```

</question_instructions>

</workflow>

</process_flow>

## Routing Logic

<routing_instructions>
  **Based on issue_type from Step 1, execute the appropriate workflow:**

  IF issue_type == "bug":
    Execute Bug Workflow above

  IF issue_type == "feature":
    Execute Feature Request Workflow above

  IF issue_type == "patch":
    Execute Patch Workflow above

  IF issue_type == "question":
    Execute Question Workflow above
</routing_instructions>

## Step Completion

<completion_checklist>
  Verify before completing:
  - [ ] Appropriate workflow executed based on classification
  - [ ] Branch created (if applicable)
  - [ ] Tests written/run (if applicable)
  - [ ] PR created OR spec created OR comment added
  - [ ] Issue linked to resolution
  - [ ] User notified of outcome
</completion_checklist>

<final_summary>
  After completing the workflow, provide a summary:

  ```
  ## Issue Processing Complete: #{issue_number}

  **Classification:** {issue_type}
  **Title:** {issue_title}

  ### Actions Taken
  [List what was done]

  ### Next Steps
  [What happens next]

  ### Links
  - Issue: {issue_url}
  - PR: {pr_url} (if created)
  - Spec: {spec_path} (if created)
  ```
</final_summary>
