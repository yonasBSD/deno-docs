---
title: "Unstable feature flags"
oldUrl:
 - /runtime/tools/unstable_flags/
 - /runtime/manual/tools/unstable_flags/
---

New features of the Deno runtime are often released behind feature flags, so
users can try out new APIs and features before they are finalized. Current
unstable feature flags are listed on this page, and can also be found in the CLI
help text by running:

```sh
deno --help
```

## Using flags at the command line

You can enable a feature flag when you run a Deno program from the command line
by passing in the flag as an option to the CLI. Here's an example of running a
program with the `--unstable-node-globals` flag enabled:

```sh
deno run --unstable-node-globals main.ts
```

## Configuring flags in `deno.json`

You can specify which unstable features you'd like to enable for your project
using a
[configuration option in `deno.json`](/runtime/fundamentals/configuration/).

```json title="deno.json"
{
  "unstable": ["bare-node-builtins", "webgpu"]
}
```

The possible values in the `unstable` array are the flag names with the
`--unstable-` prefix removed.

## Configuration via environment variables

Some flags can be enabled by setting a value (any value) for an environment
variable of a given name, rather than being passed as a flag or `deno.json`
configuration option. Flags that are settable via environment variables will be
noted below.

Here's an example of setting the `--unstable-bare-node-builtins` flag via
environment variable:

```sh
export DENO_UNSTABLE_BARE_NODE_BUILTINS=true
```

## `--unstable-bare-node-builtins`

**Environment variable:** `DENO_UNSTABLE_BARE_NODE_BUILTINS`

This flag enables you to
[import Node.js built-in modules](/runtime/fundamentals/node/#node-built-in-modules)
without a `node:` specifier, as in the example below. You can also use this flag
to enable npm packages without an `npm:` specifier if you are manually managing
your Node.js dependencies ([see `byonm` flag](#--unstable-byonm)).

```ts title="example.ts"
import { readFileSync } from "fs";

console.log(readFileSync("deno.json", { encoding: "utf8" }));
```

## `--unstable-detect-cjs`

**Environment variable:** `DENO_UNSTABLE_DETECT_CJS`

Loads `.js`, `.jsx`, `.ts`, and `.tsx` modules as possibly being CommonJS in the
following additional scenarios:

1. The _package.json_ has no `"type"` field.
1. No _package.json_ exists.

By default, Deno only loads these modules as being possibly CommonJS when you're
in a project with a _package.json_ and the closest _package.json_ has
`{ "type": "commonjs" }`.

Requires Deno >= 2.1.2

## `--unstable-node-globals`

This flags injects Node specific globals into the global scope. The injected
globals are:

- `Buffer`
- `global`
- `setImmediate`
- `clearImmediate`

Note, that `process` is already available as a global starting with Deno 2.0.

Requires Deno >= 2.1.0

## `--unstable-sloppy-imports`

**Environment variable:** `DENO_UNSTABLE_SLOPPY_IMPORTS`

This flag enables behavior which will infer file extensions from imports that do
not include them. Normally, the import statement below would produce an error:

```ts title="foo.ts"
import { Example } from "./bar";
console.log(Example);
```

```ts title="bar.ts"
export const Example = "Example";
```

Executing the script with sloppy imports enabled will remove the error, but
provide guidance that a more performant syntax should be used.

Sloppy imports will allow (but print warnings for) the following:

- Omit file extensions from imports
- Use incorrect file extensions (e.g. importing with a `.js` extension when the
  actual file is `.ts`)
- Import a directory path, and automatically use `index.js` or `index.ts` as the
  import for that directory

[`deno compile`](/runtime/reference/cli/compile/) does not support sloppy
imports.

## `--unstable-unsafe-proto`

Deno made a conscious decision to not support `Object.prototype.__proto__` for
security reasons. However there are still many npm packages that rely on this
property to work correctly.

This flag enables this property. Note that it is not recommended to use this,
but if you really need to use a package that relies on it, the escape hatch is
now available to you.

## `--unstable-webgpu`

Enable the
[`WebGPU` API](https://developer.mozilla.org/en-US/docs/Web/API/WebGPU_API) in
the global scope, as in the browser. Below is a simple example to get basic
information about the GPU using this API:

```ts
// Try to get an adapter from the user agent.
const adapter = await navigator.gpu.requestAdapter();
if (adapter) {
  // Print out some basic details about the adapter.
  const adapterInfo = await adapter.requestAdapterInfo();

  // On some systems this will be blank...
  console.log(`Found adapter: ${adapterInfo.device}`);

  // Print GPU feature list
  const features = [...adapter.features.values()];
  console.log(`Supported features: ${features.join(", ")}`);
} else {
  console.error("No adapter found");
}
```

Check out [this repository](https://github.com/denoland/webgpu-examples) for
more examples using WebGPU.

## `--unstable-broadcast-channel`

Enabling this flag makes the
[`BroadcastChannel`](https://developer.mozilla.org/en-US/docs/Web/API/BroadcastChannel)
web API available for use in the global scope, as in the browser.

## `--unstable-worker-options`

Enable unstable
[Web Worker](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers)
API options. Specifically, it enables you to specify permissions available to
workers:

```ts
new Worker(`data:application/javascript;base64,${btoa(`postMessage("ok");`)}`, {
  type: "module",
  deno: {
    permissions: {
      read: true,
    },
  },
}).onmessage = ({ data }) => {
  console.log(data);
};
```

## `--unstable-cron`

Enabling this flag makes the [`Deno.cron`](/deploy/kv/manual/cron) API available
on the `Deno` namespace.

## `--unstable-kv`

Enabling this flag makes [Deno KV](/deploy/kv/manual) APIs available in the
`Deno` namespace.

## `--unstable-net`

Enable unstable net APIs. These APIs include:

- [`WebSocketStream`](https://developer.mozilla.org/en-US/docs/Web/API/WebSocketStream)
- [`Deno.DatagramConn`](https://docs.deno.com/api/deno/~/Deno.DatagramConn)

## ``

Enable the
[OpenTelemetry integration for Deno](/runtime/fundamentals/open_telemetry).

## `--unstable`

:::caution --unstable is deprecated - use granular flags instead

The `--unstable` flag is no longer being used for new features, and will be
removed in a future release. All unstable features that were available using
this flag are now available as granular unstable flags, notably:

- `--unstable-kv`
- `--unstable-cron`

Please use these feature flags instead moving forward.

:::

Before more recent Deno versions (1.38+), unstable APIs were made available all
at once using the `--unstable` flag. Notably, [Deno KV](/deploy/kv/manual) and
other cloud primitive APIs are available behind this flag. To run a program with
access to these unstable features, you would run your script with:

```sh
deno run --unstable your_script.ts
```

It is recommended that you use the granular unstable flags instead of this, the
`--unstable` flag is now deprecated and will be removed in Deno 2.

## `--unstable-temporal`

Enable the experimental [Temporal API](https://tc39.es/proposal-temporal/docs/)
in the global scope. The Temporal API is a modern date and time API that is
being developed as a replacement for the legacy `Date` object in JavaScript. It
provides better support for time zones, calendars, and more precise date/time
calculations.

To use the Temporal API in your Deno program, run it with the flag:

```sh
deno run --unstable-temporal your_script.ts
```

Or configure it in your `deno.json`:

```json title="deno.json"
{
  "unstable": ["temporal"]
}
```

```ts title="example.ts"
// Example using Temporal API
const now = Temporal.Now.plainDateTimeISO();
console.log(`Current date and time: ${now}`);

const date = Temporal.PlainDate.from("2025-07-10");
const nextWeek = date.add({ days: 7 });
console.log(`Next week: ${nextWeek}`);

// Working with time zones
const zonedDateTime = Temporal.Now.zonedDateTimeISO("America/New_York");
console.log(`Time in New York: ${zonedDateTime}`);
```

Note that the Temporal API is still experimental and the specification may
change before it becomes stable. Use this flag only for testing and
experimentation.
