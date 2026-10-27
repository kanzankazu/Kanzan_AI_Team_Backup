---
title: PR Review Delegation Workflow
type: entity
tags: [devops, tools, automation, workflow]
created: 2026-10-27
updated: 2026-10-27
status: active
---

# PR Review Delegation Workflow

End-to-end process for delegating Bitbucket PR code reviews to Hermes fullstack-dev agent.

## Overview

**Goal:** Automated code review of Bitbucket PRs via agent delegation  
**Primary Tool:** `delegate_task` with fullstack-dev agent  
**Fallback:** REST API diff fetch when MCP tools blocked  
**Output:** Markdown review file with findings + severity + fix suggestions

## Workflow Steps

### 1. Receive PR Review Request
**Input:** Bitbucket PR URL (e.g., `https://bitbucket.bri.co.id/projects/BRIMONS/repos/brimo-ns-android/pull-requests/3680/overview`)

**Initial check:**
- Verify MCP Bitbucket server connection (`manage_connections status`)
- Confirm agent has Bitbucket access (BITBUCKET_TOKEN in profile .env)

### 2. Delegation Attempt (Primary Path)
**Tool:** `delegate_task`

```python
delegate_task(
    tasks=[{
        "goal": "Review Bitbucket PR #3680 for Android code quality",
        "context": f"PR URL: {pr_url}\nUse Bitbucket MCP to fetch diff and analyze changes."
    }]
)
```

**Expected:** Agent fetches PR via MCP tools, analyzes, generates review file

### 3. Fallback: MCP Timeout/Network Block
**Trigger:** Subagent reports MCP tool timeout or web_extract failure

**Action:** Fetch PR diff manually via Bitbucket REST API

```bash
curl -s -u "$BITBUCKET_USER:$BITBUCKET_TOKEN" \
  "https://bitbucket.bri.co.id/rest/api/1.0/projects/BRIMONS/repos/brimo-ns-android/pull-requests/3680/diff" \
  | jq -r '.diffs[] | .source.toString + "\n" + .hunks[].segments[] | select(.type == "ADDED") | .lines[].line'
```

**Result:** Raw diff text (added/modified lines with context)

### 4. Re-Delegation with Preloaded Diff
**Tool:** `delegate_task` (second attempt)

```python
delegate_task(
    tasks=[{
        "goal": "Review Android PR #3680 code quality",
        "context": f"""
PR: {pr_url}
JIRA: BRIMO300-32589

Diff (preloaded):
{diff_text}

Analyze for:
- Null-safety issues
- Logic inversions
- Lifecycle violations
- Threading safety
- Memory leaks
"""
    }]
)
```

**Expected:** Agent generates review without needing MCP access

### 5. Review Output
**File location:** `~/workplace/ai-team-backup/reviews/review_{PR}_{date}.md`

**Structure:**
- Metadata (PR, JIRA, reviewer, date)
- Severity summary (🔴 Critical, 🟡 Medium, 🟢 Minor)
- Findings table (file, line, issue, severity)
- Detailed analysis per finding
- Fix suggestions (BEFORE/AFTER code)
- Test scenarios for critical issues
- Pre-merge checklist

### 6. Delivery to Telegram
**Protocol:** Post summary + attach file to dev topic (thread 6)

**Methods:**
- Auto: `/Users/user/.hermes/scripts/post_review_to_telegram.py`
- Manual: Copy summary text + attach file via Telegram UI

**Summary format:**
```
📋 PR #{number} CODE REVIEW — {JIRA}

Status: ⚠️ BLOCKER ISSUES FOUND

Summary:
🔴 X Critical: [brief list]
🟡 X Medium: [brief list]
🟢 X Minor: [brief list]

Critical Action:
1. [Fix 1]
2. [Fix 2]
...

Files:
- File1.kt (lines X-Y)
- File2.kt (lines X-Y)

Full Report: review_{PR}_{date}.md
```

## Blockers & Solutions

| Blocker | Impact | Solution |
|---------|--------|----------|
| MCP Bitbucket timeout | Cannot fetch PR diff via tools | Fetch via curl + REST API, pass as context |
| Subagent network isolation | web_extract fails on internal URLs | Preload diff in delegation context |
| Telegram SSL error (Python) | Auto-post script fails | Manual copy-paste + file attachment |
| Message too long (4096 chars) | Cannot send full review | Send short summary + file upload |

## Configuration

### Required Environment Variables (per agent profile)
```bash
BITBUCKET_URL=https://bitbucket.bri.co.id
BITBUCKET_TOKEN=<token>
BITBUCKET_USER=<username>
JIRA_URL=https://jira.bri.co.id
JIRA_TOKEN=<token>
```

### MCP Server Config (~/.hermes/config.yaml)
```yaml
mcp_servers:
  bitbucket-server:
    command: node
    args:
      - /Users/user/.kiro/mcp-servers/bitbucket-server-mcp/build/index.js
    env:
      BITBUCKET_URL: https://bitbucket.bri.co.id
      BITBUCKET_PROJECT: BRIMONS
      BITBUCKET_TOKEN: ${BITBUCKET_TOKEN}
    read_only: true
```

### Telegram Bot Config
```python
# /Users/user/.hermes/scripts/post_review_to_telegram.py
DEV_BOT_TOKEN = "8952827360:AAEToPURHXTQTD7ZEwMUsZrBKQORB3JohCQ"
CHAT_ID = "-1004308391481"  # Kanzankazuhermesagents
THREAD_ID = 6  # dev topic
```

## Success Criteria

- ✅ Review generated within 5 minutes
- ✅ All modified files analyzed
- ✅ Critical issues have fix suggestions + test scenarios
- ✅ Review file committed to git repo (reviews/ dir)
- ✅ Summary + file posted to Telegram dev thread

## Related Pages

- [[android-review-rules-bri]] — Review standards for Android PRs
- [[hermes-agent-delegation]] — General delegate_task patterns
- [[bitbucket-mcp-setup]] — MCP server configuration

## Provenance

- **Learned from:** Session 2026-10-27 (PR #3680 review)
- **First successful run:** 2026-10-27 09:00 WIB
- **Iteration count:** 3 (MCP fail → MCP timeout → REST API success)
- **Obsidian notes:** [[2026-10-27-pr-review-session]]

## Future Improvements

- [ ] Auto-trigger on Bitbucket webhook (PR opened/updated)
- [ ] Parallel review (multiple agents for different aspects)
- [ ] Auto-fix generation (create fix commits)
- [ ] Review diff between iterations (PR updated after review)
- [ ] Integration with JIRA (auto-comment findings to ticket)
