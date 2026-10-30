# Bundling

Due to its internal architecture based on Worker Threads, it is not possible to bundle Pino *without* generating additional files.

In particular, a bundler must ensure that the following files are also bundled separately:

* `lib/worker.js` from the `thread-stream` dependency
* `file.js`
* `lib/worker.js`
* Any transport the application actually uses (for example `pino-pretty`, only if configured)

Once the files above have been generated, the bundler must also add information about the files above by injecting a code that sets `__bundlerPathsOverrides` in the `globalThis` object.

The variable is an object whose keys are an identifier for the files and the values are the paths of files relative to the currently bundle files.

Example:

```javascript
// Inject this using your bundle plugin
globalThis.__bundlerPathsOverrides = {
  'thread-stream-worker': pinoWebpackAbsolutePath('./thread-stream-worker.js')
  'pino/file': pinoWebpackAbsolutePath('./pino-file.js'),
  'pino-worker': pinoWebpackAbsolutePath('./pino-worker.js'),
  // Include pino-pretty only when the app uses that transport:
  // 'pino-pretty': pinoWebpackAbsolutePath('./pino-pretty.js'),
};
```

Note that `pino/file`, `pino-worker` and `thread-stream-worker` are required identifiers. Other identifiers are possible based on the user configuration.

<a id="pino-pretty-optional"></a>
## `pino-pretty` is optional

[`pino-pretty`](https://github.com/pinojs/pino-pretty) is a separate package.
Pino loads it only when you set `transport: { target: 'pino-pretty' }` (or an
equivalent transport target). The old `prettyPrint` option is gone; see
[Pretty Printing](pretty.md).

Bundlers that statically scan `node_modules` may still try to resolve
`pino-pretty` and report `Module not found: Can't resolve 'pino-pretty'`.
That warning is safe to silence when you are not using the pretty transport:

* Webpack: add `'pino-pretty'` to [`externals`](https://webpack.js.org/configuration/externals/), or use [pino-webpack-plugin](https://github.com/pinojs/pino-webpack-plugin).
* Rollup: ignore `pino-pretty` in the CommonJS plugin (for example `commonjs({ ignore: ['pino-pretty'] })`).
* Next.js and similar frameworks: treat `pino` / `pino-pretty` as server externals so they are not bundled into the client graph.

Install `pino-pretty` only when you want prettified development output; do not
add it to production bundles unless that transport is actually configured.

## Webpack Plugin

If you are a Webpack user, you can achieve this with [pino-webpack-plugin](https://github.com/pinojs/pino-webpack-plugin) without manual configuration of `__bundlerPathsOverrides`; however, you still need to configure it manually if you are using other bundlers.

## Esbuild Plugin

[esbuild-plugin-pino](https://github.com/davipon/esbuild-plugin-pino) is the esbuild plugin to generate extra pino files for bundling.

## Bun Plugin

[bun-plugin-pino](https://github.com/vktrl/bun-plugin-pino) is the Bun plugin to generate extra pino files for bundling.