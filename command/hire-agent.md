---
description:
  "Specialized agent creation through interactive clarification and intelligent
  configuration"
---

Create a new specialized OpenCode agent based on a description, using
interactive clarification and intelligent configuration.

Agent research patterns: @.patterns/agent-research.md

- Existing agents for reference: !`ls ~/.config/opencode/agent/`
- Current project context: !`pwd`
- Agent name and description: $ARGUMENTS

## Process

### 1. Parse Input

Extract agent name and initial description from `$ARGUMENTS`:

- Expected format: `<agent-name> "<description>"`
- Example: `api-validator "Validates API responses against OpenAPI specs"`

### 2. Interactive Clarification (Max 5 Questions)

Ask targeted questions to clarify the agent's purpose and capabilities:

**Question Categories:**

- **Primary Function**: Core task and success criteria
- **Commands & Tools**: What commands will this agent run? (Critical - commands
  go early!)
- **Scope & Boundaries**: What should the agent NEVER do? (designed
  incompetence)
- **Tool Requirements**: Reading files? Modifying code? Web access? Running
  commands?
- **Collaboration Needs**: Which other agents should it delegate to?
- **Mode**: Primary agent (user-facing) or subagent (called by others)?

**Question Format:**

Present ONE question at a time with:

- Multiple choice options (2-5 choices) in table format
- OR short answer (≤10 words)
- Record answers before proceeding

**Example Questions:**

| Q   | Question                                           | Format                                                        |
| --- | -------------------------------------------------- | ------------------------------------------------------------- |
| 1   | What is the primary task this agent performs?      | Short answer                                                  |
| 2   | What commands will this agent run most frequently? | Short answer (e.g., `npm test`, `pytest -v`)                  |
| 3   | Should this agent modify files or only analyze?    | A) Modify files B) Read-only analysis C) Both with permission |
| 4   | What level of autonomy should this agent have?     | A) Fully autonomous B) Ask before actions C) Report only      |
| 5   | What should this agent NEVER do?                   | Short answer (critical boundaries)                            |

Stop early if purpose is clear (minimum 2 questions).

### 3. Analyze Existing Agents

Research similar agents in `~/.config/opencode/agent/` to:

- Identify similar patterns and responsibilities
- Understand tool/permission configurations
- Extract relevant operational protocols
- Find appropriate collaboration patterns

**Note:** Existing agents follow a fantasy persona theme. When creating new
agents, consider adopting a similar fantasy-inspired name and persona that
reflects the agent's role.

Wizards provide consultation without implementation - perfect for agents needing
specialized guidance.

### 4. Intelligent Tool & Permission Configuration

Based on clarification answers, auto-configure:

**Tool Mapping:**

| Agent Purpose        | Tools                               | Permission                                |
| -------------------- | ----------------------------------- | ----------------------------------------- |
| Code Analysis/Review | read, grep, glob, list              | edit: deny, bash: deny                    |
| Code Generation      | read, write, edit, grep, glob       | edit: ask, bash: ask                      |
| Testing              | read, write, edit, bash, playwright | edit: allow (tests only), bash: allow     |
| Research             | read, grep, webfetch, context7      | edit: deny, webfetch: allow               |
| Documentation        | read, write, edit, grep             | edit: allow (\*.md, docs/\*\*), bash: ask |
| Optimization         | read, grep, glob, bash              | edit: ask, bash: allow                    |
| Debugging            | read, grep, glob, list, bash        | edit: deny, bash: allow                   |
| Orchestration        | todowrite, todoread, task           | edit: deny, bash: deny                    |

**Permission Patterns:**

- **Read-only agents**: `edit: deny, bash: deny, webfetch: deny`
- **Code modifiers**: `edit: ask, bash: ask` (or allow with file patterns)
- **Researchers**: `edit: deny, bash: deny, webfetch: allow`
- **Testers**: `edit: allow` for test files, `bash: allow`

**File-level permissions** (when granular control needed):

```yaml
permission:
  edit:
    "**/*.env*": "deny"
    "**/*.key": "deny"
    "**/*.secret": "deny"
    "**/*.md": "allow"
    "*": "ask"
```

### 5. Generate Agent Structure

Using research patterns from `@.patterns/agent-research.md`, create the agent
following this **critical section order**:

#### YAML Frontmatter

```yaml
---
description: "<clear, action-oriented summary from clarification>"
mode: subagent | primary | all
model: anthropic/claude-sonnet-4-5
temperature: <0.0-0.3 for deterministic, 0.3-0.7 for creative>
version: "1.0.0"
tools: <auto-configured based on purpose>
permission: <auto-configured based on purpose>
---
```

#### Markdown Body Sections (ORDER MATTERS)

**Section order is critical for effectiveness. Commands MUST appear early.**

1. **# The {Fantasy Title}** (Agent Persona)

   - Choose a fantasy-inspired name that reflects the agent's role (e.g., "The
     Shadow Tracker" for debugging, "The Code Sentinel" for review)
   - Define expertise and identity through the persona
   - Establish tone that matches the fantasy theme while remaining professional

2. **## Commands & Tools** ⚠️ **MUST BE EARLY - SECTION 2**

   Present as a table with executable commands:

   ```markdown
   | Command              | Purpose          | Flags/Options           |
   | -------------------- | ---------------- | ----------------------- |
   | `npm test`           | Run test suite   | `--coverage`, `--watch` |
   | `npm run lint --fix` | Auto-fix linting |                         |
   ```

   - Include actual flags and options, not just tool names
   - Be specific: `pytest -v --cov=src` not just "run tests"

3. **## Core Responsibilities**

   - 3-5 bulleted primary duties
   - Specific, actionable items
   - Based on clarification answers

4. **## Project Knowledge** (if applicable)

   - Tech stack with versions (e.g., "React 18.2, TypeScript 5.4")
   - File structure with purposes
   - Key directories and their roles

5. **## Operational Protocol**

   - Step-by-step execution sequence
   - Numbered steps with clear actions
   - Include tool usage points
   - Can encode CoT or ReAct patterns

6. **## Code Style Examples** (if agent writes/reviews code)

   Show, don't tell. Include before/after examples:

   ```typescript
   // ✅ Good - descriptive, proper error handling
   async function fetchUserById(id: string): Promise<User> {
     if (!id) throw new Error("User ID required");
     return api.get(`/users/${id}`);
   }

   // ❌ Bad - vague, no validation
   async function get(x) {
     return api.get("/users/" + x);
   }
   ```

7. **## Constraints and Boundaries**

   **Use three-tier format with emojis:**

   ```markdown
   - ✅ **Always**: [required behaviors, safe actions]
   - ⚠️ **Ask first**: [potentially risky, needs confirmation]
   - 🚫 **Never**: [prohibited actions, critical restrictions]
   ```

   Based on scope boundaries from clarification.

8. **## Agent Collaboration** (if applicable)

   - List specialist agents to delegate to
   - When to use each collaborator
   - Clear delegation criteria

   **Consider including these agents for collaboration:**

   - **@scout**: For codebase exploration and discovery
   - **@sage**: For research and best practices
   - **@sentinel**: For code review and security analysis
   - **@tracker**: For debugging and investigation
   - **@alchemist**: For performance optimization
   - **@bard**: For documentation needs

   **Wizards for specialized consultation:**

   - **@lit-wizard**: For Lit web component guidance
   - **@ts-wizard**: For TypeScript type system expertise
   - **@tw-wizard**: For Tailwind CSS recommendations
   - **@effect-wizard**: For Effect-TS patterns

9. **## Few-Shot Examples** (if beneficial)
   - 1-2 concrete input/output examples
   - Demonstrate expected behavior
   - Show output format

### 6. Six Core Areas Validation

Before finalizing, verify coverage of all six core areas:

| Area                  | Status | Notes                                     |
| --------------------- | ------ | ----------------------------------------- |
| **Commands**          | ☐      | Executable commands with flags early?     |
| **Testing**           | ☐      | Test/validation commands included?        |
| **Project Structure** | ☐      | File locations and purposes defined?      |
| **Code Style**        | ☐      | Examples showing good/bad patterns?       |
| **Git Workflow**      | ☐      | Commit/branch conventions if applicable?  |
| **Boundaries**        | ☐      | Three-tier (✅/⚠️/🚫) boundaries defined? |

Not all areas apply to every agent - mark N/A where appropriate.

### 7. Complexity-Based Validation

Validate generated agent against line count guidelines:

**Line Count Thresholds (from 2,500+ repo analysis):**

| Complexity | Description                | Target Lines | Max Lines |
| ---------- | -------------------------- | ------------ | --------- |
| Simple     | Single, focused task       | ≤ 100        | 120       |
| Standard   | Multiple related tasks     | ≤ 150        | 180       |
| Complex    | Orchestration, multi-phase | ≤ 200        | 230       |
| Supervisor | Multi-agent collaboration  | ≤ 250        | 280       |

**Quality Checks:**

- [ ] Clear specialization (designed incompetence)
- [ ] Commands appear in section 2 (early placement)
- [ ] Concrete code examples (not just descriptions)
- [ ] Three-tier boundaries with emojis
- [ ] Tech stack includes versions
- [ ] Proper tool/permission alignment
- [ ] Mode matches use case

**If exceeds threshold:**

- Calculate complexity score based on responsibilities
- Warn if over recommended limit
- Suggest splitting into multiple agents if too complex
- Consider extracting reference material to linked files

### 8. Determine Save Location

Check current directory:

```bash
if [ -d ".opencode/agent" ]; then
  LOCATION=".opencode/agent"  # Project-specific
  SCOPE="project"
elif [ -d "$HOME/.config/opencode/agent" ]; then
  LOCATION="$HOME/.config/opencode/agent"  # Global
  SCOPE="global"
fi
```

Ask user:

```
Agent can be saved as:
A) Project-specific (.opencode/agent/) - only available in this project
B) Global (~/.config/opencode/agent/) - available everywhere

Where should this agent be saved? [A/B]:
```

### 9. Save and Confirm

- Write agent file to chosen location: `{location}/{agent-name}.md`
- Display summary:

  ```
  ✓ Created {agent-name} agent ({scope})

  Location: {full-path}
  Mode: {mode}
  Tools: {tool-list}
  Lines: {count} ({threshold} target for {complexity} agents)

  Six Core Areas Coverage:
  ✓ Commands (early placement)
  ✓ Boundaries (three-tier)
  ✓ Code examples
  ○ Git workflow (N/A)
  ...

  Usage:
  - Primary: /{agent-name} <prompt>
  - Subagent: @{agent-name}
  - Task delegation: Use in other agents' collaboration section
  ```

### 10. Offer Next Steps

Suggest:

- Test the agent with example prompt
- Review and refine the agent definition
- Add to other agents' collaboration sections
- Create command wrapper if primary agent

## Guidelines

- **Commands early**: Always place Commands & Tools as section 2
- **Show don't tell**: Use code examples, not prose descriptions
- **Stay focused**: Each agent should have ONE clear primary purpose
- **Be specific**: Use concrete instructions with versions and flags
- **Three-tier boundaries**: Use ✅/⚠️/🚫 pattern consistently
- **Leverage patterns**: Reference existing agents for proven approaches
- **Think modular**: Design for composition with other agents
- **Validate early**: Stop if description is too broad (suggest multiple agents)

## Fantasy Persona Guidelines

When the agent name follows a fantasy theme, craft a persona that:

1. **Matches the role metaphorically:**

   - Debugging → Tracker, Hunter, Ranger (follows trails)
   - Security → Sentinel, Warden, Guardian (protects)
   - Testing → Judge, Adjudicator, Provost (renders verdicts)
   - Documentation → Bard, Chronicler, Scrivener (tells stories)
   - Performance → Alchemist (transmutes slow to fast)
   - Research → Sage, Oracle (possesses wisdom)
   - Exploration → Scout, Pathfinder (maps territory)

2. **Uses professional-fantasy tone:**

   - Serious expertise expressed through fantasy metaphor
   - Avoid whimsy; maintain technical credibility
   - Example: "You are the Shadow Tracker, a ranger who stalks bugs through the
     darkest code paths..."

3. **Weaves the metaphor through the document:**
   - Commands become "tools of the trade"
   - Errors become "barriers" or "threats"
   - Success becomes "completing the quest" or "rendering judgment"

## Example Usage

```bash
# Create a specialized validator agent (fantasy: "arbiter" or "auditor")
/hire-agent arbiter "Validates API responses against OpenAPI specifications"

# Create an orchestrator agent (fantasy: "marshal" or "commander")
/hire-agent marshal "Manages multi-step development workflows with task delegation"

# Create a security-focused agent (fantasy: "guardian" or "watcher")
/hire-agent guardian "Scans code for security vulnerabilities and compliance issues"
```

Transform agent ideas into production-ready, specialized OpenCode agents
following established patterns and best practices derived from analysis of
2,500+ successful agent definitions.
