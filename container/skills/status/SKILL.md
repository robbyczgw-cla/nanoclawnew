---
name: status
description: OpenClaw-style status dashboard — version, model, session, context, tools, and tasks. Use when the user runs /status.
---

# /status — System Status Dashboard

Generate a compact OpenClaw-style status report.

**Main-channel check:** Only the main channel has `/workspace/project` mounted. Run:

```bash
test -d /workspace/project && echo "MAIN" || echo "NOT_MAIN"
```

If `NOT_MAIN`, respond with:
> This command is available in your main chat only. Send `/status` there.

Then stop.

## How to gather the information

Run ALL commands in a single Bash call to minimize overhead:

```bash
echo "=== VERSION ==="
cat /workspace/project/package.json 2>/dev/null | grep '"version"' | head -1
git -C /workspace/project rev-parse --short HEAD 2>/dev/null || echo "no-git"

echo "=== MODEL ==="
claude --version 2>/dev/null || echo "unknown"

echo "=== SESSION ==="
echo "Timestamp: $(TZ=${TZ:-UTC} date '+%Y-%m-%d %H:%M %Z')"
echo "Session ID: ${SESSION_ID:-unknown}"
echo "Group: $(basename /workspace/group 2>/dev/null)"

echo "=== WORKSPACE ==="
test -d /workspace/project && echo "project: rw" || echo "project: none"
ls /workspace/group/ 2>/dev/null | wc -l | xargs -I{} echo "group files: {}"
ls /workspace/ipc/ 2>/dev/null | tr '\n' ', '

echo "=== TOOLS ==="
which agent-browser 2>/dev/null && echo "browser: yes" || echo "browser: no"
node --version 2>/dev/null

echo "=== TASKS ==="
cat /workspace/ipc/current_tasks.json 2>/dev/null || echo "[]"
```

Then also call `mcp__nanoclaw__list_tasks` to get the freshest task data.

## Report format

Present as a single compact message. Use this exact format, adapting values:

```
🦞 NanoClaw {version} ({git-short-hash})
🧠 Model: {claude-version}
🕐 {timestamp}
📂 Session: {group-name} · {session-id-short}

📦 Workspace:
• Project: ✓ rw | Group: {N} files | IPC: ✓

🔧 Tools:
• Core: ✓  Web: ✓  MCP: ✓  Browser: ✓/✗

📋 Tasks: {N} active / {N} paused / {N} total
{if tasks exist, list each on one line: • task-name — schedule — status}
```

Rules:
- Keep it tight — no extra blank lines or explanations
- Use ✓/✗ for boolean states
- Shorten session IDs to first 8 chars
- If no tasks, show "📋 Tasks: none"
- Show project mount as "rw" or "ro" based on whether you can write to `/workspace/project`
