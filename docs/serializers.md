# Serializers

Serializers reshape values on log lines before they are stringified. Use them to
redact fields, shrink large objects, or normalize errors and HTTP objects.

This page is a short guide with examples. The option reference lives in
[`serializers` in the API docs](api.md#opt-serializers).

## Basic usage

Register a function per top-level key. The function receives the value at that
key and must return a JSON-friendly value without throwing:

```js
const pino = require('pino')

const logger = pino({
  serializers: {
    user (value) {
      return { id: value.id }
    }
  }
})

logger.info({ user: { id: 1, name: 'Ada', password: 'secret' } }, 'login')
// user is logged as {"id":1} — name and password are dropped
```

Serializers run only for **own top-level keys** of the object passed to the log
method (or to `child()` bindings). Nested keys are not walked unless your
serializer does that itself.

A key whose value is `undefined` is skipped; its serializer does not run.

## Errors

By default Pino registers `pino.stdSerializers.err` under the `err` key. That
serializer also runs when you pass an `Error` instance as the first log
argument (`logger.info(new Error('kaboom'))`).

Change the key with [`errorKey`](api.md#opt-errorkey) if you prefer another name:

```js
const logger = pino({
  errorKey: 'error',
  serializers: {
    error: pino.stdSerializers.err
  }
})
```

## Standard serializers

`pino.stdSerializers` re-exports [pino-std-serializers](https://github.com/pinojs/pino-std-serializers)
helpers for common shapes (`err`, `req`, `res`, and related variants). Typical
HTTP setup:

```js
const pino = require('pino')

const logger = pino({
  serializers: {
    err: pino.stdSerializers.err,
    req: pino.stdSerializers.req,
    res: pino.stdSerializers.res
  }
})
```

See [`pino.stdSerializers`](api.md#pino-stdserializers).

## Child loggers

Serializers also apply to bindings passed to [`logger.child()`](api.md#child).
Those bindings are serialized **once**, when the child is created — not on every
later log call. That keeps child log lines cheap.

```js
const logger = pino({
  serializers: {
    user (value) {
      return { id: value.id }
    }
  }
})

const child = logger.child({ user: { id: 1, name: 'Ada' } })
// serializer already ran; child lines carry {"user":{"id":1}}

child.info('next request')
```

If a value must be serialized on every log call, pass it to the log method
instead of to `child()`.

Children inherit parent serializers. Passing `serializers` in the child
`options` object overrides inherited keys for that child only. See
[`options.serializers` on child](api.md#child).

## Serializing the message

A serializer registered under the configured [`messageKey`](api.md#opt-messagekey)
(`msg` by default) rewrites the message string:

```js
const logger = pino({
  serializers: {
    msg (message) {
      return String(message).toUpperCase()
    }
  }
})

logger.info('hello')
// msg is "HELLO"
```

## Whole-object transforms

`serializers[Symbol.for('pino.*')]` is deprecated. Prefer
[`formatters.log`](api.md#opt-formatters) when you need to transform the entire
log object.

## See also

* [`serializers` option](api.md#opt-serializers)
* [`pino.stdSerializers`](api.md#pino-stdserializers)
* [Child loggers](child-loggers.md)
* [Redaction](redaction.md) — for path-based stripping instead of custom serializers
