# node-flint (v5)

### Spark Bot SDK for Node JS

## News

**x/x/x IMPORTANT:**

* Flint v5 is a huge refactor from v4. Before upgrading your existing bots to
use v5, please make sure to review all docs and examples to understand the new
class methods and library structure.

* Flint no longer supports tokens from non Bot Accounts. This has become
necessary due to the various difference between a bot and person token.
Additionally Cisco does not support nor endorse using a person token for bots.
Applications that require this functionality should be defined as a "App"
integration. You can read more about the differences between bots and apps
[here](https://developer.ciscospark.com/bots.html#bots-vs-integrations). If you
are looking for a framework that uses a "person" token and integrates easier
into "App" integrations, check out either
[node-sparky](https://github.com/flint-bot/sparky) or the Cisco
[spark-js-sdk](https://github.com/ciscospark/spark-js-sdk).

**See [CHANGELOG.md](/CHANGELOG.md) for details on changes to versions of Flint.**

## Contents

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [Installation](#installation)
    - [Via Git](#via-git)
    - [Via NPM](#via-npm)
    - [Example Template Using Express](#example-template-using-express)
    - [Other Examples](#other-examples)
- [Overview](#overview)
- [Authorization](#authorization)
  - [Domain Name Authorization](#domain-name-authorization)
- [Logging](#logging)
  - [Winston Logger](#winston-logger)
- [Storage](#storage)
  - [File Store](#file-store)
  - [Redis Store](#redis-store)
  - [Mongo Store](#mongo-store)
- [Authoring Plugins](#authoring-plugins)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->
## Installation

#### Via Git
```bash
mkdir myproj
cd myproj
git clone https://github.com/flint-bot/flint
npm install ./flint
```

#### Via NPM
```bash
npm install node-flint
```
#### Example Template Using Express
```js
const Flint = require('node-flint');
const express = require('express');
const bodyParser = require('body-parser');

// config
const config = {
  token: 'abcdefg12345abcdefg12345abcdefg12345abcdefg12345abcdefg12345',
  webhookSecret: 'somesecr3t',
  webhookUrl: 'http://example.com/webhook',
  port: 8080,
};

// init flint
const flint = new Flint(config);

// string match on 'hello'
flint.hears('hello', (bot, trigger) => {
  bot.message.say().markdown(`**Hello** ${trigger.personDisplayName}!`);
});

// setup express
const app = express();
app.use(bodyParser.json());

// add route for path that is listening for web hooks
app.post('/webhook', flint.spark.webhookListen());

// start express server
const server = app.listen(config.port, () => {
  // start flint
  flint.start();
  console.log(`Flint listening on port ${config.port}`);
});

// gracefully shutdown (ctrl-c)
process.on('SIGINT', () => {
  console.log('\nStopping...');
  server.close();
  flint.stop();
});
```
#### Other Examples

The following examples are included to show the flexibility and to help with a
quick setup to see how Flint Operates. After getting the basic setup working
and a bot responding in a Room, be sure to read the rest of the documentation
to learn about The more advanced features.

* [**Basic Express with NGROK Example**](/docs/example-ngrok.md)

* [**Advanced Express Example**](/docs/example-advanced.md)

* [**Restify Example**](/docs/example-restify.md)

_More examples coming soon!_
## Overview

Most of Flint's functionality is based around the flint.hears function. This
defines the phrase or pattern the bot is listening for and what actions to take
when that phrase or pattern is matched. The flint.hears function gets a callback
than includes two objects. The bot object, and the trigger object.

A simple example of a flint.hears() function setup:

```js
flint.hears(phrase, (bot, trigger) => {
  bot.<command>
    .then((returnedValue) => {
      // do something with returnedValue
    })
    .catch(err => console.error(err));
});
```

## Authorization
By default, the authorization system used in flint allows ALL users to interact
with the bot. Other plugins can be loaded that inspect the trigger object in
order to allow or deny users from interacting with the bot based on any property
found in the trigger object. Other backend authorizations are possible by
referencing any one of the built-in storage modules and passing it to the
`flint.use()` method. Custom storage modules can be created by referencing
the template at `plugins/auth/template.js`

### Domain Name Authorization

**Example:**

```js
// require Domain authorization plugin
const DomainAuth = require('node-flint/plugins/auth/domain');

// add authorization object to flint config
const config = {
  token: 'abcdefg12345abcdefg12345abcdefg12345abcdefg12345abcdefg12345',
  webhookSecret: 'somesecr3t',
  webhookUrl: 'http://example.com/webhook',
  port: 8080,
  authorization: {
    domains: ['example.com', 'cisco.com'],
  },
};

// init flint
const flint = new Flint(config);

// load authorization module
flint.use('authorization', DomainAuth);

// start flint
flint.start();
```

## Logging
By default, the logging subsystem uses a console based logger directed at the
console. Other backend logging systems are possible by
referencing any one of the built-in logging modules and passing it to the
`flint.use()` method. Custom logging modules can be created by referencing
the template at `plugins/logging/template.js`

### Winston Logger

**Example:**

```js
// require Winston logger plugin
const WinstonLogger = require('node-flint/plugins/logger/winston');

// add logger object to flint config
const config = {
  token: 'abcdefg12345abcdefg12345abcdefg12345abcdefg12345abcdefg12345',
  webhookSecret: 'somesecr3t',
  webhookUrl: 'http://example.com/webhook',
  port: 8080,
  logger: {
    transports: [
      new (WinstonLogger.winston.transports.Console)({
        colorize: true,
        timestamp: false,
      }),
    ],
  },
};

// init flint
const flint = new Flint(config);

// load logger module
flint.use('logger', WinstonLogger);

// start flint
flint.start();
```

**See docs for winston transports for more details.**


## Storage
The storage system used in flint is a simple key/value store and resolves around
these 3 methods:

* `bot.store(key, value)` - Store a value to a bot instance where 'key' is a
  string and 'value' is a boolean, number, string, array, or object. *This does
  not not support functions or any non serializable data.* Returns a promise
  with the value.
* `bot.recall(key)` - Recall a value by 'key' from a bot instance. Returns a
  resolved promise with the value or a rejected promise if not found.
* `bot.forget([key])` - Forget (remove) value(s) from a bot instance where 'key'
  is an optional property that when defined, removes the specific key, and when
  undefined, removes all keys. Returns a resolved promise if deleted
  **or not found.**

When a bot despawns (removed from room), the key/value store for that bot
instance will automatically be removed from the store. By default, the in-memory
store is used. Other backend stores are possible by referencing any one of the
built-in storage modules and passing it to the `flint.use()` method. Custom
storage modules can be created by referencing the template at
`plugins/storage/template.js`

Other subsystems of Flint will use this same storage module for persisting data
across restarts.

**See docs for store, recall, forget for more details.**

### File Store

**Example:**

```js
// require File storage plugin
const FileStore = require('node-flint/plugins/storage/file');

// add storage object to flint config
const config = {
  token: 'abcdefg12345abcdefg12345abcdefg12345abcdefg12345abcdefg12345',
  webhookSecret: 'somesecr3t',
  webhookUrl: 'http://example.com/webhook',
  port: 8080,
  storage: {
    path: 'flint-store.json',
  },
};

// init flint
const flint = new Flint(config);

// load storage module
flint.use('storage', FileStore);

// start flint
flint.start();

```

### Redis Store

**Example:**

```js
// require Redis storage plugin
const RedisStore = require('node-flint/plugins/storage/redis');

// add storage object to flint config
const config = {
  token: 'abcdefg12345abcdefg12345abcdefg12345abcdefg12345abcdefg12345',
  webhookSecret: 'somesecr3t',
  webhookUrl: 'http://example.com/webhook',
  port: 8080,
  storage: {
    url: 'redis://localhost',
  },
};

// init flint
const flint = new Flint(config);

// load storage module
flint.use('storage', RedisStore);

// start flint
flint.start();

```

### Mongo Store

**Example:**

```js
// require Mongo storage plugin
const MongoStore = require('node-flint/plugins/storage/file');

// add storage object to flint config
const config = {
  token: 'abcdefg12345abcdefg12345abcdefg12345abcdefg12345abcdefg12345',
  webhookSecret: 'somesecr3t',
  webhookUrl: 'http://example.com/webhook',
  port: 8080,
  storage: {
    url: 'mongodb://localhost',
  },
};

// init flint
const flint = new Flint(config);

// load storage module
flint.use('storage', MongoStore);

// start flint
flint.start();

```

## Authoring Plugins

The Flint plugin architecture currently supports the following types with others being added in future updates.

* Authorization
* Logging
* Storage

When needing to create a custom plugin, the best place to start is with either
one of the existing plugins, or the template plugin found in the respective
plugin sub-directory.

For the plugin to validate, it must return a class constructor with the
required methods.

For passing configuration options to a custom plugin, these should be defined
under the Flint config object with the object key being one of the supported
plugin types. _(i.e. Storage plugins would have their config stored under
flint.config.storage)._

When the plugin is added via the `flint.use()` method, your class constructor
will be passed a instantiated flint object as the only argument. To access the
config object for your plugin, you can parse `flint.config.<plugin type>`.

For example when creating a custom Storage plugin:

**myplugin.js**

```js
class MyCustomStorage {

  constructor(flint) {
    this.config = flint.config.storage;
  }

  start() {}

  stop() {}

  create(name, key, value) {}

  read(name, key) {}

  delete(name, [key]) {}

}

module.exports = MyCustomStorage;
```

To use this plugin, your flint based app should have something similar to the
following:

**myapp.js**

```js
const MyPlugin = require('path/to/myplugin.js');

const config = {
  token: 'abcdefg12345abcdefg12345abcdefg12345abcdefg12345abcdefg12345',
  webhookSecret: 'somesecr3t',
  webhookUrl: 'http://example.com/webhook',
  port: 8080,
  storage: {
    somePluginConfig: true,
  },
};

const flint = new Flint(config);

flint.use('storage', MyPlugin);
```

After the plugin is added and validated, it is accessible from:

* `flint.storage.create(name, key, value)`
* `flint.storage.read(name, key)`
* `flint.storage.delete()`

It is also mapped to the bot object(s) with the 'name' argument forced to the
Spark Space ID (roomId):

* `bot.store(key, value)`
* `bot.recall(key)`
* `bot.forget([key])`

For a more detailed example of this, you can reference the
[example-advanced.js](/docs/example-advanced.md) app to see how various plugin
types are inserted.

# Flint Reference



 _Coming soon..._


## License

The MIT License (MIT)

Copyright (c) 2018 Nicholas Marus

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
