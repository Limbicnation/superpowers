# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Superpowers is a software development workflow system for coding agents (Claude Code, Codex, OpenCode). It provides composable "skills" - documented best practices and patterns that trigger automatically during development tasks.

**Key characteristics:**

- Documentation-driven: Skills are markdown files with YAML frontmatter, not compiled code
- Zero external dependencies: Pure bash/Node.js scripts and markdown
- Multi-platform: Claude Code (plugin), Codex (bootstrap script), OpenCode (Node.js plugin)

## Running Tests

```bash
# Run unit tests (no external dependencies)
./tests/opencode/run-tests.sh

# Run with verbose output
./tests/opencode/run-tests.sh --verbose

# Run integration tests (requires OpenCode)
./tests/opencode/run-tests.sh --integration

# Run specific test
./tests/opencode/run-tests.sh --test test-skills-core.sh
```

## Architecture

### Directory Structure

```
skills/                    # 17 skill modules (the core content)
  ├── brainstorming/
  ├── dependency-health-guardian/    # NEW: CVE scanning, upgrade paths
  ├── dispatching-parallel-agents/
  ├── executing-plans/
  ├── finishing-a-development-branch/
  ├── receiving-code-review/
  ├── refactoring-impact-predictor/  # NEW: Ripple effect analysis
  ├── requesting-code-review/
  ├── security-threat-modeler/       # NEW: OWASP Top 10, threat modeling
  ├── subagent-driven-development/
  ├── systematic-debugging/
  ├── test-driven-development/
  ├── using-git-worktrees/
  ├── using-superpowers/
  ├── verification-before-completion/
  ├── writing-plans/
  └── writing-skills/
lib/
  └── skills-core.js       # Shared utility: skill discovery, resolution, frontmatter parsing
hooks/
  ├── hooks.json           # Hook configuration (SessionStart events)
  ├── session-start.sh     # Bootstrap script injected at session start
  └── run-hook.cmd         # Cross-platform polyglot wrapper (Windows batch + bash)
commands/                  # Slash commands for Claude Code
agents/                    # Subagent definitions
.claude-plugin/            # Claude Code plugin metadata
.opencode/plugin/          # OpenCode.ai plugin implementation
.codex/                    # Codex integration scripts
```

### Skill Format

Each skill is a directory containing `SKILL.md` with this structure:

```markdown
---
name: skill-name
description: Use when [condition] - [what it does]
---

[Skill content in markdown]
```

### Skill Priority (Shadowing)

Skills resolve in order: **project > personal > superpowers**. Personal/project skills can override superpowers skills by using the same name.

### Core Module: lib/skills-core.js

Shared across platforms. Key exports:

- `extractFrontmatter(filePath)` - Parse YAML frontmatter from SKILL.md
- `findSkillsInDir(dir, sourceType, maxDepth)` - Discover skills recursively
- `resolveSkillPath(skillName, superpowersDir, personalDir)` - Handle shadowing
- `stripFrontmatter(content)` - Remove metadata for display

### Hook System

`hooks/hooks.json` defines SessionStart hooks that trigger on: `startup|resume|clear|compact`

The hook injects the `using-superpowers` skill content as JSON context at session start.

## Platform-Specific Entry Points

**Claude Code:** Plugin system with built-in Skill tool. Hook triggers `session-start.sh`.

**OpenCode:** `.opencode/plugin/superpowers.js` provides `use_skill` and `find_skills` custom tools.

**Codex:** `.codex/superpowers-codex` Node.js executable with `bootstrap`, `use-skill`, `find-skills` commands.

## Contributing Skills

1. Create `skills/{skill-name}/SKILL.md` with proper frontmatter
2. Follow `skills/writing-skills/SKILL.md` for the complete guide
3. Use `skills/testing-skills-with-subagents/SKILL.md` to validate quality
4. Test with `./tests/opencode/run-tests.sh`

## Version and Release

Current version is in `.claude-plugin/plugin.json`. Release notes are in `RELEASE-NOTES.md`.
