# Pretty Printing

By default, Pino log lines are newline delimited JSON (NDJSON). This is perfect
for production usage and long-term storage. It's not so great for development
environments. Thus, Pino logs can be prettified by using a Pino prettifier
module like [`pino-pretty`][pp]:

1. Install a prettifier module as a separate optional dependency, e.g. `npm install pino-pretty`.
   Pino does not depend on `pino-pretty`; install it only when you want prettified output.
2. Instantiate the logger with the `transport.target` option set to `'pino-pretty'`:
  ```js
  const pino = require('pino')
  const logger = pino({
    transport: {
      target: 'pino-pretty'
    },
  })

  logger.info('hi')
  ```
3. The transport option can also have an options object containing `pino-pretty` options:
  ```js
  const pino = require('pino')
  const logger = pino({
    transport: {
      target: 'pino-pretty',
      options: {
        colorize: true
      }
    }
  })

  logger.info('hi')
  ```

The legacy `prettyPrint` option was removed. Passing `prettyPrint: true` (or any
truthy value) throws. Use `transport: { target: 'pino-pretty' }` as shown above,
or pipe stdout to the `pino-pretty` CLI.

If a bundler reports `Can't resolve 'pino-pretty'` even when you are not
prettifying logs, mark `pino-pretty` as external / ignored so the bundler does
not try to resolve it. See [Bundling](bundling.md#pino-pretty-optional).

**Note:** `logger.flush()` does not flush the output of `pino-pretty`. See
[Flush Limitations with `pino-pretty`](/docs/asynchronous.md) for more details.

  [pp]: https://github.com/pinojs/pino-pretty
