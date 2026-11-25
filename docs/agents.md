# 各 Agent 接入 HookTV MCP

所有 Agent 连同一个地址，用同一种鉴权：

- 地址：`https://YOUR_HOOKTV_DOMAIN/api/mcp`（Streamable HTTP）
- 请求头：`Authorization: Bearer <访问令牌>`，令牌在 HookTV 网页创建，格式 `htv_pat_...`

建议把令牌放在环境变量 `HOOKTV_TOKEN` 里，不要写进会提交到 Git 的配置文件。

> 标“已核实”的写法来自该工具的官方文档；标“待核实”的请以该工具当前版本文档为准，确认后把标记去掉。

## Codex（插件格式参照 LibTV 公开插件；插件内 `bearer_token_env_var` 待实测）

```bash
codex plugin marketplace add https://github.com/huoke-ai/hooktv-mcp.git --ref main
codex plugin add hooktv@hooktv
```

或不装插件，直接写 `~/.codex/config.toml`：

```toml
[mcp_servers.hooktv]
url = "https://YOUR_HOOKTV_DOMAIN/api/mcp"
bearer_token_env_var = "HOOKTV_TOKEN"
```

## Claude Code（插件，已核实）

```text
/plugin marketplace add huoke-ai/hooktv-mcp
/plugin install hooktv@hooktv
```

或直接添加 MCP：

```bash
claude mcp add --transport http hooktv https://YOUR_HOOKTV_DOMAIN/api/mcp --header "Authorization: Bearer $HOOKTV_TOKEN"
```

## Cursor（待核实）

`~/.cursor/mcp.json`：

```json
{
  "mcpServers": {
    "hooktv": {
      "url": "https://YOUR_HOOKTV_DOMAIN/api/mcp",
      "headers": { "Authorization": "Bearer ${env:HOOKTV_TOKEN}" }
    }
  }
}
```

## Trae（待核实）

在 MCP 设置里手动添加，选择 HTTP / Streamable HTTP 类型，填地址与请求头。若不支持环境变量引用，直接粘贴令牌，并注意不要把该配置提交到仓库。

```json
{
  "mcpServers": {
    "hooktv": {
      "url": "https://YOUR_HOOKTV_DOMAIN/api/mcp",
      "headers": { "Authorization": "Bearer htv_pat_xxx" }
    }
  }
}
```

## CodeBuddy（待核实）

```json
{
  "mcpServers": {
    "hooktv": {
      "type": "http",
      "url": "https://YOUR_HOOKTV_DOMAIN/api/mcp",
      "headers": { "Authorization": "Bearer htv_pat_xxx" }
    }
  }
}
```

## Hermes Agent（待核实）

`~/.hermes/config.yaml`：

```yaml
mcp_servers:
  hooktv:
    url: "https://YOUR_HOOKTV_DOMAIN/api/mcp"
    headers:
      Authorization: "Bearer htv_pat_xxx"
```

## OpenClaw（待核实）

在 OpenClaw 的 MCP 服务器配置中添加一个远程（HTTP）服务器，地址与请求头同上。具体配置文件位置与字段名请以 OpenClaw 当前文档为准。

## 让 Agent 自己装

把下面这段发给支持命令行的 Agent（以 Codex 为例）：

```text
帮我安装 HookTV 插件，让我可以在这里用 HookTV 创作。
1. 执行 `codex plugin marketplace add https://github.com/huoke-ai/hooktv-mcp.git --ref main`
2. 执行 `codex plugin add hooktv@hooktv`
3. 提醒我把 HookTV 访问令牌设置到环境变量 HOOKTV_TOKEN，然后新建任务。
就绪后告诉我，可以开始创作了。
```
