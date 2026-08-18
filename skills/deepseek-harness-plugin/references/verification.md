# Verification And Debugging

## Keyless Smoke Composition

Prefer a fixture that does not require a model API key. Mount the plugin with the smallest required providers and verify an observable result. For a local Cordis fixture, the upstream tutorial uses a `cordis.yml` list and the Cordis launcher. For a Web plugin, use a patch overlay and the current Harness command documented by upstream.

At minimum verify:

- The module resolves and its export shape is correct.
- `apply` runs only after required injected services exist.
- Config defaults and invalid values behave as documented.
- Registrations are present while active and absent after disposal.
- Replacing or unloading a provider does not leave stale listeners, tools, timers, files, or processes.
- The intended service/event/tool receives the expected typed payload.

## Lifecycle Tests

For each external resource, test the disposer. For each service consumer, test provider absence and replacement. For HMR, load the plugin twice and assert that the old registration is not duplicated. For tools, test abort and invalid canonical output as well as success.

For durable features, test replay rather than only live append. For conversation nodes, cover complete history, update-only history followed by prepend, live append, stable renderer keys, and publication cadence.

## Pending Diagnosis

If a plugin prints nothing or appears inactive:

1. Confirm the actual profile/overlay was loaded.
2. Confirm the entry's module specifier resolves.
3. Confirm `id` and `disabled` fields.
4. List every `inject` dependency and its provider.
5. Inspect Cordis fiber state for `PENDING` or `FAILED`.
6. Check that a logger/exporter is mounted early enough to show loader errors.
7. Check whether the process exits because the plugin has no active resource.

`PENDING` is a valid waiting state and is often caused by an unavailable provider. A failed module resolution can be reported through logging rather than thrown directly.

## Security And Operations

External plugins execute with the permissions of the Harness process. Treat package installation, update, filesystem, subprocess, network, and credential operations as privileged:

- Validate package names, versions, URLs, paths, and archive contents.
- Pin or verify integrity when consuming remote artifacts.
- Avoid shell interpolation; use argument arrays and safe subprocess APIs.
- Do not overwrite user configuration without ownership, preview, backup, and rollback semantics.
- Keep secrets out of logs, tool results, session events, and UI payloads.
- Bound downloads, extraction, subprocess lifetime, output size, and concurrency.
- Honor cancellation and leave the system in a recoverable state after interruption.

## Final Checks

Run the smallest relevant commands from the repository, then inspect the package artifact if publishing:

```sh
pnpm test
pnpm run typecheck
pnpm run lint
pnpm run build
npm pack --dry-run
```

Use the actual package-manager scripts; these commands are examples, not a requirement to add all of them.
