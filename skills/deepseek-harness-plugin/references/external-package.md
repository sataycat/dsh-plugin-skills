# External Plugin Repository Foundations

This is guidance for repositories outside `deepseek-harness`. The upstream monorepo's package constraints, aggregate TypeScript references, vendoring rules, generated docs, and internal release gates do not automatically apply.

## Inspect Before Bootstrapping

Look for:

- `package.json`: package name, module type, exports, peer dependencies, engines, scripts
- `src/index.ts` or equivalent runtime entrypoint
- `tsconfig.json`, bundler config, and declaration output
- `README.md`: install, composition, config, permissions, and compatibility
- Tests and keyless examples
- `cordis.yml`, patch files, or profile integration docs
- Existing release automation and package manager lockfile

Preserve the repository's package manager, module format, naming, and test conventions. Do not introduce a large framework or workspace layout just because the upstream monorepo uses one.

## Minimum External Plugin Contract

An installable plugin should make these facts discoverable:

1. Which package/module the Harness loads.
2. Which Harness version range and Node runtime it supports.
3. Which `ctx` services it injects and what it provides.
4. Which `cordis.yml` entry or profile patch mounts it.
5. Which config fields are accepted, with defaults and secret behavior.
6. Which permissions, network access, filesystem access, subprocesses, or credentials it requires.
7. How to run a keyless smoke test and how to uninstall or roll back.
8. Which upstream docs/source contracts the implementation depends on.

The plugin module should be small and explicit. Keep integration-specific helpers behind the plugin boundary. If a capability is independently replaceable, define its service contract separately from the provider and consumers.

## Package Shape

There is no single mandatory external package layout. A reasonable starting point is:

```text
package.json
README.md
src/
  index.ts
  ...
test/ or tests/
```

Add a `client` entry only if the plugin contributes browser code. Add references or scripts only when the repository needs them. Keep package exports and built files aligned; verify the published tarball rather than assuming source imports are what users receive.

## Installation And Configuration

Explain the complete path from install to active behavior:

```yaml
- id: my-plugin
  name: '@scope/my-plugin'
  config:
    endpoint: 'https://example.test'
```

Do not assume users manually edit one canonical YAML file. Harness profiles, bundles, patches, settings providers, and overlays may all contribute to the final composition. The plugin should document the supported installation/configuration path for the Harness version it targets.

For a plugin that manages other plugins, separate its own installation from the managed plugin lifecycle. It must not silently rewrite unrelated user configuration or remove entries it did not own.
