[![view on npm](http://img.shields.io/npm/v/sse-broadcast.svg?style=flat-square)](https://www.npmjs.com/package/sse-broadcast)
[![downloads per month](http://img.shields.io/npm/dm/sse-broadcast.svg?style=flat-square)](https://www.npmjs.com/package/sse-broadcast)
[![node version](https://img.shields.io/badge/node-%3E=0.8-brightgreen.svg?style=flat-square)](https://nodejs.org/download)
[![build status](https://img.shields.io/travis/schwarzkopfb/sse-broadcast.svg?style=flat-square)](https://travis-ci.org/schwarzkopfb/sse-broadcast)
[![test coverage](https://img.shields.io/coveralls/schwarzkopfb/sse-broadcast.svg?style=flat-square)](https://coveralls.io/github/schwarzkopfb/sse-broadcast)
[![license](https://img.shields.io/npm/l/sse-broadcast.svg?style=flat-square)](https://github.com/schwarzkopfb/sse-broadcast/blob/master/LICENSE)

# sse-broadcast

Server-Sent Events through a Publish/Subscribe API for Node.js.

## Usage

With [Express](http://expressjs.com/):

```js
const app = require('express')(),
      sse = require('sse-broadcast')()

app.get('/events', function (req, res) {
    sse.subscribe('channel', res)
})

app.post('/event/:type', function (req, res) {
    sse.publish('channel', req.params.type, 'whoo! something happened!')
    res.send()
})

app.listen(3333)
```

![demo](/assets/demo.gif)

If you're interested about the usage with [Koa](http://koajs.com/) or
a vanilla [Node.js](https://nodejs.org/) server,
see the [examples](/examples) folder.

For more convenience, there's a helper to extend `http.ServerResponse.prototype`:

```js
const app = require('express')(),
      sse = require('sse-broadcast')

sse.proto(sse())

app
    .get('/events', function (req, res) {
        res.subscribe('channel')
    })
    .post('/event', function (req, res) {
        res.publish('channel', 'event', 'data')
           .end()
    })
    .listen(3333)
```

### Compression

This package supports `response` compression.
If you want to compress outgoing event streams then you
have to provide the `request` object for subscriptions.

```js
const app = require('express')(),
      sse = require('sse-broadcast')({ compression: true }) // !!!

app
    .get('/events', function (req, res) {
        sse.subscribe('channel', req, res) // !!!
    })
    .post('/event', function (req, res) {
        sse.publish('channel', 'event', 'data')
        res.end()
    })
    .listen(3333)
```

The `compression` option can be set to `true` or an object containing settings
for the [compression](https://github.com/expressjs/compression#options) module.

## Using multiple nodes

SSE is a [long-polling](https://en.wikipedia.org/wiki/Push_technology#Long_polling) solution,
consequently if you want to broadcast events to every client subscribed to a given channel
you’ll need some way of passing messages between processes or computers.

You can implement your own mechanism to do this or simply use [sse-broadcast-redis](https://github.com/schwarzkopfb/sse-broadcast-redis)
to distribute events on top of [Redis](http://redis.io/):

```js
const os      = require('os'),
      cluster = require('cluster')

if (cluster.isMaster)
    for (var i = os.cpus().length; i--;)
        cluster.fork()
else {
    const app = require('express')(),
          sse = require('sse-broadcast')()

    require('sse-broadcast-redis')(sse, { host: 'localhost', port: 6379 })

    app.get('/events', function (req, res) {
        sse.subscribe('channel', res)
    })

    app.post('/event', function (req, res) {
        sse.publish('channel', 'event', 'data')
        res.send()
    })

    app.listen(3333)
}
```

**Note:** options are passed to [redis](https://github.com/NodeRedis/node_redis) directly.

## Installation

With npm:

    npm install sse-broadcast

## License

[MIT](/LICENSE)
