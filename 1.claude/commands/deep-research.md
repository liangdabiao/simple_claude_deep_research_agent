---
description: "Conduct comprehensive deep research on any topic using parallel subagents and web search"
---

# /deep-research

Conduct comprehensive research on any topic using parallel subagents, web search, and intelligent synthesis.

## Usage

```
/deep-research <your research question>
```

## Examples

- `/deep-research What are the most effective treatments for depression?`
- `/deep-research Compare the top 3 cloud providers by market share and features`
- `/deep-research What caused the 2008 financial crisis?`
- `/deep-research List all Fortune 500 companies and their CEOs`

## How It Works

1. **Query Analysis**: The Research Lead Agent analyzes your question and determines the optimal research strategy
2. **Research Planning**: Based on query type (depth-first, breadth-first, or straightforward), a detailed plan is created
3. **Parallel Execution**: Multiple research subagents work in parallel to gather information from the web
4. **Synthesis**: The Lead Agent synthesizes all findings into a comprehensive, well-structured report

## Query Types

- **Depth-first**: Complex questions requiring multiple perspectives (e.g., "What causes obesity?")
- **Breadth-first**: Multi-part questions with distinct sub-topics (e.g., "Compare Nordic economic systems")
- **Straightforward**: Focused fact-finding (e.g., "What is Tokyo's population?")

## Tips

- Be specific in your question for better results
- For complex topics, the system will automatically deploy multiple subagents
- Research time varies by complexity: straightforward (~1-2 min), standard (~3-5 min), complex (~5-10 min)
- All information is sourced from the web and synthesized for accuracy

## Related Skills

- `deep-research`: Main research coordination skill
- `research-subagent`: Internal subagent for parallel research tasks
