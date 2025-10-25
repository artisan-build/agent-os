# get-pr-feedback Command

## Purpose

Retrieves feedback from a GitHub pull request, addresses all identified issues, and documents the resolution.

## When to Use

Use this command when:
- A PR has received review comments with feedback
- You agree with the reviewers and want all issues addressed
- You want comprehensive resolution with proper documentation
- You need quality checks to pass before pushing

## What It Does

1. **Retrieves feedback** from PR comments (both general and line-specific)
2. **Analyzes and categorizes** issues into Critical, Quality, Improvements, Tests/Docs
3. **Addresses all concerns** systematically
4. **Runs quality checks** (`composer ready`) until passing
5. **Commits and pushes** changes
6. **Documents resolution** in a PR comment for context

## Important Understanding

### Ignores Approval Status

Even if a reviewer says "LGTM but...", this command focuses on the "but" part. When you run this command, you're indicating you agree with the reviewers that the concerns should be addressed, regardless of approval status.

### Addresses Everything

This command is thorough - it addresses:
- Bugs and critical issues
- Code quality concerns
- Suggestions and improvements
- Test coverage gaps
- Documentation needs
- Questions that require code changes

### Quality Gates

Will not push changes until:
- ✅ All tests pass
- ✅ PHPStan clean
- ✅ Code properly formatted
- ✅ No new issues introduced

## Usage

### Multi-Agent Mode (Claude Code)

```bash
/get-pr-feedback [pr-number]
```

**With PR number:**
```bash
/get-pr-feedback 123
```

**Without PR number (auto-detects from current branch):**
```bash
/get-pr-feedback
```

### Single-Agent Mode (Other Tools)

Run the command file directly:
```bash
agent-os/profiles/laravel/commands/get-pr-feedback/single-agent/get-pr-feedback.md
```

The command will:
1. Detect PR from current branch (or use provided number)
2. Show you analysis of all feedback
3. Wait for your confirmation
4. Address all issues
5. Document resolution

## Example Workflow

### Scenario: Task PR Receives Feedback

```bash
# You created Task 1 PR targeting spec branch
# Reviewer left 5 comments with concerns

# On the task branch:
git checkout 1-database-schema

# Get and address feedback:
/get-pr-feedback

# Agent analyzes feedback:
# "Found 5 actionable items:
#  - 2 critical issues
#  - 2 code quality improvements
#  - 1 test missing
# Ready to proceed? (yes/no)"

# You confirm: "yes"

# Agent addresses all issues, runs composer ready, pushes

# Agent posts comment documenting what was done

# Reviewers are notified automatically
```

## What Gets Analyzed

### Comments Analyzed

- **General PR comments**: Comments on the PR as a whole
- **Review comments**: Line-specific code review comments
- **Both** are checked and all feedback is addressed

### What Gets Extracted

**Critical Issues:**
- Bugs
- Security vulnerabilities
- Breaking changes
- Test failures

**Code Quality:**
- Type safety issues
- Error handling gaps
- Standards violations
- Naming concerns

**Improvements:**
- Refactoring suggestions
- Performance optimizations
- Better patterns
- Cleaner code

**Tests/Documentation:**
- Missing test coverage
- Test improvements needed
- Documentation gaps

### What Gets Ignored

- "LGTM" / "Approved" / "Looks good"
- Generic compliments
- "Ready to merge" (without concerns)
- Non-actionable commentary

## Output

### During Execution

You'll see:
1. PR identification and retrieval
2. Feedback analysis with categorization
3. Action plan for confirmation
4. Progress as issues are addressed
5. Quality check results
6. Final summary

### After Completion

**In Git:**
- Commit with detailed message listing all fixes
- Changes pushed to PR branch

**On GitHub:**
- New commit appears in PR
- Detailed comment documents all resolutions
- Reviewers notified automatically

**Example PR Comment:**
```markdown
## 🔧 Feedback Addressed

Thank you for the review! I've addressed all the feedback:

### Critical Issues ✓

**Type safety issue in getUserData** (@reviewer1)
- Fixed by: Added proper return type annotation
- Location: app/Services/UserService.php:45
- Changes: Method now returns User|null with proper PHPDoc

### Code Quality Improvements ✓

**Inconsistent error handling** (@reviewer2)
- Improved by: Standardized exception handling across controller
- Location: app/Http/Controllers/UserController.php
- Changes: All methods now throw consistent exceptions

### Tests Added ✓

**Missing test for edge case** (@reviewer1)
- Added: Test for null user scenario
- Coverage: Now tests getUserData with invalid ID
- Location: tests/Feature/UserServiceTest.php

---

### Quality Verification

All checks passing:
- ✅ composer ready passes
- ✅ All tests pass
- ✅ No new issues introduced

### Summary

Addressed 5 items from 2 reviewers. Ready for re-review.

🤖 Generated with Claude Code
```

## Prerequisites

### Required

- GitHub CLI (`gh`) installed and authenticated
- Write access to the repository
- Pull request exists with feedback

### Recommended

- On the PR's branch (or will switch for you)
- Working directory clean (or commit/stash changes first)

## Quality Guarantees

Before pushing, this command ensures:

1. **All tests pass**: Runs full test suite
2. **Static analysis clean**: PHPStan passes
3. **Code formatted**: Pint formatting applied
4. **Standards compliant**: Follows all project standards
5. **No regressions**: Existing functionality preserved

If `composer ready` fails, the command will:
- Show you the errors
- Attempt to fix them
- Run checks again
- Will not push until passing

## Best Practices

### When to Use

✅ **Good times to use:**
- After receiving thorough review feedback
- When you agree with reviewer concerns
- Before requesting re-review
- To ensure systematic issue resolution

❌ **Don't use when:**
- You disagree with feedback (discuss first)
- Feedback is unclear (ask for clarification first)
- You want to address only some concerns (do manually)
- PR has no comments

### Tips

1. **Review the action plan** carefully before confirming
2. **Check the quality output** - ensure all checks pass
3. **Read the resolution comment** before re-requesting review
4. **Use with task PRs** - great for iterative review cycles
5. **Run on spec branch** - can address feedback on overall spec too

## Error Scenarios

### Cannot Find PR

If PR cannot be detected:
- Provide PR number explicitly: `/get-pr-feedback 123`
- Check you're on correct branch
- Verify PR exists and is open

### No Feedback Found

If no actionable feedback:
- Command will inform you and stop
- Nothing to address = nothing to do
- This is normal for PRs with only "LGTM"

### Quality Checks Fail

If `composer ready` keeps failing:
- Command will show errors
- May ask you to review manually
- Could indicate complex conflicts
- Might need reviewer clarification

### Partial Resolution

If some issues can't be fully addressed:
- Command will document partial resolution
- Will tag reviewer with questions
- Will still push other fixes
- Reviewer can provide more context

## Integration with Git Workflow

This command works seamlessly with the Laravel profile's git workflow:

**Task PR Workflow:**
```
1. Create task branch from spec branch
2. Implement task
3. Create PR targeting spec branch
4. Receive feedback
5. → get-pr-feedback  ← You are here
6. Reviewers see resolution
7. Merge when approved
```

**Spec PR Workflow:**
```
1. Create spec branch
2. Create draft PR
3. Receive feedback on spec itself
4. → get-pr-feedback  ← Can use here too
5. Mark ready when approved
6. Begin task implementation
```

## Command Options

### PR Number (Optional)

**Provide explicitly:**
```bash
/get-pr-feedback 123
```

**Auto-detect from branch:**
```bash
# On branch 1-database-schema
/get-pr-feedback
# Finds PR for 1-database-schema automatically
```

## Standards Compliance

All fixes and improvements follow user standards:
- Coding style standards
- Best practices
- Framework conventions
- Testing standards
- Documentation standards

## Related Commands

- `/create-spec` - Creates spec with draft PR
- `/implement-spec` - Implements tasks with PRs
- This command complements the review cycle for those PRs

## Example Output

```
✅ PR Feedback Successfully Addressed

PR: #124 - Task 1: Database Schema
Branch: 1-database-schema
Status: Changes pushed and documented

Summary of Changes

Critical Issues Fixed: 2
- Fixed type safety in UserService::getUserData
- Added missing null check in UserController

Code Quality Improved: 2
- Standardized error handling across controller
- Improved method naming for clarity

Improvements Implemented: 1
- Refactored database query for better performance

Tests/Docs Added: 1
- Added test for null user edge case

Verification Results

✅ All tests passing
✅ composer ready clean
✅ No new issues introduced
✅ Changes committed and pushed
✅ Resolution documented in PR

What Happens Next

1. Reviewers notified of new commit automatically
2. They'll see detailed resolution comment
3. Can re-review the changes
4. Merge when approved

View updated PR: [URL]
```
