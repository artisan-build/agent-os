# Step 1: Run Quality Analysis

You are running a comprehensive quality analysis on a legacy Laravel project to identify all issues that need to be addressed before the project can be considered ready for modern development.

## Execute Analysis Command

Run the following command:

```bash
composer report
```

This command executes the quality tool suite in sequence:

1. **IDE Helper Generation** - `php artisan ide-helper:generate`
2. **IDE Helper Meta** - `php artisan ide-helper:meta`
3. **IDE Helper Models** - `php artisan ide-helper:models -M`
4. **Rector** - `composer rector` (applies refactorings automatically)
5. **Duster** - `vendor/bin/duster fix` (applies Laravel convention fixes automatically)
6. **PHPStan** - `composer stan` (static analysis Level 6)
7. **Pest** - `composer test` (test suite)
8. **Security Check** - `composer update --dry-run roave/security-advisories`

## Capture Output AND Exit Code

**Critical**: You need both:
1. The complete output from the command
2. The exit code (success or failure)

## Check for Critical Failures

Before proceeding to Step 2, check if the command failed catastrophically.

### Critical Failure Indicators

**Exit Code Non-Zero** - Command failed

**Fatal Errors:**
- PHP Fatal error
- PHP Parse error
- Class not found errors
- Composer autoload failures
- Missing dependencies

**Tool Failures:**
- IDE Helper crashes
- Rector crashes
- PHPStan cannot analyze
- Tests cannot run

### If Critical Failure Detected

**DO NOT PROCEED** to Step 2. Instead, display this message to the user:

```
⚠️  Critical Failure Detected - Cannot Create Spec

The `composer report` command encountered a critical failure that prevents analysis:

[DESCRIBE THE SPECIFIC FAILURE YOU OBSERVED]

**What This Means:**
The codebase has blocking issues that must be resolved before we can assess modernization needs.

**Recommended Actions:**

[Choose the appropriate guidance:]

**For Syntax/Parse Errors:**
1. Review the error output above
2. Fix the syntax error in [FILE]
3. Run `composer report` again to verify it completes
4. Then restart this command sequence from Step 1

**For Missing Dependencies:**
1. Run `composer install` to ensure all dependencies are installed
2. Check that all required packages are in composer.json
3. Run `composer report` again
4. Then restart from Step 1

**For Autoload Issues:**
1. Run `composer dump-autoload`
2. Verify PSR-4 autoload configuration in composer.json
3. Run `composer report` again
4. Then restart from Step 1

**For Tool Installation Issues:**
1. Ensure all required tools are installed:
   - barryvdh/laravel-ide-helper
   - rector/rector
   - tightenco/duster
   - larastan/larastan
   - pestphp/pest
2. Run `composer install`
3. Run `composer report` again
4. Then restart from Step 1

**General Approach:**
Work through the error interactively until `composer report` can complete successfully.
Once it runs (even with thousands of reported issues), return to Step 1 to begin the analysis.

---

Need help resolving the blocking issue? Share the error and I can help you fix it.
```

**STOP HERE** if there's a critical failure. Do not create a spec. Do not proceed to Step 2.

## If Command Succeeded

Even if there are thousands of errors reported, if the command completed without crashing, that's success!

### Understanding What Happens

**Automatic fixes applied during this command:**
- Rector applies refactorings (imports, modernizations, etc.)
- Duster applies Laravel convention fixes
- IDE Helper generates model docblocks for PHPStan
- These changes are saved to your codebase

**Issues reported for manual intervention:**
- Security vulnerabilities in dependencies
- PHPStan errors that remain after automatic fixes
- Test failures
- Any issues the automated tools couldn't fix

## Important Notes

- Rector and Duster WILL modify your code automatically
- The files changed by these tools will be shown in the output
- Focus on the REMAINING issues that need manual fixes
- Review what was automatically changed to understand context
- Large output (thousands of lines) is normal for legacy projects

## Expected Result

Analysis is complete when:
- The `composer report` command has finished executing
- You have the complete output showing both:
  - What was automatically fixed
  - What remains to be fixed manually
- Your working directory has changes from Rector/Duster (check with `git status`)
- The command did NOT crash with a fatal error

## Next Step

**If command succeeded** (even with many errors reported):
Proceed to step 2: `2-analyze-issues.md`

**If command failed critically**:
Work with the user to fix the blocking issue, then re-run Step 1.

Display to user (if successful):

```
✓ Analysis complete.

Automatic fixes have been applied by Rector and Duster.
Check git status to see modified files.

Next: Run command 2-analyze-issues.md to categorize remaining issues.
```

## Standards Reference

Follow these standards during analysis:

{{standards/global/*}}
{{standards/backend/*}}
{{standards/testing/*}}
