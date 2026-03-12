# NanoClaw 飞书频道

[飞书](https://www.feishu.cn/) 频道集成，为 [NanoClaw](https://github.com/qwibitai/nanoclaw) 提供 AI 助手消息收发能力。

## 功能特点

- **WebSocket 长连接** - 无需公网域名或服务器
- **群聊支持** - 将机器人添加到任意飞书群
- **私聊支持** - 用户可直接与机器人对话
- **多频道并行** - 可与 WhatsApp、Telegram、Slack 等同时运行

## 安装步骤

### 1. 添加远程仓库并合并

```bash
cd /path/to/nanoclaw

git remote add feishu https://github.com/qdongxu/nanoclaw-feishu.git
git fetch feishu main
git merge feishu/main
```

如果遇到 package-lock 冲突：

```bash
git checkout --theirs package-lock.json
git add package-lock.json
git merge --continue
```

### 2. 安装依赖并编译

```bash
npm install
npm run build
```

### 3. 创建飞书应用

1. 访问 [飞书开放平台](https://open.feishu.cn/app)
2. 点击「创建企业自建应用」
3. 填写应用名称和描述
4. 创建完成后，进入「凭证与基础信息」
5. 复制 **App ID** 和 **App Secret**

### 4. 配置应用

**启用机器人能力：**
1. 进入「应用功能」
2. 启用「机器人」

**配置事件订阅：**
1. 进入「事件与回调」→「事件配置」
2. 订阅方式选择：**使用长连接接收事件**
3. 添加事件：`im.message.receive_v1`

**添加权限：**
1. 进入「权限管理」
2. 搜索并添加：
   - `im:message` - 获取与读取消息
   - `im:message:send_as_bot` - 以应用身份发送消息

### 5. 设置环境变量

在 `.env` 文件中添加：

```bash
FEISHU_APP_ID=cli_xxxxxxxxxxxx
FEISHU_APP_SECRET=xxxxxxxxxxxxxxxxxx
```

如果使用 launchd（macOS），还需要在 `~/Library/LaunchAgents/com.nanoclaw.plist` 中添加：

```xml
<key>FEISHU_APP_ID</key>
<string>cli_xxxxxxxxxxxx</string>
<key>FEISHU_APP_SECRET</key>
<string>xxxxxxxxxxxxxxxxxx</string>
```

### 6. 同步环境变量并重启

```bash
cp .env data/env/env
launchctl unload ~/Library/LaunchAgents/com.nanoclaw.plist
launchctl load ~/Library/LaunchAgents/com.nanoclaw.plist
```

### 7. 获取 Chat ID 并注册

1. 创建或打开一个飞书群聊
2. 将机器人添加到群（群设置 → 添加机器人 → 选择你的应用）
3. 在群里发送一条消息
4. 查看日志获取 Chat ID：

```bash
tail -f logs/nanoclaw.log
```

5. 注册群聊：

```bash
# 主群（响应所有消息）
npx tsx setup/index.ts --step register -- \
  --jid "feishu:<chat-id>" \
  --name "feishu_main" \
  --folder "feishu_main" \
  --trigger "@Andy" \
  --channel feishu \
  --no-trigger-required \
  --is-main

# 或触发词群（仅响应触发词）
npx tsx setup/index.ts --step register -- \
  --jid "feishu:<chat-id>" \
  --name "my_group" \
  --folder "feishu_my_group" \
  --trigger "@Andy" \
  --channel feishu
```

### 8. 重启并测试

```bash
launchctl kickstart -k gui/$(id -u)/com.nanoclaw
```

在飞书群中发送消息，机器人应该会回复！

## 环境变量说明

| 变量 | 必填 | 说明 |
|------|------|------|
| `FEISHU_APP_ID` | 是 | 飞书应用 ID（以 `cli_` 开头） |
| `FEISHU_APP_SECRET` | 是 | 飞书应用密钥 |

## Chat ID 格式

- 群聊：`oc_xxxxxx`
- 私聊：`ou_xxxxxx`
- NanoClaw JID 格式：`feishu:<chat_id>`

## 已知限制

- **仅支持文本消息** - 暂不支持图片、文件、富文本
- **无输入状态提示** - 飞书 API 不提供此功能
- **话题扁平化** - 回复消息显示为普通消息
- **发送者名称** - 可能显示用户 ID 而非昵称

## 常见问题

### 机器人无响应

1. 检查 `.env` 和 `data/env/env` 中是否都设置了凭证
2. 确认群聊已注册：`sqlite3 store/messages.db "SELECT * FROM registered_groups WHERE jid LIKE 'feishu:%'"`
3. 查看日志：`tail -f logs/nanoclaw.log`
4. 确认 WebSocket 已连接（日志中应显示 "Feishu WebSocket connected"）

### WebSocket 连接失败

1. 确认 App ID 和 Secret 正确
2. 确认飞书开发者后台已启用「使用长连接接收事件」
3. 确认已订阅 `im.message.receive_v1` 事件
4. 确认已启用机器人能力

### 收不到消息

1. 确认机器人已添加到群
2. 检查权限是否包含 `im:message`
3. 私聊需要用户先发起对话

## 配合[智谱 GLM](https://www.bigmodel.cn/glm-coding?ic=GHWBRW2KW5) 或其他大模型使用

如果你使用[智谱 GLM](https://www.bigmodel.cn/glm-coding?ic=GHWBRW2KW5) 或其他大模型（非 Anthropic），需要设置 `ANTHROPIC_API_KEY`（不能只设置 `ANTHROPIC_AUTH_TOKEN`），以便凭证代理使用 `x-api-key` 请求头：

```bash
ANTHROPIC_API_KEY=your_api_key
ANTHROPIC_BASE_URL=https://open.bigmodel.cn/api/anthropic
```

## 许可证

MIT License - 详见 [LICENSE](LICENSE)

## 贡献

欢迎提交 Issue 和 Pull Request！
