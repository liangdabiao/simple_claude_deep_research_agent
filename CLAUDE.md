# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This repository contains a **simplified multi-agent deep research system** for Claude Code. The system uses skills to orchestrate parallel research tasks through specialized agents.

**Design Philosophy**: Medium complexity, web-focused, practical - not too simple, not overly complex.

## Quick Start

### Using the Deep Research Skill

```
deep-research What are the most effective treatments for depression?
```

or:

```
@deep-research Compare AWS, Azure, and Google Cloud
```

**Important**: Do NOT use `/deep-research` - slash commands are built-in and cannot be customized.

### Adding Citations

```
citations [paste your research report]
```

## Architecture

### Agent Types

1. **Research Lead Agent** (`.claude/skills/deep-research.md`)
   - Analyzes queries and determines query type
   - Plans research approach
   - Uses `Task` tool to launch parallel subagents
   - Synthesizes findings into final report

2. **Research Subagent** (`.claude/skills/research-subagent.md`)
   - Receives tasks from lead agent
   - Uses web tools (web_search, web_fetch, Playwright MCP)
   - Reports findings in dense format
   - Limited to 20 tool calls

3. **Citations Agent** (`.claude/skills/citations.md`)
   - Adds citations to completed reports
   - Uses footnote or inline citation style
   - Preserves original content

### Query Types

| Type | Description | Subagents | Example |
|------|-------------|-----------|---------|
| Straightforward | Simple fact-finding | 1 | "What is Tokyo's population?" |
| Breadth-first | Independent sub-topics | 3 | "Compare AWS, Azure, GCP" |
| Depth-first | Multiple perspectives | 4 | "What caused the 2008 crisis?" |

## Tools Used

| Tool | Purpose | Used By |
|------|---------|---------|
| `web_search` | Search web | Lead, Subagent |
| `web_fetch` | Get full page content | Subagent |
| `mcp__playwright__navigate` | Load JS pages | Subagent |
| `mcp__playwright__snapshot` | Get rendered content | Subagent |
| `Task` | Launch subagents | Lead |

**Note**: Only web tools are used - no internal tools (GDrive, Gmail, Slack).

## File Structure

```
simple_deep_research/
├── .claude/
│   └── skills/
│       ├── deep-research.md       # Lead agent (research coordination)
│       ├── research-subagent.md   # Subagent (research execution)
│       └── citations.md           # Citations agent (citation addition)
├── research_lead_agent.md         # Original prompt (reference only)
├── research_subagent.md           # Original prompt (reference only)
├── citations_agent.md             # Original prompt (reference only)
├── README.md                      # User documentation
└── CLAUDE.md                      # This file
```

## Development Workflow

### Modifying Agent Behavior

**To change research strategy**: Edit `.claude/skills/deep-research.md`
- Query classification logic (lines 19-33)
- Subagent deployment rules (lines 52-96)
- Synthesis approach (lines 98-111)

**To modify subagent behavior**: Edit `.claude/skills/research-subagent.md`
- Tool usage patterns (lines 40-56)
- Source quality evaluation (lines 58-67)
- Report format (lines 69-94)

**To modify citations**: Edit `.claude/skills/citations.md`
- Citation style (lines 30-44)
- When to cite rules (lines 19-28)

### Skill File Format

All skills require YAML frontmatter:

```yaml
---
description: "Brief description of when to use this skill"
---
```

Followed by detailed instructions.

### Testing Changes

1. Edit skill files
2. Test with: `deep-research <test query>`
3. Iterate based on results

## Key Implementation Details

### Task Tool Usage

Lead agent launches subagents using:

```
Task(
  subagent_type="general-purpose",
  prompt="<clear task description>",
  model="sonnet"  # optional
)
```

**Parallel execution**: Launch 2-6 Task calls in a single message.

### Task Description Format

Each task must include:
1. **Specific objective** (1 core objective per subagent)
2. **Output format** (e.g., "list of facts", "detailed report")
3. **Background context** (why this task matters)
4. **Key questions** (what to answer)
5. **Suggested sources** (where to look)
6. **Scope boundaries** (what NOT to research)

### Subagent Constraints

- **Tool call budget**: 3-15 calls depending on complexity
- **Hard limit**: 20 tool calls maximum (terminated if exceeded)
- **Minimum**: 3 tool calls for meaningful research
- **Parallel**: Use 2+ web_search calls simultaneously

### Source Quality Evaluation

Subagents evaluate sources for:
- **Speculation**: "could", "may", "might" indicate predictions
- **Source type**: Original sources > aggregators
- **Bias**: Marketing language, cherry-picked data
- **Recency**: Current info preferred for time-sensitive topics
- **Consistency**: Cross-reference conflicting facts

### Report Format

**Subagent Report**:
```markdown
## Key Findings

- Fact 1 with source
- Fact 2 with source

## Summary

[Brief summary]

## Sources

[URLs]
```

**Final Report** (Lead agent):
```markdown
# [Title]

## Section 1

[Content with data points]

## Section 2

[Content with comparisons]

...
```

## Common Tasks

### Running a Research Query

```
deep-research <your question>
```

or:

```
@deep-research <your question>
```

### Adding Citations

After research completes:

```
citations <paste report here>
```

### Debugging Issues

**Problem**: "Unknown skill"
- **Solution**: Check `.claude/skills/` directory, verify YAML frontmatter, restart Claude Code

**Problem**: Subagents not launching
- **Solution**: Verify Task tool syntax, check `general-purpose` subagent_type, check permissions

**Problem**: Poor research quality
- **Solution**: Try different query wording, verify query classification, check web_search

**Problem**: Tool call limit exceeded
- **Solution**: Subagent hit 20-call limit - this is intentional to prevent runaway execution

### Customizing Subagent Count

Edit `deep-research.md` lines 54-58:

```
- Straightforward: 1 subagent
- Standard complexity: 2-3 subagents
- Medium complexity: 3-4 subagents
- High complexity: 4-6 subagents (maximum 10)
```

## Important Constraints

1. **Web-only**: No internal tools (GDrive, Gmail, Slack)
2. **Tool limits**: Subagents max 20 calls each
3. **Lead writes report**: Never delegate report writing
4. **Parallel required**: Always launch subagents simultaneously
5. **No citations in report**: Separate agent handles citations
6. **Use correct syntax**: `deep-research <query>` NOT `/deep-research`

## Differences from Original Prompts

This is a **simplified implementation** of the original prompts:

| Original | Simplified | Reason |
|----------|------------|--------|
| 5 main sections | 3 sections (core only) | Focus on essentials |
| Internal tools (GDrive, Gmail) | Web tools only | Simpler, more reliable |
| 20 max subagents | 10 max subagents | Medium complexity target |
| `run_blocking_subagent` | `Task` tool | Claude Code standard |
| Detailed Bayesian reasoning | Simplified synthesis | Practical approach |
| `complete_task` tool | Direct return | Simpler workflow |

## Design Rationale

### Why Medium Complexity?

- **Not too simple**: Retains core value (parallel research, synthesis)
- **Not too complex**: Avoids edge cases, easier to maintain
- **Practical**: Focuses on what works reliably

### Why Web Tools Only?

- **Universal**: Works for any user without special setup
- **Reliable**: web_search and web_fetch are stable
- **Sufficient**: Covers most research needs

### Why Three Query Types?

- **Straightforward**: Simple questions don't need parallelization
- **Breadth-first**: Independent topics benefit from parallel agents
- **Depth-first**: Complex questions need multiple perspectives

## Testing Examples

### Simple Query (1 subagent)
```
deep-research What is the current population of Tokyo?
```

### Comparison Query (3 subagents)
```
deep-research Compare AWS, Azure, and Google Cloud pricing
```

### Analysis Query (4 subagents)
```
deep-research What are the most effective treatments for depression?
```

## MCP Configuration (Optional)

To enable Playwright MCP for JavaScript-heavy pages:

1. Install Playwright MCP server
2. Add to Claude Code MCP configuration
3. Restart Claude Code

The system will automatically use Playwright when `web_fetch` doesn't get complete content.

## Summary

This is a **practical, medium-complexity deep research system** that:
- Uses 3 specialized agents (lead, subagent, citations)
- Leverages web tools (web_search, web_fetch, Playwright)
- Executes parallel research via Task tool
- Targets 2-6 subagents per query
- Synthesizes findings into comprehensive reports

For detailed usage, see README.md.
For original requirements, see research_*.md files (reference only).
