[![Build Status](https://travis-ci.org/mojo-js/crudlet-pubnub.svg)](https://travis-ci.org/mojo-js/crudlet-pubnub) [![Coverage Status](https://coveralls.io/repos/mojo-js/crudlet-pubnub/badge.svg?branch=master)](https://coveralls.io/r/mojo-js/crudlet-pubnub?branch=master) [![Dependency Status](https://david-dm.org/mojo-js/crudlet-pubnub.svg)](https://david-dm.org/mojo-js/crudlet-pubnub)

A streamable interface for [Pubnub](http://www.pubnub.com/). This library also works nicely with [crudlet](https://github.com/mojo-js/crudlet.js), and other crudlet adapters.

#### Example

```javascript
var pubnub      = require("crudlet-pubnub");
var crud        = require("crudlet");

var pubStream = pubnub({
  subscribeKey: "sub key"
  publishKey: "pub key",
  channel: "streamChannel"
});

// tail all remote messages
pubStream(crud.operation("tail", { name: "message" })).on("data", function(operation) {
  console.log(operation.data.message); // "hello"
});

// publish a remote message to the world
pubStream("message", { message: "hello" });
```


#### db pubnub(options[, reject])

Creates a new pubnub streamer.

- `options`
  - `subscribeKey` - your pubnub subscription key
  - `publishKey` - your pubnub publish key
  - `channel` - (optional) the channel to subscribe to
- `reject` - set of commands to reject - default is `[load]`

```javascript
var pubStream = pubnub({
  subscribeKey: "sub key"
  publishKey: "pub key",
  channel: "streamChannel"
}, ["load", "anotherCommandToIgnore"]);

// does not get broadcasted
pubStream(crud.operation("anotherCommandToIgnore"));
```

#### db.addChannel(channel)

adds a new channel to subscribe to.

```javascript
pubStream.addChannel(crud.operation("someChannel"));
pubStream.addChannel(crud.operation("anotherChannel")_;
```

#### [stream.Readable](https://nodejs.org/api/stream.html#stream_class_stream_readable) db(operationName, options)

Publishes a new operation to pubnub.

```javascript
pubStream({ name: "hello", data: { name: "world" }});
pubStream({ name "doSomething", data: { name: "world" }});
```

#### [stream.Readable](https://nodejs.org/api/stream.html#stream_class_stream_readable) db(tail, filter)

Tails a remote operation. This is your subscription function.

```
db({ name: "tail" }).on("data", function(operation) {

});

```

Or you can do something like synchronizing databases between clients:

```javascript
var crud   = require("crudlet");
var loki   = require("crudlet-loki");
var pubnub = require("crudlet-pubnub");

var pubdb = pubnub({
  subscribeKey: "sub key"
  publishKey: "pub key",
  channel: "streamChannel"
});

var db = crud.tailable(loki());

// listen for local operations on lokidb - pass to pubnub
db(crud.operation("tail")).pipe(crud.open(pubdb));

// listen for remote operations on pubnub - pass to lokidb
pubdb(crud.operation("tail")).pipe(crud.open(db));

// stored in loki & synchronized across clients
db(crud.operation("insert", { data: { name: "Juice" }}));
```
