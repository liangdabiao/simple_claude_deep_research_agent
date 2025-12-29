# Claude Deep Research Agent System

A simplified multi-agent research system for Claude Code that conducts comprehensive web-based research using parallel subagents.

## Overview

This system uses three specialized agent types:

- **Research Lead Agent**: Orchestrates research, plans tasks, delegates to subagents, and synthesizes findings
- **Research Subagents**: Execute focused research tasks in parallel using web tools
- **Citations Agent**: Adds proper citations to research reports

## Quick Start

### Using the Deep Research Skill

```
deep-research What are the most effective treatments for depression?
```

or with @ syntax:

```
@deep-research Compare AWS, Azure, and Google Cloud by market share
```

**Important**: Do NOT use `/deep-research` - slash commands are built-in and cannot be customized.

### Adding Citations

After research completes, add citations:

```
citations [paste your research report here]
```

## Features

### ✅ Implemented

- **Three query types**: Depth-first, breadth-first, straightforward
- **Parallel subagent execution**: 2-6 subagents working simultaneously
- **Web tools integration**: web_search, web_fetch, Playwright MCP
- **Source quality evaluation**: Critical assessment of sources
- **Citation generation**: Automatic citation addition
- **Flexible complexity**: From simple fact-finding to multi-perspective analysis

### 🎯 Design Goals

- **Medium complexity**: Not too simple, not overly complex
- **Web-focused**: Uses web tools only (no internal tools like GDrive, Gmail)
- **Practical**: Focuses on what works reliably
- **Efficient**: Parallel execution, tool call budgets, clear stopping criteria

## Query Types

### 1. Straightforward Queries
Simple fact-finding with clear answers.

- **Example**: `deep-research What is the population of Tokyo?`
- **Approach**: 1 subagent with focused instructions
- **Duration**: ~1-2 minutes

### 2. Breadth-First Queries
Multi-part questions with distinct sub-topics.

- **Example**: `deep-research Compare AWS, Azure, and Google Cloud`
- **Approach**: 3 subagents researching different aspects
- **Duration**: ~3-5 minutes

### 3. Depth-First Queries
Complex questions requiring multiple perspectives.

- **Example**: `deep-research What are the most effective treatments for depression?`
- **Approach**: 4 subagents exploring different viewpoints
- **Duration**: ~5-10 minutes

## Architecture

```
User invokes: deep-research <query>
    ↓
Research Lead Agent
  ├─ Analyzes query
  ├─ Determines query type
  ├─ Creates research plan
  ├─ Launches 2-6 subagents in parallel (Task tool)
  │   ├─ Subagent 1 → web_search → web_fetch → Report
  │   ├─ Subagent 2 → web_search → web_fetch → Report
  │   └─ Subagent 3 → web_search → Playwright → Report
  └─ Synthesizes findings → Final Markdown report
      ↓
Citations Agent (optional)
  └─ Adds citations to report
```

## Tools Used

| Tool | Purpose | When to Use |
|------|---------|-------------|
| `web_search` | Search the web | Initial research |
| `web_fetch` | Get full page content | After web_search, for static pages |
| `mcp__playwright__navigate` | Load JavaScript pages | Dynamic content |
| `mcp__playwright__snapshot` | Get rendered content | JavaScript-heavy pages |
| `Task` | Launch subagents | Parallel research |

## Automatic Playwright MCP Usage

The system **automatically detects** and uses Playwright MCP:

### Detection Conditions
Automatically switches to Playwright when `web_fetch` returns:
- Incomplete content (truncated HTML)
- JavaScript placeholder text
- "Enable JavaScript" messages
- Empty or very short content
- Login walls or paywalls

### Websites that Prefer Playwright
- Single Page Applications (React/Vue/Angular apps)
- News sites with dynamic content loading
- Social platforms (Twitter/X, LinkedIn, Reddit)
- E-commerce sites
- Infinite scroll or lazy loading sites
- Pages requiring user interaction

## File Structure

```
simple_claude_deep_research_agent/
├── .claude/
│   └── skills/
│       ├── deep-research.md       # Lead agent skill
│       ├── research-subagent.md   # Subagent skill
│       └── citations.md           # Citations agent skill
├── research_lead_agent.md         # Original prompt (reference)
├── research_subagent.md           # Original prompt (reference)
├── citations_agent.md             # Original prompt (reference)
├── README.md                      # Chinese documentation
└── README_en.md                   # This file (English)
```

## Examples

### Example 1: Simple Query

```
deep-research What is the current population of Tokyo?
```

**Expected output**:
- Single subagent searches for population data
- Returns current population with source
- Total time: ~1 minute

### Example 2: Comparison Query

```
deep-research Compare AWS, Azure, and Google Cloud pricing for compute instances
```

**Expected output**:
- 3 subagents, each researching one provider
- Lead agent synthesizes comparison table
- Total time: ~3-5 minutes

### Example 3: Deep Analysis

```
deep-research What caused the 2008 financial crisis?
```

**Expected output**:
- 4 subagents exploring: economic, regulatory, behavioral, historical perspectives
- Comprehensive multi-perspective analysis
- Total time: ~5-8 minutes

## Configuration

### MCP Server Setup (Optional)

To use Playwright MCP for dynamic pages:

1. Install Playwright MCP server (if not already installed)
2. Add to your Claude Code MCP configuration
3. Restart Claude Code

The system will automatically use Playwright when needed for JavaScript-heavy pages.

### Skill Files

Skills are automatically loaded from `.claude/skills/`:
- `deep-research.md`: Main research coordination
- `research-subagent.md`: Subagent research logic
- `citations.md`: Citation addition

## Development Guide

### Modifying Behavior

**To change research strategy**: Edit `.claude/skills/deep-research.md`
- Query classification logic
- Subagent deployment rules
- Synthesis approach

**To modify subagent behavior**: Edit `.claude/skills/research-subagent.md`
- Tool usage patterns
- Source quality evaluation
- Report format

**To modify citations**: Edit `.claude/skills/citations.md`
- Citation style
- When to cite rules

### Testing

1. Edit skill files
2. Test with: `deep-research <test query>`
3. Iterate based on results

## Limitations

- **Web-only**: Uses web tools only (no GDrive, Gmail, Slack integration)
- **Tool call limits**: Subagents limited to 20 tool calls each
- **No persistent storage**: Research not cached between sessions
- **Language**: Primarily optimized for English queries

## Future Enhancements

Possible improvements:

- [ ] Add more query types (e.g., temporal analysis, trend detection)
- [ ] Implement result caching
- [ ] Add export formats (PDF, DOCX)
- [ ] Multi-language support
- [ ] Research history tracking

## Troubleshooting

### "Unknown skill" error

- Ensure skill files are in `.claude/skills/`
- Check YAML frontmatter is correct
- Restart Claude Code

### Subagents not launching

- Check Task tool syntax
- Ensure `general-purpose` subagent_type is used
- Verify Claude Code has permissions

### Poor research quality

- Try different query wording
- Check if query type is correctly classified
- Verify web_search is working

### Playwright MCP not being used

- Check if Playwright MCP is installed and configured
- Confirm the page actually requires JavaScript rendering
- Verify subagent detected incomplete content

## Tips for Best Results

1. **Be specific**: "What are the pros and cons of React vs Vue?" is better than "Tell me about frontend frameworks"
2. **Set context**: Include time constraints if relevant (e.g., "as of 2025")
3. **Use appropriate complexity**: Simple questions don't need multiple subagents
4. **Add citations**: Use the citations skill after research for authoritative reports
5. **Iterate**: Refine your query based on initial results

## Workflow

### Typical Research Process

1. **Initiate query**: Use `deep-research <question>`
2. **Analyze & plan**: Lead agent analyzes query type and complexity
3. **Parallel research**: 2-6 subagents research different aspects simultaneously
4. **Synthesize report**: Lead agent integrates all findings
5. **Add citations** (optional): Use `citations` to add references

### Best Practices

- **Start broad**: Use broad queries first, then refine based on results
- **Check sources**: Evaluate information quality, look for bias and speculation
- **Verify facts**: Cross-reference key facts
- **Save results**: Save important reports to files

## Technical Details

### Query Classification

The system classifies queries into three types:

| Type | Description | Subagents | Example |
|------|-------------|-----------|---------|
| Straightforward | Simple fact-finding | 1 | "What is Tokyo's population?" |
| Breadth-first | Independent sub-topics | 3 | "Compare AWS, Azure, GCP" |
| Depth-first | Multiple perspectives | 4 | "What caused the 2008 crisis?" |

### Tool Call Strategy

**Static content**: `web_search` → `web_fetch`
- Blogs, articles, documentation
- Simple HTML websites

**Dynamic content**: `web_search` → Playwright MCP
- React/Vue/Angular applications
- Dynamically loaded websites
- Pages requiring interaction

### Subagent Constraints

- **Tool call budget**: 3-15 calls depending on complexity
- **Hard limit**: 20 tool calls maximum
- **Minimum**: 3 tool calls for meaningful research
- **Parallel**: Use 2+ web_search calls simultaneously

## Design Philosophy

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

## Contributing

Contributions are welcome! Feel free to:

1. Submit issue reports
2. Suggest new features
3. Submit improvement requests

## License

This is a demonstration project for educational purposes.

---

**Documentation Version**: 1.0
**Last Updated**: 2025-12-29
**Maintainer**: Claude Code Community
