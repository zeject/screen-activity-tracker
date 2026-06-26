# Screen Activity Tracker

屏幕活动追踪器 - 自动截图、AI 分析、记录到本地或思源笔记

[English Documentation](README_EN.md)

## ✨ 功能特点

- 📸 **自动截图** - 定时捕捉屏幕，了解自己在做什么
- 🤖 **AI 视觉分析** - 使用多模态模型（如 Qwen3.5-9B）自动分析截图内容
- 📝 **双后端支持** - 可选择记录到本地 Markdown 或思源笔记
- 🔒 **隐私保护** - 应用黑名单，自动跳过敏感应用
- ⏸️ **闲置检测** - 锁屏或离开时自动跳过，节省存储
- 📊 **每日总结** - 一键生成当天活动报告
- 🔍 **历史搜索** - 搜索过去的活动记录
- 🗑️ **自动清理** - 截图保留 N 天后自动删除（默认 7 天）

## 📦 安装步骤

### 1. 安装依赖

```bash
# 安装 peekaboo（截图工具）
npm install -g @steipete/peekaboo

# 确认 Python 3 已安装
python3 --version
```

### 2. 安装 Skill

```bash
# 方法一：从本地路径安装到 OpenClaw
openclaw skills install /path/to/screen-activity-tracker

# 方法二：直接复制到 OpenClaw skills 目录
cp -r /path/to/screen-activity-tracker ~/.openclaw/skills/
```

### 3. 配置

```bash
cd /path/to/screen-activity-tracker
cp config.example.json config.json
python3 setup.py
```

配置向导会询问：
- **后端选择**：`local`（本地 Markdown）或 `siyuan`（思源笔记）
- **思源配置**（如果选择 siyuan 后端）：URL、API Token、笔记本 ID
- **MLX 模型地址**：如 `http://192.168.1.198:18000/v1`
- **分析模式**：`simple` / `category` / `productivity` / `custom`
- **截图间隔**：分钟数（默认 5）
- **截图保留天数**：0 = 永久保留
- **应用黑名单**：敏感应用关键词

### 4. 开始追踪

在 OpenClaw 中说：

```
开始追踪屏幕
```

OpenClaw 会自动：
- 检查依赖和配置
- 创建定时任务（cron job）
- 开始截图和记录

### 5. 停止追踪

在 OpenClaw 中说：

```
停止追踪屏幕
```

## 🎯 使用指南

### 基本命令

| 命令 | 说明 |
|---|---|
| `开始追踪屏幕` | 启动定时截图和记录 |
| `停止追踪屏幕` | 暂停追踪 |
| `今天屏幕活动总结` | 生成当天活动报告 |
| `我上次用 Figma 是什么时候` | 搜索历史记录 |

### 分析模式

在 `config.json` 中设置 `analysis_mode`：

| 模式 | 说明 | 示例输出 |
|---|---|---|
| `simple` | 简单描述（默认） | `[Safari] 浏览 OpenClaw 文档` |
| `category` | 活动分类 | `[work] 在 VS Code 中编码` |
| `productivity` | 生产力评分 | `[7] VS Code - 实现新功能` |
| `custom` | 自定义提示词 | 在 `analysis_prompt` 中设置 |

### 后端配置

#### 本地后端（默认）

记录到本地 Markdown 文件：

```json
{
  "backend": "local",
  "local": {
    "output_dir": "~/screen-activity",
    "keep_days": 7
  }
}
```

文件结构：
```
~/screen-activity/
├── 2026-06-26.md          # 今日活动记录
├── 2026-06-27.md
└── screenshots/
    ├── 2026-06-26/
    │   ├── 143000.png
    │   └── 143500.png
    └── 2026-06-27/
        └── 090300.png
```

#### 思源笔记后端

记录到思源笔记，每日自动创建两个文档：

- **操作记录** - 时间线列表，含截图链接
- **截图存档** - 完整截图嵌入，可点击查看

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

## 🔧 手动运行（不通过 OpenClaw）

```bash
# 运行一次截图+分析+记录
bash /path/to/screen-activity-tracker/scripts/screen-activity.sh

# 生成每日总结
python3 /path/to/screen-activity-tracker/scripts/activity-analyzer.py summary

# 搜索历史
python3 /path/to/screen-activity-tracker/scripts/activity-analyzer.py search "Figma"

# 分析指定截图
python3 /path/to/screen-activity-tracker/scripts/activity-analyzer.py analyze /path/to/screenshot.png
```

## 🔒 隐私设置

### 应用黑名单

在 `config.json` 中配置 `app_blacklist`：

```json
{
  "app_blacklist": ["银行", "密码", "支付宝", "微信支付", "登录", "信用卡"]
}
```

匹配黑名单的应用会：
- ❌ 不截图
- ✅ 仍记录应用名称和时间（无截图）

### 闲置检测

当检测到用户闲置（无键盘/鼠标输入）超过 5 分钟时，自动跳过截图。

可在 `config.json` 中禁用：
```json
{
  "idle_detection": false
}
```

## 📊 每日总结示例

问：`今天屏幕活动总结`

答：
```
📊 2026-06-26 活动总结

总追踪时间：6 小时 30 分钟
有效活动：5 小时 10 分钟
闲置时间：1 小时 20 分钟

主要应用：
  - VS Code       2 小时 30 分钟  [work]
  - Safari        1 小时 45 分钟  [work]
  - Terminal      45 分钟         [work]
  - 微信          30 分钟         [communication]

活动分类：
  - 工作：4 小时 20 分钟 (84%)
  - 沟通：30 分钟 (9%)
  - 娱乐：20 分钟 (7%)

生产力评分：7.5/10
```

## 🔍 搜索历史

问：`我上周三在做什么？`

答：
```
🔍 搜索结果：2026-06-19 (周三)

09:30 [VS Code] 实现用户登录功能
10:15 [Safari] 查阅 OAuth2 文档
11:00 [Terminal] 运行单元测试
...
```

问：`上次用 Figma 是什么时候？`

答：
```
🔍 搜索结果：Figma

最近一次：2026-06-25 14:30
  [Figma] 设计新版首页原型

历史记录：
  - 2026-06-25 14:30
  - 2026-06-20 10:15
  - 2026-06-18 16:45
```

## ⚙️ 配置文件说明

`config.json` 完整配置：

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

| 字段 | 说明 | 默认值 |
|---|---|---|
| `backend` | 后端类型：`local` 或 `siyuan` | `local` |
| `local.output_dir` | 本地后端输出目录 | `~/screen-activity` |
| `local.keep_days` | 截图保留天数（0 = 永久） | `7` |
| `siyuan.url` | 思源笔记地址 | - |
| `siyuan.token` | 思源笔记 API Token | - |
| `siyuan.notebook_id` | 思源笔记本 ID | - |
| `mlx_url` | MLX 模型服务地址 | - |
| `analysis_mode` | 分析模式 | `simple` |
| `analysis_prompt` | 自定义分析提示词（mode=custom 时） | - |
| `interval_minutes` | 截图间隔（分钟） | `5` |
| `idle_detection` | 是否启用闲置检测 | `true` |
| `app_blacklist` | 应用黑名单关键词列表 | 见上方 |

## 🐛 故障排除

### 截图失败

**问题**：`Screenshot failed`

**解决**：
1. 打开"系统设置" > "隐私与安全性" > "屏幕录制"
2. 确保运行 OpenClaw 的应用（Terminal 或 OpenClaw）已勾选
3. 重启应用使权限生效

### VL 分析失败

**问题**：`VL analysis failed` 或超时

**解决**：
1. 检查 MLX 服务是否运行：`curl http://YOUR_MLX_URL/v1/models`
2. 确认模型支持图片输入（多模态模型）
3. 检查网络连接和防火墙设置

### Cron 任务未运行

**问题**：定时任务没有执行

**解决**：
1. 检查任务状态：`openclaw cron list`
2. 查看任务日志：`openclaw cron runs --id <任务ID>`
3. 如果任务被禁用，启用它：`openclaw cron edit <任务ID> --enable`
4. 可能需要在 OpenClaw Dashboard 中批准任务

### 思源笔记写入失败

**问题**：`SiYuan write failed`

**解决**：
1. 检查思源是否运行：`curl http://YOUR_SIYUAN_URL/api/system/version`
2. 确认 API Token 正确（在思源"设置" > "API" 中查看）
3. 确认笔记本 ID 正确（在思源 URL 中查看）
4. 检查 Token 有写入权限

### peekaboo 命令找不到

**问题**：`peekaboo: command not found`

**解决**：
```bash
# 确认安装
npm list -g @steipete/peekaboo

# 如果已安装但找不到，检查 PATH
echo $PATH

# 手动添加到 PATH（添加到 ~/.zshrc）
export PATH="$HOME/.npm-global/bin:$PATH"
```

## 📝 开发笔记

### 项目结构

```
screen-activity-tracker/
├── SKILL.md                      # OpenClaw skill 定义（给模型读）
├── README.md                     # 中文用户说明文档
├── README_EN.md                  # 英文用户说明文档
├── config.example.json           # 配置模板
├── setup.py                      # 交互式配置向导
└── scripts/
    ├── screen-activity.sh       # 主脚本（截图+分析+写入）
    ├── siyuan-logger.py        # 双后端写入（本地+思源）
    └── activity-analyzer.py     # 每日汇总+搜索历史
```

### 技术栈

- **截图**：peekaboo（macOS 截图 API）
- **视觉分析**：OpenAI 兼容 API（MLX-LM / Qwen3.5-9B）
- **后端**：本地 Markdown / 思源笔记 API
- **定时任务**：OpenClaw Cron

## 📄 许可证

MIT License

## 🙏 致谢

- [OpenClaw](https://openclaw.ai) - AI 助手框架
- [Peekaboo](https://github.com/steipete/peekaboo) - macOS 截图工具
- [SiYuan Note](https://b3log.org/siyuan/) - 本地优先的笔记软件

---

**问题反馈**：在 GitHub Issues 中提交问题或建议

**注意**：此 skill 需要 macOS 和 OpenClaw。由于 peekaboo 依赖，暂不支持 Windows/Linux。
