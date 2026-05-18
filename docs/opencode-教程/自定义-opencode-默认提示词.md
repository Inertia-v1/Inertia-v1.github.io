# 自定义 OpenCode 默认提示词

OpenCode 默认提示词决定了 AI 助手的基本行为和风格。本文介绍两种自定义方式：完全覆盖和追加扩展。

## 方法一：通过 Webhook 查看并覆盖

此方法适合需要完全重写提示词的场景。

### 1. 获取 Webhook URL

打开 [webhook.site](https://webhook.site)，页面会自动生成一个唯一 URL，形如：

```
https://webhook.site/12345678-1234-1234-1234-123456789101
```

复制这个地址备用。

### 2. 修改 baseURL

编辑 opencode 配置文件，找到模型配置，将 `baseURL` 改为 webhook 地址：

```json
{
  "providers": {
    "openai": {
      "baseUrl": "https://webhook.site/你的唯一ID"
    }
  }
}
```

### 3. 捕获系统提示词

在 opencode 中发送任意消息，由于请求被重定向到 webhook，opencode 不会有正常输出。

回到 webhook.site 页面，可以看到完整的 JSON 请求体，其中 `messages` 数组的第一个元素即为系统提示词：

```json
{
  "messages": [
    {
      "role": "system",
      "content": "这里是完整的系统提示词..."
    }
  ]
}
```

复制并保存这个内容，作为自定义的基础。

### 4. 覆盖默认提示词

在配置文件中添加：

```json
"agent": {
  "build": {
    "prompt": "你的自定义提示词内容"
  }
}
```

`prompt` 字段支持两种形式：
- **直接文本**：填写完整的提示词内容
- **文件路径**：指向一个 `.md` 或 `.txt` 文件

```json
"agent": {
  "build": {
    "prompt": "C:\\Users\\用户名\\.config\\opencode\\custom-prompt.md"
  }
}
```

{{< callout type="warning" >}}
使用 `prompt` 会**完全覆盖**默认提示词，请确保包含必要的指令。
{{< /callout >}}

## 方法二：追加提示词

如果只想添加额外指令而不修改默认内容，使用 `instructions`：

```json
"instructions": [
  "C:\\Users\\用户名\\.config\\opencode\\AGENTS.md"
]
```

### 工作原理

`instructions` 中的内容会追加到默认提示词末尾，形成：

```
[默认提示词]

[你追加的内容]
```

### 优先级顺序

opencode 会按以下顺序拼接提示词：

1. 全局默认提示词
2. 全局 `instructions` 配置
3. 项目级 `.opencode/AGENTS.md`
4. 项目级 `instructions` 配置

{{< callout type="info" >}}
追加方式不会影响项目级自定义配置，适合添加通用规则。
{{< /callout >}}

## 配置文件位置

| 平台 | 全局配置 | 项目配置 |
|------|---------|---------|
| Windows | `%APPDATA%\opencode\opencode.json` | `.opencode/opencode.json` |
| macOS/Linux | `~/.config/opencode/opencode.json` | `.opencode/opencode.json` |

项目配置会覆盖全局配置中的同名项。

## 常见问题

### Q: 两种方法可以同时使用吗？

可以。`prompt` 负责覆盖核心内容，`instructions` 继续追加补充。

### Q: 如何恢复默认？

删除配置文件中的 `agent.build.prompt` 字段即可恢复默认提示词。

### Q: 提示词支持哪些格式？

支持纯文本、Markdown 格式。文件路径支持 `.md`、`.txt` 扩展名。

