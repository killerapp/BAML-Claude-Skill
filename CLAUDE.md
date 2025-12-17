# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

BAML Code Generation Skill for Claude Code - generates production-ready BAML (Boundary ML) applications from natural language requirements. The skill queries official BoundaryML repositories via MCP servers for real-time pattern matching and code generation.

**Key Stats**: 95%+ compilation success, 50-70% token optimization, 6 languages, 10+ frameworks

## Architecture

This is a **documentation-as-code** project where the skill logic is defined in markdown files that Claude Code reads and executes:

```
baml-codegen/
├── .claude-plugin/
│   └── plugin.json       # Plugin metadata (name, version, author, license)
├── SKILL.md              # Core skill definition (~3800 tokens) - Claude reads this
├── metadata.yaml         # Activation rules, capabilities, caching config
├── lib/                  # 18 algorithm modules as markdown
│   ├── mcp_interface.md  # MCP query patterns for BoundaryML repos
│   ├── code_generator.md # BAML synthesis algorithms
│   ├── validator.md      # 5-layer validation pipeline
│   └── ...               # Other modules
├── templates/            # Example patterns (extraction, classification, RAG, agents)
└── cache/                # Pattern cache structure
```

### Code Generation Flow

1. **Requirement Parser** - NL → structured spec
2. **Pattern Matcher** - Query MCP for similar patterns
3. **Code Generator** - Synthesize BAML types, functions, clients
4. **Test Generator** - Create pytest/Jest/RSpec tests
5. **Integration Builder** - Framework-specific code (FastAPI, Next.js, Rails, etc.)
6. **Validator** - 5-layer check (syntax, types, semantics, completeness, performance)
7. **Optimizer** - Token reduction

### Multi-Tier Caching

- **Tier 1 (Embedded)**: Core syntax in SKILL.md, never expires
- **Tier 2 (Session)**: Recent patterns, 15 min TTL
- **Tier 3 (Persistent)**: Top 20 patterns in `cache/`, 7 day TTL
- **Tier 4 (MCP Live)**: Real-time repository queries

## MCP Server Setup

```bash
# Required - Core BAML repository
claude mcp add "baml_Docs" --scope user -- npx -y mcp-remote https://gitmcp.io/BoundaryML/baml

# Optional - Examples repository
claude mcp add "baml_Examples" -- npx -y mcp-remote https://gitmcp.io/BoundaryML/baml-examples
```

The skill works offline with 80% functionality if MCP is unavailable.

## Token Budget

**Critical**: SKILL.md must stay under 4000 tokens (~3800 currently). When modifying the skill definition, verify token count stays within budget.

## Key Constraints

- **Multimodal Gotcha**: Image parameters MUST be included in prompt body:
  ```baml
  function ExtractFromImage(page_image: image) -> Result {
    prompt #"
      {{ page_image }}  // ← Required!
      {{ ctx.output_format }}
    "#
  }
  ```

- **Validation Pipeline**: All generated code goes through 5-layer validation before output

- **Offline Mode**: When MCP unavailable, use `lib/fallback_patterns.md` templates

## Local Development

### Install for Testing

```bash
# Add this repository as a local marketplace
/plugin marketplace add /home/vaskin/projects/BAML-Claude-Skill

# Install the plugin locally
/plugin install baml-codegen@baml-skills

# Restart Claude Code to load changes
```

### Development Cycle

```bash
# 1. Make changes to skill files
# 2. Uninstall current version
/plugin uninstall baml-codegen@baml-skills

# 3. Reinstall to pick up changes
/plugin install baml-codegen@baml-skills

# 4. Restart Claude Code
```

### Debug Mode

```bash
claude --debug
```

### Testing the Skill

Trigger the skill with keywords like:
- "Generate BAML to extract invoice data"
- "Create a sentiment classifier in BAML"
- "Build a RAG system with BAML"

## Plugin Publishing

Plugin metadata is in `.claude-plugin/marketplace.json`. Install via:
```
/plugin install baml-codegen@baml-skills
```

### Versioning

- **Marketplace version** (`marketplace.json` root): Package/distribution version
- **Plugin version** (`plugins[0].version`): Feature version (currently 1.2.0)
- **Skill version** (`SKILL.md`, `metadata.yaml`): Should match plugin version

## Release Automation

This repository uses [release-please](https://github.com/googleapis/release-please) for automated releases.

### How It Works

1. Use [Conventional Commits](https://www.conventionalcommits.org/) for commit messages:
   - `feat: add new feature` → bumps minor version
   - `fix: bug fix` → bumps patch version
   - `feat!: breaking change` → bumps major version

2. On push to `main`, release-please creates/updates a release PR with:
   - Updated version in `.claude-plugin/marketplace.json`
   - Updated `CHANGELOG.md`

3. Merging the release PR creates a GitHub release and tag

### Configuration Files

- `release-please-config.json` - Release configuration
- `.release-please-manifest.json` - Current version tracking
- `.github/workflows/release-please.yml` - GitHub Actions workflow

### Manual Version Updates

When updating versions manually, update these files:
- `.claude-plugin/marketplace.json` (`$.plugins[0].version`)
- `baml-codegen/.claude-plugin/plugin.json` (`$.version`)
- `baml-codegen/SKILL.md` (Version line)
- `baml-codegen/metadata.yaml` (`skill.version` and `version_info.current`)
- `.release-please-manifest.json`

## Licensing

- Repository: MIT
- Skill (baml-codegen/): Apache 2.0
