# MongoDB

[![GitHub stars](https://img.shields.io/github/stars/feathersjs/feathers-mongodb.svg?style=social&label=Star)](https://github.com/feathersjs/feathers-mongodb/)
[![npm version](https://img.shields.io/npm/v/feathers-mongodb.svg?style=flat-square)](https://www.npmjs.com/package/feathers-mongodb)
[![Changelog](https://img.shields.io/badge/changelog-.md-blue.svg?style=flat-square)](https://github.com/feathersjs/feathers-mongodb/blob/master/CHANGELOG.md)

[feathers-mongodb](https://github.com/feathersjs/feathers-mongodb) is a database adapter for [MongoDB](https://www.mongodb.org/). It uses the [official NodeJS driver for MongoDB](https://www.npmjs.com/package/mongodb).

```bash
$ npm install --save mongodb feathers-mongodb
```

> **Important:** To use this adapter you also want to be familiar with the [common interface](./common.md) for database adapters.

> This adapter also requires a [running MongoDB](https://docs.mongodb.com/getting-started/shell/#) database server.


## API

### `service([options])`

Returns a new service instance intitialized with the given options. `Model` has to be a MongoDB collection.

```js
const MongoClient = require('mongodb').MongoClient;
const service = require('feathers-mongodb');

MongoClient.connect('mongodb://localhost:27017/feathers').then(db => {
  app.use('/messages', service({
    Model: db.collection('messages')
  }));
  app.use('/messages', service({ Model, id, events, paginate }));
});
```

__Options:__

- `Model` (**required**) - The MongoDB collection instance
- `id` (*optional*, default: `'_id'`) - The name of the id field property.
- `events` (*optional*) - A list of [custom service events](../real-time/events.md#custom-events) sent by this service
- `paginate` (*optional*) - A [pagination object](pagination.md) containing a `default` and `max` page size


## Example

Here is an example of a Feathers server with a `messages` endpoint that writes to the `feathers` database and the `messages` collection.

```
$ npm install feathers feathers-errors feathers-rest feathers-socketio feathers-mongodb mongodb body-parser
```

In `app.js`:

```js
const feathers = require('feathers');
const errorHandler = require('feathers-errors/handler');
const rest = require('feathers-rest');
const socketio = require('feathers-socketio');
const bodyParser = require('body-parser');
const MongoClient = require('mongodb').MongoClient;
const service = require('feathers-mongodb');

// Create a feathers instance.
const app = feathers()
  // Enable Socket.io
  .configure(socketio())
  // Enable REST services
  .configure(rest())
  // Turn on JSON parser for REST services
  .use(bodyParser.json())
  // Turn on URL-encoded parser for REST services
  .use(bodyParser.urlencoded({extended: true}));

// Connect to your MongoDB instance(s)
MongoClient.connect('mongodb://localhost:27017/feathers').then(function(db){
  // Connect to the db, create and register a Feathers service.
  app.use('/messages', service({
    Model: db.collection('messages'),
    paginate: {
      default: 2,
      max: 4
    }
  }));

  // A basic error handler, just like Express
  app.use(errorHandler());

  // Create a dummy Message
  app.service('messages').create({
    text: 'Message created on server',
    completed: false
  }).then(message => console.log('Created message', message));

  // Start the server.
  const port = 3030;

  app.listen(port, () => {
    console.log(`Feathers server listening on port ${port}`);
  });
}).catch(error => console.error(error));
```

Run the example with `node app` and go to [localhost:3030/messages](http://localhost:3030/messages).


## Querying

Additionally to the [common querying mechanism](./querying.md) this adapter also supports [MongoDB's query syntax](https://docs.mongodb.com/v3.2/tutorial/query-documents/) and the `update` method also supports MongoDB [update operators](https://docs.mongodb.com/v3.2/reference/operator/update/).
