---
title: Node API
description: API docs for using deepstream within your own node application
---

API when using deepstream as a Node.js package via NPM.

### `constructor(options)`
Instantiate a new deepstream server instance. You can pass an optional object
which contains the configuration or a filePath to the configuration file. Missing options will be merged with default values.

If you omit the argument, deepstream will use default values. Read more about
the [configuration and default values](/docs/server/configuration/).

{{#table mode="api"}}
-
  arg: options
  typ: Object | String
  opt: true
  des: Either the configuration object or a filepath to the configuration file
{{/table}}

**Please note** calling `server = new Deepstream()` only creates the instance, to actually start the server, you still need to call `server.start();`

```javascript
const Deepstream = require('deepstream.io')
const server = new Deepstream({port:8000})
```

## Events

### `started`
Emitted once `deepstream.start()` has been called and the startup procedure has completed succesfully.

### `stopped`
Emitted once `deepstream.stop()` has been called and the server has been completely shut down.

---

## Methods

### `start()`
Starts the server.

### `stop()`
Stops the server.

### `set(key, value)`
This method allows you to overwrite particular configuration options which were built via the
configuration initialization step.

💡 **NOTE:** If deepstream is initialized with a configuration object, `set()` will override the keys in your initial configuration. This is useful for passing in objects which are shared between deepstream and the rest of your application, such as a cache connector. You can override any of the options using the same name within your [configuration](/docs/server/configuration/), except for the notable difference(s) below.

**NOTE:** 
httpServer parameter removed in 2.x

```
{{#table mode="api"}}
-
  arg: key
  typ: String
  opt: false
  des: The configuration option that should be set
-
  arg: value
  typ: various
  opt: false
  des: The value that should be used
{{/table}}
```

##### Differences when using `set(key, value)`

If you use an configuration object its properties will be treated as a file path.
Here the `value` is treated as a string for these options:

- `sslCert`
- `sslKey`
- `sslCA`

Actually these options can be passed by an configuration object, but if you use a file-based configuration it only works with a `.js` file. YAML and JSON config files are not supporting these options.

These options might have a different name and location in the structure of the configuration object. If you use `set()` you also need to provide the instantiated instance as the `value`.

- `authenticationHandler`
- `permissionHandler`
- `logger`
- `message`
- `cache`
- `storage`

Make sure you run `server.start()` after you set all your options.

Some examples:

```javascript

/**
* The public key to use if using ssl
* Must have an associated sslKey set
*
* @type String
*/
server.set('sslCert', fs.readFileSync('./keys/cert.pem', 'utf8'))


/**
* An object that exposes a isValidUser function.
*
* @type authenticationHandler
* @default OpenPermissionHandler (same as `{auth:{type: none}}` in the default config)
*/
server.set('authenticationHandler', new OAuthHandler())

/**
* An object that that exposes a canPerformAction function.
*
* @type permissionHandler
* @default ConfigPermissionHandler (with arguments from the default config)
*/
server.set('permissionHandler', new LdapPermissionHandler())

/**
* A logger
*
* @type Logger
* @default DeepstreamWinstonLogger
*/
server.set('logger', new FileLogger())

/**
* MessageConnectors connect deepstream to a message bus
* (e.g. AMQP, Redis, Kafka) thus allowing for clustering.
*
* See the "Connectors" section above for details
*
* @type MessageConnector
* @default NoopMessageConnector
*/
server.set('messageConnector', new RedisMessageConnector({
  port: 6379,
  host: 'localhost'
}))


/**
* A regular expression that determines which records will not be
* stored in the database.
*
* This is useful to improve performance for fast-updating records
* that do not need to be stored in the long-term, e.g. stock prices
*
* Any record whose name matches the specified RegExp will be read / written
* directly to cache
*
* @type RegExp
* @default null
*/
server.set('storageExclusion', /^dont-store\/*./)
```
