# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This repository contains a multi-agent deep research system for Claude Code. The system uses skills to orchestrate parallel research tasks through specialized agent types that work collaboratively to answer complex user queries.

## Quick Start

### Using the Deep Research Skill

```
deep-research What are the most effective treatments for depression?
```

or with @ syntax:

```
@deep-research Compare AWS, Azure, and Google Cloud by market share
```

**Important**: Do NOT use `/deep-research` - slash commands are built-in to Claude Code and cannot be customized.

## Implementation Architecture

### Claude Code Skills (`.claude/skills/`)

The system uses Claude Code's native skill framework:

- **`deep-research.md`**: Main research coordination skill
  - Analyzes user queries
  - Determines query type (depth-first, breadth-first, straightforward)
  - Uses the `Task` tool to launch parallel subagents
  - Synthesizes findings into comprehensive reports

- **`research-subagent.md`**: Internal subagent skill
  - Automatically invoked by the lead agent
  - Executes focused research tasks
  - Uses `web_search` and `web_fetch` tools
  - Reports condensed findings back

### How It Works

```
User invokes: deep-research <query>
         ↓
Research Lead Agent (deep-research skill)
  ├─ Analyzes query
  ├─ Classifies query type
  ├─ Creates research plan
  ├─ Uses Task tool to launch subagents (parallel)
  │   ├─ Subagent 1: research-subagent skill
  │   ├─ Subagent 2: research-subagent skill
  │   └─ Subagent 3: research-subagent skill
  └─ Synthesizes all findings → Final report
```

## Agent Architecture

The research system follows a hierarchical agent pattern:

### Lead Agent (deep-research skill)

The orchestrator that manages the entire research process:
- **Query analysis**: Breaks down user questions and determines query type
- **Research planning**: Creates detailed research plans with clear task allocation
- **Delegation**: Uses the `Task` tool to launch subagents in parallel
- **Synthesis**: Integrates findings and writes the final report (never delegates this)
- **Subagent count guidelines**:
  - Simple queries: 1 subagent
  - Standard complexity: 2-3 subagents
  - Medium complexity: 3-5 subagents
  - High complexity: 5-10 subagents (maximum 20)

### Subagent (research-subagent skill)

The worker that executes specific research tasks:
- Receives clear task descriptions from the lead agent via `Task` tool
- Uses `web_search` and `web_fetch` in a core loop
- **Tool call budget**: 5-15 calls, maximum 20 hard limit
- **Minimum**: At least 5 distinct tool calls for comprehensive research
- Reports condensed results back via `complete_task` (or returns findings)
- Never generates final reports—only researches and reports findings

### Citations Agent (citations_agent.md)

Original prompt for post-processing agent that adds citations (not currently implemented as skill).

## Research Query Types

The system classifies queries into three categories:

1. **Depth-first**: Single topic requiring multiple perspectives
   - Examples: "What caused the 2008 financial crisis?", "What are the most effective treatments for depression?"
   - Approach: Parallel agents exploring different viewpoints
   - Synthesis: Multi-perspective analysis

2. **Breadth-first**: Distinct, independent sub-questions
   - Examples: "Compare Nordic economic systems", "Compare AWS, Azure, GCP"
   - Approach: Parallel agents each handling separate sub-topics
   - Synthesis: Comparative aggregation

3. **Straightforward**: Focused, well-defined questions
   - Examples: "What is Tokyo's population?", "List Fortune 500 companies"
   - Approach: Single agent with clear fact-finding
   - Synthesis: Direct report

## Key Patterns

### Task Tool Usage

The `Task` tool is used to launch subagents:

```
Task(
  subagent_type="general-purpose",
  prompt="Clear task description with objectives, context, and expected output"
)
```

Launch multiple subagents in parallel for efficiency:
```
Task(subagent_type="general-purpose", prompt="Task 1...")
Task(subagent_type="general-purpose", prompt="Task 2...")
Task(subagent_type="general-purpose", prompt="Task 3...")
```

### Tool Usage Pattern

- Core research loop: `web_search` → `web_fetch` URLs → analyze → repeat
- Subagents must use `web_fetch` to retrieve full page content
- Maximum 20 tool calls per subagent to prevent runaway execution

### Reporting Pattern

- Subagents return condensed, fact-based findings
- Lead agent synthesizes all findings into final Markdown report
- Lead agent NEVER delegates report writing

## File Structure

```
simple_deep_research/
├── .claude/
│   ├── skills/
│   │   ├── deep-research.md          # Main research coordination skill
│   │   └── research-subagent.md      # Subagent research skill
│   └── settings.local.json           # Claude Code settings
├── research_lead_agent.md            # Original lead agent prompt
├── research_subagent.md              # Original subagent prompt
├── citations_agent.md                # Original citations agent prompt
├── README.md                         # User documentation
└── CLAUDE.md                         # This file
```

## Development Workflow

### Modifying Agent Behavior

1. **To change research strategy**: Edit `.claude/skills/deep-research.md`
2. **To modify subagent behavior**: Edit `.claude/skills/research-subagent.md`

### Testing Changes

1. Edit the skill file
2. Restart Claude Code or reload skills
3. Test with: `deep-research <test query>`

### Skill File Format

Skills require YAML frontmatter:

```yaml
---
description: "Brief description of when to use this skill"
---

# Skill Name

Detailed instructions for the agent...
```

## Common Tasks

### Running a Research Query

```
deep-research <your question>
```

or

```
@deep-research <your question>
```

### Debugging Research Issues

1. Check query type classification in the skill logic
2. Verify subagent tasks are clear and non-overlapping
3. Ensure subagents stay under tool call limits (max 20)
4. Confirm parallel execution is happening via Task tool

### Customizing Research Depth

Adjust subagent counts in the research planning section of `deep-research.md`:
- Straightforward: 1 subagent
- Standard: 2-3 subagents
- Medium: 3-5 subagents
- High: 5-10 subagents (max 20)

## Important Constraints

- Lead agents **NEVER** delegate final report writing—that's their core responsibility
- Subagents have hard tool call limits (20 max) and must stop when reaching diminishing returns
- Maximum 20 subagents for high-complexity queries (avoid over-delegation overhead)
- Parallel execution via Task tool is required for efficiency
- Use `deep-research <query>` NOT `/deep-research` (slash commands cannot be customized)
