---
summary: "在外掛中撰寫 agent 工具（schemas、選用工具、允許清單）"
read_when:
  - You want to add a new agent tool in a plugin
  - 你需要透過允許清單讓工具成為選用
title: "外掛代理程式工具"
---

# 外掛代理程式工具

OpenClaw 外掛可以註冊 **代理程式工具**（JSON‑schema 函式），在代理程式執行期間向 LLM 曝露。工具可以是 **必要**（永遠可用）或 **選用**（需選擇加入）。 Tools can be **required** (always available) or
**optional** (opt‑in).

Agent 工具可在主設定中的 `tools` 下設定，或在各個 agent 之下設定
`agents.list[].tools`. The allowlist/denylist policy controls which tools the agent
can call.

## 基本工具

```ts
import { Type } from "@sinclair/typebox";

export default function (api) {
  api.registerTool({
    name: "my_tool",
    description: "Do a thing",
    parameters: Type.Object({
      input: Type.String(),
    }),
    async execute(_id, params) {
      return { content: [{ type: "text", text: params.input }] };
    },
  });
}
```

## 選用工具（需選擇加入）

選用工具 **永遠不會** 自動啟用。使用者必須將其加入代理程式的允許清單。 Users must add them to an agent
allowlist.

```ts
export default function (api) {
  api.registerTool(
    {
      name: "workflow_tool",
      description: "Run a local workflow",
      parameters: {
        type: "object",
        properties: {
          pipeline: { type: "string" },
        },
        required: ["pipeline"],
      },
      async execute(_id, params) {
        return { content: [{ type: "text", text: params.pipeline }] };
      },
    },
    { optional: true },
  );
}
```

在 `agents.list[].tools.allow`（或全域的 `tools.allow`）中啟用選用工具：

```json5
{
  agents: {
    list: [
      {
        id: "main",
        tools: {
          allow: [
            "workflow_tool", // specific tool name
            "workflow", // plugin id (enables all tools from that plugin)
            "group:plugins", // all plugin tools
          ],
        },
      },
    ],
  },
}
```

其他會影響工具可用性的設定旋鈕：

- 僅列出外掛工具的允許清單會被視為外掛的選用啟用；核心工具仍將
保持啟用，除非你也在允許清單中加入核心工具或工具群組。
- `tools.profile` / `agents.list[].tools.profile`（基礎允許清單）
- `tools.byProvider` / `agents.list[].tools.byProvider`（提供者特定的允許／拒絕）
- `tools.sandbox.tools.*`（在沙箱隔離時的沙箱工具政策）

## 規則＋提示

- 工具名稱 **不得** 與核心工具名稱衝突；發生衝突的工具會被略過。
- 允許清單中使用的外掛 ID 不得與核心工具名稱衝突。
- 對於會觸發副作用或需要額外二進位檔／認證的工具，優先使用 `optional: true`。
