# Step 2: Analyze Remaining Issues

You are analyzing the output from `composer report` to understand what issues remain AFTER the automatic fixes have been applied by Rector, Duster, and IDE Helper.

## Important Context Note

The output may be very large (thousands of errors in legacy projects). You should:
1. Store the full output for reference
2. Work with summaries and patterns for analysis
3. Focus on categorization and counts rather than listing every error

## Review the Output

Carefully read through the complete output from the `composer report` command that you captured in Step 1.

**Remember**: Rector and Duster have already applied fixes. Focus on issues that remain.

## First: Document What Was Automatically Fixed

Before analyzing remaining issues, document what was automatically corrected:

### Rector Refactorings Applied
List the files and changes Rector made automatically:
- Import optimizations
- Code modernizations
- Pattern updates
- Count of changes

### Duster Laravel Convention Fixes Applied
List the files and changes Duster made automatically:
- Laravel convention fixes
- Code style improvements
- Count of changes

### IDE Helper Model Docblocks Generated
List which models had docblocks generated:
- Model names
- Property and relationship annotations added
- This helps PHPStan understand your models

## Second: Categorize Remaining Issues

Group issues that REMAIN AFTER automatic fixes:

### 1. Security Vulnerabilities
Dependencies with known security issues:
- Package name and current version
- Vulnerability description (CVE reference if available)
- Severity level
- Recommended upgrade path or workaround
- Impact if not fixed

**Document for this category:**
- Total vulnerable packages
- Severity breakdown
- Which require major version upgrades
- Any breaking changes expected

### 2. PHPStan Errors (By Level 0-6)

**Critical**: Break down PHPStan errors by level (0-6), not just as one group.

The spec will create incremental tasks where you fix Level 0 completely, then Level 1, and so on.

**For each level, run PHPStan at that level and count errors:**

You may need to run these commands to get counts:
- `composer stan -- --level=0` - Count errors at Level 0
- `composer stan -- --level=1` - Count errors at Level 1
- `composer stan -- --level=2` - Count errors at Level 2
- `composer stan -- --level=3` - Count errors at Level 3
- `composer stan -- --level=4` - Count errors at Level 4
- `composer stan -- --level=5` - Count errors at Level 5
- `composer stan -- --level=6` - Count errors at Level 6 (already ran)

**Document for each level:**
- Error count at that level
- Common error types at that level
- Complexity assessment (low/medium/high)

**Also document overall:**
- Total PHPStan errors at Level 6
- Top 10 problem files across all levels
- Most common error patterns
- Files with most errors

**Important**: Model property access should already work via generated docblocks. PHPStan errors that remain are:
- Missing type declarations on methods
- Incorrect or ambiguous return types
- Parameter types that need specification
- Complex array/collection types needing PHPDoc for specificity
- Logic that's unclear to static analysis

### 3. Test Failures
Tests that fail after the automatic refactorings:
- Number of failing tests
- Which tests are failing
- Root causes of failures
- Whether failures indicate actual bugs or just outdated tests
- Test setup/teardown problems

**Document for this category:**
- Total failing tests
- Failure patterns (similar root causes)
- Tests broken by Rector changes vs. actual bugs
- Tests that need updates vs. code that needs fixes

## Create Summary Statistics

Calculate and document:
- **Automatic fixes applied**: What Rector/Duster/IDE Helper changed
- **Remaining issues by category**:
  - Security vulnerabilities: [count]
  - PHPStan errors by level (0-6): [counts for each]
  - Test failures: [count]
- **Files requiring manual work**: Unique count
- **Top 10 problem files**: Files with most remaining issues
- **Estimated complexity**: Overall assessment (low/medium/high)
- **Estimated effort**: Rough time estimate per phase

## Identify Implementation Strategy

Based on your analysis:
- Security vulnerabilities should be addressed first (Phase 1)
- PHPStan errors will be fixed incrementally Level 0→6 (Phases 2-8)
- Test failures come after PHPStan is clean (Phase 9)
- Final verification (Phase 10)

## Save Analysis

Structure your complete analysis. You'll use this to create the specification in the next step.

Include:
1. What was automatically fixed (with counts)
2. PHPStan error breakdown by level (0-6)
3. Security vulnerabilities (with details)
4. Test failures (with categories)
5. Implementation phase recommendations

## Next Step

Once you've completed the analysis and have:
1. Documented what was automatically fixed
2. Broken down PHPStan errors by level (0-6)
3. Categorized security vulnerabilities and test failures
4. Identified top problem files

Proceed to step 3: `3-create-specification.md`

Display to user:

```
✓ Issue analysis complete.

## Automatic Fixes Applied
- Rector refactorings: [count]
- Duster convention fixes: [count]
- Model docblocks generated: [count] models

## Remaining Issues (Manual Intervention Required)
- Security vulnerabilities: [count]
- PHPStan errors (by level):
  - Level 0: [count]
  - Level 1: [count]
  - Level 2: [count]
  - Level 3: [count]
  - Level 4: [count]
  - Level 5: [count]
  - Level 6: [count]
- Test failures: [count]

Files requiring manual work: [count]

## Implementation Strategy
- Phase 1: Security Vulnerabilities
- Phases 2-8: PHPStan Levels 0→6 (incremental)
- Phase 9: Test Suite
- Phase 10: Final Verification

Next: Run command 3-create-specification.md to create the remediation spec.
```

## Standards Reference

Follow these standards during analysis:

{{standards/global/*}}
{{standards/backend/*}}
{{standards/testing/*}}
