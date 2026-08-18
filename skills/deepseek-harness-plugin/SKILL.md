---
name: deepseek-harness-plugin
description: Create, bootstrap, extend, debug, and maintain external DeepSeek Harness plugins built on Cordis. Use when working with dsh-plugin repositories, cordis.yml composition, Cordis services, inject dependencies, lifecycle effects, typed events, Schemastery config, Harness tools, LLM adapters, settings cards, conversation nodes, protocol drivers, HMR, plugin installation, or pending plugin fibers.
---

# DeepSeek Harness Plugins

Use this skill for **external** DeepSeek Harness plugin repositories. Do not apply the repository-internal package checklist from the `deepseek-harness` monorepo unless the user explicitly asks about that monorepo.

DeepSeek Harness is a Cordis application where everything is a plugin. An external plugin should attach to documented services and events, not patch or fork the agent loop.

## Source Of Truth

The Harness is in developer preview and APIs can change. Treat the current upstream English documentation and source as authoritative. Read the relevant page before coding and inspect the installed package declarations when the docs do not answer a question.

- [First external plugin](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/user/develop/basic/index.md)
- [Cordis tutorial](https://github.com/deepseek-ai/deepseek-harness/tree/master/docs/cordis-tutorial)
- [Cordis primer](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/cordis-primer.md)
- [Extension cookbook](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/cookbook/extension-cookbook.md)
- [Tool authoring](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/cookbook/adding-a-tool.md)
- [Harness architecture](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/architecture.md)

Load a reference below only when the task needs it. Do not copy large upstream documents into the skill; link to them so the agent can follow API changes.

## Workflow

1. Inspect the existing repository before proposing structure. Read `README.md`, `package.json`, TypeScript config, source entrypoints, tests, and any `cordis.yml` or patch files. Identify whether the repository is a package, a standalone plugin module, or a host/client combination.
2. Clarify the boundary. State what the plugin owns, what Harness capability it consumes or provides, and whether behavior must be model-visible, durable in the session log, live-only, or browser-only.
3. Choose the documented extension point. Use the decision guide in [references/extension-shapes.md](references/extension-shapes.md). Prefer one small function plugin. Use a Cordis `Service` only when the plugin exposes a reusable capability to other plugins.
4. Inspect the current upstream contract for the chosen service/event. Do not invent `ctx` keys, event names, payloads, or package APIs from memory.
5. Express dependencies with `inject`. Plugin load order in `cordis.yml` is not a dependency mechanism. A plugin with a missing required service remains `PENDING`.
6. Implement registrations through Cordis APIs. They must unwind when the plugin unloads. Wrap timers, connections, watchers, subprocesses, and other external resources in `ctx.effect()` with a disposer.
7. Add configuration only when needed. Export a Schemastery schema named `Config`; validate resource/provider references early and do not start half-configured.
8. Test the lifecycle and the contract, not only the happy path. Cover activation, missing dependencies, config failure, disposal/HMR, and the selected service or event behavior.
9. Run the repository's own package, typecheck, lint, build, and test commands. If the repository has no checks, create a minimal keyless smoke composition and document how to run it.
10. Report which upstream docs and source declarations were used, along with any API uncertainty or assumptions.

## Non-Negotiable Cordis Rules

- A function plugin normally exports `name`, optional `inject`, optional `Config`, and `apply(ctx, config)`.
- `ctx.plugin(child)` returns a fiber. Dispose it when code owns a child plugin lifetime; parent disposal recursively disposes children.
- `ctx.on(...)`, `ctx.plugin(...)`, service registrations, and Harness registries are lifecycle effects. Do not add manual cleanup for resources already owned by Cordis.
- `inject` is for hard dependencies. For optional capabilities, use `ctx.get('serviceName')` at the point of use.
- Use services for direct capability calls and events for decoupled observation, interception, and policy.
- A waterfall listener that only observes or annotates must call `next()`. Returning without `next()` intentionally short-circuits downstream behavior.
- Keep model-visible input reconstructable from durable session events. Do not hide model context in process-local state.
- Tool `execute` returns the canonical value declared by `output.schema`; `output.render` creates model-facing content. Do not make callers parse prose to recover ids or fields.
- Honor operational `AbortSignal`s and dispose owned agents/resources to quiescence.

## Progressive References

- [Cordis foundations](references/cordis-foundations.md): plugin shapes, services, events, effects, config, composition, HMR, and pending fibers.
- [Extension shapes](references/extension-shapes.md): tools, hooks, adapters, settings, conversation nodes, protocol drivers, and package-level capabilities.
- [External package foundations](references/external-package.md): what to inspect and the minimum concerns for bootstrapping an external plugin repository without copying monorepo-only rules.
- [Verification and debugging](references/verification.md): keyless smoke tests, lifecycle checks, dependency diagnosis, and security concerns.
