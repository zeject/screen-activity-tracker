---
name: screen-activity-tracker
description: >
  Screen activity tracking skill. IMPORTANT: This skill ONLY uses `cron` tool and `bash` shell commands.
  NEVER call `skill_workshop`, NEVER create tasks via skill_workshop, NEVER use action/create/goal/status/taskName parameters.
  The only valid tools are: 1) `cron` tool for start/stop/list cron jobs, 2) bash/shell for running summary/search scripts.
  Trigger when user asks: track screen activity, start/stop tracking, daily screen summary, search activity history,
  today's activity log, 整理今天的操作, 开始追踪屏幕, 停止追踪, 屏幕活动总结.
  For "start tracking": call cron tool with operation=add. For "summary": run activity-analyzer.py via bash.
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

## ⚠️ CRITICAL TOOL RULES — VIOLATION WILL CAUSE ERRORS

### FORBIDDEN (never do this):
- ❌ **NEVER call `skill_workshop`** — it will fail with "must not have additional properties" error
- ❌ **NEVER use `action`, `goal`, `taskName`, `status`, `reason`, `description` as top-level JSON keys**
- ❌ **NEVER say "I'll start tracking for you" without actually creating a cron job**
- ❌ **NEVER propose to create/register/install/setup anything via skill_workshop**

### ALLOWED (only these):

| User Intent | Tool | Example |
|-------------|------|---------|
| Start tracking | `cron` tool | `{"operation":"add","name":"...","command":"...","cron":"*/5 * * * *"}` |
| Stop tracking | `cron` tool | `{"operation":"edit","jobId":"...","enabled":false}` |
| List jobs | `cron` tool | `{"operation":"list"}` |
| Daily summary | **bash command** | `python3 <SKILL_DIR>/scripts/activity-analyzer.py summary` |
| Search history | **bash command** | `python3 <SKILL_DIR>/scripts/activity-analyzer.py search "keyword"` |
| One-shot screenshot | **bash command** | `bash <SKILL_DIR>/scripts/screen-activity.sh` |

---

## Start Tracking (when user says 开始追踪 / start tracking)

### Step 1: Read config
```bash
cat <SKILL_DIR>/config.json
```
Get `interval_minutes`. Default: 5 → `*/5 * * * *`

### Step 2: Call cron tool directly
```json
{
  "operation": "add",
  "name": "屏幕活动记录",
  "command": "sh -lc '/bin/bash /Users/zeject/ai/skills/mySkills/screen-activity-tracker/scripts/screen-activity.sh'",
  "cron": "*/5 * * * *",
  "timeoutSeconds": 90,
  "description": "每5分钟截图分析屏幕活动"
}
```
Replace `<SKILL_DIR>` with actual path. Adjust cron expression from config.json `interval_minutes`.

### Step 3: Report result
Tell user: ✅ Cron job created, job ID, interval, backend target.

## Stop / Edit Tracking

```json
// Stop: {"operation": "edit", "jobId": "<id>", "enabled": false}
// Resume: {"operation": "edit", "jobId": "<id>", "enabled": true}
// Change interval: {"operation": "edit", "jobId": "<id>", "cron": "*/10 * * * *"}
// List all: {"operation": "list"}
```

## Daily Summary & Search (use Bash tool)

```bash
# Today's summary
python3 <SKILL_DIR>/scripts/activity-analyzer.py summary

# Search history
python3 <SKILL_DIR>/scripts/activity-analyzer.py search "<query>"
```

## Configuration

Edit `<SKILL_DIR>/config.json`:

```json
{
  "backend": "siyuan",
  "siyuan": { "url": "http://192.168.1.100:6806", "token": "YOUR_TOKEN", "notebook_id": "YOUR_NOTEBOOK_ID" },
  "local": { "output_dir": "~/screen-activity", "keep_days": 7 },
  "mlx_url": "http://192.168.1.198:18000/v1",
  "analysis_mode": "simple",
  "interval_minutes": 5,
  "idle_detection": true,
  "app_blacklist": ["银行", "密码", "支付宝", "微信支付", "登录"]
}
```

## Quick Reference

| User says | Action | Tool |
|-----------|--------|------|
| "开始追踪屏幕" | Create cron job | `cron` |
| "停止追踪屏幕" | Disable cron job | `cron` |
| "今天屏幕活动总结"/"整理今天的操作" | Run summary script | **bash** |
| "搜索 X" | Run search script | **bash** |

## License

MIT
