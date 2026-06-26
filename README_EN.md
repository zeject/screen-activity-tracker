# Screen Activity Tracker

Automatic screen capture, AI vision analysis, and activity logging to local markdown or SiYuan Note.

## ✨ Features

- 📸 **Automatic Screenshots** - Periodically capture screen to track activities
- 🤖 **AI Vision Analysis** - Use multimodal models (e.g., Qwen3.5-9B) to analyze screenshots
- 📝 **Dual Backend** - Log to local Markdown files or SiYuan Note
- 🔒 **Privacy Protection** - App blacklist to skip sensitive applications
- ⏸️ **Idle Detection** - Automatically skip when user is away
- 📊 **Daily Summary** - Generate activity reports with one command
- 🔍 **History Search** - Search past activity records
- 🗑️ **Auto Cleanup** - Delete old screenshots after N days (default: 7)

## 📦 Installation

### Prerequisites

```bash
# Install peekaboo (screenshot tool)
npm install -g @steipete/peekaboo

# Verify Python 3 is installed
python3 --version
```

### Install Skill

```bash
# Method 1: Install from local path to OpenClaw
openclaw skills install /path/to/screen-activity-tracker

# Method 2: Copy to OpenClaw skills directory
cp -r /path/to/screen-activity-tracker ~/.openclaw/skills/
```

### Configure

```bash
cd /path/to/screen-activity-tracker
cp config.example.json config.json
python3 setup.py
```

The setup wizard will ask for:
- **Backend**: `local` (markdown files) or `siyuan` (SiYuan Note)
- **SiYuan config** (if using siyuan backend): URL, API Token, notebook ID
- **MLX model URL**: e.g., `http://192.168.1.198:18000/v1`
- **Analysis mode**: `simple` / `category` / `productivity` / `custom`
- **Interval**: minutes between screenshots (default: 5)
- **Retention days**: days to keep screenshots (0 = forever)
- **App blacklist**: keywords for sensitive apps

### Start Tracking

Say to OpenClaw:

```
开始追踪屏幕
```

OpenClaw will automatically:
- Check dependencies and configuration
- Create a cron job (scheduled task)
- Start capturing and logging

### Stop Tracking

Say to OpenClaw:

```
停止追踪屏幕
```

## 🎯 Usage

### Basic Commands

| Command | Description |
|---|---|
| `开始追踪屏幕` | Start periodic screenshot and logging |
| `停止追踪屏幕` | Pause tracking |
| `今天屏幕活动总结` | Generate today's activity report |
| `我上次用 Figma 是什么时候` | Search history |

### Analysis Modes

Set `analysis_mode` in `config.json`:

| Mode | Description | Example Output |
|---|---|---|
| `simple` (default) | Simple description | `[Safari] Browsing OpenClaw docs` |
| `category` | Activity classification | `[work] Coding in VS Code` |
| `productivity` | Productivity score | `[7] VS Code - implementing feature X` |
| `custom` | Custom prompt | Set `analysis_prompt` in config |

### Backend Configuration

#### Local Backend (Default)

Logs to local Markdown files:

```json
{
  "backend": "local",
  "local": {
    "output_dir": "~/screen-activity",
    "keep_days": 7
  }
}
```

File structure:
```
~/screen-activity/
├── 2026-06-26.md          # Today's activity log
├── 2026-06-27.md
└── screenshots/
    ├── 2026-06-26/
    │   ├── 143000.png
    │   └── 143500.png
    └── 2026-06-27/
        └── 090300.png
```

#### SiYuan Note Backend

Logs to SiYuan Note, automatically creates two documents daily:

- **Activity Log** - Timeline with screenshot links
- **Screenshot Archive** - Embedded screenshots with descriptions

```json
{
  "backend": "siyuan",
  "siyuan": {
    "url": "http://192.168.1.100:6806",
    "token": "YOUR_API_TOKEN",
    "notebook_id": "YOUR_NOTEBOOK_ID"
  }
}
```

## 🔧 Manual Usage (Without OpenClaw)

```bash
# Run one screenshot + analysis + log
bash /path/to/screen-activity-tracker/scripts/screen-activity.sh

# Generate daily summary
python3 /path/to/screen-activity-tracker/scripts/activity-analyzer.py summary

# Search history
python3 /path/to/screen-activity-tracker/scripts/activity-analyzer.py search "Figma"

# Analyze specific screenshot
python3 /path/to/screen-activity-tracker/scripts/activity-analyzer.py analyze /path/to/screenshot.png
```

## 🔒 Privacy Settings

### App Blacklist

Configure `app_blacklist` in `config.json`:

```json
{
  "app_blacklist": ["银行", "密码", "支付宝", "微信支付", "登录", "信用卡"]
}
```

Matched apps will:
- ❌ Not be screenshot
- ✅ Still logged (app name + time, no screenshot)

### Idle Detection

Automatically skip screenshots when user is idle (no keyboard/mouse input) for > 5 minutes.

Disable in `config.json`:
```json
{
  "idle_detection": false
}
```

## 📊 Daily Summary Example

Query: `今天屏幕活动总结`

Output:
```
📊 2026-06-26 Activity Summary

Total tracking time: 6 hours 30 minutes
Active time: 5 hours 10 minutes
Idle time: 1 hour 20 minutes

Top apps:
  - VS Code       2 hours 30 minutes  [work]
  - Safari        1 hour 45 minutes   [work]
  - Terminal      45 minutes          [work]
  - WeChat        30 minutes          [communication]

Activity categories:
  - Work: 4 hours 20 minutes (84%)
  - Communication: 30 minutes (9%)
  - Entertainment: 20 minutes (7%)

Productivity score: 7.5/10
```

## 🔍 Search History

Query: `我上周三在做什么？`

Output:
```
🔍 Search results: 2026-06-19 (Wednesday)

09:30 [VS Code] Implementing user login
10:15 [Safari] Reading OAuth2 docs
11:00 [Terminal] Running unit tests
...
```

Query: `上次用 Figma 是什么时候？`

Output:
```
🔍 Search results: Figma

Last used: 2026-06-25 14:30
  [Figma] Designing new homepage prototype

History:
  - 2026-06-25 14:30
  - 2026-06-20 10:15
  - 2026-06-18 16:45
```

## ⚙️ Configuration Reference

Complete `config.json`:

```json
{
  "backend": "local",
  "local": {
    "output_dir": "~/screen-activity",
    "keep_days": 7
  },
  "siyuan": {
    "url": "http://127.0.0.1:6806",
    "token": "YOUR_API_TOKEN",
    "notebook_id": "YOUR_NOTEBOOK_ID"
  },
  "mlx_url": "http://192.168.1.198:18000/v1",
  "analysis_mode": "simple",
  "analysis_prompt": "",
  "interval_minutes": 5,
  "idle_detection": true,
  "app_blacklist": [
    "银行",
    "密码",
    "支付宝",
    "微信支付",
    "登录"
  ]
}
```

| Field | Description | Default |
|---|---|---|
| `backend` | Backend type: `local` or `siyuan` | `local` |
| `local.output_dir` | Local backend output directory | `~/screen-activity` |
| `local.keep_days` | Days to keep screenshots (0 = forever) | `7` |
| `siyuan.url` | SiYuan Note URL | - |
| `siyuan.token` | SiYuan Note API Token | - |
| `siyuan.notebook_id` | SiYuan notebook ID | - |
| `mlx_url` | MLX model service URL | - |
| `analysis_mode` | Analysis mode | `simple` |
| `analysis_prompt` | Custom analysis prompt (when mode=custom) | - |
| `interval_minutes` | Screenshot interval (minutes) | `5` |
| `idle_detection` | Enable idle detection | `true` |
| `app_blacklist` | App blacklist keywords | See above |

## 🐛 Troubleshooting

### Screenshot fails

**Issue**: `Screenshot failed`

**Solution**:
1. Open "System Settings" > "Privacy & Security" > "Screen Recording"
2. Enable the app running OpenClaw (Terminal or OpenClaw)
3. Restart the app to apply permissions

### VL analysis fails

**Issue**: `VL analysis failed` or timeout

**Solution**:
1. Check MLX service is running: `curl http://YOUR_MLX_URL/v1/models`
2. Verify model supports image input (multimodal model)
3. Check network connection and firewall settings

### Cron job not running

**Issue**: Scheduled task not executing

**Solution**:
1. Check job status: `openclaw cron list`
2. View job logs: `openclaw cron runs --id <JOB_ID>`
3. If disabled, enable it: `openclaw cron edit <JOB_ID> --enable`
4. May need approval in OpenClaw Dashboard

### SiYuan Note write fails

**Issue**: `SiYuan write failed`

**Solution**:
1. Check SiYuan is running: `curl http://YOUR_SIYUAN_URL/api/system/version`
2. Verify API Token (in SiYuan "Settings" > "API")
3. Verify notebook ID (in SiYuan URL)
4. Check token has write permission

### peekaboo command not found

**Issue**: `peekaboo: command not found`

**Solution**:
```bash
# Verify installation
npm list -g @steipete/peekaboo

# If installed but not found, check PATH
echo $PATH

# Manually add to PATH (add to ~/.zshrc)
export PATH="$HOME/.npm-global/bin:$PATH"
```

## 📝 Project Structure

```
screen-activity-tracker/
├── SKILL.md                      # OpenClaw skill definition (for AI model)
├── README.md                     # English documentation
├── README_CN.md                  # Chinese documentation
├── config.example.json           # Configuration template
├── setup.py                      # Interactive setup wizard
└── scripts/
    ├── screen-activity.sh       # Main script (screenshot + analysis + log)
    ├── siyuan-logger.py        # Dual backend logger (local + SiYuan)
    └── activity-analyzer.py     # Daily summary + history search
```

## 📄 License

MIT License

## 🙏 Acknowledgments

- [OpenClaw](https://openclaw.ai) - AI assistant framework
- [Peekaboo](https://github.com/steipete/peekaboo) - macOS screenshot tool
- [SiYuan Note](https://b3log.org/siyuan/) - Local-first note-taking app

## 📧 Support

- Report issues on GitHub
- Discuss on OpenClaw Discord

---

**Note**: This skill requires macOS and OpenClaw. Windows/Linux support is not available due to peekaboo dependency.
