# Guidelines for Building High-Quality AI Agents

This document provides a comprehensive framework and set of best practices for constructing effective, reliable, and maintainable AI agents. It is designed to be used as a primary context source for an LLM tasked with creating new agents.

**Source:** Insights derived from analysis of 2,500+ agent files across public repositories, combined with industry best practices.

## 1. The .agent.md Master Template: Structure and Best Practices

The standard format for defining an agent is a Markdown file with a YAML frontmatter block, typically saved with a `.agent.md` or `.md` extension. This hybrid approach separates machine-readable configuration from human-and-AI-readable instructions, providing clarity and efficiency.

### 1.1 Part I: YAML Frontmatter for Configuration

The YAML frontmatter, enclosed by `---`, contains structured metadata that the agent execution framework parses to configure the agent's environment and core parameters.

**Key Configuration Parameters:**

| Parameter   | Description                                                                                      | Example                                            |
| ----------- | ------------------------------------------------------------------------------------------------ | -------------------------------------------------- |
| description | A concise, human-readable summary of the agent's purpose. (Required)                             | `description: "Refactors Python code to PEP8"`     |
| mode        | The agent's mode: "primary", "subagent", or "all" (defaults to "all" if not specified).          | `mode: "subagent"`                                 |
| model       | The identifier for the LLM to be used (e.g., provider/model-name).                               | `model: "anthropic/claude-sonnet-4-5"`             |
| temperature | A float between 0.0 (deterministic) and 1.0 (creative) that controls response randomness.        | `temperature: 0.1`                                 |
| tools       | A map of available tool names enabled (true) or disabled (false).                                | `tools: { bash: true, edit: true }`                |
| permission  | A map of permissions ("edit", "bash", or "webfetch") to permission levels ("allow", "ask", "deny"). | `permission: { edit: "ask", bash: "deny" }`     |
| disable     | Set to true to disable the agent.                                                                | `disable: false`                                   |
| prompt      | Path to custom system prompt file using {file:path} syntax.                                      | `prompt: "{file:./prompts/review.txt}"`            |
| version     | Semantic version for change tracking.                                                            | `version: "1.2.0"`                                 |

### 1.2 Part II: Markdown Body for System Prompts

The Markdown body contains the system prompt—the detailed, natural language instructions that guide the LLM's behavior. Using a structured format with clear sections helps the LLM follow instructions more reliably.

**Critical Ordering Principle:** Research shows that **commands and tools should appear early** in the agent definition. Agents reference these frequently, and LLMs give more weight to information at the beginning and end of context.

**Recommended Section Order:**

1. **Persona** - Identity and expertise
2. **Commands & Tools** - Executable commands with flags (EARLY placement critical)
3. **Core Responsibilities** - Primary duties
4. **Project Knowledge** - Tech stack, file structure
5. **Operational Protocol** - Step-by-step workflow
6. **Code Style & Examples** - Show, don't tell
7. **Constraints & Boundaries** - Three-tier (Always/Ask/Never)
8. **Agent Collaboration** - Delegation patterns
9. **Few-Shot Examples** - Concrete input/output demos

### 1.3 Recommended Template Structure

```md
---
description: "A concise description of what the agent does"
mode: subagent
model: "anthropic/claude-sonnet-4-5"
temperature: 0.1
version: "1.0.0"
tools:
  write: false
  edit: false
  bash: false
permission:
  edit: deny
  bash: ask
---

# <AGENT> Persona

Define the agent's identity, expertise, and tone. Example: "You are an expert Senior TypeScript Engineer and a meticulous code reviewer."

## Commands & Tools

**Put executable commands EARLY - agents reference these frequently.**

| Command             | Purpose                    | Flags/Options              |
| ------------------- | -------------------------- | -------------------------- |
| `npm test`          | Run test suite             | `--coverage`, `--watch`    |
| `npm run lint`      | Check code style           | `--fix` for auto-repair    |
| `npm run build`     | Compile TypeScript         | Outputs to `dist/`         |

## Core Responsibilities

Use a bulleted list to clearly enumerate the agent's primary duties:

- Analyze the code against the provided linting rules
- Generate a structured report of all violations
- Provide actionable recommendations

## Project Knowledge

**Tech Stack:** React 18, TypeScript 5.4, Vite 5, Tailwind CSS 3.4

**File Structure:**
- `src/` – Application source code
- `tests/` – Unit and integration tests
- `docs/` – Documentation files

## Operational Protocol

Provide a precise, step-by-step sequence of actions:

1. **Ingest Context**: Read the provided file content and coding standards
2. **Analyze**: Systematically scan for issues
3. **Report**: Create structured findings in specified format

## Code Style Examples

**Show, don't tell.** One real code snippet beats three paragraphs of description.

```typescript
// ✅ Good - descriptive names, proper error handling
async function fetchUserById(id: string): Promise<User> {
  if (!id) throw new Error('User ID required');
  const response = await api.get(`/users/${id}`);
  return response.data;
}

// ❌ Bad - vague names, no error handling
async function get(x) {
  return await api.get('/users/' + x).data;
}
```

## Constraints and Boundaries

Use three-tier boundary system for clarity:

- ✅ **Always**: Write to `docs/`, follow style examples, run validation
- ⚠️ **Ask first**: Before modifying existing documents, adding dependencies
- 🚫 **Never**: Modify code in `src/`, edit config files, commit secrets

## Agent Collaboration

List specialist agents for delegation:

- **@reviewer**: Code quality and security analysis
- **@optimizer**: Performance assessment
- **@researcher**: Best practices research

## Few-Shot Examples

Provide concrete input-to-output examples demonstrating desired behavior.
```

## 2. Advanced Techniques for Agent Control

### 2.1 The "Commands First" Principle

Research from 2,500+ repositories shows that the most effective agents place executable commands early in their definition. This serves multiple purposes:

1. **Frequent Reference**: Agents constantly need to recall available commands
2. **Context Attention**: LLMs weight information at the beginning more heavily
3. **Quick Orientation**: Developers can immediately understand what the agent can do

**Bad Practice:**
```md
# Agent Persona
You are a helpful assistant...

## Responsibilities  
- Review code
- Run tests

## Background
[Three paragraphs of context]

## Commands (buried at bottom)
npm test
```

**Good Practice:**
```md
# Agent Persona
You are a test engineer...

## Commands & Tools
| Command | Purpose | Flags |
|---------|---------|-------|
| `npm test` | Run tests | `--coverage` |
| `npm run lint --fix` | Auto-fix style | |
| `pytest -v` | Verbose test output | |

## Core Responsibilities
...
```

### 2.2 Modular and Hierarchical Context

To keep agent definitions clean and maintainable, separate extensive or reusable instructions into modular files.

**Dynamic Context Injection:** Use Markdown links (`[link text](./path/to/context.md)`) within the agent's protocol. The execution framework should resolve these links at runtime, injecting the content of the linked file into the prompt.

**Hierarchical Agent Context:** OpenCode supports hierarchical agent configuration:

- **Global agents**: Stored in `~/.config/opencode/agent/` directory
- **Project agents**: Stored in `.opencode/agent/` directory within your project

Agent markdown files (`.md`) placed in these locations are automatically loaded. The filename becomes the agent name (e.g., `review.md` creates a "review" agent).

Project-specific agents can override global agents with the same name.

### 2.3 Encoding Complex Reasoning Patterns

Encode advanced reasoning frameworks directly into the Operational Protocol section:

**Chain-of-Thought (CoT):** Instruct the agent to "think step-by-step" before providing a final answer. Use `<thinking>...</thinking>` blocks for reasoning.

**ReAct (Reason + Act):** Script a loop of thought, action (tool use), and observation directly into the protocol's numbered steps.

### 2.4 Code Examples Over Explanations

Research shows that concrete code examples are significantly more effective than prose descriptions.

**Principle:** One real code snippet showing your style beats three paragraphs describing it.

**Ratio Target:** Aim for at least 2:1 code examples to prose when defining style or patterns.

**Show the transformation:**
```typescript
// ❌ Current (problematic)
const query = `SELECT * FROM users WHERE email = '${userEmail}'`;

// ✅ Fixed (secure)
const query = "SELECT * FROM users WHERE email = ?";
const result = await db.query(query, [userEmail]);
```

### 2.5 Tech Stack Specificity

Be precise about your technology stack. Vague descriptions lead to inconsistent outputs.

| Specificity Level | Example                                    | Effectiveness |
| ----------------- | ------------------------------------------ | ------------- |
| Low               | "React project"                            | Poor          |
| Medium            | "React with TypeScript"                    | Moderate      |
| High              | "React 18, TypeScript 5.4, Vite 5"         | Good          |
| Optimal           | "React 18.2 + TypeScript 5.4 + Vite 5 + Tailwind CSS 3.4" | Excellent |

## 3. Lifecycle Management and Operational Best Practices

Treat agent definitions as first-class software artifacts by adopting a "Prompts as Code" paradigm.

### 3.1 Mitigating "Context Rot" and Managing File Size

LLM performance can degrade as prompt length increases ("context rot"). Information in the middle of a long prompt is often given less weight.

**Guidelines:**
- **Be Concise**: Keep total assembled prompt under 3000 tokens when possible
- **Structure for Attention**: Place critical instructions at the very beginning and end
- **Modularize**: Extract lengthy reference material into linked files

### 3.2 Line Count Guidelines

Based on analysis of 2,500+ repositories, these thresholds correlate with agent effectiveness:

| Agent Complexity | Description                        | Recommended Lines | Max Lines |
| ---------------- | ---------------------------------- | ----------------- | --------- |
| Simple           | Single, focused task               | ≤ 100             | 120       |
| Standard         | Multiple related tasks             | ≤ 150             | 180       |
| Complex          | Orchestration, multi-phase         | ≤ 200             | 230       |
| Supervisor       | Multi-agent collaboration          | ≤ 250             | 280       |

**If exceeding thresholds:**
1. Consider splitting into multiple specialized agents
2. Extract reference material to linked files
3. Remove redundant explanations (favor code examples)
4. Validate that complexity justifies length

### 3.3 Version Control and Management

Agent definitions are dynamic assets requiring rigorous management.

**Store in Git:** All `.agent.md` and associated instruction files must be version controlled.

**Use Semantic Versioning (SemVer):**
- **Major (x.0.0)**: Breaking changes to behavior or output structure
- **Minor (0.x.0)**: New, non-breaking features or enhancements
- **Patch (0.0.x)**: Minor tweaks, bug fixes, typo corrections

### 3.4 The Six Core Areas Checklist

Top-performing agents cover these six areas. Use as a validation checklist:

| Area              | Description                                     | Priority  |
| ----------------- | ----------------------------------------------- | --------- |
| **Commands**      | Executable commands with flags/options          | Critical  |
| **Testing**       | Test commands, validation approaches            | High      |
| **Project Structure** | File locations, directory purposes          | High      |
| **Code Style**    | Examples of good/bad patterns                   | High      |
| **Git Workflow**  | Commit conventions, branch patterns             | Medium    |
| **Boundaries**    | Clear always/ask/never rules                    | Critical  |

### 3.5 Three-Tier Boundary Pattern

The most effective boundary definitions use a three-tier system with visual indicators:

```md
## Boundaries

- ✅ **Always do**: Write to `src/` and `tests/`, run tests before commits, follow naming conventions
- ⚠️ **Ask first**: Database schema changes, adding dependencies, modifying CI/CD config  
- 🚫 **Never do**: Commit secrets or API keys, edit `node_modules/`, modify vendor files
```

**Why this works:**
1. **Visual scanning**: Emojis enable quick comprehension
2. **Clear hierarchy**: Graduated permission levels
3. **Explicit boundaries**: "Never commit secrets" was the most common helpful constraint in analyzed repos

### 3.6 Tooling and Evaluation

A mature agentic workflow requires dedicated tooling:

- **Prompt Management**: Centralized versioning and collaboration
- **Automated Evaluation**: Test against benchmark tasks before deployment
- **Production Monitoring**: Track cost, latency, tool error rates, output quality

## 4. Agent Collaboration: Working in a Team

Complex tasks are rarely solved by a single agent. The most effective AI systems are built on collaboration, where multiple specialized agents work together.

### 4.1 The Principle of Specialization ("Designed Incompetence")

Design a team of hyper-focused agents, each an expert in its own domain. An agent designed as a "Code Refactoring Specialist" should not also handle database administration.

**Advantages:**
- **Improved Performance**: Hyper-focused prompts prevent hallucinations
- **Modularity**: Specialized agents can be reused across workflows
- **Robustness**: Distributed systems are more resilient to failure

### 4.2 Hierarchical Collaboration: Supervisors and Specialists

The most common pattern is a hierarchical one where a "Supervisor" agent coordinates "Specialist" agents.

- **Supervisor/Orchestrator**: Analyzes tasks, breaks them down, delegates to specialists
- **Specialist Agents (Sub-Agents)**: Execute specific sub-tasks with high precision

### 4.3 Defining Collaboration in the Agent File

```md
## Agent Collaboration

This agent acts as a Supervisor. When a task requires specialized skills, delegate to:

### Available Collaborators:

| Agent | Description | When to Use |
|-------|-------------|-------------|
| @code-generator | Writes clean, documented code from specifications | New code from scratch |
| @code-reviewer | Reviews code for quality and security | Auditing existing code |
| @test-writer | Generates comprehensive tests with pytest | Writing tests for code |
```

**Critical:** The `description` field in each agent's YAML frontmatter enables automatic delegation. The Supervisor uses descriptions to determine which Specialist fits a sub-task.

### 4.4 Wizard Pattern (Consultant Agents)

Some agents act as read-only consultants, providing expertise without implementation:

| Wizard        | Expertise                    | Use Case                              |
| ------------- | ---------------------------- | ------------------------------------- |
| @ts-wizard    | TypeScript type systems      | Complex types, generics, type errors  |
| @lit-wizard   | Lit framework web components | Shadow DOM, reactive properties       |
| @tw-wizard    | Tailwind CSS                 | Utility patterns, theming             |
| @effect-wizard | Effect-TS patterns          | Functional programming with Effect    |
| @a11y-wizard  | Accessibility (WCAG)         | ARIA patterns, inclusive design       |

Wizards provide recommendations that other agents implement.

## 5. Starter Templates

### 5.1 Minimal Agent Template

```md
---
name: agent-name
description: "[One sentence: what this agent does]"
---

You are an expert [role] for this project.

## Commands & Tools
| Command | Purpose |
|---------|---------|
| `npm test` | Run test suite |
| `npm run lint --fix` | Auto-fix linting |

## Core Responsibilities
- [Primary duty 1]
- [Primary duty 2]

## Boundaries
- ✅ **Always**: [required behaviors]
- ⚠️ **Ask first**: [cautious actions]
- 🚫 **Never**: [prohibited actions]
```

### 5.2 Documentation Agent Template

```md
---
name: docs-agent
description: "Expert technical writer for project documentation"
mode: subagent
tools:
  read: true
  write: true
  edit: true
  bash: false
permission:
  edit: allow
  bash: deny
---

# Documentation Agent Persona

You are an expert technical writer. You read code from `src/` and generate documentation in `docs/`.

## Commands & Tools
| Command | Purpose | Notes |
|---------|---------|-------|
| `npm run docs:build` | Build documentation | Checks for broken links |
| `npx markdownlint docs/` | Lint markdown | Validates your work |

## Project Knowledge
- **Tech Stack:** React 18, TypeScript, Vite, Tailwind CSS
- **Source Code:** `src/` (READ from here)
- **Documentation:** `docs/` (WRITE to here)
- **Tests:** `tests/` (reference for examples)

## Documentation Standards

Write for developers new to this codebase. Be concise, specific, value-dense.

```typescript
// ✅ Good documentation example
/**
 * Fetches user data by ID with automatic retry.
 * @param id - User's unique identifier
 * @returns User object or throws if not found
 * @example
 * const user = await fetchUser('abc123');
 */
```

## Boundaries
- ✅ **Always**: Write to `docs/`, follow style examples, run markdownlint
- ⚠️ **Ask first**: Before major modifications to existing docs
- 🚫 **Never**: Modify code in `src/`, edit config files, commit secrets
```

### 5.3 Test Agent Template

```md
---
name: test-agent
description: "Writes comprehensive tests for TypeScript/JavaScript code"
mode: subagent
tools:
  read: true
  write: true
  edit: true
  bash: true
permission:
  bash: allow
  edit: allow
---

# Test Agent Persona

You are a QA engineer specializing in test coverage. You write tests, you never remove failing tests.

## Commands & Tools
| Command | Purpose | Flags |
|---------|---------|-------|
| `npm test` | Run all tests | |
| `npm test -- --coverage` | Run with coverage | |
| `npm test -- --watch` | Watch mode | |
| `npm test path/to/file.test.ts` | Run specific file | |

## Project Knowledge
- **Test Framework:** Vitest 4.0+
- **Test Location:** `tests/` directory
- **Source Code:** `src/` (READ only)

## Test Standards

```typescript
// ✅ Good test structure (AAA pattern)
describe('UserService', () => {
  it('should return user by ID', async () => {
    // Arrange
    const mockUser = { id: '1', name: 'Test' };
    
    // Act
    const result = await service.getUser('1');
    
    // Assert
    expect(result).toEqual(mockUser);
  });
});
```

## Boundaries
- ✅ **Always**: Write to `tests/`, run tests after writing, use AAA pattern
- ⚠️ **Ask first**: Before adding new test dependencies
- 🚫 **Never**: Modify source code in `src/`, remove failing tests, skip test execution
```

## 6. Anti-Patterns to Avoid

### 6.1 Vague Personas
```md
# ❌ Bad
You are a helpful coding assistant.

# ✅ Good
You are an expert test engineer who writes tests for React components, 
follows these examples, and never modifies source code.
```

### 6.2 Missing Commands
```md
# ❌ Bad
Run the tests and fix any issues.

# ✅ Good
| Command | Purpose |
|---------|---------|
| `npm test -- --coverage` | Run tests with coverage report |
| `npm run lint --fix` | Auto-fix linting issues |
```

### 6.3 Ambiguous Boundaries
```md
# ❌ Bad
Be careful with sensitive files.

# ✅ Good
- 🚫 **Never**: Edit `*.env*`, `*.key`, `*.secret`, `node_modules/`
```

### 6.4 Description-Heavy, Example-Light
```md
# ❌ Bad
Use descriptive variable names that clearly indicate the purpose 
of the variable. Names should be self-documenting and follow 
camelCase convention. Avoid abbreviations unless they are 
universally understood...

# ✅ Good
```typescript
// ✅ Good
const userEmailAddress = 'user@example.com';
const maxRetryAttempts = 3;

// ❌ Bad  
const e = 'user@example.com';
const max = 3;
```
```

## 7. Quick Reference

### Effective Agent Checklist

Before deploying an agent, verify:

- [ ] Description is action-oriented and specific
- [ ] Commands appear early with flags/options
- [ ] Tech stack includes versions
- [ ] Code examples demonstrate style (not just describe it)
- [ ] Three-tier boundaries defined (✅/⚠️/🚫)
- [ ] Line count within threshold for complexity level
- [ ] Six core areas addressed (commands, testing, structure, style, git, boundaries)

### Line Count Quick Reference

| Complexity | Target | Max  |
| ---------- | ------ | ---- |
| Simple     | ≤100   | 120  |
| Standard   | ≤150   | 180  |
| Complex    | ≤200   | 230  |
| Supervisor | ≤250   | 280  |

### Three-Tier Boundary Emojis

| Tier | Emoji | Meaning |
| ---- | ----- | ------- |
| Always | ✅ | Required behaviors, safe actions |
| Ask | ⚠️ | Needs confirmation, potentially risky |
| Never | 🚫 | Prohibited, dangerous, off-limits |
