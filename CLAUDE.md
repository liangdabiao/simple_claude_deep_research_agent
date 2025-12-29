# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This repository contains a multi-agent deep research system with both prompt definitions and a Python implementation. The system orchestrates research tasks through specialized agent types that work collaboratively to answer complex user queries using Claude Code's agent framework.

## Quick Start

### Using the `/deep-research` Command

```bash
/deep-research What are the most effective treatments for depression?
```

The command automatically invokes the `deep-research` skill, which:
1. Analyzes your query
2. Determines the optimal research strategy
3. Deploys parallel subagents via the `Task` tool
4. Synthesizes a comprehensive report

### Using the Python Implementation

```bash
python research_coordinator.py
python example_usage.py
```

## Implementation Architecture

### Claude Code Integration (`.claude/`)

The Claude Code integration consists of:

**Skills** (`.claude/skills/`):
- `deep-research.md`: Main research coordination skill - Use this for complex research tasks
- `research-subagent.md`: Internal subagent skill - Automatically invoked by lead agent

**Commands** (`.claude/commands/`):
- `deep-research.md`: Slash command definition for `/deep-research`

**Settings** (`.claude/settings.local.json`):
- Permissions configuration for allowed operations

### Python Implementation (`research_coordinator.py`)

The Python code provides a reference implementation with three main classes:

- `ResearchLeadAgent`: Main coordinator (see `research_lead_agent.md` for prompt details)
- `ResearchSubagent`: Worker that executes research tasks (see `research_subagent.md`)
- `CitationsAgent`: Adds citations to reports (see `citations_agent.md`)

Key classes:
- `QueryType`: Enum for query classification (DEPTH_FIRST, BREADTH_FIRST, STRAIGHTFORWARD)
- `ResearchTask`: Dataclass for subagent tasks
- `ResearchPlan`: Dataclass for research plans

## Agent Architecture

The research system follows a hierarchical agent pattern:

### Lead Agent (`research_lead_agent.md`)
The orchestrator that manages the entire research process. Key responsibilities:
- **Query analysis**: Breaks down user questions and determines query type (depth-first, breadth-first, or straightforward)
- **Research planning**: Creates detailed research plans with clear task allocation across subagents
- **Delegation**: Deploys research subagents in parallel using the `run_blocking_subagent` tool
- **Synthesis**: Integrates findings from all subagents and writes the final report (never delegates this)
- **Subagent count guidelines**:
  - Simple queries: 1 subagent
  - Standard complexity: 2-3 subagents
  - Medium complexity: 3-5 subagents
  - High complexity: 5-10 subagents (maximum 20)

The lead agent follows a structured research process: assessment/breakdown → query type determination → plan development → methodical execution.

### Subagent (`research_subagent.md`)
The worker that executes specific research tasks. Key characteristics:
- Receives clear, focused `<task>` descriptions from the lead agent
- Uses web_search and web_fetch tools in a core loop (search → fetch detailed content → analyze)
- **Tool call budget**: Adapted to task complexity (5-15 calls, maximum 20 hard limit)
- **Minimum requirements**: At least 5 distinct tool calls, up to 10 for complex queries
- Reports condensed, information-dense results back to lead agent via `complete_task` tool
- Never generates the final report—only researches and reports findings

Subagents use an OODA loop (observe, orient, decide, act) and must use `web_fetch` to get complete website contents, not just search snippets.

### Citations Agent (`citations_agent.md`)
Post-processing agent that adds citations to completed research reports. Key responsibilities:
- Takes synthesized research report from `<synthesized_text>` tags
- Adds citations using inline citation format
- **Critical constraint**: Must NOT modify the original text whatsoever—only adds citations
- Places citations at sentence ends for complete thoughts/claims
- Avoids over-citing common knowledge or fragmenting sentences with multiple citations

## Research Query Types

The system classifies queries into three categories that determine the research strategy:

1. **Depth-first**: Single topic requiring multiple perspectives (e.g., "What caused the 2008 financial crisis?")
   - Uses parallel agents exploring different viewpoints on the same question
   - Synthesizes diverse methodological approaches

2. **Breadth-first**: Distinct, independent sub-questions (e.g., "Compare Nordic countries' economic systems")
   - Uses parallel agents each handling separate sub-topics
   - Aggregates findings into coherent comparison

3. **Straightforward**: Focused, well-defined questions (e.g., "What is Tokyo's population?")
   - Single subagent with clear fact-finding instructions
   - Lead agent may collaborate directly on simple tasks

## Key Patterns

### Delegation Pattern
- Lead agents deploy subagents via `run_blocking_subagent(prompt="<clear_task_description>")`
- Subagents receive extremely specific instructions including: objectives, output format, context, key questions, suggested sources, scope boundaries
- Parallel execution is mandatory for efficiency—run 3+ subagents simultaneously except for straightforward queries

### Tool Usage Pattern
- Core research loop: `web_search` → `web_fetch` URLs from results → analyze → repeat
- Subagents must use `web_fetch` to retrieve full page content, not rely on search snippets
- Internal tools (GDrive, Gmail, Slack, etc.) take priority over web search when available and relevant
- Maximum 20 tool calls per subagent to prevent system overload

### Reporting Pattern
- Subagents use `complete_task` tool to return condensed, fact-based reports
- Lead agents use `complete_task` tool to submit final research reports
- Citations agent outputs text within `<exact_text_with_citation>` tags

## Template Variables

The agent prompts use Go template syntax:
- `{{.CurrentDate}}`: Injected at runtime to provide agents with current date context

## Important Constraints

- Lead agents **NEVER** delegate final report writing—that's their core responsibility
- Subagents have hard tool call limits (20 max) and must stop when reaching diminishing returns
- Citations agent must preserve original text exactly—no whitespace changes
- Maximum 20 subagents for high-complexity queries (avoid over-delegation overhead)
- Parallel tool calls are required for efficiency

## File Structure

```
simple_deep_research/
├── .claude/
│   ├── skills/
│   │   ├── deep-research.md          # Main research coordination skill
│   │   └── research-subagent.md      # Subagent research skill
│   ├── commands/
│   │   └── deep-research.md          # /deep-research command
│   └── settings.local.json           # Claude Code settings
├── research_coordinator.py           # Python implementation
├── example_usage.py                  # Usage examples
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
3. **To adjust Python logic**: Edit `research_coordinator.py`

### Testing Changes

1. Test skills using: `/deep-research <test query>`
2. Test Python code: `python example_usage.py`
3. Verify both produce similar results

### Adding New Query Types

1. Add to `QueryType` enum in `research_coordinator.py`
2. Create `_plan_<new_type>()` method in `ResearchLeadAgent`
3. Update skill documentation with examples

## Common Tasks

### Running a Research Query

**Via Command**:
```
/deep-research <your question>
```

**Via Python**:
```python
from research_coordinator import ResearchLeadAgent
agent = ResearchLeadAgent()
report = await agent.conduct_research("<your question>")
```

### Debugging Research Issues

1. Check query type classification in `determine_query_type()`
2. Review task allocation in `create_research_plan()`
3. Verify subagent tool calls stay under limits (max 20)
4. Ensure parallel execution is working

### Customizing Research Depth

Adjust subagent counts in `create_research_plan()`:
- Straightforward: 1 subagent
- Standard: 2-3 subagents
- Medium: 3-5 subagents
- High: 5-10 subagents (max 20)
