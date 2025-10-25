Now that we've prepared this spec's `spec.md` and `tasks.md`, we must verify that this spec is fully aligned with the current requirements to ensure we're ready for implementation.

Follow these instructions:

{{workflows/specification/verify-spec}}

## Create Spec Branch and Draft PR

After verification is complete:

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

## Display confirmation and next step

Display the following message to the user:

```
✅ Spec Created and Ready for Review

Your spec verification report is ready at `agent-os/specs/[this-spec]/verification/spec-verification.md`.

**Spec Branch:** [spec-name]
**Draft PR:** [PR URL]

### Next Steps

1. Review the draft PR at [PR URL]
2. Request any changes or iterations needed
3. When satisfied, mark the PR as "Ready for Review" or let me know to begin implementation

Each task will be implemented in its own branch/PR targeting the spec branch for isolated review.

Next step: Run the command `implement-spec.md` to begin implementation.
```

## User Standards & Preferences Compliance

IMPORTANT: Ensure that your verifications are ALIGNED and DOES NOT CONFLICT with the user's preferences and standards as detailed in the following files:

{{standards/*}}
