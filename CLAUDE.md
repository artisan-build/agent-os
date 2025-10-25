# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Agent OS is a framework for spec-driven agentic development that transforms AI coding agents into productive developers through structured workflows. It's a shell script-based system that installs into user codebases and provides standardized processes for product planning, spec creation, and task execution.

## Architecture

### Dual-Mode System

Agent OS v2.0 operates in two modes:

- **Multi-Agent Mode**: For tools that support subagents (Claude Code). Commands orchestrate multiple specialized agents.
- **Single-Agent Mode**: For tools without subagent support (Cursor, etc.). Commands deliver step-by-step prompts to a single agent.

Mode configuration is stored in `config.yml`.

### Installation Model

Agent OS uses a two-tier installation:

1. **Base Installation** (`~/agent-os`): User's main installation containing all profiles, standards, workflows, and scripts
2. **Project Installation** (`.agent-os/` in project): Self-contained copy of selected profile installed into each project

### Profile System

Profiles define complete workflow configurations:

- **Location**: `profiles/{profile-name}/`
- **Structure**:
  - `agents/` - Agent definitions and templates
  - `commands/` - Command entry points (multi-agent and single-agent subdirs)
  - `roles/` - Role definitions (implementers.yml, verifiers.yml)
  - `standards/` - Code standards organized by domain (global/, backend/, frontend/, testing/)
  - `workflows/` - Workflow steps (planning/, specification/, implementation/)
  - `profile-config.yml` - Profile metadata (inherits_from)

**Built-in Profiles**:
- `default` - Framework-agnostic profile
- `laravel` - Laravel/PHP-specific profile (doesn't inherit from default)

### Key Components

**Scripts** (`scripts/`):
- `base-install.sh` - Installs Agent OS to user's system
- `project-install.sh` - Installs Agent OS into a project from base installation
- `project-update.sh` - Updates existing project installation
- `create-profile.sh` - Creates new profiles
- `create-role.sh` - Creates new agent roles
- `common-functions.sh` - Shared utility functions

**Configuration** (`config.yml`):
```yaml
version: 2.0.3
base_install: true
multi_agent_mode: true
multi_agent_tool: claude-code
single_agent_mode: false
single_agent_tool: generic
profile: laravel
```

**Roles System** (`profiles/{profile}/roles/`):
- `implementers.yml` - Defines specialized implementation agents (database-engineer, api-engineer, ui-designer, testing-engineer)
- `verifiers.yml` - Defines verification agents (backend-verifier, frontend-verifier)
- Each role specifies: description, responsibilities, tools, standards to follow, and which verifier validates their work

## Development Workflow

### Working with Installation Scripts

All installation logic uses bash scripts with these patterns:

1. Source `common-functions.sh` for shared utilities
2. Use color-coded output functions: `print_step`, `print_success`, `print_error`, `print_warning`
3. Handle dry-run mode with `should_execute` checks
4. Track installed files in `INSTALLED_FILES` array
5. Use `set -e` for error handling (exit on error)

### Testing Installation Scripts

```bash
# Test base installation (dry run)
./scripts/base-install.sh --dry-run

# Test project installation (dry run)
./scripts/project-install.sh --dry-run --verbose

# Test with specific profile
./scripts/project-install.sh --profile laravel --dry-run

# Test update process
./scripts/project-update.sh --dry-run --overwrite-all
```

### Working with Profiles

**Creating a new profile**:
```bash
./scripts/create-profile.sh
# Follow interactive prompts
```

**Profile structure** must contain:
- All subdirectories: agents/, commands/, roles/, standards/, workflows/
- Both multi-agent and single-agent command versions
- Complete standards hierarchy (global/, backend/, frontend/, testing/)
- profile-config.yml with inherits_from setting

**Testing profile installation**:
```bash
./scripts/project-install.sh --profile your-profile-name --dry-run
```

### Working with Roles

**Creating a new role**:
```bash
./scripts/create-role.sh
# Select profile, role type (implementer/verifier), follow prompts
```

**Role definition structure** (YAML):
```yaml
role_type:
  - id: unique-identifier
    description: One-line description
    your_role: Agent's perspective of their role
    tools: Comma-separated tool list
    model: inherit or specific model
    color: terminal color name
    areas_of_responsibility: [List of what they do]
    example_areas_outside_of_responsibility: [List of what they don't do]
    standards: [List of standard globs like "global/*"]
    verified_by: [List of verifier IDs]
```

### Command Structure

Commands exist in two versions:

**Multi-agent**: `commands/{command-name}/multi-agent/{command-name}.md`
- Uses subagent delegation
- Orchestrates specialized agents
- Returns summarized results

**Single-agent**: `commands/{command-name}/single-agent/*.md`
- Step-by-step instructions
- May be split across multiple files (1-file.md, 2-file.md, etc.)
- Direct execution by single agent

## Common Development Tasks

### Adding a new command

1. Create command directories:
   ```bash
   mkdir -p profiles/{profile}/commands/{command-name}/{multi-agent,single-agent}
   ```

2. Create multi-agent version:
   - Single markdown file with subagent delegation logic
   - Use existing commands as templates

3. Create single-agent version:
   - Can be one file or split across multiple steps
   - Include XML-structured instructions with clear step markers

4. Test both modes:
   ```bash
   # Install with multi-agent mode
   ./scripts/project-install.sh --multi-agent-mode true --dry-run

   # Install with single-agent mode
   ./scripts/project-install.sh --single-agent-mode true --dry-run
   ```

### Modifying installation scripts

1. Edit the appropriate script in `scripts/`
2. Follow existing patterns in `common-functions.sh`
3. Test with `--dry-run` and `--verbose` flags
4. Verify all edge cases:
   - Fresh installation
   - Re-installation (--re-install)
   - Update scenarios
   - Missing directories
   - Permission issues

### Updating standards

Standards are organized hierarchically:

- `global/` - Universal standards (coding-style.md, conventions.md, naming.md, etc.)
- `backend/` - Backend-specific (api.md, migrations.md, models.md, queries.md)
- `frontend/` - Frontend-specific (accessibility.md, components.md, css.md, responsive.md)
- `testing/` - Testing standards (test-writing.md)

Edit standards in the appropriate profile, then use `project-update.sh` to push changes to projects.

### Version Updates

When updating version in `config.yml`:
1. Update version number in `config.yml`
2. Add changelog entry in `CHANGELOG.md`
3. Update version references in installation scripts if needed
4. Test installation flow end-to-end

## Script Execution Patterns

### Common Flags

All installation scripts support:
- `--dry-run` - Show what would happen without executing
- `--verbose` - Show detailed output
- `--help` - Display usage information

**project-install.sh specific**:
- `--profile PROFILE` - Use specific profile
- `--multi-agent-mode [true/false]` - Enable/disable multi-agent mode
- `--multi-agent-tool TOOL` - Specify tool (claude-code)
- `--single-agent-mode [true/false]` - Enable/disable single-agent mode
- `--single-agent-tool TOOL` - Specify tool (generic, cursor)
- `--re-install` - Delete and reinstall
- `--overwrite-all` - Overwrite all files during update
- `--overwrite-standards` - Overwrite only standards
- `--overwrite-commands` - Overwrite only commands
- `--overwrite-agents` - Overwrite only agents

### File Operations Pattern

Scripts follow this pattern for file operations:

```bash
# Check if should execute (handles dry-run)
if should_execute; then
  # Perform operation
  mkdir -p "$target_dir"
  cp "$source_file" "$target_file"

  # Track what was done
  INSTALLED_FILES+=("$target_file")
fi
```

## Important Conventions

1. **Date Formatting**: YYYY-MM-DD format for spec folders and timestamps
2. **Branch Naming**: Spec folder names without date prefix (e.g., `password-reset` from `2025-03-15-password-reset`)
3. **File Encoding**: All markdown files use UTF-8 with LF line endings
4. **Indentation**: 2 spaces for YAML, varies by language for code
5. **Agent Model Setting**: Use "inherit" instead of hardcoded model names to respect user's current Claude Code model selection

## Testing Checklist

Before committing changes to installation scripts:

- [ ] Run with `--dry-run` to verify output
- [ ] Test fresh installation path
- [ ] Test update path with `--re-install`
- [ ] Verify file permissions are correct
- [ ] Check that INSTALLED_FILES tracking works
- [ ] Test error handling (missing dirs, bad permissions)
- [ ] Verify config.yml is read correctly
- [ ] Test with different profile selections
- [ ] Ensure colored output works in terminal

## File Organization

```
agent-os/
├── config.yml                 # Main configuration
├── scripts/                   # Installation and management scripts
├── profiles/                  # Profile definitions
│   ├── default/              # Framework-agnostic profile
│   └── laravel/              # Laravel-specific profile
├── CHANGELOG.md              # Version history
└── README.md                 # User documentation
```

Project installations create:
```
project-root/
└── .agent-os/                # Agent OS installation
    ├── agents/               # Compiled agents (multi-agent mode)
    ├── commands/             # Compiled commands
    ├── instructions/         # Core instruction files
    ├── roles/                # Role definitions
    ├── standards/            # Code standards
    ├── workflows/            # Workflow definitions
    └── product/              # Product-specific docs (created by user)
        ├── mission.md
        ├── roadmap.md
        └── tech-stack.md
```
