---
name: hooktv-canvas
description: Use when the user wants to create images, videos or copy with HookTV, or work on their HookTV canvases (list, create, add nodes, connect, quote, run, fetch results, upload reference files). Requires the hooktv MCP server.
---

# HookTV 画布创作

通过 `hooktv` MCP 工具在用户自己的 HookTV 画布上工作。生成会扣用户的积分，结果会同步出现在 HookTV 网页的同一张画布上。

## 基本流程

1. 找画布：`list_canvases`，或 `create_canvas` 新建。改已有画布前先 `get_canvas` 看清现有节点和连线。
2. 选模型：每类生成节点先 `list_models`（`text_generator` / `image_generator` / `video_generator`），只用返回里的模型编码，参数只用返回的可选值。
3. 搭节点：`edit_canvas` 一次提交多个操作，按顺序执行，任一失败整批不生效：
   - `add_node`：自定义 `node_id`（字母数字、`_`、`-`）方便同一批里连线；
   - `connect`：把素材或上游生成节点连到生成节点，返回 `edge_id`，提示词里用 `{{ref:<edge_id>}}` 引用这路输入；
   - `update_node` / `remove_nodes` / `disconnect` 用于修改。
4. 估价并征得同意：`quote_run` 返回每个节点的积分与余额。**必须把总积分告诉用户，用户同意后才能调 `start_run`。** 余额不足时如实告知，不要反复重试。
5. 运行与取结果：`start_run` 返回运行 ID；每隔几秒 `get_run`，直到各节点 `status` 为终态。成功节点的 `original_asset_urls` 是原图/原视频地址，`texts` 是文本结果；失败节点看 `error_code`。

## 节点数据

| 节点类型 | data |
|---|---|
| `text` | `{"text": "..."}` |
| `image` / `video` / `audio` | `{"asset_key": "<对象键>"}`，只能用本人已上传或画布已有的素材 |
| `text_generator` / `image_generator` / `video_generator` | `{"model": "<模型编码>", "prompt": "...", "params": {...}, "num": 1}`，`params`、`num` 按 `list_models` 的可选值填 |

运行范围 `run_scope`：`node` 只跑目标节点；`upstream` 连同从未成功过的上游一起跑；`dag` 连同全部上游；`group` 跑分组；`canvas` 跑整张画布。`node`/`upstream`/`dag` 要填 `target_node_ids`。

## 上传本地素材

1. `create_upload`（`content_type`、`size` 字节数、可选 `filename`）拿到 `url`、`fields`、`key`；
2. 向 `url` 发 multipart POST：`fields` 原样作为表单字段，文件字段名 `file` 放在最后；
3. `confirm_upload(object_key=key)`；
4. 用 `key` 作为图片/视频/音频节点的 `asset_key`。

## 出错时

工具错误文本形如 `错误码: 说明`。常见：
- `error.canvas.points_insufficient`：积分不足，告诉用户去 HookTV 充值；
- `error.rate_limit.exceeded`：请求太频繁，稍等再试；
- `error.canvas.version_conflict`：画布刚被别处修改，`get_canvas` 重新看一遍再改；
- 连接返回 401：访问令牌无效或已吊销，请用户到 HookTV 重新创建并更新 `HOOKTV_TOKEN`。
