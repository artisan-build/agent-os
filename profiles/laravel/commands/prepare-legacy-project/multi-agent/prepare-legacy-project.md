# Prepare Legacy Project

You are preparing a legacy Laravel project for modern development standards by running comprehensive quality analysis and creating a detailed specification for remediation work.

This command analyzes the current state of a legacy project using the full suite of quality tools and creates a specification document that details all the work required to bring the project up to standard.

**IMPORTANT**: This command DOES NOT fix any issues. It only analyzes and documents what needs to be fixed. The actual fixes will be implemented through the normal spec implementation process.

## Process Overview

PHASE 1. Run quality analysis tools
PHASE 2. Check for critical failures
PHASE 3. Analyze and categorize issues
PHASE 4. Create remediation specification
PHASE 5. Display results to user

Follow each phase in sequence:

## PHASE 1: Run Quality Analysis

Execute the following command to generate a comprehensive quality report:

```bash
composer report
```

This command runs the following tools in sequence:
1. **IDE Helper Generation** - `php artisan ide-helper:generate`
2. **IDE Helper Meta** - `php artisan ide-helper:meta`
3. **IDE Helper Models** - `php artisan ide-helper:models -M`
4. **Rector** - `composer rector` (applies automated refactorings)
5. **Duster** - `vendor/bin/duster fix` (applies Laravel convention fixes)
6. **PHPStan** - `composer stan` (static analysis at Level 6)
7. **Pest** - `composer test` (test suite)
8. **Security Check** - `composer update --dry-run roave/security-advisories`

Capture the complete output AND the exit code from this command.

## PHASE 2: Check for Critical Failures

Before proceeding with analysis, check if `composer report` completed successfully.

### Critical Failure Detection

Check for these blocking conditions:

**Exit Code Non-Zero:**
- Command failed catastrophically
- Cannot proceed with spec creation

**Fatal Errors in Output:**
- PHP Fatal error
- PHP Parse error
- Class not found errors
- Composer autoload failures
- Missing dependencies

**Tool Failures:**
- IDE Helper cannot generate files
- Rector crashes
- PHPStan cannot analyze code
- Tests cannot run

### If Critical Failure Detected

**DO NOT CREATE A SPEC**. Instead, display an error message to the user:

```
⚠️  Critical Failure Detected - Cannot Create Spec

The `composer report` command encountered a critical failure that prevents analysis:

[DESCRIBE THE SPECIFIC FAILURE]

**What This Means:**
The codebase has blocking issues that must be resolved before we can assess modernization needs.

**Recommended Actions:**

[Choose appropriate action based on failure type:]

**For Syntax/Parse Errors:**
1. Review the error output above
2. Fix the syntax error in [FILE]
3. Run `composer report` again to verify it completes
4. Then re-run this command

**For Missing Dependencies:**
1. Run `composer install` to ensure all dependencies are installed
2. Check that all required packages are in composer.json
3. Run `composer report` again
4. Then re-run this command

**For Autoload Issues:**
1. Run `composer dump-autoload`
2. Verify PSR-4 autoload configuration in composer.json
3. Run `composer report` again
4. Then re-run this command

**For Tool Installation Issues:**
1. Ensure all required tools are installed:
   - barryvdh/laravel-ide-helper
   - rector/rector
   - tightenco/duster
   - larastan/larastan
   - pestphp/pest
2. Run `composer install`
3. Run `composer report` again
4. Then re-run this command

**General Approach:**
Work through the error interactively until `composer report` can complete successfully.
Once it runs (even with thousands of reported issues), return here to create the modernization spec.

---

Need help resolving the blocking issue? Share the error and I can help you fix it.
```

**STOP HERE** - Do not proceed to Phase 3 if there are critical failures.

## PHASE 3: Analyze Quality Report

Only proceed here if `composer report` completed successfully (even if with many issues).

Read and analyze the complete output from `composer report`.

**Important**: The output may be very large (thousands of errors in legacy projects). Store the full output in a file for reference, then work with summaries and patterns for the spec creation.

Categorize the REMAINING issues (after automatic fixes) into these groups:

### 1. Security Vulnerabilities
Dependencies with known security issues:
- Package name and version
- Vulnerability description
- Recommended action

### 2. PHPStan Errors (By Level)
Static analysis errors remain after IDE helper model docblocks are generated.

**Break down by PHPStan level (0-6):**
- Count errors that would appear at each level
- Identify which levels have the most errors
- Common error patterns per level

This breakdown will create incremental tasks: fix Level 0, then Level 1, up to Level 6.

**For the spec, document:**
- Total PHPStan errors at Level 6
- Error count at each level (0-6)
- Top problem files
- Common error patterns
- Complexity assessment per level

### 3. Test Failures
Tests that fail after automatic refactorings:
- Number of failing tests
- Root causes
- Categories of failures

### 4. Changes Applied
Document what was automatically fixed:
- Rector refactorings applied
- Duster convention fixes applied
- Model docblocks generated

For each category of REMAINING issues, identify:
- Total number of issues
- Files affected
- Estimated complexity
- Whether issues indicate bugs or technical debt

## PHASE 4: Create Remediation Specification

Create a new spec folder following AgentOS conventions:

```
agent-os/specs/YYYY-MM-DD-legacy-modernization/
```

Create the following files in this folder:

### File 1: `spec.md`

Create a comprehensive specification document with these sections:

**## Overview**
- Brief summary of legacy project state
- What `composer report` automatically fixed
- What remains to be fixed manually
- Overall goal: achieve clean `composer report` with zero remaining errors

**## Automatic Fixes Applied**

Document what tools fixed automatically:
- Rector refactorings (import statements, modernizations)
- Duster Laravel convention fixes
- Model docblocks generated for PHPStan
- IDE helper files created

**## Remaining Issues Analysis**

**### Security Vulnerabilities: [COUNT]**
- Package name, version, CVE, upgrade path for each
- Impact if not fixed

**### PHPStan Errors: [TOTAL_COUNT]**

**Strategy: Incremental Level Progression**

This project will address PHPStan errors incrementally, starting at Level 0 and progressing to Level 6. This approach makes the work manageable and provides clear checkpoints.

**Error Count by Level:**
- Level 0: [count] errors
- Level 1: [count] errors
- Level 2: [count] errors
- Level 3: [count] errors
- Level 4: [count] errors
- Level 5: [count] errors
- Level 6: [count] errors (current - includes all above)

**Top Problem Files:**
1. [file] - [error count]
2. [file] - [error count]
3. [file] - [error count]

**Common Error Patterns:**
- [Pattern] - occurs in [count] places
- [Pattern] - occurs in [count] places

**Note**: Each level builds on the previous. Fix Level 0 completely before moving to Level 1.

**### Test Failures: [COUNT]**
- Total failing tests
- Categories of failures
- Root causes

**## Spec Scope**

What this spec will address:
1. **Security Vulnerabilities** - Upgrade or replace vulnerable dependencies
2. **Type Safety (Incremental)** - Fix PHPStan errors level by level (0→6)
3. **Test Suite** - Fix all failing tests
4. **Verification** - Ensure `composer report` completes successfully

**## Out of Scope**

- Code style issues (already fixed by Duster/Pint)
- Missing declare(strict_types) (already added by Pint)
- Basic refactorings (already applied by Rector)
- Model docblocks (already generated by IDE Helper)
- Additional improvements not surfaced by the tools

**## Success Criteria**

- [ ] `composer report` completes successfully
- [ ] No security vulnerabilities reported
- [ ] PHPStan Level 6 analysis passes with zero errors
- [ ] All tests in the test suite pass
- [ ] Application runs without errors
- [ ] No critical functionality has been broken

**## Implementation Strategy**

### Recommended Phase Order

**Phase 1: Security Vulnerabilities**
Address all security vulnerabilities first.
Estimated effort: [time]

**Phase 2-8: PHPStan Levels (0→6)**
Fix PHPStan errors incrementally:
- Phase 2: PHPStan Level 0
- Phase 3: PHPStan Level 1
- Phase 4: PHPStan Level 2
- Phase 5: PHPStan Level 3
- Phase 6: PHPStan Level 4
- Phase 7: PHPStan Level 5
- Phase 8: PHPStan Level 6

Each level must pass completely before moving to the next.
Estimated effort per level: [varies by error count]

**Phase 9: Test Suite**
Fix all failing tests.
Estimated effort: [time]

**Phase 10: Final Verification**
Run `composer report` and verify everything passes.
Estimated effort: [time]

**Total estimated effort**: [total time]

### Testing Strategy

- Run PHPStan at current level after each fix
- Run full test suite after completing each PHPStan level
- Test critical user paths manually after major changes
- Monitor for regressions

**## Risk Assessment**

### High-Risk Changes

Files requiring careful attention:
- Files with PHPStan errors in critical business logic
- Core models with complex relationships
- Controllers handling financial/sensitive operations
- Services with external integrations

**Recommended mitigation**:
- Extra manual testing for these areas
- Deploy to staging first
- Have rollback plan ready

### Rollback Strategy

- Each PHPStan level should be its own PR
- Tag releases before each merge
- Maintain ability to revert per-phase

**## Estimated Effort**

Based on error counts:

- Phase 1: [time] - Security vulnerabilities
- Phase 2: [time] - PHPStan Level 0 ([count] errors)
- Phase 3: [time] - PHPStan Level 1 ([count] errors)
- Phase 4: [time] - PHPStan Level 2 ([count] errors)
- Phase 5: [time] - PHPStan Level 3 ([count] errors)
- Phase 6: [time] - PHPStan Level 4 ([count] errors)
- Phase 7: [time] - PHPStan Level 5 ([count] errors)
- Phase 8: [time] - PHPStan Level 6 ([count] errors)
- Phase 9: [time] - Test suite fixes
- Phase 10: [time] - Final verification

**Total**: [total time]

### File 2: `tasks.md`

Create a detailed task breakdown:

```markdown
# Legacy Modernization Tasks

Complete these tasks in order. Each PHPStan level must be 100% clean before proceeding to the next.

## Understanding What Was Automatically Fixed

- [ ] Review Rector changes from composer report output
- [ ] Review Duster changes from composer report output
- [ ] Verify model docblocks were generated
- [ ] Understand baseline for remaining work

## Phase 1: Security Vulnerabilities

- [ ] 1.1 Document all vulnerable dependencies
- [ ] 1.2 Research upgrade paths for each vulnerability
- [ ] 1.3 Test dependency upgrades in isolation
- [ ] 1.4 Update composer.json with fixed versions
- [ ] 1.5 Run composer update
- [ ] 1.6 Test affected functionality
- [ ] 1.7 Verify security check passes
- [ ] 1.8 Run full test suite

## Phase 2: PHPStan Level 0

- [ ] 2.1 Configure PHPStan to Level 0
- [ ] 2.2 Run PHPStan at Level 0: `composer stan -- --level=0`
- [ ] 2.3 Fix Level 0 errors (basic checks, unknown classes)
- [ ] 2.4 Verify Level 0 passes: `composer stan -- --level=0`
- [ ] 2.5 Run full test suite
- [ ] 2.6 Commit Level 0 fixes

## Phase 3: PHPStan Level 1

- [ ] 3.1 Run PHPStan at Level 1: `composer stan -- --level=1`
- [ ] 3.2 Fix Level 1 errors (possibly undefined variables)
- [ ] 3.3 Verify Level 1 passes: `composer stan -- --level=1`
- [ ] 3.4 Run full test suite
- [ ] 3.5 Commit Level 1 fixes

## Phase 4: PHPStan Level 2

- [ ] 4.1 Run PHPStan at Level 2: `composer stan -- --level=2`
- [ ] 4.2 Fix Level 2 errors (unknown methods on objects)
- [ ] 4.3 Verify Level 2 passes: `composer stan -- --level=2`
- [ ] 4.4 Run full test suite
- [ ] 4.5 Commit Level 2 fixes

## Phase 5: PHPStan Level 3

- [ ] 5.1 Run PHPStan at Level 3: `composer stan -- --level=3`
- [ ] 5.2 Fix Level 3 errors (return types, dead code)
- [ ] 5.3 Verify Level 3 passes: `composer stan -- --level=3`
- [ ] 5.4 Run full test suite
- [ ] 5.5 Commit Level 3 fixes

## Phase 6: PHPStan Level 4

- [ ] 6.1 Run PHPStan at Level 4: `composer stan -- --level=4`
- [ ] 6.2 Fix Level 4 errors (parameter types, property types)
- [ ] 6.3 Verify Level 4 passes: `composer stan -- --level=4`
- [ ] 6.4 Run full test suite
- [ ] 6.5 Commit Level 4 fixes

## Phase 7: PHPStan Level 5

- [ ] 7.1 Run PHPStan at Level 5: `composer stan -- --level=5`
- [ ] 7.2 Fix Level 5 errors (more strict type checking)
- [ ] 7.3 Verify Level 5 passes: `composer stan -- --level=5`
- [ ] 7.4 Run full test suite
- [ ] 7.5 Commit Level 5 fixes

## Phase 8: PHPStan Level 6

- [ ] 8.1 Run PHPStan at Level 6: `composer stan -- --level=6`
- [ ] 8.2 Fix Level 6 errors (strict type checking, mixed types)
- [ ] 8.3 Verify Level 6 passes: `composer stan -- --level=6`
- [ ] 8.4 Run full test suite
- [ ] 8.5 Return PHPStan config to Level 6 permanently
- [ ] 8.6 Commit Level 6 fixes

## Phase 9: Test Suite

- [ ] 9.1 Document all currently failing tests
- [ ] 9.2 Fix tests affected by Rector refactorings
- [ ] 9.3 Fix tests affected by Duster changes
- [ ] 9.4 Fix tests that found real bugs
- [ ] 9.5 Address test infrastructure issues
- [ ] 9.6 Verify full test suite passes
- [ ] 9.7 Run `composer test` to confirm

## Phase 10: Final Verification

- [ ] 10.1 Run `composer report` - must complete successfully
- [ ] 10.2 Verify no security vulnerabilities
- [ ] 10.3 Verify PHPStan Level 6 passes
- [ ] 10.4 Verify all tests pass
- [ ] 10.5 Manually test critical user paths
- [ ] 10.6 Verify application runs without errors
- [ ] 10.7 Document any remaining warnings (not errors)
- [ ] 10.8 Create summary of work completed

## Post-Implementation

- [ ] Deploy to staging environment
- [ ] Run smoke tests on staging
- [ ] Monitor for any issues
- [ ] Create PR for production deployment
- [ ] Document lessons learned
```

Adjust the task breakdown based on specific error counts at each level.

### File 3: `analysis/composer-report-output.md`

Store the complete output from `composer report`:

```markdown
# Composer Report Output

Generated: [CURRENT_DATE]
Command: `composer report`

## Full Command Output

[PASTE COMPLETE COMPOSER REPORT OUTPUT HERE]

## Summary Statistics

**Automatic Fixes Applied:**
- Rector refactorings: [count] changes
- Duster fixes: [count] changes
- Model docblocks: [count] models updated
- IDE helper files: generated

**Remaining Issues:**
- Security vulnerabilities: [count]
- PHPStan errors: [count total at Level 6]
- Test failures: [count]

## PHPStan Error Breakdown by Level

- Level 0: [count] errors
- Level 1: [count] additional errors
- Level 2: [count] additional errors
- Level 3: [count] additional errors
- Level 4: [count] additional errors
- Level 5: [count] additional errors
- Level 6: [count] additional errors

## Files Modified by Automatic Tools

### Rector Changes
[LIST FILES MODIFIED BY RECTOR]

### Duster Changes
[LIST FILES MODIFIED BY DUSTER]

### Model Docblocks Generated
[LIST MODELS UPDATED]

## Remaining Issues Breakdown

### Security Vulnerabilities
[PASTE SECURITY ADVISORY OUTPUT]

### PHPStan Errors
[PASTE PHPSTAN OUTPUT]

**Common error patterns**:
- [Pattern 1] - occurs in [count] places
- [Pattern 2] - occurs in [count] places

**Files with most errors**:
1. [file path] - [error count]
2. [file path] - [error count]
3. [file path] - [error count]

### Test Failures
[PASTE TEST FAILURE OUTPUT]

**Failure categories**:
- [Category] - [count] tests
- [Category] - [count] tests

## Implementation Notes

- Automatic fixes already applied
- Focus on incremental PHPStan level progression
- Each level must be 100% clean before moving forward
- Test thoroughly after each level
```

## PHASE 5: Display Results

Display to the user:

```
✓ Legacy project analysis complete!

Created specification at: agent-os/specs/YYYY-MM-DD-legacy-modernization/

## Summary

Automatic fixes applied:
- Rector refactorings: [count]
- Duster convention fixes: [count]
- Model docblocks generated: [count] models

Remaining issues requiring manual intervention:
- Security vulnerabilities: [count]
- PHPStan errors: [count] (will be addressed incrementally Level 0→6)
- Test failures: [count]

## PHPStan Strategy: Incremental Levels

PHPStan errors will be fixed incrementally:
- Level 0: [count] errors
- Level 1: [count] errors
- Level 2: [count] errors
- Level 3: [count] errors
- Level 4: [count] errors
- Level 5: [count] errors
- Level 6: [count] errors

Each level must pass completely before proceeding to the next.

## Files Created

- spec.md - Comprehensive remediation specification
- tasks.md - Phased task breakdown (10 phases)
- analysis/composer-report-output.md - Complete tool output

## Next Steps

1. Review the specification at agent-os/specs/YYYY-MM-DD-legacy-modernization/spec.md
2. Note that Rector and Duster have already applied automatic fixes
3. The remaining issues require manual code changes
4. When ready to begin fixes, use the standard spec implementation process:
   - Phase 1: Security Vulnerabilities
   - Phases 2-8: PHPStan Levels 0→6 (one level at a time)
   - Phase 9: Test Suite
   - Phase 10: Final Verification

## Important Notes

- Automatic fixes (Rector, Duster, docblocks) are already applied
- PHPStan errors addressed incrementally - don't skip levels
- Each PHPStan level should be its own PR
- Test thoroughly after each level
- After all phases complete, `composer report` must run cleanly

Ready to begin implementation when you are!
```

## Standards Compliance

Ensure all analysis and documentation follows:

{{standards/global/*}}
{{standards/backend/*}}
{{standards/testing/*}}
