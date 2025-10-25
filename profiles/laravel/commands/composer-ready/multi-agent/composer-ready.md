# Composer Ready - Fix Quality Issues

You are running quality checks and fixing all identified issues to make the code pass `composer ready`.

**Context:**
- This is typically used during active spec implementation
- Quality checks are failing and need to be fixed
- Output should be minimal (recent changes, not legacy codebase)
- If output is large, user should use `/prepare-legacy-project` instead

**Important:**
- Do not create a spec from this analysis
- Focus on fixing issues, not documenting them
- Must keep running `composer ready` until it passes cleanly

## Process Overview

PHASE 1: Run composer ready and capture output
PHASE 2: Analyze and categorize issues
PHASE 3: Fix all issues systematically
PHASE 4: Verify and confirm clean run

Follow each phase in sequence:

## PHASE 1: Run Composer Ready and Capture Output

### Step 1: Check Context

Display to user:
```
🔍 Running Quality Checks

This command will:
1. Run `composer ready` to identify issues
2. Analyze and categorize the issues
3. Fix all issues systematically
4. Verify everything passes

Note: If you see hundreds/thousands of issues, this is not the right tool.
Use `/prepare-legacy-project` for large-scale legacy code cleanup.

Running checks...
```

### Step 2: Execute Composer Ready

```bash
composer ready 2>&1 | tee /tmp/composer-ready-output.txt
EXIT_CODE=${PIPESTATUS[0]}
```

**Capture both:**
- The complete output
- The exit code (success/failure)

### Step 3: Check for Critical Failures

**If composer ready cannot run at all:**
- Fatal PHP errors
- Missing dependencies
- Composer configuration issues
- Tool crashes

Display:
```
❌ Critical Failure - Cannot Run Quality Checks

The `composer ready` command failed to run:

[ERROR OUTPUT]

**What This Means:**
There's a blocking issue preventing quality checks from running.

**Recommended Actions:**

[Choose appropriate guidance:]

**For PHP Fatal Errors:**
1. Fix the syntax/fatal error in [FILE]
2. Run this command again

**For Missing Dependencies:**
1. Run `composer install`
2. Run this command again

**For Tool Issues:**
1. Check composer.json scripts
2. Verify all tools are installed
3. Run this command again

Cannot proceed until `composer ready` can execute.
```

**STOP HERE** if critical failure. Do not attempt to fix or proceed.

### Step 4: Check Output Size

Count approximate issues found:
- Lines in output
- Error count if reported
- General sense of scale

**If issues are extensive (hundreds+):**

Display:
```
⚠️  Large Number of Issues Detected

`composer ready` found approximately [N] issues across many files.

**This suggests:**
- This is a legacy codebase needing systematic modernization
- Recent changes introduced widespread issues
- Wrong tool for the job

**Recommended:**

If this is a legacy project that hasn't been modernized:
→ Use `/prepare-legacy-project` instead

If recent changes introduced many issues:
→ Consider reverting and re-implementing more carefully

This command is designed for fixing issues in recently changed files,
not for large-scale codebase cleanup.

Should I proceed anyway, or would you like to use a different approach?
```

Wait for user decision.

## PHASE 2: Analyze and Categorize Issues

### Step 1: Parse Output

Review the output and identify issues from each tool:

**Pint (Code Formatting):**
- Files with formatting issues
- Types of formatting problems

**PHPStan (Static Analysis):**
- Files with type errors
- Error messages and severities
- Common error patterns

**Pest (Tests):**
- Failing tests
- Error messages
- Test files affected

**Security Advisories:**
- Vulnerable dependencies
- Severity levels

### Step 2: Categorize by File

Group issues by the file they're in:

```
app/Services/UserService.php:
  - PHPStan: Method getUserData() has no return type
  - PHPStan: Parameter $id has no type declaration
  - Pint: Indentation incorrect on lines 45-50

tests/Feature/UserServiceTest.php:
  - Pest: Test "it retrieves user data" is failing
  - PHPStan: Undefined property $user

app/Models/User.php:
  - Pint: Missing blank line after class declaration
```

### Step 3: Prioritize Issues

Order by:
1. **Failing tests** (highest priority - code is broken)
2. **Type errors causing confusion** (blocking other work)
3. **Type safety issues** (correctness)
4. **Formatting issues** (easiest to fix)
5. **Security advisories** (if any)

### Step 4: Create Fix Plan

Display to user:

```
📋 Quality Issues Analysis

Found issues in [N] files that need fixing.

## Failing Tests ([count])

**tests/Feature/UserServiceTest.php**
- Test "it retrieves user data" failing
- Error: [error message]
- Priority: HIGH (functionality is broken)

## Type Safety Issues ([count])

**app/Services/UserService.php**
- Method getUserData() missing return type
- Parameter $id missing type declaration
- Priority: MEDIUM (static analysis failing)

**app/Http/Controllers/UserController.php**
- Property $userService missing type
- Method store() has ambiguous return type
- Priority: MEDIUM

## Formatting Issues ([count])

**app/Services/UserService.php**
- Lines 45-50: Incorrect indentation
- Line 32: Missing space after comma
- Priority: LOW (auto-fixable)

**app/Models/User.php**
- Line 12: Missing blank line
- Priority: LOW

## Security Advisories ([count])

[Only if present]

---

**Total:** [N] issues across [M] files
**Estimated time:** [quick/moderate/extended]

Ready to fix all issues? (yes/no)
```

Wait for user confirmation.

## PHASE 3: Fix All Issues Systematically

### Step 1: Fix in Priority Order

Work through issues from highest to lowest priority:

**For each file with issues:**

1. **Open the file**
2. **Address all issues in that file**:
   - Fix type declarations
   - Add return types
   - Fix logic errors
   - Update tests
   - Let Pint auto-format

3. **Test the fixes**:
   - If tests were failing, run those specific tests
   - If type errors, verify they're resolved
   - Don't move to next file until this one is clean

### Step 2: Run Composer Ready Frequently

After fixing each file (or group of related issues):

```bash
composer ready
```

**Track progress:**
- Which issues are now resolved
- Which issues remain
- Any new issues introduced

**Display progress:**
```
✓ Fixed app/Services/UserService.php
  - Added return types
  - Fixed parameter types
  - Pint formatted

✓ Fixed tests/Feature/UserServiceTest.php
  - Updated test assertions
  - Test now passing

Running composer ready... [3 of 5 files fixed]
```

### Step 3: Handle Pint Separately

For pure formatting issues, can run Pint directly:

```bash
./vendor/bin/pint
```

This auto-fixes formatting issues without manual intervention.

### Step 4: Address Test Failures

**For failing tests:**

1. **Understand the failure** - read error message carefully
2. **Check if test is wrong** - maybe test needs updating for new behavior
3. **Check if code is wrong** - maybe implementation has a bug
4. **Fix the root cause** - don't just make test pass artificially

**Common scenarios:**
- Test expects old behavior → update test
- Code has actual bug → fix code
- Test setup is wrong → fix test setup
- Mock/fake needs updating → update mock

### Step 5: Keep User Informed

As you work, provide updates:

```
Working on app/Services/UserService.php...
- Added return type: User|null
- Added parameter type: int
- Fixed indentation
✓ Done

Working on tests/Feature/UserServiceTest.php...
- Updated test assertion for new return type
- Test now passing
✓ Done

Running composer ready to verify...
```

## PHASE 4: Verify and Confirm Clean Run

### Step 1: Final Composer Ready

After all fixes applied:

```bash
composer ready
```

### Step 2: Verify Success

**If it passes:**

Display:
```
✅ All Quality Checks Passing!

composer ready completed successfully with zero errors.

## What Was Fixed

**Files Modified:** [N]
- [file 1] - [brief description of fixes]
- [file 2] - [brief description of fixes]
- [file 3] - [brief description of fixes]

**Issues Resolved:**
- ✓ [N] type safety issues
- ✓ [N] formatting issues
- ✓ [N] test failures
- ✓ [N] static analysis errors

**Quality Status:**
- ✅ All tests passing (Pest)
- ✅ Static analysis clean (PHPStan)
- ✅ Code formatted (Pint)
- ✅ No security vulnerabilities

## Next Steps

Your code is now ready to:
- Commit and push
- Create a pull request
- Pass CI checks

The fixes have been applied. Review the changes and commit when ready.
```

**If it still fails:**

```
⚠️  Quality Checks Still Failing

After fixing identified issues, `composer ready` is still reporting errors:

[SHOW REMAINING ERRORS]

This might mean:
- New issues were introduced while fixing others
- Some issues are more complex than initially assessed
- There are cascading effects from the changes

Options:
1. Let me analyze the new errors and fix them (recommended)
2. Review the changes manually and provide guidance
3. Revert changes and approach differently

What would you like to do?
```

### Step 3: Offer to Continue

If issues remain and user chooses to continue:

Return to PHASE 2 with the new output, analyze remaining issues, fix them, and repeat until `composer ready` passes.

**Maximum iterations:** 3 cycles

After 3 cycles, if still failing:
```
⚠️  Unable to Resolve All Issues

After 3 attempts, quality checks are still failing.

This suggests the issues are more complex than can be automatically resolved.

**Current status:**
- [N] issues remaining
- Files affected: [list]

**Recommendation:**
Review the errors manually and either:
- Fix the remaining issues yourself
- Provide specific guidance on how to resolve them
- Consider reverting changes and re-implementing

Would you like to see the current errors in detail?
```

## Important Reminders

### This Is Not prepare-legacy-project

- **composer-ready**: Fix issues in recently changed files during active development
- **prepare-legacy-project**: Analyze and create spec for modernizing legacy codebase

If you see hundreds of errors, wrong tool. Use prepare-legacy-project.

### Must Pass Before Proceeding

Do not consider the task complete until `composer ready` passes with zero errors.

### Fix, Don't Document

This command fixes issues, it doesn't create specifications or documentation about them.

### Stay Focused

Focus only on making `composer ready` pass. Don't:
- Refactor unrelated code
- Add new features
- Optimize for performance
- Make stylistic improvements beyond what's required

## Standards Compliance

All fixes must follow:

{{standards/*}}
