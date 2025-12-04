---
description: "Assemble an adventuring party of agents for a quest"
---

# Quest Briefing

> $ARGUMENTS

Analyze this quest and recommend an optimal party of agents to tackle it, using
interactive clarification before embarking.

## Realm Context

- **Guild Hall**:
  !`ls ~/.config/opencode/agent/ | sed 's/\.md$//' | tr '\n' ', ' | sed 's/, $//'`
- **Current Location**: !`pwd | xargs basename`
- **Project Root**: !`pwd`
- **Active Branch**:
  !`git branch --show-current 2>/dev/null || echo "Not a git repo"`

### Detected Technologies

!`cat package.json 2>/dev/null | grep -E '"(name|dependencies|devDependencies)"' -A 20 | head -25 || cat requirements.txt 2>/dev/null | head -15 || cat Cargo.toml 2>/dev/null | head -15 || echo "No package manifest found"`

### Recent Activity

!`git log --oneline -5 2>/dev/null || echo "No git history"`

## Agent Guild Roster

The guild's agents follow a fantasy persona theme. Match the quest requirements
to agent expertise. Discover all the agents and their personas by listing the
files in `~/.config/opencode/agent/` and reading their descriptions:

`ls ~/.config/opencode/agent/ | sed 's/\.md$//' | xargs -I {} sh -c 'echo -n "{}: "; head -3 ~/.config/opencode/agent/{}.md | grep description: | sed "s/description: //"'`

## Process

### 1. Quest Analysis

Analyze the quest briefing above (in the blockquote) to identify:

- **Quest Type**: Feature, Bug Fix, Refactor, Research, Documentation, Testing,
  Performance, Security, Accessibility
- **Technologies Involved**: Languages, frameworks, libraries mentioned or
  detected
- **Complexity Level**: Simple (1-2 agents), Standard (2-4 agents), Epic (4+
  agents)
- **Risk Factors**: Security-sensitive, performance-critical, user-facing, data
  integrity

### 2. Interactive Clarification (Max 3 Questions)

Ask targeted questions to better understand the quest. Present ONE question at a
time:

**Example Questions:**

| #   | Question                                                | Format                                                                             |
| --- | ------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| 1   | What is the primary goal of this quest?                 | Short answer                                                                       |
| 2   | Are there specific technologies or frameworks involved? | Short answer or "Use detected"                                                     |
| 3   | What is the risk level of this quest?                   | A) Low - internal tooling B) Medium - user-facing C) High - security/data critical |

Stop early if the quest is clear (minimum 1 question, maximum 3).

### 3. Party Formation

Based on the quest analysis, recommend a party with these roles:

**Party Roles:**

| Role           | Description                                         | Count |
| -------------- | --------------------------------------------------- | ----- |
| **Champion**   | Leads the quest, primary implementer                | 0-1   |
| **Vanguard**   | Core party members, essential for quest completion  | 1-3   |
| **Support**    | Provides assistance, handles secondary concerns     | 0-2   |
| **Consultant** | Wizards called for specialized guidance (read-only) | 0-2   |

**Selection Criteria:**

- Match agent expertise to quest requirements
- Consider detected project technologies
- Include reviewers for high-risk quests
- Add wizards for specialized technology guidance
- Prefer smaller parties for simpler quests

### 4. Present Party Recommendation

Output the recommended party in this format:

```markdown
# Quest: [Brief Quest Title]

## Quest Analysis

- **Type:** [Quest Type]
- **Complexity:** [Simple/Standard/Epic]
- **Technologies:** [Detected/mentioned technologies]
- **Risk Level:** [Low/Medium/High]

## Recommended Party

### Champion

> [If applicable - agent who should lead]

**@agent-name** - The [Persona Title]

- **Role:** [Why they're leading]
- **Primary Tasks:** [What they'll focus on]

### Vanguard

> Core party members essential for this quest

**@agent-name** - The [Persona Title]

- **Role:** [Their contribution]
- **Primary Tasks:** [Specific responsibilities]

[Repeat for each vanguard member]

### Support

> [If applicable - secondary assistance]

**@agent-name** - The [Persona Title]

- **Role:** [Support function]

### Consultants

> [If applicable - wizards for specialized guidance]

**@wizard-name** - The [Persona Title]

- **Consult for:** [Specific expertise needed]

## Collaboration Strategy

1. [First step - which agent, what action]
2. [Second step]
3. [Continue as needed]

## Quick Reference

Invoke this party with:

@agent1 @agent2 @agent3 [your instructions]

Or individually:

- @agent1 for [task type]
- @agent2 for [task type]
```

### 5. Party Confirmation

After presenting the recommendation, ask:

```txt
## Party Confirmation

Does this party composition suit your quest?

A) **Embark** - Begin the quest with this party
B) **Adjust** - Modify the party (add/remove members)
C) **Explain** - Learn more about a specific agent's role
D) **Abandon** - Cancel and reformulate the quest

Your choice:
```

**If Adjust:** Ask which agents to add or remove, then re-present the updated
party.

**If Explain:** Provide more detail about the selected agent's capabilities and
why they were chosen.

**If Embark:** Provide the final party summary with ready-to-use invocation
syntax:

```markdown
## Quest Commenced!

Your party is assembled and ready. Begin your quest with:

**Full Party Invocation:**

@sentinel @judge @ts-wizard Review and test the authentication module, ensuring
type safety and security best practices.

**Or invoke individually as needed:**

- `@sentinel` - For security review
- `@judge` - For test coverage
- `@ts-wizard` - For type system guidance

May your quest be successful, adventurer!
```

## Guidelines

- **Smaller is better**: Recommend the minimum viable party for the quest
- **Match expertise**: Only include agents whose skills directly apply
- **Consider workflow**: Order collaboration strategy logically (research →
  implement → test → review)
- **Wizard sparingly**: Only include wizard consultants when specialized
  expertise is truly needed
- **Context matters**: Use detected project technologies to inform
  recommendations
- **Risk-appropriate**: High-risk quests need reviewers (@sentinel) and testers
  (@judge)

## Example Invocations

```bash
# Feature quest
/form-party Implement user authentication with JWT tokens

# Bug hunt
/form-party Debug why payment API returns 500 errors

# Performance quest
/form-party Optimize dashboard load time (currently 8s)

# Documentation quest
/form-party Document the plugin API for developers
```

---

_Assemble the right heroes for every quest!_
