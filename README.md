# HookTV Marketplace

HookTV 的 Agent 插件目录，只提供一个插件 `hooktv`，同时支持 Codex 与 Claude Code。插件本身不含本地服务，只把 Agent 连到 HookTV 的 Remote MCP：

| 安装标识 | 连接地址 | 版本 |
|---|---|---|
| `hooktv` | `https://YOUR_HOOKTV_DOMAIN/api/mcp` | `1.0.0` |

## 准备访问令牌

1. 登录 HookTV 网页，在访问令牌管理里创建一个令牌（只显示一次，立即复制）。
2. 设置环境变量：

```bash
# macOS / Linux
export HOOKTV_TOKEN="htv_pat_..."
```

```powershell
# Windows（对新开的终端生效）
setx HOOKTV_TOKEN "htv_pat_..."
```

## 安装

### Codex

```bash
codex plugin marketplace add https://github.com/huoke-ai/hooktv-mcp.git --ref main
codex plugin add hooktv@hooktv
```

装完重新加载插件或新建 Codex 任务。

### Claude Code

```text
/plugin marketplace add huoke-ai/hooktv-mcp
/plugin install hooktv@hooktv
```

### 其他 Agent

Cursor、Trae、CodeBuddy、Hermes、OpenClaw 等见 [docs/agents.md](docs/agents.md)。

## 随包技能

| 技能 | 用途 |
|---|---|
| `hooktv-canvas` | 告诉 Agent 怎么搭画布、选模型、先估价征得同意再生成、上传素材、处理常见错误 |

## 更新

```bash
codex plugin marketplace upgrade hooktv
codex plugin add hooktv@hooktv
```

Claude Code：`/plugin marketplace update hooktv`。

## 排查

- 连接返回 `401` 并带 `WWW-Authenticate: Bearer`：令牌缺失、无效、过期或已吊销，重新创建并更新 `HOOKTV_TOKEN`。
- 工具返回 `error.canvas.points_insufficient`：积分不足。
- 工具返回 `error.rate_limit.exceeded`：请求过于频繁，稍后再试。

## 发布前替换占位

仓库里的 `YOUR_HOOKTV_DOMAIN`（线上域名）与 `YOUR_GITHUB_ORG`（GitHub 账号或组织）是占位，发布前全部替换：

```bash
grep -rl "YOUR_HOOKTV_DOMAIN\|YOUR_GITHUB_ORG" .
```
