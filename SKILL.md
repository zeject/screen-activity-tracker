---
name: screen-activity-tracker
description: Screen activity tracking with periodic screenshots, VL analysis, and daily logs. Trigger when user asks to track screen activity, record screen usage, start/stop tracking, view daily summary, or search activity history.
version: "1.0.0"
license: "MIT"
author: "zeject"
homepage: "https://github.com/zeject-google/screen-activity-tracker"
metadata:
  openclaw:
    emoji: "🖥️"
    os: ["darwin"]
    requires:
      bins: ["peekaboo", "python3"]
    install:
      - kind: "npm-global"
        pkg: "@steipete/peekaboo"
        bins: ["peekaboo"]
        label: "Install Peekaboo (npm)"
      - id: "python3"
        kind: "system"
        label: "Python 3.x required"
---

# Screen Activity Tracker

Periodically capture screenshots, analyze with a VL model, and log activity to SiYuan Note or local markdown files.

## ⚠️ Mandatory Rules (CRITICAL — READ BEFORE ANY ACTION)

**Rule 1: Use ONLY `cron` tool for start/stop tracking**

When user says "start tracking" / "开始追踪" / "record screen" / "开始记录", you **MUST** call the `cron` tool with `operation: "add"`.

**Rule 2: NEVER use `skill_workshop` tool**

Do NOT call `skill_workshop` under any circumstances. This skill does NOT need skill registration, task creation, or proposal submission via skill_workshop. The `skill_workshop` tool will reject your call with validation errors.

**Rule 3: ONLY these 2 tools are allowed for this skill**

| Purpose | Tool | When |
|---------|------|------|
| Start/stop/change/list tracking jobs | `cron` | User asks to start/stop tracking |
| Run summary/search/one-shot scripts | `bash` (or shell command) | User asks for summary, search, or single screenshot |

**Rule 4: No verbal promises**

You do NOT have background threads. Saying "I'll start tracking for you" without calling `cron` tool is a LIE. Always create the actual cron job first, then confirm.

---

## Using the Cron Tool

### Create a tracking cron job (start tracking)

Use the `cron` tool with these parameters:

| Parameter | Required | Description |
|-----------|----------|-------------|
| `operation` | Yes | `"add"` to create, `"edit"` to update, `"list"` to list, `"remove"` to delete |
| `name` | Yes | Human-readable name, e.g. "屏幕活动记录" |
| `command` | Yes | Shell command to run the tracker script |
| `cron` | Yes | Cron expression, default `*/5 * * * *` (read `interval_minutes` from config.json to adjust) |
| `timeoutSeconds` | No | Default 90 |
| `description` | No | Description of the task |

**Step-by-step workflow when user says "开始追踪屏幕" / "start tracking":**

1. **Read config first** to get the skill directory and interval:
   ```bash
   cat <SKILL_DIR>/config.json
   ```
   Parse the `interval_minutes` field. If it's 5, cron expression is `*/5 * * * *`. If 10, `*/10 * * * *`, etc.

   If `config.json` doesn't exist, copy from `config.example.json` and tell the user to configure it.

2. **Build the command** using the skill directory path:
   ```
   sh -lc '/bin/bash <SKILL_DIR>/scripts/screen-activity.sh'
   ```
   Replace `<SKILL_DIR>` with the actual skill directory.

3. **Call the cron tool** to create the job:
   ```json
   {
     "operation": "add",
     "name": "屏幕活动记录",
     "command": "sh -lc '/bin/bash /Users/zeject/ai/skills/mySkills/screen-activity-tracker/scripts/screen-activity.sh'",
     "cron": "*/5 * * * *",
     "timeoutSeconds": 90,
     "description": "每5分钟截图分析屏幕活动，记录到思源笔记或本地文件"
   }
   ```

4. **After creation**, tell the user:
   - Job ID (from cron tool response)
   - Running interval
   - Backend target (siyuan or local)
   - The cron tool will return a job ID like `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`

### List existing tracking jobs

```json
{ "operation": "list" }
```

Check if a "屏幕活动记录" job already exists before creating a duplicate.

### Stop tracking (disable)

```json
{
  "operation": "edit",
  "jobId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "enabled": false
}
```

### Resume tracking (re-enable)

```json
{
  "operation": "edit",
  "jobId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "enabled": true
}
```

### Change interval

```json
{
  "operation": "edit",
  "jobId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "cron": "*/10 * * * *"
}
```

---

## Other Script Commands (use Bash tool)

### Daily summary

When user says "今天屏幕活动总结" / "daily summary":

```bash
python3 <SKILL_DIR>/scripts/activity-analyzer.py summary
```

### Search history

When user says "搜索历史" / "我上次用X是什么时候" / "search activity":

```bash
python3 <SKILL_DIR>/scripts/activity-analyzer.py search "<query>"
```

### One-shot screenshot + analysis (no cron)

```bash
bash <SKILL_DIR>/scripts/screen-activity.sh
```

---

## Features

| Feature | Description |
|---|---|
| Dual backend | SiYuan Note or local markdown (configurable) |
| Analysis modes | simple / category / productivity / custom prompt |
| Idle detection | Skip screenshots when user is idle |
| App blacklist | Skip sensitive apps (banking, passwords, etc.) |
| Daily summary | `activity-analyzer.py summary` |
| Search history | `activity-analyzer.py search "keyword"` |
| Auto cleanup | Keep screenshots for N days (default: 7) |

## Configuration

Edit `config.json` in the skill directory:

```json
{
  "backend": "siyuan",
  "siyuan": {
    "url": "http://192.168.1.100:6806",
    "token": "YOUR_TOKEN",
    "notebook_id": "YOUR_NOTEBOOK_ID"
  },
  "local": {
    "output_dir": "~/screen-activity",
    "keep_days": 7
  },
  "mlx_url": "http://192.168.1.198:18000/v1",
  "analysis_mode": "simple",
  "interval_minutes": 5,
  "idle_detection": true,
  "app_blacklist": ["银行", "密码", "支付宝", "微信支付", "登录"]
}
```

### Analysis Modes

- **simple**: `[app] description` — what the user is doing
- **category**: `[work|娱乐|学习|休息] description` — classify activity
- **productivity**: `[score/10] description` — rate productivity
- **custom**: Use `analysis_prompt` field for custom instructions

## Quick Reference

| User says | Action |
|-----------|--------|
| "开始追踪屏幕" | Create cron job with `cron` tool |
| "停止追踪屏幕" | Disable cron job |
| "今天屏幕活动总结" | Run `activity-analyzer.py summary` |
| "搜索X" | Run `activity-analyzer.py search "X"` |
| "截图分析" | Run `screen-activity.sh` once |

## License

MIT
