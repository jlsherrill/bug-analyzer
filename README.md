# Jira Bug Analyzer

Automated Jira bug triage using Claude Code. Polls Jira every 5 minutes for bugs labeled `ai-triage`, analyzes them using local source code, and posts analysis with fix suggestions back to Jira.

## Features

- Polls Jira Cloud every 5 minutes for `ai-triage` labeled issues
- Analyzes up to 3 issues per cycle
- Searches across multiple local repositories for relevant code
- Provides medium-depth analysis with:
  - Relevant files and line numbers
  - Execution path analysis
  - 2-3 potential fix approaches
  - Effort estimates
- Posts markdown-formatted analysis as Jira comments
- Updates labels (`ai-triage` → `ai-analyzed`) to prevent re-processing
- Runs while Claude Code session is active

## Prerequisites

1. **Claude Code** installed and running
2. **MCP Atlassian Server** configured with environment variables:
   - `JIRA_URL` - Your Jira Cloud instance URL (e.g., `https://yourcompany.atlassian.net`)
   - `JIRA_EMAIL` - Your Jira account email
   - `JIRA_API_TOKEN` - Your Jira API token ([generate here](https://id.atlassian.com/manage-profile/security/api-tokens))
3. **Local repositories** - The codebases you want to analyze must be locally accessible

## Setup

### 1. Configure Repositories

Copy the example repositories file and customize it:

```bash
cp repositories.md.example repositories.md
```

Edit `repositories.md` with your actual repository paths:

```markdown
# Bug Solver Repositories

## /home/user/projects/my-app
**Purpose:** Main web application
**Stack:** React, TypeScript, Node.js
**Covers:** Frontend bugs, API client issues
**Key areas:**
- src/components/ - React components
- src/api/ - API client
```

Add all repositories you want the analyzer to search.

### 2. Verify Environment Variables

Ensure your Jira credentials are set:

```bash
echo $JIRA_URL
echo $JIRA_EMAIL
echo $JIRA_API_TOKEN
```

If not set, add them to your shell profile or `.env` file.

### 3. Test MCP Connection

In Claude Code, verify the mcp-atlassian server is working:

```
Can you list my recent Jira issues?
```

If this works, you're ready to run the analyzer.

## Usage

### Start the Analyzer

From this directory in Claude Code, run:

```bash
/loop 5m analyze-jira-bugs
```

Or with the full prompt:

```bash
/loop 5m "$(cat analyzer-prompt.md)"
```

### Monitor Progress

The analyzer will report:
- Number of issues found each cycle
- Analysis progress for each issue
- Success/failure status
- Any errors encountered

Example output:
```
Found 2 issues with ai-triage label
Analyzing PROJ-123: Login fails with 500 error
- Searching repositories...
- Found relevant code in backend-api
- Posting analysis to Jira
- Updated labels
Analyzed PROJ-123: Login fails with 500 error

Analyzing PROJ-124: UI rendering broken on mobile
- Searching repositories...
- Found relevant code in mobile-app
- Posting analysis to Jira
- Updated labels
Analyzed PROJ-124: UI rendering broken on mobile

Analyzed 2 issues this cycle
```

### Stop the Analyzer

Press `Ctrl+C` in Claude Code to stop the loop.

## Workflow

### Creating Issues for Analysis

1. Create or find a Jira issue
2. Add the `ai-triage` label to it
3. Wait up to 5 minutes for the analyzer to process it
4. Check the issue for the AI Analysis comment
5. The label will change from `ai-triage` to `ai-analyzed`

### Using Analysis Results

The analyzer posts a structured comment with:
- **Relevant Files** - Where to look in the codebase
- **Analysis** - What's likely causing the bug
- **Potential Fixes** - Multiple approaches with trade-offs
- **Effort Estimate** - How complex the fix might be

Use this as a starting point for investigation and fixing the bug.

## Configuration

### Polling Interval

Change the interval by modifying the loop command:

```bash
/loop 10m analyze-jira-bugs   # Check every 10 minutes
/loop 2m analyze-jira-bugs    # Check every 2 minutes (more responsive)
```

### Issues Per Cycle

The analyzer processes max 3 issues per cycle by default. To change this, edit `analyzer-prompt.md` and modify the limit in step 1.

### Analysis Depth

Currently set to medium-depth analysis. To adjust:
- **Lighter**: Reduce the analysis steps in `analyzer-prompt.md`
- **Deeper**: Add more investigation steps (but watch for longer cycle times)

## Troubleshooting

### "No MCP tools found"

The mcp-atlassian server isn't configured. Check:
1. MCP server is installed
2. Environment variables are set
3. Restart Claude Code

### "Repository not found"

Check `repositories.md`:
1. Paths are absolute (not relative)
2. Paths exist and are readable
3. No typos in the paths

### "Analysis takes too long"

If processing 3 issues takes longer than 5 minutes:
1. Reduce issues per cycle
2. Increase loop interval to 10m
3. Simplify repository structure (fewer repos or key areas)

### "Same issue analyzed multiple times"

Label update might have failed. Check:
1. MCP server has write permissions
2. Check Jira issue history for label changes
3. Manually verify labels are being updated

## Limitations

- **Session-dependent**: Only runs while Claude Code is active
- **Medium-depth only**: Not exhaustive root cause analysis
- **No auto-fix**: Provides suggestions but doesn't implement fixes
- **Local repos only**: Can't clone or access remote repositories
- **Rate limited**: Max 3 issues per 5 minutes

## Architecture

See `docs/superpowers/specs/2026-04-27-jira-bug-analyzer-design.md` for detailed design documentation.

## Future Enhancements

Potential improvements:
- Configurable analysis depth per issue priority
- Auto-generate fix patches
- Integration with git blame to identify code authors
- Automatic assignment to developers
- Slack/email notifications for critical bugs
- Analysis accuracy tracking

## Contributing

This is a personal automation tool. Customize `analyzer-prompt.md` and `repositories.md` to fit your workflow.

## License

MIT
