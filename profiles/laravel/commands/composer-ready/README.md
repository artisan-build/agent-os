# composer-ready Command

## Purpose

Runs quality checks and fixes all identified issues to make `composer ready` pass cleanly.

## When to Use

✅ **Use this command when:**
- Working on a spec and quality checks are failing
- CI is failing on your PR due to quality issues
- Need to fix issues in recently changed files (< 20 issues typically)
- `composer ready` is reporting manageable errors

❌ **Don't use this when:**
- You see hundreds/thousands of errors → Use `/prepare-legacy-project`
- Working on a legacy codebase needing systematic modernization
- Haven't made any recent changes (nothing to fix)

## What It Does

1. **Runs `composer ready`** and captures all output
2. **Analyzes issues** from Pint, PHPStan, Pest, and security advisories
3. **Categorizes by priority**: Tests → Types → Formatting → Security
4. **Groups by file** for systematic fixing
5. **Fixes all issues** in priority order
6. **Runs checks repeatedly** until `composer ready` passes
7. **Reports success** with summary of fixes

## Key Difference from prepare-legacy-project

| Feature | composer-ready | prepare-legacy-project |
|---------|---------------|----------------------|
| **Purpose** | Fix issues in active development | Modernize legacy codebase |
| **Scope** | Recently changed files | Entire codebase |
| **Output** | Fixes applied directly | Creates spec for systematic work |
| **Issue Count** | < 20 typically | Hundreds/thousands |
| **Context** | During spec implementation | Before starting feature work |
| **Creates Spec?** | No | Yes |

## Usage

### Multi-Agent Mode (Claude Code)

```bash
/composer-ready
```

### Single-Agent Mode (Other Tools)

Run the command file:
```bash
agent-os/profiles/laravel/commands/composer-ready/single-agent/composer-ready.md
```

## Typical Workflow

### Scenario: Task Implementation with Quality Issues

```bash
# Implement Task 1
# Create PR
# CI fails with quality check errors

# On the task branch:
/composer-ready

# Agent analyzes output:
# "Found 8 issues in 3 files:
#  - 2 failing tests
#  - 4 type safety issues
#  - 2 formatting issues
# Ready to fix? (yes/no)"

# You confirm: "yes"

# Agent fixes all issues systematically:
# "Fixing tests/Feature/UserServiceTest.php..."
# "✓ Fixed"
# "Fixing app/Services/UserService.php..."
# "✓ Fixed"
# "Running Pint for auto-formatting..."
# "✓ Done"
# "Running composer ready..."
# "✅ All checks passing!"

# Now you can commit and push
git add .
git commit -m "Fix quality issues"
git push

# CI will now pass
```

## What Gets Fixed

### Failing Tests (Priority 1)
- Tests that are currently failing
- Broken test setup/teardown
- Incorrect assertions after code changes
- Mock/fake issues

### Type Safety Issues (Priority 2)
- Missing return types on methods
- Missing parameter types
- Missing property types
- Ambiguous types needing clarification
- PHPDoc needed for complex types (array shapes, generics)

### Code Formatting (Priority 3)
- Indentation issues
- Spacing problems
- Line breaks
- PSR-12 compliance
- Most auto-fixed by Pint

### Security Advisories (Priority 4)
- Vulnerable dependencies
- Required version upgrades
- Security patches needed

## Process Details

### Step-by-Step

1. **Run composer ready**
   ```bash
   composer ready
   ```
   Captures all output for analysis.

2. **Check for critical failures**
   - If can't even run → provides guidance to fix blocking issue
   - If hundreds of errors → warns and suggests `/prepare-legacy-project`

3. **Parse and categorize**
   - Groups by tool (Pint, PHPStan, Pest, Security)
   - Organizes by file for efficient fixing
   - Prioritizes (tests → types → formatting → security)

4. **Display fix plan**
   - Shows all issues organized
   - Waits for your confirmation
   - Clear about what will be done

5. **Fix systematically**
   - One file at a time
   - All issues in that file at once
   - Test after each file
   - Track progress

6. **Auto-format**
   - Runs Pint to fix formatting issues automatically

7. **Verify repeatedly**
   - Runs `composer ready` after fixes
   - Continues until passing (max 3 iterations)
   - Reports any issues that can't be auto-resolved

8. **Report success**
   - Summary of all fixes applied
   - List of files modified
   - Quality status confirmed

## Example Output

### Initial Analysis

```
📋 Quality Issues Found

Total: 8 issues in 3 files

## Priority 1: Failing Tests (2)

**tests/Feature/UserServiceTest.php**
- Test "it retrieves user data" failing
- Error: Expected User but got null
- Fix: Update assertion for nullable return type

**tests/Unit/UserTest.php**
- Test "it validates email" failing
- Error: Undefined property $email
- Fix: Add property to test setup

## Priority 2: Type Safety Issues (4)

**app/Services/UserService.php** (3 issues)
- Line 45: Method getUserData() missing return type
- Line 45: Parameter $id missing type
- Line 67: Property $repository missing type

**app/Http/Controllers/UserController.php** (1 issue)
- Line 89: Method store() ambiguous return type

## Priority 3: Code Formatting (2)

**app/Services/UserService.php**
- Lines 50-55: Incorrect indentation
- Line 32: Missing space after comma

---

Fixing Strategy:
1. Fix failing tests first (code is broken)
2. Add missing types (static analysis)
3. Run Pint to auto-fix formatting
4. Run `composer ready` to verify

Ready to proceed? (yes/no)
```

### Progress Updates

```
Fixing tests/Feature/UserServiceTest.php...
✓ Updated assertions for nullable return types
✓ Test now passing

Fixing tests/Unit/UserTest.php...
✓ Added $email property to test setup
✓ Test now passing

Fixing app/Services/UserService.php...
✓ Added return type User|null to getUserData()
✓ Added parameter type int to $id
✓ Added property type UserRepository to $repository
✓ PHPStan passes for this file

Running Pint for automatic formatting...
✓ Formatted 2 files automatically

Progress: All fixes complete
Running final composer ready...
```

### Success Message

```
✅ All Quality Checks Passing!

composer ready completed successfully with zero errors.

## Summary of Fixes

**Files Modified:** 3

**tests/Feature/UserServiceTest.php**
- Updated assertions for new return types
- Test now passing

**tests/Unit/UserTest.php**
- Added $email property to setup
- Test now passing

**app/Services/UserService.php**
- Added return type: User|null
- Added parameter types: int, UserRepository
- Fixed indentation and spacing via Pint

**app/Http/Controllers/UserController.php**
- Added return type: JsonResponse
- Pint formatted

**Totals:**
- ✓ 2 test failures corrected
- ✓ 4 type issues resolved
- ✓ 2 formatting issues fixed

## Quality Status

All checks passing:
- ✅ Pest: All tests pass
- ✅ PHPStan: No type errors
- ✅ Pint: Code properly formatted
- ✅ Security: No vulnerabilities

## Next Steps

Your code is ready to:
1. Commit changes
2. Push to remote
3. Pass CI quality checks

Review the changes and commit when satisfied.
```

## Common Scenarios

### CI Failing on PR

```bash
# PR created but CI fails
# Check CI output: "composer ready failed"

# On your branch:
/composer-ready

# Agent fixes all issues
# Commit and push
# CI now passes
```

### Before Creating PR

```bash
# Implemented task
# Want to ensure quality before PR

/composer-ready

# Agent confirms everything passes
# Or fixes any issues found
# Now create PR with confidence
```

### After Addressing PR Feedback

```bash
# Used /get-pr-feedback to address review comments
# But maybe introduced some type issues

/composer-ready

# Agent catches and fixes any quality issues
# Push clean code
```

### During Active Development

```bash
# Making changes, not sure if tests still pass
# Want to check before committing

/composer-ready

# Agent runs checks, fixes anything broken
# Can continue with confidence
```

## Error Handling

### Critical Failures

If `composer ready` can't even run:
```
❌ Critical Failure

The command failed: Fatal error in app/Services/UserService.php

Fix the syntax error first, then run this command again.
```

### Too Many Issues

If hundreds of errors detected:
```
⚠️  Large Number of Issues Detected

This looks like a legacy codebase issue, not recent changes.

Recommendation: Use /prepare-legacy-project instead.

This command is for fixing issues in recently changed files.
```

### Can't Auto-Resolve

If issues persist after 3 fix attempts:
```
⚠️  Manual Intervention Needed

After 3 attempts, some issues remain:
[list remaining issues]

These need human judgment to resolve.
```

## Tips for Best Results

1. **Run frequently**: Don't wait until PR to check quality
2. **Commit before running**: Easy to see what changed
3. **Review the fixes**: Agent shows what it did, make sure it makes sense
4. **Use during development**: Catch issues early
5. **Let Pint handle formatting**: Don't manually fix spacing issues

## Quality Standards

The command ensures code meets:
- ✅ All tests passing
- ✅ PHPStan Level 6 (strictest)
- ✅ PSR-12 formatting via Pint
- ✅ No security vulnerabilities
- ✅ User coding standards in `{{standards/*}}`

## Prerequisites

- `composer ready` script configured in composer.json
- Quality tools installed (PHPStan/Larastan, Pest, Pint)
- Working in a Laravel project
- Recent changes that may have broken quality checks

## Integration with Workflow

Fits seamlessly into the development cycle:

```
Implement task → Check quality → Fix issues → Create PR → CI passes
                      ↓
                 /composer-ready
```

Or during review:

```
Get PR feedback → Address feedback → Check quality → Push clean code
                                          ↓
                                    /composer-ready
```

## Related Commands

- `/get-pr-feedback` - Address PR review comments
- `/prepare-legacy-project` - For large-scale legacy modernization
- `/implement-spec` - Implements tasks (should pass composer ready)

This command ensures your code meets quality standards before pushing or creating PRs, making the review process smoother.
