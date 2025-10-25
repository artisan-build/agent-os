# Get PR Feedback and Address Issues (Single-Agent)

This command retrieves feedback from a pull request and addresses all identified issues.

**Important Context:**
- You're addressing this feedback because the user agrees with reviewers
- Ignore approval/merge status - focus on ALL concerns mentioned
- Must pass all quality checks before pushing
- Document what was done in a PR comment

## Before You Begin

**Prerequisites:**
- GitHub CLI (`gh`) is installed and authenticated
- You have access to the repository
- PR exists and has feedback comments

**Optional:**
- PR number (if not provided, will detect from current branch)

## Process

### Step 1: Identify the PR

**If user provided PR number:**
```bash
PR_NUMBER=[user-provided-number]
```

**If NOT provided, detect from current branch:**
```bash
CURRENT_BRANCH=$(git branch --show-current)
PR_NUMBER=$(gh pr list --head $CURRENT_BRANCH --json number --jq '.[0].number')

if [ -z "$PR_NUMBER" ]; then
  echo "❌ No PR found for branch: $CURRENT_BRANCH"
  echo "Please specify PR number: get-pr-feedback [number]"
  exit 1
fi
```

### Step 2: Get PR Information

```bash
gh pr view $PR_NUMBER --json number,title,baseRefName,headRefName,state,url
```

Display to user:
```
📋 Analyzing PR #[number]: [title]
Branch: [headRefName] → [baseRefName]
URL: [url]

Retrieving feedback...
```

### Step 3: Retrieve All Feedback

**Get general PR comments:**
```bash
gh pr view $PR_NUMBER --json comments --jq '.comments[] | "Author: \(.author.login)\nDate: \(.createdAt)\nComment:\n\(.body)\n---"'
```

**Get review comments (line-specific):**
```bash
gh api repos/:owner/:repo/pulls/$PR_NUMBER/comments --jq '.[] | "Author: \(.user.login)\nFile: \(.path):\(.line)\nComment:\n\(.body)\n---"'
```

**Check if any feedback exists:**
If both commands return empty, display:
```
ℹ️  No feedback found on PR #[number]

This PR has no comments yet. Nothing to address.
```
Stop here.

### Step 4: Parse and Categorize Feedback

Read through ALL comments and identify actionable items.

**Ignore these:**
- "LGTM" / "Approved" / "Looks good"
- General compliments without specifics
- "Ready to merge" statements

**Extract these actionable items:**

1. **Critical Issues:**
   - "This bug..."
   - "Security issue..."
   - "This breaks..."
   - "Test is failing..."

2. **Code Quality Issues:**
   - "This should be typed as..."
   - "Missing error handling..."
   - "Violates standard..."
   - "Consider renaming..."

3. **Improvements:**
   - "Could be refactored..."
   - "Better approach would be..."
   - "Performance could improve..."

4. **Questions Requiring Changes:**
   - "Why isn't this..." (if answer requires changing code)
   - "Should this handle..." (if answer is yes)

5. **Tests/Documentation:**
   - "Missing test for..."
   - "Should add test..."
   - "Documentation unclear..."

### Step 5: Create Action Plan

Display to user:

```
📝 PR #[number] Feedback Analysis

Found [N] actionable items from [M] reviewers.

## Critical Issues ([count])

1. **[Brief description]**
   - Reviewer: @[username]
   - Location: [file:line or "general"]
   - Issue: [full description]
   - Action: [what you'll do]

## Code Quality Issues ([count])

1. **[Brief description]**
   - Reviewer: @[username]
   - Location: [file:line]
   - Issue: [description]
   - Action: [what you'll do]

## Improvements Suggested ([count])

1. **[Brief description]**
   - Reviewer: @[username]
   - Location: [file:line]
   - Suggestion: [description]
   - Action: [what you'll do]

## Tests/Documentation ([count])

1. **[Brief description]**
   - Reviewer: @[username]
   - Need: [description]
   - Action: [what you'll add]

---

Total items to address: [N]

Ready to proceed with addressing all feedback? (yes/no)
```

**WAIT for explicit user confirmation** before proceeding.

### Step 6: Switch to PR Branch

```bash
PR_BRANCH=$(gh pr view $PR_NUMBER --json headRefName --jq '.headRefName')
CURRENT_BRANCH=$(git branch --show-current)

if [ "$CURRENT_BRANCH" != "$PR_BRANCH" ]; then
  echo "Switching to PR branch: $PR_BRANCH"
  git checkout $PR_BRANCH
  git pull origin $PR_BRANCH
fi
```

### Step 7: Address Each Issue Systematically

Work through issues in priority order (Critical → Quality → Improvements → Tests).

**For each issue:**

1. **Locate the relevant code**
   - If file:line provided, start there
   - Otherwise, search for relevant code

2. **Understand the concern**
   - Re-read reviewer's comment
   - Understand their perspective
   - Determine correct fix

3. **Implement the fix**
   - Follow coding standards: `{{standards/*}}`
   - Make targeted, focused changes
   - Don't introduce new issues

4. **Test the fix**
   - Run relevant tests
   - Add new tests if needed
   - Verify functionality

5. **Track what was done**
   - Keep notes for commit message
   - Keep notes for PR comment

**As you work, update a tracking list:**
```
✓ Fixed: [Issue 1 description]
✓ Improved: [Issue 2 description]
✓ Implemented: [Suggestion 1 description]
✓ Added test: [Test description]
```

### Step 8: Run Quality Checks

After addressing all issues, verify everything passes:

```bash
composer ready
```

**If it fails:**
1. Review the errors
2. Fix the issues
3. Run `composer ready` again
4. Repeat until it passes

**Must pass:**
- ✓ All tests (Pest)
- ✓ Static analysis (PHPStan)
- ✓ Code formatting (Pint)

Do not proceed until `composer ready` passes cleanly.

### Step 9: Commit Changes

Create a detailed commit message:

```bash
git add .

git commit -m "Address PR feedback from review

Addressed all feedback from PR #$PR_NUMBER review:

Critical Issues Fixed:
- [Issue 1 fixed]
- [Issue 2 fixed]

Code Quality Improvements:
- [Improvement 1]
- [Improvement 2]

Suggestions Implemented:
- [Suggestion 1]
- [Suggestion 2]

Tests/Documentation Added:
- [Test 1 added]
- [Documentation updated]

Reviewers: @[username1] @[username2]

All quality checks passing:
✓ Tests pass
✓ PHPStan clean
✓ Code formatted

🤖 Generated with Claude Code (https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>"
```

### Step 10: Push Changes

```bash
git push origin $PR_BRANCH
```

### Step 11: Document Resolution in PR Comment

Create a comprehensive comment documenting what was addressed:

```bash
gh pr comment $PR_NUMBER --body "$(cat <<'EOF'
## 🔧 Feedback Addressed

Thank you for the thorough review! I've addressed all the feedback:

### Critical Issues ✓

**[Issue title/description]** (@[reviewer])
- **Fixed by:** [explanation of how it was fixed]
- **Location:** [file:line]
- **Changes:** [specific changes made]

**[Another issue if applicable]** (@[reviewer])
- **Fixed by:** [explanation]
- **Location:** [file:line]
- **Changes:** [what changed]

### Code Quality Improvements ✓

**[Issue description]** (@[reviewer])
- **Improved by:** [what was done]
- **Location:** [file:line]
- **Changes:** [specific improvements]

### Suggestions Implemented ✓

**[Suggestion description]** (@[reviewer])
- **Implemented:** [how it was implemented]
- **Location:** [file:line]
- **Changes:** [what was added/changed]

### Tests/Documentation Added ✓

**[Gap identified]** (@[reviewer])
- **Added:** [what was added]
- **Coverage:** [what's now covered]
- **Location:** [test file or doc file]

---

### Quality Verification

All checks passing:
- ✅ `composer ready` passes
- ✅ All tests pass (Pest)
- ✅ Static analysis clean (PHPStan)
- ✅ Code formatted (Pint)
- ✅ No new issues introduced

### Summary

Addressed **[N] items** from **[M] reviewers**:
- **[N1]** critical issues fixed
- **[N2]** code quality improvements
- **[N3]** suggestions implemented
- **[N4]** tests/documentation added

All feedback has been incorporated. Ready for re-review.

🤖 Generated with Claude Code (https://claude.com/claude-code)
EOF
)"
```

### Step 12: Display Completion Summary

Show final summary to user:

```
✅ PR Feedback Successfully Addressed

**PR:** #[number] - [title]
**Branch:** [branch-name]
**Status:** Changes pushed and documented

### Summary of Changes

**Critical Issues Fixed:** [N]
- [Brief item 1]
- [Brief item 2]

**Code Quality Improved:** [N]
- [Brief item 1]
- [Brief item 2]

**Improvements Implemented:** [N]
- [Brief item 1]
- [Brief item 2]

**Tests/Docs Added:** [N]
- [Brief item 1]
- [Brief item 2]

### Verification Results

✅ All tests passing
✅ composer ready clean
✅ No new issues introduced
✅ Changes committed and pushed
✅ Resolution documented in PR

### What Happens Next

1. Reviewers will be automatically notified of your new commit
2. They'll see your detailed resolution comment
3. They can re-review the changes
4. You can merge when approved

**View updated PR:** [PR URL]
```

## Error Handling

### Cannot Find PR

```
❌ Error: Could not find PR #[number]

Please check:
- PR number is correct
- You have access to the repository
- PR is not closed/deleted

Try: gh pr list
```

### No Feedback Found

```
ℹ️  No feedback to address

PR #[number] has no comments or all comments are non-actionable (e.g., just "LGTM").

Nothing to do!
```

### Quality Checks Fail

If `composer ready` continues to fail after multiple fix attempts:

```
❌ Cannot proceed: Quality checks failing

After addressing feedback, `composer ready` is still failing:

[Show error output]

This may indicate:
1. Feedback created conflicts with existing code
2. Additional issues were introduced
3. Issue is more complex than initial analysis

Please review the errors and either:
- Fix manually
- Revert problematic changes
- Ask for clarification from reviewers
```

### Partial Resolution

If some issues cannot be fully addressed:

In the PR comment, include:

```markdown
### Items Needing Clarification ⚠️

**[Issue description]** (@[reviewer])
- **Partially addressed:** [what was done]
- **Question:** [what needs clarification]
- **Status:** Awaiting reviewer input

@[reviewer] Could you clarify [specific question]?
```

## Usage Examples

**With PR number:**
```bash
get-pr-feedback 123
```

**Without PR number (auto-detect from branch):**
```bash
# On branch `1-database-schema`
get-pr-feedback
# Will find PR for current branch automatically
```

## Important Reminders

- **Address everything** - User invoked this because they agree with reviewers
- **Ignore "approved" status** - Still address concerns even if PR is approved
- **Quality gates** - Must pass `composer ready` before pushing
- **Document thoroughly** - Help future reviewers understand what changed
- **Credit reviewers** - Thank them and @mention in resolutions
- **Be specific** - Don't just say "fixed", explain how/why

## Standards Compliance

All fixes must follow user standards:

{{standards/*}}
