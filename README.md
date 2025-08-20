# 百度网盘自动转存

> **声明**：本项目是使用 Cursor AI 辅助编写的。本人是一名剪辑师，非专业程序员，精力有限。如遇到使用问题，请先查阅文档并尝试自行解决，谢谢理解。

一个基于Flask的百度网盘自动转存系统，支持多用户管理、定时任务调度和通知推送。

## 主要特性

- 🔄 自动转存：支持自动转存百度网盘分享链接到指定目录
- 👥 多用户管理：支持添加多个百度网盘账号
- ⏰ 定时任务：支持全局定时和单任务定时规则
- 📱 消息推送：支持25+种通知方式和自定义WEBHOOK
- 🎯 任务分类：支持对任务进行分类管理
- 📊 状态监控：实时显示任务执行状态和进度
- 🔍 智能去重：自动跳过已转存的文件
- 💾 容量监控：监控网盘容量并在超过阈值时发送通知
- 📋 链接复制：支持一键复制分享链接到剪贴板
- 🤖 智能填充：自动获取分享链接的文件夹名称并填充任务名称
- 🔍 正则处理：支持文件过滤和重命名的正则表达式功能
- 🎨 美观界面：响应式设计，支持移动端访问

## 系统要求

- Python 3.10（baidupcs-py-0.7.6只支持3.10）
- Windows/Linux/MacOS

## 通信模式

项目支持两种前后端通信模式：

1. **轮询模式（默认）**：前端定期向后端发送请求获取最新状态，适用于所有环境，不需要额外的依赖。
2. **WebSocket模式**：使用WebSocket实时通信，需要安装`gevent-websocket`依赖。

默认情况下，项目使用轮询模式。如果需要启用WebSocket模式，请确保：
1. 在`requirements.txt`中取消对`gevent-websocket`的注释
2. 在`static/main.js`中将`WS_CONFIG.enabled`设置为`true`

## 安装说明

1. 克隆仓库：
```bash
git clone https://github.com/your-username/baidu-autosave.git
cd baidu-autosave
```

2. 安装依赖：
```bash
pip install -r requirements.txt
```

3. 运行应用：
```bash
python web_app.py
```

4. 访问Web界面：
```
http://localhost:5000
```

## Docker 部署

### 使用 docker-compose 部署（推荐）

1. 创建 `docker-compose.yml` 文件：
```yaml
version: '3'

services:
  baidu-autosave:
    image: kokojacket/baidu-autosave:latest
    container_name: baidu-autosave
    restart: unless-stopped
    ports:
      - "5000:5000"
    volumes:
      - ./config:/app/config
      - ./log:/app/log
    environment:
      - TZ=Asia/Shanghai
```

2. 创建必要目录：
```bash
mkdir -p config log
```

3. 启动服务：
```bash
docker-compose up -d
```

4. 查看日志：
```bash
docker-compose logs -f
```

5. 访问Web界面：
```
http://localhost:5000
```

> 默认登录账号：admin  
> 默认登录密码：admin123

### 使用 Docker CLI 部署

1. 创建必要目录：
```bash
mkdir -p config log
```

2. 启动容器：
```bash
docker run -d \
  --name baidu-autosave \
  --restart unless-stopped \
  -p 5000:5000 \
  -v $(pwd)/config:/app/config \
  -v $(pwd)/log:/app/log \
  -e TZ=Asia/Shanghai \
  kokojacket/baidu-autosave:latest
```

3. 查看日志：
```bash
docker logs -f baidu-autosave
```

4. 访问Web界面：
```
http://localhost:5000
```
> 默认登录账号：admin  
> 默认登录密码：admin123

### 目录结构说明

```
baidu-autosave/
├── config/                # 配置文件目录
│   ├── config.json       # 运行时配置文件（自动生成）
│   └── config.template.json  # 配置文件模板
├── log/                  # 日志目录
│   └── web_app_*.log    # 应用日志文件
├── static/               # 静态资源目录
│   ├── style.css        # 样式表
│   ├── main.js          # 主脚本文件
│   └── favicon/         # 网站图标资源
├── templates/            # 模板文件目录
│   ├── index.html       # 主页面模板
│   └── login.html       # 登录页面模板
├── docs/                 # 文档目录
├── Dockerfile           # Docker构建文件
├── docker-compose.yml   # Docker编排文件
├── requirements.txt     # 项目依赖
├── LICENSE              # 许可证文件
├── web_app.py           # Web应用主程序
├── storage.py           # 存储管理模块
├── scheduler.py         # 任务调度模块
├── utils.py             # 工具函数
└── notify.py            # 通知模块
```

### 主要模块说明

- **web_app.py**: Web应用核心，处理HTTP请求和WebSocket通信
- **storage.py**: 管理百度网盘API调用和数据存储
- **scheduler.py**: 处理定时任务的调度和执行
- **notify.py**: 实现各种通知方式
- **utils.py**: 提供通用工具函数

## 使用说明

### 1. 添加用户

1. 登录百度网盘网页版
2. 按F12打开开发者工具获取cookies
3. 在系统中添加用户，填入cookies

### 2. 添加任务

1. 点击"添加任务"按钮
2. 填写任务信息：
   - 任务名称（可选，支持自动获取分享链接的文件夹名称）
   - 分享链接（必填，输入完成后会自动获取文件夹名称）
   - 保存目录（必填，会根据任务名称智能同步）
   - 定时规则（可选）
   - 分类（可选）
   - 过滤表达式（可选，正则表达式过滤需要转存的文件）
   - 重命名表达式（可选，正则表达式重命名转存的文件）

**智能功能说明：**
- **自动填充任务名称**：输入分享链接后，系统会自动获取分享内容的文件夹名称并填充到任务名称
- **保存目录同步**：任务名称变化时会自动更新保存目录的最后一级文件夹名
- **编辑检测**：一旦手动编辑保存目录，当前任务将停止自动同步；新建任务时重新启用
- **分享链接复制**：任务创建后，可在任务列表中一键复制分享链接
- **正则文件过滤**：使用正则表达式筛选需要转存的文件，如 `^(\d+)\.mp4$` 只转存以数字开头的mp4文件
- **正则文件重命名**：使用正则表达式重命名转存的文件，如 `第\1集.mp4` 将匹配的数字替换为"第X集"格式

**⚠️ 重命名功能注意事项：**
百度网盘API对重命名操作有严格的频率限制，频繁重命名可能导致失败或触发风控。建议优先通过调整分享源文件名来满足需求，谨慎使用重命名功能。

### 3. 定时设置

- 全局定时规则：适用于所有未设置自定义定时的任务
- 单任务定时：可为每个任务设置独立的定时规则
- 定时规则使用cron表达式，例如：
  - `*/5 * * * *` : 每5分钟执行一次
  - `0 */1 * * *` : 每小时执行一次
  - `0 8,12,18 * * *` : 每天8点、12点、18点执行

### 4. 通知设置

系统支持25+种通知方式，包括但不限于：
- **PushPlus**: 访问 [pushplus.plus](http://www.pushplus.plus) 获取Token
- **Bark**: iOS平台推送服务
- **钉钉机器人**: 企业钉钉群组通知
- **飞书机器人**: 企业飞书群组通知
- **企业微信**: 企业微信应用和机器人推送
- **Telegram**: Telegram机器人推送
- **SMTP邮件**: 支持各种邮件服务商
- **自定义WEBHOOK**: 支持任意HTTP接口推送
- **其他**: Gotify、iGot、ServerJ、PushDeer等

**配置方法：**
1. 在系统设置的"通知设置"部分启用通知功能
2. 添加对应通知服务的字段配置（如`PUSH_PLUS_TOKEN`）
3. 对于WEBHOOK配置，支持快速添加和简化输入格式
4. 点击"测试通知"验证配置是否正确
5. 支持同时配置多种通知方式

**WEBHOOK配置说明：**
- 系统提供"添加WEBHOOK配置"按钮，可一键添加所有必需的WEBHOOK字段
- WEBHOOK_BODY字段支持简化输入格式：`title: "$title"content: "$content"source: "项目名"`
- 保存时系统会自动转换为标准多行格式，无需手动处理换行符和转义
- 支持自定义HTTP方法、请求头和请求体格式

**通知延迟合并功能：**
1. 当多个任务在短时间内执行完成时，系统会将通知合并为一条发送
2. 可在系统设置页面的"通知设置"部分设置通知延迟时间（默认30秒）
3. 延迟时间越长，越有可能将更多任务的通知合并在一起
4. 这有助于减少频繁的通知推送，提高用户体验

### 5. 网盘容量监控

系统支持自动监控网盘容量并在超过阈值时发送通知：
1. 在系统设置中启用"网盘容量提醒"
2. 设置容量提醒阈值（默认90%）
3. 设置检查时间（默认每天00:00）
4. 当网盘使用量超过设定阈值时，系统将通过已配置的通知渠道发送警告

## 配置文件说明

`config.json` 包含以下主要配置（示例）：

```json
{
    "baidu": {
        "users": {},          // 用户信息
        "current_user": "",   // 当前用户
        "tasks": []          // 任务列表
    },
    "retry": {
        "max_attempts": 3,    // 最大重试次数
        "delay_seconds": 5    // 重试间隔
    },
    "cron": {
        "default_schedule": [   // 默认定时规则，支持多个 cron 表达式
            "0 10 * * *"
        ],
        "auto_install": true    // 自动启动定时
    },
    "notify": {
        "enabled": true,      // 启用通知
        "notification_delay": 30, // 通知延迟合并时间（秒），设置为0禁用合并
        "direct_fields": {     // 直接映射到通知库的字段（推荐）
            "PUSH_PLUS_TOKEN": "",        // PushPlus Token
            "PUSH_PLUS_USER": "",         // PushPlus 群组编码（可选）
            "BARK_PUSH": "",              // Bark 推送地址或设备码
            "DD_BOT_TOKEN": "",           // 钉钉机器人 Token
            "DD_BOT_SECRET": "",          // 钉钉机器人 Secret
            "TG_BOT_TOKEN": "",           // Telegram 机器人 Token
            "TG_USER_ID": "",             // Telegram 用户 ID
            "SMTP_SERVER": "",            // SMTP 服务器地址
            "SMTP_EMAIL": "",             // SMTP 发件邮箱
            "SMTP_PASSWORD": "",          // SMTP 登录密码
            "WEBHOOK_URL": "",            // 自定义 Webhook 地址
            "WEBHOOK_METHOD": "POST",     // 自定义 Webhook 请求方法
            "WEBHOOK_CONTENT_TYPE": "application/json", // 请求内容类型
            "WEBHOOK_HEADERS": "Content-Type: application/json", // 请求头
            "WEBHOOK_BODY": "title: \"$title\"\ncontent: \"$content\"\nsource: \"项目名\"" // 请求体格式
        },
        "custom_fields": {     // 自定义字段，可通过界面动态添加
        }
        // 兼容旧格式：也支持 { "channels": { "pushplus": { "token": "", "topic": "" } } }
    },
    "quota_alert": {
        "enabled": true,      // 是否启用容量提醒
        "threshold_percent": 90, // 容量提醒阈值百分比
        "check_schedule": "0 0 * * *" // 检查时间（默认每天00:00）
    },
    "scheduler": {
        "max_workers": 1,     // 最大工作线程数
        "misfire_grace_time": 3600,  // 错过执行的容错时间
        "coalesce": true,     // 合并执行错过的任务
        "max_instances": 1    // 同一任务的最大并发实例数
    }
}
```

## 常见问题

1. **任务执行失败**
   - 检查分享链接是否有效
   - 确认账号登录状态
   - 查看错误日志了解详细原因

2. **定时任务不执行**
   - 确认定时规则格式正确
   - 检查系统时间是否准确
   - 查看调度器日志

3. **通知推送失败**
   - 验证通知服务的Token/配置是否正确（如PushPlus Token、Bark设备码等）
   - 使用"测试通知"功能验证配置
   - 检查网络连接和防火墙设置
   - 查看日志了解具体错误信息

4. **WEBHOOK配置问题**
   - 使用"添加WEBHOOK配置"按钮可自动添加所有必需字段
   - WEBHOOK_BODY支持简化格式输入，系统会自动转换
   - 确认目标服务器能正常接收JSON格式的POST请求
   - 检查WEBHOOK_URL地址是否可访问
   - 验证WEBHOOK_HEADERS配置是否符合目标服务器要求

5. **通知未合并**
   - 确认通知延迟时间设置合理（默认30秒）
   - 检查任务执行时间间隔是否超过了设定的延迟时间
   - 容量警告等重要通知不会被合并，会立即发送

6. **正则表达式使用问题**
   - 确认正则表达式语法正确，可使用在线正则测试工具验证
   - 过滤表达式用于筛选文件，重命名表达式用于文件重命名
   - 重命名功能可能因频率限制失败，建议谨慎使用
   - 示例：过滤 `\.mp4$` 只转存mp4文件，重命名 `(.+)\.mp4$` → `\1_renamed.mp4`

## 开发说明

### 主要模块说明

- **web_app.py**: Web应用核心，处理HTTP请求和WebSocket通信
- **storage.py**: 管理百度网盘API调用和数据存储
- **scheduler.py**: 处理定时任务的调度和执行
- **notify.py**: 实现各种通知方式
- **utils.py**: 提供通用工具函数

## 更新日志

### v1.1.2
- 修复正则表达式字段二次编辑保存失败的问题
- 修复定时任务运行时手动执行任务可能出现提取码错误的并发问题
- 优化分享链接失效时的错误信息显示，将复杂的API错误转换为用户友好的提示
- 改进错误处理机制，提升用户体验

### v1.1.1
- 新增WEBHOOK配置自动格式化功能，简化用户配置过程
- 添加"添加WEBHOOK配置"快速按钮，一键添加所有必需的WEBHOOK字段
- WEBHOOK_BODY字段支持简化输入格式，系统会自动转换为标准多行格式
- 优化前端WEBHOOK配置体验，移除复杂的格式化逻辑
- 后端自动处理WEBHOOK_BODY字段的格式转换，确保兼容性
- 改进通知配置的用户体验，降低配置难度

### v1.1.0
- 新增正则表达式功能，支持文件过滤和重命名处理
- 添加通知延迟合并功能，解决同一时间多个任务执行时发送多条通知的问题
- 优化通知发送逻辑，提高用户体验
- 添加通知延迟时间配置选项到系统设置页面，无需手动修改配置文件
- 默认延迟时间为30秒，可根据需要调整
- 添加分享链接一键复制功能，提升使用便利性
- 优化任务名称与保存目录单向同步逻辑，支持智能编辑检测
- 增强分享链接自动填充功能，每次输入链接时都会自动更新任务名称
- 完善通知功能集成，支持25+种通知方式和自定义WEBHOOK
- 添加重命名频率限制提醒，避免触发百度网盘API风控

### v1.0.9
- 添加网盘容量监控功能，支持在容量超过阈值时发送通知
- 优化前端界面交互体验
- 修复配置加载和保存的问题

### v1.0.2
- 修复任务编辑时提取码丢失的问题
- 优化任务更新逻辑，确保密码信息正确保存
- 改进任务调度更新机制

### v1.0.1
- 优化界面交互体验
- 修复已知问题
- 提升系统稳定性

### v1.0.0
- 初始版本发布
- 实现基本功能：任务管理、定时执行、通知推送

## 许可证

MIT License

## 致谢

- [Flask](https://flask.palletsprojects.com/)
- [APScheduler](https://apscheduler.readthedocs.io/)
- [baidupcs-py](https://github.com/PeterDing/BaiduPCS-Py)
- [quark-auto-save](https://github.com/Cp0204/quark-auto-save) - 夸克网盘自动转存项目，提供了很好的参考