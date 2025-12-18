# BAML Dev Plugin for Claude Code

Type-safe LLM extraction with BAML — schema design, testing, and debugging tools based on Vaibhav Gupta's reliability-first philosophy.

## What is BAML?

[BAML](https://docs.boundaryml.com/) (Boundary ML) is a domain-specific language for building reliable LLM applications. Instead of fighting with JSON parsing and prompt engineering, you define typed schemas and let BAML handle the complexity.

### The Core Philosophy

1. **Schema Is The Prompt** — Your BAML types automatically become extraction instructions via `{{ ctx.output_format }}`
2. **Types Over Strings** — Use enums, unions, and structured types instead of parsing free-form text
3. **Fuzzy Parsing Is BAML's Job** — BAML's parser handles malformed JSON, missing quotes, and LLM quirks
4. **Transpiler Not Library** — BAML compiles to native Python/TypeScript with full type safety

## Installation

```bash
# Add the marketplace
/plugin marketplace add /path/to/claude-code-docs/baml-dev

# Install the plugin
/plugin install baml-dev@baml-dev
```

Then restart Claude Code.

## Features

### Skills

- **baml-philosophy** — Foundation skill covering Vaibhav Gupta's reliability-first approach
- **baml-implementation** — Core patterns with context-aware references for Python, TypeScript, Go, Ruby

### Agents

| Agent | Use When |
|-------|----------|
| `baml-architect` | Designing new schemas, converting types to BAML, visualizing with mermaid |
| `baml-debugger` | Extractions return wrong data, validation fails, provider issues |
| `baml-tester` | Running test suites, analyzing failures, validating changes |

### Commands

| Command | Description |
|---------|-------------|
| `/baml-init` | Initialize BAML in a project |
| `/baml-test [filter]` | Run BAML tests |
| `/baml-diagnose` | Diagnose configuration issues |
| `/baml-schema [task]` | Visualize or design schemas |

## Quick Start

### 1. Initialize a Project

```
/baml-init
```

Creates `baml_src/` with generators, clients, and example schema.

### 2. Design a Schema

Ask Claude to design a schema:

```
I need to extract invoice data from images including vendor, line items, and totals
```

Claude will use the `baml-architect` agent to:
- Design appropriate BAML types
- Create mermaid visualization
- Generate complete function and tests

### 3. Run Tests

```
/baml-test
```

Or filter:

```
/baml-test Invoice
```

### 4. Debug Issues

If something isn't working:

```
/baml-diagnose
```

Or ask Claude directly — it will use the `baml-debugger` agent.

## Example Schema

```baml
class LineItem {
  description string
  quantity int @assert(this > 0)
  unit_price float
  subtotal float
}

class Invoice {
  invoice_number string
  vendor string
  date string @description("ISO8601 format")
  line_items LineItem[]
  total float @assert(this > 0)
}

function ExtractInvoice(doc: image) -> Invoice {
  client GPT4o
  prompt #"
    {{ _.role("user") }}
    Extract invoice data:
    {{ doc }}
    {{ ctx.output_format }}
  "#
}
```

## Reference Documentation

The `baml-implementation` skill includes references for:

| Reference | Topics |
|-----------|--------|
| `providers.md` | OpenAI, Anthropic, Ollama, Azure, retries, fallbacks |
| `types-and-schemas.md` | Classes, enums, unions, maps, optionals |
| `validation.md` | @assert, @check, @@assert patterns |
| `advanced-patterns.md` | Hierarchical extraction, tool calling, streaming |
| `languages-python.md` | Python/Pydantic setup, async, streaming |
| `languages-typescript.md` | TypeScript setup, React hooks |
| `languages-other.md` | Go, Ruby, OpenAPI generators |
| `frameworks-langgraph.md` | LangGraph integration |
| `usecase-image-forms.md` | Receipt/invoice extraction, form filling |
| `usecase-workflows.md` | Agent loops, state machines |

## Examples

See [baml-examples](https://github.com/BoundaryML/baml-examples) for complete projects:

- `python-chatbot/` — Agent with tool selection
- `form-filler/` — Multi-turn form extraction
- `java-gradle-openapi-starter/` — Comprehensive patterns
- `nextjs-starter/` — Next.js integration

## Philosophy Deep Dive

Read the `baml-philosophy` skill for the full explanation of:

- The Reliability Problem (90% → 99.9%)
- Schema as DSL for prompts
- Why types beat string parsing
- The transpiler advantage

## Contributing

This plugin is part of the claude-code-docs project. Contributions welcome!

## License

MIT
