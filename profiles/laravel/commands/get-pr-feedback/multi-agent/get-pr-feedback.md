# Get PR Feedback and Address Issues

You are retrieving feedback from a pull request and addressing all identified issues.

**Important Context:**
- The user is asking you to address PR feedback because they agree with the reviewers
- Ignore approval/merge status - focus on addressing ALL concerns mentioned
- After addressing issues, document what was done in a PR comment

## Process Overview

PHASE 1: Identify the PR and retrieve feedback
PHASE 2: Analyze feedback and create action plan
PHASE 3: Address all identified issues
PHASE 4: Verify quality and push changes
PHASE 5: Document resolution in PR comment

Follow each phase in sequence:

## PHASE 1: Identify PR and Retrieve Feedback

### Step 1: Determine Which PR

**If user provided PR number:**
- Use that PR number

**If user did NOT provide PR number:**
- Get current branch name: `git branch --show-current`
- Find PR for that branch: `gh pr list --head [branch-name] --json number,title,url --jq '.[0]'`
- If no PR found: Error and ask user to specify PR number

### Step 2: Get PR Details

Retrieve PR information:

```bash
gh pr view [pr-number] --json number,title,baseRefName,headRefName,state,url
```

Display to user:
```
📋 PR #[number]: [title]
Branch: [headRefName] → [baseRefName]
Status: [state]
URL: [url]

Retrieving feedback...
```

### Step 3: Retrieve All Comments

Get all comments on the PR:

```bash
gh pr view [pr-number] --json comments --jq '.comments[] | {author: .author.login, body: .body, createdAt: .createdAt}'
```

Also get review comments (on specific lines of code):

```bash
gh api repos/:owner/:repo/pulls/[pr-number]/comments --jq '.[] | {author: .user.login, body: .body, path: .path, line: .line, createdAt: .created_at}'
```

### Step 4: Check for Feedback

**If no comments found:**
```
ℹ️  No feedback found on PR #[number]

This PR has no comments yet. Nothing to address.
```
Stop here.

**If comments found:**
Continue to Phase 2.

## PHASE 2: Analyze Feedback and Create Action Plan

### Step 1: Parse All Feedback

Review ALL comments (both general and line-specific) and extract:
- Concerns raised
- Issues identified
- Suggestions made
- Questions asked

**Ignore:**
- Approval status (e.g., "LGTM", "Approved")
- Merge readiness statements
- Compliments without actionable items

**Focus on:**
- "This should..." statements
- "Consider changing..." suggestions
- "There's a bug..." reports
- "Missing..." observations
- Questions that require code changes
- Security concerns
- Performance issues
- Code quality feedback
- Test coverage gaps

### Step 2: Categorize Issues

Group feedback into categories:

1. **Critical Issues** (must fix)
   - Bugs
   - Security vulnerabilities
   - Broken functionality
   - Failing tests

2. **Code Quality Issues**
   - Type safety problems
   - Missing error handling
   - Code organization
   - Naming concerns
   - Standards violations

3. **Improvements Suggested**
   - Refactoring suggestions
   - Performance optimizations
   - Better patterns
   - Clearer code

4. **Questions Requiring Changes**
   - "Why is this..." (if answer requires code change)
   - "Should this..." (if answer is yes)

5. **Documentation/Tests**
   - Missing tests
   - Test improvements
   - Documentation gaps

### Step 3: Create Action Plan

Display to user:

```
📝 PR Feedback Analysis

Found [N] items to address across [M] comments.

### Critical Issues ([count])
- [Issue 1]
  - Location: [file:line or general]
  - Reviewer: @[username]
  - Action: [what needs to be done]

### Code Quality Issues ([count])
- [Issue 1]
  - Location: [file:line]
  - Reviewer: @[username]
  - Action: [what needs to be done]

### Improvements Suggested ([count])
- [Suggestion 1]
  - Location: [file:line]
  - Reviewer: @[username]
  - Action: [what will be done]

### Tests/Documentation ([count])
- [Item 1]
  - Reviewer: @[username]
  - Action: [what needs to be added]

---

Ready to address all [N] items. Shall I proceed? (yes/no)
```

**Wait for user confirmation** before proceeding.

## PHASE 3: Address All Identified Issues

### Step 1: Ensure on Correct Branch

```bash
# Get the PR's head branch
PR_BRANCH=$(gh pr view [pr-number] --json headRefName --jq '.headRefName')

# Check current branch
CURRENT_BRANCH=$(git branch --show-current)

# If not on PR branch, switch to it
if [ "$CURRENT_BRANCH" != "$PR_BRANCH" ]; then
  git checkout $PR_BRANCH
  git pull origin $PR_BRANCH
fi
```

### Step 2: Address Issues Systematically

Work through each issue in priority order (Critical → Quality → Improvements → Tests/Docs):

**For each issue:**

1. **Locate the code** (if file/line specified)
2. **Understand the concern**
3. **Implement the fix** following standards
4. **Update or add tests** if needed
5. **Verify the fix** works as intended
6. **Note the resolution** for documentation

**Use appropriate implementer subagent if needed:**
- For complex refactorings
- For new functionality
- For test additions

**Keep track of:**
- Which issues were addressed
- What changes were made
- Any issues that couldn't be fully resolved (document why)

### Step 3: Run Tests and Quality Checks

After addressing all issues:

```bash
composer ready
```

**If it fails:**
- Fix the issues
- Run again until it passes

**Ensure:**
- All tests pass
- No lint errors
- No type errors
- No formatting issues

## PHASE 4: Verify Quality and Push Changes

### Step 1: Final Verification

Double-check that:
- [ ] All identified issues have been addressed
- [ ] `composer ready` passes cleanly
- [ ] All tests pass
- [ ] No new issues introduced

### Step 2: Commit Changes

```bash
git add .
git commit -m "Address PR feedback from review

Addressed all feedback from PR #[number]:

Critical Issues:
- [Fixed issue 1]
- [Fixed issue 2]

Code Quality:
- [Improved issue 1]
- [Fixed issue 2]

Improvements:
- [Implemented suggestion 1]
- [Refactored as suggested]

Tests/Documentation:
- [Added test for X]
- [Updated documentation]

All reviewers: @[username1] @[username2]

Quality checks: All passing ✓

🤖 Generated with Claude Code (https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>"
```

### Step 3: Push Changes

```bash
git push origin [branch-name]
```

## PHASE 5: Document Resolution in PR Comment

### Step 1: Create Resolution Comment

Post a detailed comment to the PR documenting what was addressed:

```bash
gh pr comment [pr-number] --body "$(cat <<'EOF'
## 🔧 Feedback Addressed

Thank you for the review! I've addressed all the feedback:

### Critical Issues ✓

**[Issue description from reviewer]** (@[reviewer])
- Fixed by: [explanation of fix]
- Location: [file:line]
- Changes: [brief description]

### Code Quality Improvements ✓

**[Issue description]** (@[reviewer])
- Improved by: [explanation]
- Location: [file:line]
- Changes: [brief description]

### Suggestions Implemented ✓

**[Suggestion description]** (@[reviewer])
- Implemented: [explanation]
- Location: [file:line]
- Changes: [brief description]

### Tests Added ✓

**[Test gap identified]** (@[reviewer])
- Added: [test description]
- Coverage: [what's now tested]

---

### Verification

- ✓ `composer ready` passes
- ✓ All tests pass
- ✓ No new issues introduced

### Summary

Addressed **[N]** items from **[M]** reviewers:
- [N1] critical issues fixed
- [N2] code quality improvements
- [N3] suggestions implemented
- [N4] tests/docs added

Ready for re-review.

🤖 Generated with Claude Code (https://claude.com/claude-code)
EOF
)"
```

### Step 2: Display Completion Summary

Show user:

```
✅ PR Feedback Addressed

**PR:** #[number] - [title]
**Branch:** [branch-name]
**Changes pushed:** Yes

### What Was Done

**Critical Issues:** [N] fixed
- [Brief list]

**Code Quality:** [N] improved
- [Brief list]

**Improvements:** [N] implemented
- [Brief list]

**Tests/Documentation:** [N] added
- [Brief list]

### Verification

✓ All tests passing
✓ composer ready passes
✓ Changes pushed
✓ Resolution documented in PR comment

### Next Steps

The PR has been updated with all feedback addressed. Reviewers will be notified automatically of the new commit and comment.

You can view the updated PR at: [PR URL]
```

## Error Handling

### If PR Not Found

```
❌ Error: Could not find PR

Please specify a PR number:
  get-pr-feedback [pr-number]

Or ensure you're on a branch with an open PR.
```

### If Cannot Address an Issue

If an issue cannot be fully addressed:

1. Document why in the resolution comment
2. Tag the reviewer with a question
3. Note it in the summary
4. Don't block other fixes

Example:
```
**[Issue description]** (@[reviewer])
- ⚠️ Partially addressed: [what was done]
- Question: [what needs clarification]
- @[reviewer] Could you clarify [specific question]?
```

### If Quality Checks Fail

If `composer ready` fails and can't be fixed after multiple attempts:

1. Document what's failing
2. Revert problematic changes if needed
3. Report to user with details
4. Ask for guidance

## Important Notes

- **Address ALL concerns** - user asked for this because they agree with reviewers
- **Ignore approval status** - "LGTM but..." means address the "but" part
- **Document thoroughly** - future reviewers need context
- **Quality first** - must pass all checks before pushing
- **Credit reviewers** - mention them in commit and comment
- **Be thorough** - better to over-communicate than under

## Standards Compliance

Ensure all fixes follow:

{{standards/*}}
