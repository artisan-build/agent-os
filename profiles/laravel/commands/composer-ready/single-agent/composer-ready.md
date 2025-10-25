# Composer Ready - Fix Quality Issues (Single-Agent)

This command runs quality checks and fixes all identified issues to make `composer ready` pass cleanly.

**Use this when:**
- Working on a spec and quality checks are failing
- CI is failing due to quality issues
- Need to fix issues in recently changed files
- Output is manageable (not hundreds of errors)

**Don't use this when:**
- You see hundreds/thousands of errors (use `/prepare-legacy-project`)
- Working on a legacy codebase that needs systematic modernization

## Important Context

This command **fixes issues**, it doesn't create specs or documentation. It's designed for active development when you need to get `composer ready` passing before committing/pushing.

## Process

### Step 1: Run Quality Checks

Execute composer ready and capture all output:

```bash
composer ready 2>&1 | tee /tmp/composer-ready-output.txt
EXIT_CODE=${PIPESTATUS[0]}
echo "Exit code: $EXIT_CODE"
```

**Important**: Capture both stdout and stderr, and note the exit code.

### Step 2: Check for Critical Failures

Before analyzing issues, check if the command could even run:

**Critical failures include:**
- PHP fatal errors
- Parse errors
- Missing dependencies
- Tool crashes (PHPStan, Pest, etc.)
- Composer configuration errors

**If critical failure detected:**

Display to user:
```
❌ Critical Failure - Cannot Run Quality Checks

The `composer ready` command failed to execute:

[SHOW ERROR]

**Recommended Fix:**

[Choose based on error type:]

**For Fatal/Parse Errors:**
Fix the syntax error in [FILE] then run this command again.

**For Missing Dependencies:**
Run `composer install` then run this command again.

**For Tool Issues:**
Verify tools are installed and composer.json scripts are correct.
```

**STOP HERE** - do not proceed if critical failure. User must resolve the blocking issue first.

### Step 3: Assess Output Scale

Look at the output and estimate the number of issues:

**If output is extensive (rough indicators):**
- More than 100 lines of errors
- Many different files affected
- Errors across the entire codebase (not just recent changes)

**Display warning:**
```
⚠️  Large Number of Issues Detected

This appears to be a widespread quality issue affecting many files.

**Context check:**
- Is this a legacy project that hasn't been modernized? → Use `/prepare-legacy-project`
- Did recent changes introduce widespread issues? → Consider reverting and re-approaching
- Are these only in files you just changed? → This tool is appropriate

**This command is designed for:**
Fixing issues in recently modified files (typically < 20 issues)

**Not designed for:**
Large-scale legacy codebase modernization (hundreds of issues)

Continue with this command? (yes/no)
```

Wait for user confirmation before proceeding.

### Step 4: Parse and Categorize Issues

Read through the entire output and categorize issues by tool:

**Pint (Code Formatting):**
- List files with formatting issues
- Note specific problems (indentation, spacing, etc.)

**PHPStan (Static Analysis):**
- List files with type errors
- Group similar errors
- Note severity levels

**Pest (Tests):**
- List failing tests
- Note error messages
- Identify broken test files

**Security Advisories:**
- List vulnerable dependencies (if any)
- Note severity and required versions

### Step 5: Group by File

Reorganize issues by file for systematic fixing:

**Example:**
```
app/Services/UserService.php:
  - PHPStan: Method getUserData() has no return type (line 45)
  - PHPStan: Parameter $id has no type (line 45)
  - Pint: Indentation incorrect (lines 50-55)

tests/Feature/UserServiceTest.php:
  - Pest: Test "it retrieves user" failing
  - Error: Expected User but got null

app/Http/Controllers/UserController.php:
  - PHPStan: Property $userService has no type (line 12)
  - Pint: Missing space after comma (line 67)
```

### Step 6: Create and Display Fix Plan

Show organized plan to user:

```
📋 Quality Issues Found

Total: [N] issues in [M] files

## Priority 1: Failing Tests ([count])

Tests are currently broken - highest priority to fix.

**tests/Feature/UserServiceTest.php**
- Test "it retrieves user data" failing
- Error: [full error message]
- Fix: [what needs to be done]

## Priority 2: Type Safety Issues ([count])

Static analysis failing - need to add/fix types.

**app/Services/UserService.php** ([count] issues)
- Line 45: Method getUserData() missing return type
- Line 45: Parameter $id missing type
- Fix: Add proper type declarations

**app/Http/Controllers/UserController.php** ([count] issues)
- Line 12: Property $userService missing type
- Line 89: Method store() ambiguous return type
- Fix: Add type declarations

## Priority 3: Code Formatting ([count])

Code formatting issues - Pint can auto-fix most of these.

**app/Services/UserService.php**
- Lines 50-55: Incorrect indentation
- Line 32: Missing space after comma

**app/Models/User.php**
- Line 12: Missing blank line after class declaration

## Priority 4: Security ([count])

[Only if present]
- Vulnerable dependency: [package]
- Severity: [level]
- Required action: [upgrade to version X]

---

**Fixing Strategy:**
1. Fix failing tests first (code is broken)
2. Add missing types (static analysis)
3. Run Pint to auto-fix formatting
4. Address any security advisories
5. Run `composer ready` to verify

Ready to proceed? (yes/no)
```

Wait for explicit user confirmation.

### Step 7: Fix Issues Systematically

Work through issues in priority order:

**For each file:**

1. **Identify all issues in that file**
2. **Open and fix them all at once**
3. **For type issues:**
   - Add return types to methods
   - Add parameter types
   - Add property types
   - Add PHPDoc for complex types (array shapes, generics)

4. **For test failures:**
   - Read error message carefully
   - Determine if test or code is wrong
   - Fix the root cause (don't just make test pass)

5. **Save changes**

6. **Run checks for that specific area:**
   ```bash
   # If you fixed a test:
   ./vendor/bin/pest tests/Feature/UserServiceTest.php

   # If you fixed type issues:
   ./vendor/bin/phpstan analyze app/Services/UserService.php
   ```

7. **Confirm fix worked before moving to next file**

**Display progress as you work:**
```
Fixing app/Services/UserService.php...
✓ Added return type User|null to getUserData()
✓ Added parameter type int to $id
✓ Fixed indentation issues

Testing the fixes...
✓ PHPStan passes for this file

Moving to tests/Feature/UserServiceTest.php...
✓ Updated test assertion for new return type
✓ Test now passing

Progress: 2 of 5 files complete
```

### Step 8: Handle Formatting with Pint

For pure formatting issues, run Pint to auto-fix:

```bash
./vendor/bin/pint
```

Pint will automatically fix:
- Indentation
- Spacing
- Line breaks
- Other PSR-12 formatting issues

Display:
```
Running Pint for automatic formatting...
✓ Formatted [N] files automatically
```

### Step 9: Run Composer Ready After Each Major Fix

Don't wait until all fixes are done. Run `composer ready` periodically:

```bash
composer ready
```

**After fixing each file or logical group:**
- Run composer ready
- See if those specific errors are resolved
- Check for any new errors introduced
- Adjust approach if needed

**Track remaining issues:**
```
Running composer ready... (after fixing 2 of 5 files)

Remaining issues: [N] (down from [M])
Still need to fix:
- tests/Feature/AccountTest.php (test failure)
- app/Models/Account.php (type issues)
- app/Services/AccountService.php (type issues)
```

### Step 10: Final Verification

After all fixes applied:

```bash
composer ready
```

**If it passes (exit code 0):**

Display success:
```
✅ All Quality Checks Passing!

composer ready completed successfully with zero errors.

## Summary of Fixes

**Files Modified:** [N]

**app/Services/UserService.php**
- Added return type: User|null
- Added parameter types: int, string
- Fixed indentation and spacing

**tests/Feature/UserServiceTest.php**
- Updated assertions for new return types
- Test now passing

**app/Http/Controllers/UserController.php**
- Added property type: UserService
- Added return types to methods
- Pint formatted

**Totals:**
- ✓ [N] type issues resolved
- ✓ [N] formatting issues fixed
- ✓ [N] test failures corrected
- ✓ [N] static analysis errors resolved

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

**If it still fails:**

Show remaining errors:
```
⚠️  Some Issues Remain

After applying fixes, `composer ready` still has errors:

[SHOW REMAINING ERRORS - first 20 lines]

**Possible reasons:**
- Fixes introduced new issues
- Some problems are more complex than initially assessed
- Cascading effects from changes

**Options:**
1. Analyze these new errors and fix them (recommended)
2. Show me all remaining errors for manual review
3. Revert changes and try a different approach

What would you like to do? (1/2/3)
```

### Step 11: Iterate If Needed

If issues remain and user chooses option 1:

**Return to Step 4** with the new output:
- Parse remaining issues
- Categorize them
- Create fix plan
- Apply fixes
- Verify

**Maximum 3 iterations**: After 3 full cycles, if still failing, stop and ask for manual intervention:

```
⚠️  Manual Intervention Needed

After 3 attempts, quality checks are still failing with [N] remaining issues.

**Remaining errors:**
[SHOW ALL REMAINING ERRORS]

**Assessment:**
These issues are more complex than can be automatically resolved.

**Recommendations:**
1. Review the remaining errors carefully
2. Consider if recent changes need to be reverted
3. Fix remaining issues manually with your expertise
4. Consult with team if errors are unclear

I've fixed what I could, but these need human judgment.
```

## Key Principles

### Fix Everything

Don't stop until `composer ready` exits with code 0. No exceptions.

### Failing Tests First

If tests are failing, code is broken. Fix that before anything else.

### Add Types, Don't Suppress

When PHPStan reports type issues, add proper types. Don't:
- Use `@phpstan-ignore`
- Cast away errors
- Use mixed types unless truly necessary

### Let Pint Do Its Job

Don't manually fix spacing/indentation. Let Pint handle it:
```bash
./vendor/bin/pint
```

### Stay Focused

Only fix what's needed to pass `composer ready`. Don't:
- Refactor unrelated code
- Add new features
- Over-engineer solutions
- Change code not related to the errors

### Test Your Fixes

After fixing, verify:
```bash
# Run specific test:
./vendor/bin/pest tests/Feature/UserServiceTest.php

# Run PHPStan on specific file:
./vendor/bin/phpstan analyze app/Services/UserService.php

# Run full suite:
composer ready
```

## Common Issues and Solutions

### "Method has no return type"

**Add explicit return type:**
```php
// Before:
public function getUser($id)

// After:
public function getUser(int $id): ?User
```

### "Parameter has no type"

**Add parameter types:**
```php
// Before:
public function updateUser($id, $data)

// After:
public function updateUser(int $id, array $data): User
```

### "Property has no type"

**Add property type:**
```php
// Before:
protected $userService;

// After:
protected UserService $userService;
```

### "Cannot call method on mixed"

**Type the variable properly:**
```php
// Before:
$user = $this->getUser($id);
$user->update($data); // Error: $user is mixed

// After:
public function getUser(int $id): ?User  // Add return type
{
    return User::find($id);
}
```

### Test Failing After Adding Types

**Update test expectations:**
```php
// If method now returns ?User instead of User|null|array
// Update test assertions accordingly
$this->assertInstanceOf(User::class, $result);
// or
$this->assertNull($result);
```

## Error Messages

### Cannot Find Composer

```
❌ Error: composer command not found

Please ensure Composer is installed and available in your PATH.
```

### Composer Ready Script Not Found

```
❌ Error: 'ready' script not found in composer.json

Please ensure your composer.json has a 'ready' script that runs:
- PHPStan (or Larastan)
- Pest
- Pint

Example:
{
  "scripts": {
    "ready": [
      "vendor/bin/pint",
      "vendor/bin/phpstan analyze",
      "vendor/bin/pest"
    ]
  }
}
```

## Standards Compliance

All fixes must follow:

{{standards/*}}
