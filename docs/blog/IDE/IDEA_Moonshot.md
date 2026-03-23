# 手把手教你在IDEA中使用Moonshot API Key接入AI

## 📌 写在前面

本文旨在帮助你在 IntelliJ IDEA 中通过 Moonshot API（按量付费）接入 AI 辅助编程，无需订阅 Kimi Code 会员，按实际使用 token 付费即可。Moonshot API 完全兼容 OpenAI 格式，因此支持自定义 API 端点的插件都可以接入。

---

## 🎯 方案选择

目前主流的接入方式有两种：

| 方案 | 优点 | 缺点 | 适用场景 |
|------|------|------|----------|
| **ACP + Kimi Code CLI** | 支持 MCP 工具、官方支持 | 配置稍复杂 | 需要 MCP 工具的重度用户 |
| **Continue 插件** | 配置简单、功能完整 | 不支持 MCP 工具 | 日常编程、代码补全 |

本文主要介绍 **Continue 插件方案**，配置最简单，成功率最高。

---

## 🚀 方案一：Continue 插件（推荐）

### 第一步：获取 Moonshot API Key

1. 访问 [Moonshot AI 开放平台](https://platform.moonshot.cn)
2. 注册/登录账号
3. 进入 **API Keys** 页面，点击 **「创建新密钥」**
4. 复制生成的密钥（格式：`sk-...`），**注意关闭页面后不再显示，请妥善保存**

> 新用户注册后通常有 15 元体验额度，约 125 万 tokens（8k 模型）。

### 第二步：安装 Continue 插件

1. 打开 IDEA，进入 `File` → `Settings` → `Plugins`
2. 搜索 **Continue**，点击 **Install**
3. 安装完成后，重启 IDEA

> **如果搜索不到 Continue**：由于网络原因，可能无法访问国外插件市场。可以尝试：
> - 在 IDEA 中设置代理：`Settings` → `Appearance & Behavior` → `System Settings` → `HTTP Proxy`
> - 或者手动从 [Continue GitHub Releases](https://github.com/continuedev/continue/releases) 下载插件包，通过 `Install Plugin from Disk` 安装

### 第三步：配置 Continue

配置文件位置：`%USERPROFILE%\.continue\config.json`（Windows）或 `~/.continue/config.json`（macOS/Linux）

如果文件不存在，手动创建并写入以下内容：

```json
{
  "models": [
    {
      "title": "Moonshot Kimi",
      "provider": "openai",
      "model": "kimi-k2-turbo-preview",
      "apiBase": "https://api.moonshot.cn/v1",
      "apiKey": "sk-你的Moonshot-API-Key"
    }
  ],
  "tabAutocompleteModel": {
    "title": "Moonshot Kimi",
    "provider": "openai",
    "model": "kimi-k2-turbo-preview",
    "apiBase": "https://api.moonshot.cn/v1",
    "apiKey": "sk-你的Moonshot-API-Key"
  }
}
```

**配置说明**：

| 字段 | 说明 |
|------|------|
| `title` | 显示名称，可自定义 |
| `provider` | 固定为 `openai`（Moonshot 兼容 OpenAI 格式） |
| `model` | 模型名称，可选值见下表 |
| `apiBase` | Moonshot API 地址 |
| `apiKey` | 你的 Moonshot API Key |

**可用的模型**：

| 模型名称 | 特点 | 适用场景 |
|----------|------|----------|
| `kimi-k2-turbo-preview` | 高速版，响应快 | 日常编程、代码补全 |
| `kimi-k2-thinking-turbo` | 推理增强版 | 复杂逻辑分析、代码调试 |
| `moonshot-v1-128k` | 超长上下文 | 处理大文件、长文档 |

### 第四步：使用 Continue

1. 重启 IDEA（配置保存后建议重启）
2. 在 IDEA 右下角找到 Continue 图标（或通过 `View` → `Tool Windows` → `Continue`）
3. 点击图标，在对话框中输入问题即可开始使用

---

## 🔧 方案二：ACP + Kimi Code CLI

如果你需要 MCP 工具支持（如数据库查询、文件操作等），可以使用此方案。

### 第一步：安装 Kimi Code CLI

在 PowerShell（Windows）或终端（macOS/Linux）中执行：

```bash
# Windows
Invoke-RestMethod https://code.kimi.com/install.ps1 | Invoke-Expression

# macOS/Linux
curl -fsSL https://code.kimi.com/install.sh | sh
```

或者通过 Python 包管理器安装：

```bash
uv tool install kimi-cli
```

### 第二步：配置 Kimi Code CLI

在终端中执行 `kimi`，进入交互式界面。根据提示选择平台：

```
Select a platform:
> 1. Kimi Code
> 2. Moonshot AI Open Platform (moonshot.cn)
> 3. Moonshot AI Open Platform (moonshot.ai)
```

选择 `2. Moonshot AI Open Platform (moonshot.cn)`，然后输入你的 Moonshot API Key。

配置完成后输入 `/exit` 退出。

### 第三步：配置 IDEA ACP

配置文件位置：`%USERPROFILE%\.jetbrains\acp.json`（Windows）或 `~/.jetbrains/acp.json`（macOS/Linux）

创建或编辑该文件，写入以下内容：

```json
{
  "default_mcp_settings": {
    "use_idea_mcp": true
  },
  "agent_servers": {
    "Kimi Code CLI": {
      "command": "kimi",
      "args": ["acp"]
    }
  }
}
```

> **注意**：如果 `kimi` 命令不在系统 PATH 中，请使用完整路径，例如：
> ```json
> "command": "C:\\Users\\你的用户名\\.local\\bin\\kimi.exe"
> ```

### 第四步：启用 Registry 设置

如果你没有 JetBrains AI 订阅，需要启用一个注册表项：

1. 在 IDEA 中**连按两次 Shift** 键
2. 输入 `registry` 并选择 **Registry**
3. 搜索 `llm.enable.mock.response`
4. **勾选启用**
5. 点击 **Close**，重启 IDEA

### 第五步：使用

重启 IDEA 后，打开 AI Chat 工具窗口，在 Agent 下拉菜单中选择 **Kimi Code CLI**，即可开始使用。

---

## ❗ 常见问题

### 1. Continue 插件安装失败或搜索不到

**原因**：IDEA 插件市场在国外，国内网络可能无法访问。

**解决方案**：
- 设置代理：`Settings` → `Appearance & Behavior` → `System Settings` → `HTTP Proxy`
- 或手动下载插件包：访问 [Continue Releases](https://github.com/continuedev/continue/releases)，下载最新的 `.zip` 文件，通过 `Install Plugin from Disk` 安装

### 2. Kimi Code CLI 在 ACP 模式下一直要求登录

**原因**：Kimi Code CLI 的 ACP 模式强制要求 OAuth 认证（Kimi Code 会员），无法直接使用 Moonshot API Key 绕过。这是官方设计，目前无解。

**解决方案**：
- 切换到 **Continue 插件方案**（推荐）
- 或订阅 Kimi Code 会员，使用 `sk-kimi-xxx` 格式的 API Key

### 3. 报错 `No such option: -config-file`

**原因**：`acp.json` 中的 `args` 参数格式错误，或使用了不存在的选项。

**解决方案**：使用最简配置，去掉 `--config-file` 等额外参数：

```json
{
  "args": ["acp"]
}
```

### 4. API Key 无效或返回 401

**原因**：API Key 错误、过期，或使用了错误类型的 Key。

**解决方案**：
- 确认 Key 来自 [platform.moonshot.cn](https://platform.moonshot.cn)，格式为 `sk-...`
- 在终端测试 Key 是否有效：
  ```bash
  curl https://api.moonshot.cn/v1/chat/completions \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer sk-你的Key" \
    -d '{"model":"kimi-k2-turbo-preview","messages":[{"role":"user","content":"hi"}]}'
  ```

### 5. 环境变量不生效

**原因**：ACP 模式读取的是系统环境变量，而不是 `acp.json` 中的 `env` 字段。

**解决方案**：在 Windows 系统设置中永久添加环境变量：
- `ANTHROPIC_BASE_URL` = `https://api.moonshot.cn/v1`
- `ANTHROPIC_API_KEY` = `sk-你的Key`
- 然后**重启电脑**

### 6. 模型响应慢或超时

**原因**：网络问题或模型选择不当。

**解决方案**：
- 使用高速版模型：`kimi-k2-turbo-preview`
- 检查网络连通性，必要时配置代理

---

## 📊 两种方案对比

| 维度 | Continue 插件 | ACP + Kimi Code CLI |
|------|--------------|---------------------|
| 配置复杂度 | ⭐ 简单 | ⭐⭐⭐ 中等 |
| 需要额外安装 | ❌ 否 | ✅ 是（Kimi CLI） |
| 代码自动补全 | ✅ 支持 | ❌ 不支持 |
| MCP 工具支持 | ❌ 不支持 | ✅ 支持 |
| 认证方式 | API Key | 需要 OAuth（Kimi Code 会员） |
| 推荐度 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐（仅限需要 MCP 的用户） |

---

## 💡 总结

- **日常使用**：推荐 **Continue 插件方案**，配置简单，功能完整，支持代码补全
- **需要 MCP 工具**：可以尝试 **ACP + Kimi Code CLI**，但需要注意 ACP 模式可能需要 Kimi Code 会员订阅
- **无论哪种方案**，都需要先从 [Moonshot 开放平台](https://platform.moonshot.cn) 获取 API Key

---

## 📚 参考资料

- [Moonshot AI 开放平台](https://platform.moonshot.cn)
- [Continue 插件文档](https://docs.continue.dev)
- [Kimi Code CLI 文档](https://code.kimi.com/docs)

---

*本文档基于实际配置经验整理，如有问题欢迎交流讨论。*