# Jira Bug Analyzer Design

**Date:** 2026-04-27  
**Status:** Draft

## Overview

An automated system that polls Jira every 5 minutes for bugs labeled `ai-triage`, analyzes them using local source code from multiple repositories, and posts analysis results back to Jira as comments with effort estimates.

## Goals

- Automatically triage bugs labeled `ai-triage` in Jira Cloud
- Analyze bugs using code from multiple local repositories
- Provide medium-depth analysis: relevant files, execution paths, potential fixes, effort estimates
- Post findings as markdown comments in Jira
- Update labels after analysis to prevent re-processing
- Run continuously while Claude Code session is active

## Non-Goals

- 24/7 monitoring (runs only while Claude Code is active)
- Deep root cause analysis (medium depth only)
- Automatic fix implementation
- Integration with CI/CD or deployment systems

## Architecture

### Components

**1. Claude Code Loop**
- Uses `/loop 5m <prompt>` to trigger analysis every 5 minutes
- No external scripts - pure Claude execution via MCP and built-in tools

**2. MCP Integration**
- Uses existing `mcp-atlassian` server for all Jira operations
- Authenticates via environment variables (already configured)

**3. Repository Configuration**
- `repositories.md` file in working directory describes all repositories
- Contains repo paths, purpose, stack, and key areas
- Parsed by Claude at runtime

**4. Code Exploration**
- Claude uses grep, find, and Read tools to explore repositories
- Iterative search - can refine and dig deeper as needed
- Searches across all configured repositories for each issue

### Data Flow

```
┌─────────────────┐
│ /loop 5m prompt │
└────────┬────────┘
         │
         ▼
┌─────────────────────────────────┐
│ Query Jira via MCP              │
│ for issues with 'ai-triage'     │
│ (max 3 per cycle)               │
└────────┬────────────────────────┘
         │
         ▼
┌─────────────────────────────────┐
│ Read repositories.md            │
│ to understand repo context      │
└────────┬────────────────────────┘
         │
         ▼
┌─────────────────────────────────┐
│ For each issue (up to 3):       │
│ - Extract keywords              │
│ - Search all repos for relevant │
│   files (grep, find)            │
│ - Read relevant code            │
│ - Follow imports/calls          │
└────────┬────────────────────────┘
         │
         ▼
┌─────────────────────────────────┐
│ Analyze bug:                    │
│ - Identify likely cause         │
│ - Trace execution paths         │
│ - Propose potential fixes       │
│ - Estimate effort               │
└────────┬────────────────────────┘
         │
         ▼
┌─────────────────────────────────┐
│ Post markdown comment to Jira   │
│ via MCP                         │
└────────┬────────────────────────┘
         │
         ▼
┌─────────────────────────────────┐
│ Update labels via MCP:          │
│ - Remove 'ai-triage'            │
│ - Add 'ai-analyzed'             │
└────────┬────────────────────────┘
         │
         ▼
┌─────────────────────────────────┐
│ Report progress to session      │
│ Continue to next cycle          │
└─────────────────────────────────┘
```

## Configuration

### Environment Variables

Required (already configured for mcp-atlassian):
- `JIRA_URL` - Jira Cloud instance URL
- `JIRA_EMAIL` - User email for authentication
- `JIRA_API_TOKEN` - API token for authentication

### repositories.md Format

Located in working directory where `/loop` runs.

```markdown
# Bug Solver Repositories

## /absolute/path/to/repo1
**Purpose:** Brief description of what this repo does
**Stack:** Technologies used (e.g., React, TypeScript, Next.js)
**Covers:** What kinds of issues this repo handles
**Key areas:**
- path/to/important/dir - description
- path/to/other/dir - description

## /absolute/path/to/repo2
**Purpose:** Backend API service
**Stack:** Python, FastAPI, PostgreSQL
**Covers:** API endpoints, database operations
**Key areas:**
- app/routes/ - API endpoints
- app/models/ - Database models
```

Claude parses this at runtime to:
1. Know which directories to search
2. Understand what each repo covers (helps prioritize searches)
3. Include relevant tech stack in analysis

## Analysis Process

### Discovery Phase

For each issue, Claude:
1. Extracts keywords from issue summary and description
   - Error messages
   - Component names
   - Function names
   - Stack traces
2. Searches all repositories for these keywords using grep
3. Identifies potentially relevant files
4. Reads files to understand context
5. Follows imports and function calls if needed
6. Refines search based on findings

### Analysis Phase

Claude performs medium-depth analysis:
1. **Relevant Files** - List files involved with line numbers
2. **Execution Path** - Trace likely code path that triggers the bug
3. **Potential Cause** - Hypothesis about root cause
4. **Proposed Fixes** - 2-3 potential solutions with trade-offs
5. **Effort Estimate** - Small/Medium/Large based on:
   - Complexity of fix
   - Number of files affected
   - Testing requirements
   - Potential side effects

### Output Format

Markdown comment posted to Jira:

```markdown
## AI Analysis

**Analyzed:** YYYY-MM-DD HH:MM UTC

### Relevant Files
- `repo1/path/to/file.py:45-67` - Authentication logic
- `repo2/path/to/other.ts:123` - API client call

### Analysis
[2-3 paragraphs explaining the likely cause and execution path]

### Potential Fixes

**Option 1: [Approach name]**
- Description of fix
- Trade-offs
- Files affected

**Option 2: [Alternative approach]**
- Description
- Trade-offs

### Effort Estimate
**Medium** (X-Y hours)
- Reasoning for estimate

---
*Generated by Claude Code Bug Analyzer*
```

## Processing Rules

### Issue Selection
- Query: `labels = "ai-triage" ORDER BY created ASC`
- Limit: 3 issues per 5-minute cycle
- If >3 issues exist, process oldest 3 first (FIFO)

### Label Management
After successful analysis:
1. Remove `ai-triage` label
2. Add `ai-analyzed` label
3. This prevents re-processing the same issue

If analysis fails (error accessing code, etc.):
- Leave labels unchanged
- Report error to Claude Code session
- Issue will be retried in next cycle

### Error Handling

Errors are reported to the Claude Code session but don't stop the loop:
- Jira API failures → report and continue to next cycle
- Repository access issues → skip that repo, continue with others
- File read failures → note in analysis, continue
- MCP tool failures → report and retry next cycle

## Limitations

### Session-Dependent
- Only runs while Claude Code session is active
- If CLI/app is closed, monitoring stops
- Suitable for "while I'm working" scenarios, not 24/7 monitoring

### Analysis Depth
- Medium depth analysis only (not exhaustive root cause investigation)
- May miss complex edge cases or race conditions
- Estimates are educated guesses, not guarantees

### Repository Access
- Repositories must be locally accessible
- No remote repository cloning
- No git operations (analysis uses current state)

### Rate Limiting
- Limited to 3 issues per cycle to avoid overwhelming Jira API
- 5-minute interval may not be suitable for high-volume bug tracking
- Each analysis could take 1-3 minutes depending on code complexity

## Future Enhancements

Potential improvements not in initial scope:

- Configurable analysis depth per issue priority
- Support for attaching fix patches/diffs to Jira
- Integration with git blame to identify code authors
- Automatic assignment to relevant developers
- Slack/email notifications for critical bugs
- Historical tracking of analysis accuracy
- Learning from accepted/rejected fix suggestions

## Success Criteria

The system is successful if:

1. **Accuracy** - Identifies relevant code in >80% of cases
2. **Usefulness** - Analysis provides actionable insights (not generic suggestions)
3. **Reliability** - Handles errors gracefully, doesn't crash the loop
4. **Efficiency** - Processes 3 issues in <5 minutes (doesn't backup)
5. **Non-intrusive** - Runs quietly in background, only surfaces errors/results
