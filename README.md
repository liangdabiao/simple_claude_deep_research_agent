# Deep Research Agent System

A multi-agent research system for conducting comprehensive web-based research using parallel subagents.

## Overview

This system implements a hierarchical agent architecture for deep research:

- **Research Lead Agent**: Orchestrates research, plans tasks, delegates to subagents, and synthesizes findings
- **Research Subagents**: Execute focused research tasks in parallel using web search and fetch
- **Citations Agent**: Adds accurate citations to research reports (optional)

## Quick Start

### Using the `/deep-research` Command

The simplest way to use this system is through the Claude Code command:

```
/deep-research What are the most effective treatments for depression?
```

The system will automatically:
1. Analyze your query
2. Determine the optimal research strategy
3. Deploy parallel subagents to gather information
4. Synthesize a comprehensive report

### Using the Python API

```python
import asyncio
from research_coordinator import ResearchLeadAgent

async def main():
    agent = ResearchLeadAgent()
    report = await agent.conduct_research(
        "What caused the 2008 financial crisis?"
    )
    print(report)

asyncio.run(main())
```

## Query Types

The system classifies queries into three types:

### 1. Straightforward Queries
Simple fact-finding with clear answers.
- **Example**: "What is the population of Tokyo?"
- **Approach**: Single subagent with focused instructions
- **Duration**: ~1-2 minutes

### 2. Breadth-First Queries
Multi-part questions with distinct sub-topics.
- **Example**: "Compare AWS, Azure, and Google Cloud"
- **Approach**: Multiple subagents researching independent topics
- **Duration**: ~3-5 minutes

### 3. Depth-First Queries
Complex questions requiring multiple perspectives.
- **Example**: "What are the most effective treatments for depression?"
- **Approach**: Multiple subagents exploring different viewpoints
- **Duration**: ~5-10 minutes

## Architecture

```
User Query
    ↓
Research Lead Agent
    ├─ Analyze Query
    ├─ Determine Query Type
    ├─ Create Research Plan
    ├─ Deploy Subagents (Parallel)
    │   ├─ Subagent 1 → Web Search → Fetch → Report
    │   ├─ Subagent 2 → Web Search → Fetch → Report
    │   └─ Subagent 3 → Web Search → Fetch → Report
    └─ Synthesize Findings → Final Report
```

## File Structure

```
simple_deep_research/
├── .claude/
│   ├── skills/
│   │   ├── deep-research.md          # Main research skill
│   │   └── research-subagent.md      # Subagent skill
│   └── commands/
│       └── deep-research.md          # Command definition
├── research_coordinator.py           # Main coordinator implementation
├── example_usage.py                  # Usage examples
├── research_lead_agent.md            # Original lead agent prompt
├── research_subagent.md              # Original subagent prompt
├── citations_agent.md                # Original citations agent prompt
└── README.md                         # This file
```

## Configuration

### Skill Files

Skills are defined in `.claude/skills/` and contain:

- `deep-research.md`: Main research coordination skill
- `research-subagent.md`: Internal subagent for parallel research

### Command Files

Commands are defined in `.claude/commands/` and provide:

- `deep-research.md`: The `/deep-research` slash command

### Settings

Claude Code settings are in `.claude/settings.local.json`.

## Examples

Run the example file to see the system in action:

```bash
python example_usage.py
```

## Development

### Extending the System

To add new research capabilities:

1. **Add new query types**: Extend the `QueryType` enum in `research_coordinator.py`
2. **Customize planning logic**: Modify the `_plan_*` methods in `ResearchLeadAgent`
3. **Add new tools**: Extend the `ResearchSubagent` class with new tool capabilities

### Integration with Claude Code

The skills and commands in `.claude/` are automatically available in Claude Code:

- Use the `deep-research` skill when you need comprehensive research
- Invoke `/deep-research <query>` as a command

## Research Process

### For Users

1. **Provide a clear question**: The more specific your query, the better the results
2. **Wait for completion**: Research time varies by complexity
3. **Review the report**: The system provides comprehensive, sourced answers

### For Developers

To understand the internal flow:

1. `ResearchLeadAgent.analyze_query()` - Understands the question
2. `ResearchLeadAgent.determine_query_type()` - Classifies the approach
3. `ResearchLeadAgent.create_research_plan()` - Creates the task list
4. `ResearchLeadAgent.execute_research()` - Launches subagents in parallel
5. `ResearchLeadAgent.synthesize_report()` - Creates the final report

## Limitations

- **Tool call limits**: Subagents are limited to 20 tool calls each
- **Web-only**: Current implementation only uses web search/fetch
- **No persistent storage**: Research results are not cached between sessions

## Future Enhancements

Possible improvements to the system:

- [ ] Add support for internal tools (GDrive, Gmail, Slack)
- [ ] Implement caching for repeated queries
- [ ] Add automatic citation generation
- [ ] Support for multi-language queries
- [ ] Export reports to different formats (PDF, DOCX)
- [ ] Web UI for research queries

## License

This is a demonstration project for educational purposes.
