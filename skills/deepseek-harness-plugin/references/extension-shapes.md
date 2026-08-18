# Extension Shapes

Use the current [extension cookbook](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/cookbook/extension-cookbook.md), [architecture guide](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/architecture.md), and owning subsystem docs as the API authority.

## Choose The Mechanism

| Goal | Mechanism |
| --- | --- |
| Add a model-facing capability | Register a tool on `ctx.tools`. |
| Intercept or authorize a tool | `tools/pre-execute` or `ctx.tools.guard()` for a final monotonic denial. |
| Wrap tool execution | `tools/execute` for timeout, retry, or metrics. |
| Transform tool output | `tools/post-execute`. |
| Observe final tool results | `tools/result`. |
| Add a model provider | Register an `LlmAdapter` on `ctx.llm`. |
| Observe or intercept agent work | An owning `agent/*` event. |
| Add durable session facts | Extend the session event map and render from the log. |
| Add live UI behavior | Listen to `session/event` and use `ctx.agents`. |
| Add a Web Client Chat row | Register a `ConversationNodeDefinition` and keyed renderer. |
| Add a plugin settings card | Register a Host settings namespace and a client slot contribution. |
| Bridge an external client | Build a protocol driver through `ctx.agents`. |
| Add an independent reusable capability | Provide a Cordis service and document its contract. |

Do not change the agent loop when an existing seam expresses the behavior. If no seam fits, inspect the current upstream architecture and document why a new service or event is required.

## Tools

The minimal tool pattern is:

```ts
import { defineTool } from '@deepseek-ai/dsh-tools'

export const name = 'example-tool'
export const inject = ['tools']

export function apply(ctx: Context) {
  ctx.tools.register(defineTool({
    name: 'example',
    description: 'Describe what the model can do.',
    parameters: {
      input: { type: 'string', required: true },
    },
    output: {
      schema: { type: 'string' },
      render: (_args, value) => [{ type: 'text', text: value }],
    },
    async execute(args, exec) {
      return doWork(args.input, exec.signal)
    },
  }))
}
```

Read the [tool authoring reference](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/cookbook/adding-a-tool.md) before implementing non-trivial tools. Inputs are validated from the parameter schema. Return one canonical JSON-compatible value matching `output.schema`; keep prose in `output.render`. Honor `exec.signal`. Background work needs the Harness jobs runtime and a typed handle, not an unbounded promise.

Use pure presentation functions for UI cards. They run during live rendering and replay, so they must not perform I/O or read mutable session state.

## Hooks And Events

Choose the narrowest event that owns the behavior. Use waterfalls for cooperative interception and policy. An observation listener must delegate; a policy that owns a decision may short-circuit. Use `tools/result` for immutable final outcomes instead of transforming an earlier phase by accident.

## LLM Adapters

An adapter extends the current `LlmAdapter` contract and registers one or more provider routes through `ctx.llm`. Read the [adapter guide](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/cookbook/adding-an-llm-adapter.md) and the current `StreamChunk` declarations before coding. Preserve stream ordering, raw tool-call argument fragments, finish/usage rules, errors, abort signals, unsupported options, and provider replay state.

## Settings Cards

A settings feature usually has two halves in one external package:

- Host: define and register a namespace, schema, validation, and change behavior.
- Client: register a keyed settings slot, bind the namespace through the settings scope, and own the card UI.

Read [adding a settings card](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/cookbook/adding-a-settings-card.md) for current package exports, client metadata, revision fencing, and secret handling. Never expose secret values in responses or model-visible content.

## Conversation Nodes

For browser rows derived from durable session events, define a stable business id, an event family, replayable immutable state, location data, a stable renderer key, and focused replace/prepend/append tests. Do not scan the full session window or latest unfinished context on every append. Read [adding a conversation node](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/cookbook/adding-a-conversation-node.md) before implementing one.

## Protocol Drivers

A protocol driver adapts an external wire protocol to `ctx.agents`. Separate enqueue receipts from open-ended event streams, publish whole-agent status separately, honor cancellation, and dispose owned agents to quiescence. Read the protocol-driver section of the [extension cookbook](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/cookbook/extension-cookbook.md).
