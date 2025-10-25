# prepare-legacy-project Command

## Purpose

This command analyzes a legacy Laravel project using comprehensive quality tools and creates a detailed specification for bringing the project up to modern development standards.

**Important**: This command does NOT manually fix issues - `composer report` applies automatic fixes via Rector and Duster, and this command documents what remains to be fixed manually.

## What It Does

1. Runs `composer report` which executes:
   - **IDE Helper** - Generates autocomplete helpers and model docblocks
   - **Rector** - **Applies automated refactorings automatically** (imports, modernizations)
   - **Duster** - **Applies Laravel convention fixes automatically**
   - **PHPStan** - Static analysis at Level 6 (reports remaining errors)
   - **Pest** - Runs test suite
   - **Security Check** - Checks for vulnerable dependencies via roave/security-advisories

2. Analyzes output to categorize:
   - **What was automatically fixed** (Rector/Duster changes, model docblocks)
   - **What remains** (security vulnerabilities, PHPStan errors, test failures)

3. Creates a comprehensive spec focusing on manual interventions needed:
   - Security vulnerabilities requiring dependency upgrades
   - PHPStan errors needing type declarations or code clarification
   - Test failures needing fixes
   - Implementation phases with clear priorities

## Key Understanding

**Automatic fixes happen during `composer report`:**
- Rector refactors code (imports, patterns, modernizations)
- Duster fixes Laravel conventions
- IDE Helper generates model docblocks for PHPStan
- These changes are saved to disk automatically

**Manual fixes are what the spec documents:**
- Security vulnerabilities (dependency upgrades)
- PHPStan errors after model docblocks are in place
- Test failures after refactorings
- Code that needs type clarification

## When to Use

Use this command when:
- You've just pulled AgentOS into an existing Laravel project
- The project hasn't been using modern quality tools
- You need to understand what work remains after automatic fixes
- You want to establish a baseline before starting feature work

## Prerequisites

The project must have a `composer report` script configured with:
```json
{
  "scripts": {
    "report": [
      "@php artisan ide-helper:generate",
      "@php artisan ide-helper:meta",
      "@php artisan ide-helper:models -M",
      "composer rector",
      "vendor/bin/duster fix",
      "composer stan",
      "composer test",
      "composer update --dry-run roave/security-advisories"
    ]
  }
}
```

Tools must be installed:
- barryvdh/laravel-ide-helper
- rector/rector with driftingly/rector-laravel
- tightenco/duster
- larastan/larastan (PHPStan)
- pestphp/pest
- roave/security-advisories

## Multi-Agent Mode (Claude Code)

In Claude Code, this command will be available as `/prepare-legacy-project`.

It will:
1. Execute `composer report` (which applies automatic fixes)
2. Analyze what was fixed vs. what remains
3. Create specification documenting manual work needed
4. Display summary

## Single-Agent Mode (Other Tools)

The command is split into 3 sequential steps:

1. **1-run-analysis.md** - Executes `composer report` and captures output (applies automatic fixes)
2. **2-analyze-issues.md** - Categorizes automatic fixes vs. remaining issues
3. **3-create-specification.md** - Creates spec documentation for manual work

Run each step in order, providing the output from previous steps.

## Output

Creates a spec at `agent-os/specs/YYYY-MM-DD-legacy-modernization/` with:

```
agent-os/specs/YYYY-MM-DD-legacy-modernization/
├── spec.md                           # Complete specification with:
│                                     #   - What was automatically fixed
│                                     #   - What remains for manual work
│                                     #   - Implementation phases
│                                     #   - Risk assessment
├── tasks.md                          # Phased task breakdown:
│                                     #   - Phase 1: Security Vulnerabilities
│                                     #   - Phases 2-8: PHPStan Levels 0→6 (incremental)
│                                     #   - Phase 9: Test Suite
│                                     #   - Phase 10: Final Verification
└── analysis/
    └── composer-report-output.md     # Full composer report output with analysis
```

## Expected Results

The specification will document:

**Automatic Fixes Applied:**
- Rector refactorings (count and files affected)
- Duster convention fixes (count and files affected)
- Model docblocks generated (count of models)

**Remaining Issues (Manual Work):**
- Security vulnerabilities with upgrade paths
- PHPStan errors broken down by level (0-6) for incremental fixing
- Test failures by root cause
- Estimated effort per phase

**Success Criteria:** `composer report` must run cleanly with zero remaining errors

## After Running This Command

1. Review what Rector and Duster automatically fixed (via git diff)
2. Review the specification for remaining manual work
3. Commit the automatic fixes
4. Begin manual fixes using standard AgentOS spec implementation workflow:
   - Phase 1: Security vulnerabilities
   - Phases 2-8: PHPStan errors level-by-level (0→6)
   - Phase 9: Test failures
   - Phase 10: Verification
5. Create separate PRs for each phase (or combine PHPStan levels as appropriate)

## Important Notes

- **Rector and Duster modify code automatically** - review changes before committing
- The spec documents **only what remains** after automatic fixes
- **Model docblocks are auto-generated** - you don't need to write them
- **declare(strict_types)** is added by Pint automatically
- PHPStan errors that remain often surface real bugs
- Test failures may be actual bugs found by better type safety
- Focus is on `composer report` output only - not additional improvements

## What's Out of Scope

This command only analyzes `composer report` output. It does NOT look for:
- Additional refactoring opportunities beyond what Rector suggests
- Code quality issues not caught by the tools
- Architecture improvements
- Performance optimizations
- Additional test coverage opportunities

These may be addressed by future AgentOS commands, but `prepare-legacy-project` is strictly focused on achieving a clean `composer report`.
