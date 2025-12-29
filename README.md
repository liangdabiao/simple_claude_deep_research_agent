# Deep Research Agent System

A multi-agent research system for Claude Code that conducts comprehensive web-based research using parallel subagents.

## Overview

This system implements a hierarchical agent architecture for deep research:

- **Research Lead Agent**: Orchestrates research, plans tasks, delegates to subagents, and synthesizes findings
- **Research Subagents**: Execute focused research tasks in parallel using web search and fetch

## Quick Start

### Using the Deep Research Skill

Invoke the deep research skill directly in Claude Code:

```
deep-research What are the most effective treatments for depression?
```

or with @ syntax:

```
@deep-research Compare AWS, Azure, and Google Cloud by market share
```

The system will automatically:
1. Analyze your query
2. Determine the optimal research strategy
3. Deploy parallel subagents to gather information from the web
4. Synthesize a comprehensive report

## Query Types

The system classifies queries into three types:

### 1. Straightforward Queries
Simple fact-finding with clear answers.
- **Example**: `deep-research What is the population of Tokyo?`
- **Approach**: Single subagent with focused instructions
- **Duration**: ~1-2 minutes

### 2. Breadth-First Queries
Multi-part questions with distinct sub-topics.
- **Example**: `deep-research Compare AWS, Azure, and Google Cloud`
- **Approach**: Multiple subagents researching independent topics
- **Duration**: ~3-5 minutes

### 3. Depth-First Queries
Complex questions requiring multiple perspectives.
- **Example**: `deep-research What are the most effective treatments for depression?`
- **Approach**: Multiple subagents exploring different viewpoints
- **Duration**: ~5-10 minutes

## Architecture

```
User Query
    ↓
Research Lead Agent (deep-research skill)
    ├─ Analyze Query
    ├─ Determine Query Type
    ├─ Create Research Plan
    ├─ Deploy Subagents (Parallel via Task tool)
    │   ├─ Subagent 1 → Web Search → Web Fetch → Report
    │   ├─ Subagent 2 → Web Search → Web Fetch → Report
    │   └─ Subagent 3 → Web Search → Web Fetch → Report
    └─ Synthesize Findings → Final Report
```

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
├── README.md                         # This file
└── CLAUDE.md                         # Development guide
```

## Configuration

### Skill Files

Skills are defined in `.claude/skills/`:

- **`deep-research.md`**: Main research coordination skill
  - Analyzes queries and determines research strategy
  - Uses the `Task` tool to launch parallel subagents
  - Synthesizes findings into final reports

- **`research-subagent.md`**: Internal subagent skill
  - Executes focused research tasks
  - Uses `web_search` and `web_fetch` tools
  - Reports condensed findings back to lead agent

### Settings

Claude Code settings in `.claude/settings.local.json` configure permissions.

## Examples

### Example 1: Straightforward Query

```
deep-research What is the current population of Tokyo?
```

### Example 2: Breadth-First Query

```
deep-research Compare the top 3 cloud providers (AWS, Azure, GCP) by market share, pricing, and key features
```

### Example 3: Depth-First Query

```
deep-research What caused the 2008 financial crisis?
```

### Example 4: Complex Research

```
deep-research What are the most effective treatments for depression, and what does the latest research say about their efficacy?
```

## Research Process

### For Users

1. **Invoke the skill**: Use `deep-research <your question>` or `@deep-research <your question>`
2. **Wait for completion**: Research time varies by complexity (1-10 minutes)
3. **Review the report**: The system provides comprehensive, sourced answers in Markdown

### How It Works Internally

1. **Query Analysis**: The Research Lead Agent breaks down your question
2. **Type Classification**: Determines if it's depth-first, breadth-first, or straightforward
3. **Research Planning**: Creates detailed task allocation for subagents
4. **Parallel Execution**: Launches multiple subagents simultaneously using the `Task` tool
5. **Synthesis**: Integrates all findings into a comprehensive report

## Development

### Modifying Research Behavior

**To change research strategy**: Edit `.claude/skills/deep-research.md`
- Adjust query classification logic
- Modify subagent deployment rules
- Change synthesis approach

**To modify subagent behavior**: Edit `.claude/skills/research-subagent.md`
- Adjust tool usage patterns
- Modify source quality evaluation
- Change reporting format

### Skill File Format

Skills use YAML frontmatter for metadata:

```yaml
---
description: "Brief description of when to use this skill"
---

# Skill Name

Detailed instructions for the agent...
```

### Testing Changes

1. Edit the skill file
2. Restart Claude Code or reload skills
3. Test with: `deep-research <test query>`

## Important Notes

- **No Slash Commands**: Use `deep-research <query>` not `/deep-research`
- **Parallel Execution**: The system automatically deploys multiple subagents for efficiency
- **Tool Limits**: Subagents are limited to ~20 tool calls each to prevent runaway execution
- **Web-Only**: Current implementation uses web_search and web_fetch tools

## Future Enhancements

Possible improvements to the system:

- [ ] Add support for internal tools (GDrive, Gmail, Slack)
- [ ] Implement caching for repeated queries
- [ ] Add automatic citation generation
- [ ] Support for multi-language queries
- [ ] Export reports to different formats (PDF, DOCX)

## License

This is a demonstration project for educational purposes.
