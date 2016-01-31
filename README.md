# Electrode Server

This is an imaginatively named, configurable web server using Hapi.js atop Node.js.

The aim is to provide a standardized node web server that can be used to serve your web
application without the need for duplicating from another example, or starting from scratch.

The intention is that you will extend via configuration, such that this provides the baseline
functionality of a Hapi web server with React rendering, and within your own application you
will add on the features, logic, etc unique to your situation.


## Why

Why do we have electrode-server that's not a plugin, and apparently duplicating a core Hapi module [glue]?

It's true that the part to start a Hapi server is similar to [glue] and fairly trivial.  It basically sets up a Hapi server, adds connections, and starts the server.

The key feature electrode-server brings is composable plugins registration.

Electrode platform brings many standard capabilities through Hapi plugins, and these are automatically provided by default.  You can choose to turn them off if you want.  With Hapi's plugins registration through an array, it's not very flexible because arrays are not easily composable.  

Electrode server instead allow you to specify the plugins you want in an object, which is easily composable.  With an `enable` flag and a `priority` value, electrode server then can combine plugin registrations and turn them into an array sorted by priority, and then pass that to Hapi.

## Versioning

This module require Node v4.2.x+.

## Installing

`npm i --save @walmart/electrode-server`

## Usage

Electrode Server comes with enough defaults such that you can spin up a Hapi server at `http://localhost:3000` with one call:

```js
require("@walmart/electrode-server")();
```

Of course that doesn't do much but getting a 404 response from `http://localhost:3000`, so see below for configuring it. 

## Configuration

You can pass in a config object that controls every aspect of the Hapi server.

```js
const config = {};
require("@walmart/electrode-server")(config);
```

It's recommended that you use a configuration management tool to control
your configurations based on `NODE_ENV`.

[node-config] is one such tool.  [see below for more details](#node-config)

## Configuration Options

Here's what you can configure:

All properties are optional (if not present, the default values shown below will be used).

* `server` (Object) Server options to pass to [Hapi's `Hapi.Server`]

   * _default_
      * `server.app.config` is set to a object that's the combination of your config with `electrode-server's` defaults applied.

    ```js    
    {
      server: {
        app: {
          electrode: true
        }
      }
    }
    ```

   
* `connections` (Object) Connections to setup for the Hapi server.  Each connection should be an object field and its key is used as the labels of the connection. 

   * _default_

    ```js
    {
        default: {
          host: process.env.HOST,
          address: process.env.HOST_IP || "0.0.0.0",
          port: parseInt(process.env.PORT, 10) || 3000,
          routes: {
            cors: true
          }
        }
    }
    ```

* `plugins` (Object) plugin registration objects, converted to an array of its values and passed to [Hapi's `server.register`]

   * _default_

    ```js
    {
      plugins: {
        appConfig: {
          priority: 10
        },
        csrf: {
          priority: 15,
          module: "crumb"
        },
        inert: {
          priority: 100
        },
        staticPaths: {
          priority: 120
        }
      }
    }
    ```
    
  * Unless you have a reason to, please avoid specifying priority.  If you do, unless you have a reason otherwise, please use >= 200.

## Default plugins

`electrode-server` registers a few plugins by default.

`csrf` is using [Hapi crumb plugin] to provide CSRF support.

`inert` is using [Hapi inert plugin] to handle static files.

These two are `electrode-server's` internal plugins: 

   * `appConfig` sets `req.app.config` to `server.app.config` as soon as a request comes in.  That's why its priority is low.
   * `staticPaths` is a simple plugin that serves static files from `dist/js`, `dist/images`, and `dist/html`.

If you want to turn off one of these default plugins, you can set its `enable` flag to false.  For example, to turn off
the `csrf` plugin, in your config, do:

```js
{
  plugins: {
    csrf: {
      enable: false
    }
  }
}
```

> Please refer to the code for [latest default plugins](lib/config/default.js#L39).

## node-config

To keep your environment specific configurations manageable, you should use a tool like [node-config].

It supports many different formats for your config files, from different flavors of JSON to Yaml to a full blown JavaScript file, it's up to you.

Once you have your config files setup like [node-config recommended config setup], you can simply pass node-config to electrode-server:

```js
const config = require("config");
require("@walmart/electrode-server")(config);
```

## Adding a Hapi plugin 

You can have `electrode-server` register any Hapi plugin that you want
through your configuration file. Here's an example using the Store Info Plugin:

First, install the plugin as you normally would from `npm`:

`npm i --save @walmart/store-info-plugin`

Then, add your plugin to the config `plugins` section.

```js
{
  plugins: {
    "@walmart/store-info-plugin": {
      priority: 210
    }
  }
}
```

Above config tells `electrode-server` to use the plugin's field name `@walmart/store-info-plugin` as the name of 
the plugin's module to load for registration with Hapi.

### Plugin configs

   * `priority` - integer value to indicate the plugin's registration order
      * lower value ones are register first
      * Default to `Infinity` if this field is missing or has no valid integer value (`NaN`) (string of number accepted)
   * `enable` - if set to `false` then this plugin won't be registered.
   * plugin field name - generally use as the name of the plugin module to load for registration

### Other plugin configs

  If a plugin's field name is not desired as its module name, then you can optionally specify one of the following 
  to provide the plugin's module for registration:

   * `register` - if specified, then treat as the plugin's `register` function to pass to Hapi
   * `module` - if specified and `register` is not, then treat it as the name of the plugin module to load for registration.
      * If you absolutely do not want electrode server to try loading any module for this plugin, then set `module` to false.

## hostname and IP

`electrode-server` will add a `electrode` object to the config.

Inside it will lookup the hostname with `os.hostname` and IP with `dns.lookup` and set them to:

```js
{
  electrode: {
    hostIP: "<ip>",
    hostname: "<hostname>"
  }
}
```

## API

The electrode server exports a single API.

### [electrodeServer](#electrodeserver)

`electrodeServer(config, [callback])`

   * `config` is the [electrode server config](#configuration-options)
   * Returns a promise resolving to the Hapi server
   
   * `callback` is an optional errback with the signature `function (err, server)`
      * where `server` is the Hapi server
      * If callback is provided then promise is _not_ returned.

## Support/Contact

Dave Stevens <dstevens@walmartlabs.com> Slack: @dstevens

Joel Chen <xchen@walmartlabs.com> Slack: @joelchen

Also see Slack Channel `#electrode` or `#react`

[glue]: https://github.com/hapijs/glue
[node-config]: https://github.com/lorenwest/node-config
[Hapi crumb plugin]: https://github.com/hapijs/crumb
[Hapi's `Hapi.Server`]: http://hapijs.com/api#new-serveroptions 
[Hapi's `server.register`]: http://hapijs.com/api#serverregisterplugins-options-callback
[Hapi inert plugin]: https://github.com/hapijs/inert
[node-config recommended config setup]: https://github.com/lorenwest/node-config/wiki/Configuration-Files#config-directory 
[Hapi inert plugin]: https://github.com/hapijs/inert
