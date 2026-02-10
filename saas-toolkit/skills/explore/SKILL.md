---
description: Codebase exploration using specialized explore agents
allowed-tools:
  - Task
  - Read
  - Grep
  - Glob
user-invocable: true
---

# /explore — Codebase Exploration

Explore and understand the codebase using specialized agents for different exploration needs.

## Behavior

When the user asks to explore, determine the type of exploration needed and delegate to the appropriate agent:

### Codebase structure
Use the **explore-codebase** agent for:
- Finding files and directories
- Understanding project structure
- Searching for patterns, functions, or components
- Mapping dependencies between modules

### Database schema
Use the **explore-db** agent for:
- Listing tables and their schemas
- Understanding relationships and foreign keys
- Reviewing RLS policies
- Inspecting database functions

### Library documentation
Use the **explore-docs** agent for:
- Looking up API documentation
- Finding usage examples for libraries
- Checking framework conventions
- Understanding library configuration

### General research
Use the **websearch** agent for:
- Finding solutions to specific problems
- Researching best practices
- Looking up error messages
- Comparing approaches

## Process

1. **Understand the question** — What does the user want to explore?
2. **Choose the right agent** — Match the question to the most appropriate exploration agent.
3. **Delegate** — Launch the agent with a clear, specific prompt.
4. **Synthesize** — If multiple agents are needed, combine their findings into a coherent answer.

## Rules

- Use agents in parallel when exploring multiple independent things
- Keep the scope focused — don't explore everything at once
- Return structured, actionable findings
