# Cordis Foundations

Read the current [Cordis primer](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/cordis-primer.md) and [Cordis tutorial](https://github.com/deepseek-ai/deepseek-harness/tree/master/docs/cordis-tutorial) when an API detail matters. This file is a compact mental model for external plugin work.

## Plugin Module

The default shape is a named module export:

```ts
import type { Context } from '@deepseek-ai/cordis'

export const name = 'example-plugin'
export const inject = ['tools']

export function apply(ctx: Context) {
  // Register behavior through ctx.
}
```

Cordis also accepts an object with `apply` and a `Service` subclass. Use the function form unless the plugin provides a named service or needs class lifecycle behavior.

`cordis.yml` entries identify module specifiers and may include `id`, `config`, and `disabled`. Entries can be patched or overlaid by the host profile. Explicit stable ids matter for config diffing and HMR.

## Services And Dependencies

A service is a named capability on `ctx`. Providers register a service; consumers declare `inject` and access the ready service in `apply`.

```ts
export const inject = ['myService']

export function apply(ctx: Context) {
  ctx.myService.run()
}
```

When defining a service, pair runtime registration with TypeScript declaration merging:

```ts
declare module '@deepseek-ai/cordis' {
  interface Context {
    myService: MyService
  }
}
```

The declaration merge is compile-time only. The service is available at runtime only after the provider mounts it. Service names share a flat namespace; use a distinctive name for external capabilities.

Use `inject` for required capabilities. Use `ctx.get('myService')` when absence is a supported mode. Do not import a concrete provider merely to call its capability; dependency injection keeps providers replaceable.

## Effects And Lifecycle

Cordis unloads a plugin when configuration changes, HMR replaces it, a parent is disposed, or a required service disappears. Registrations made through Cordis APIs are effects and are undone automatically.

For resources Cordis does not own:

```ts
ctx.effect(() => {
  const resource = acquireResource()
  return () => resource.close()
})
```

Use one disposer when teardown steps must be ordered. Async disposers may run concurrently, so do not rely on separate effects for sequencing. A child plugin mounted through `ctx.plugin()` is disposed with its parent.

## Events

Events are typed through declaration merging and should use a readable `namespace/action` name. The dispatch mode is part of the contract:

| Mode | Call | Use |
| --- | --- | --- |
| `emit` | `ctx.emit` | Synchronous observation; values are not collected. |
| `parallel` | `await ctx.parallel` | Independent async listeners. |
| `serial` | `await ctx.serial` | Ordered listeners with a result/decision. |
| `bail` | `ctx.bail` | Synchronous first-answer behavior. |
| `waterfall` | `await ctx.waterfall` | Around-middleware and interception. |

For a waterfall listener, call `next()` when delegating. Returning without it vetoes or replaces downstream behavior. Use the event map and owning subsystem docs to determine payloads and mode.

## Configuration

Use a Schemastery schema and an interface with the same name:

```ts
import Schema from '@deepseek-ai/schemastery'

export interface Config {
  endpoint: string
}

export const Config: Schema<Config> = Schema.object({
  endpoint: Schema.string(),
})

export function apply(ctx: Context, config: Config) {
  // config is validated before this runs.
}
```

Defaults belong in the schema. Fail loudly on invalid config and validate constraints the schema cannot express, such as reachability or provider availability, before starting work.

## Composition And Pending Fibers

YAML order does not guarantee load order. Cordis starts plugins when their injected services are available. A missing provider leaves the consumer `PENDING`; this is not necessarily an error because the provider may mount later.

When a plugin appears to do nothing, inspect:

- The module specifier and export shape
- The active profile and patch/overlay actually being used
- The plugin entry's stable `id` and `disabled` state
- Every name in `inject` and which plugin provides it
- Fiber state and loader logs
- Whether the process exits because nothing is keeping the event loop alive

HMR is safe only when every registration and external resource has correct effect disposal. A replacement should unload the old instance fully before the new instance owns the same registration.
