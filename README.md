# CEX.IO Spot Trading

The official Node.js client for CEX.IO Spot Trading API (https://trade.cex.io/docs)

## Features

- Easy to use, requires only key-secret pair to setup
- Handle all transport work, just call required action
- Popular protocols supported, REST and WebSocket onboard

## Installation

```bash
npm install @cex-io/cexio-spot-trading
```

## Rest client

```js
const { RestClient } = require('@cex-io/cexio-spot-trading')
const defaultClient = new RestClient()
const authenticatedClient = new RestClient(apiKey, apiSecret, options)
```

Arguments for RestClient are optional. For private actions you need to generate apiKey and apiSecret pair from UI terminal.

- `apiKey` _string_ - Api key for specific account.
- `apiSecret` _string_ - Api secret for specific account.
- `options` _object_ - Additional settings for client.

Available client options described below, they all are optional:

- `apiLimit` _integer_ - Rate limit value for apiKey. Default is 300.
  Client will check requests count and prevent from spam the server. You can ask to increase this limit.
- `timeout` _integer_ - Request timeout in milliseconds. Default is 30000.
- `rejectUnauthorized` _boolean_ - This option useful when you test demo env. Default is true.
- `host` _string_ - Can be changed to test your bot on demo environment. Default is 'https://trade.cex.io/api/spot/'
- `apiUrlPublic` _string_ - Use a concrete url for public API calls. This option overrides `host` value. Default is 'https://trade.cex.io/api/spot/rest-public/'
- `apiUrl` _string_ - Use a concrete url for private API calls. This option overrides `host` value. Default is 'https://trade.cex.io/api/spot/rest/'


### Public actions

To make a public request use `async callPublic(action, params)` method.
This method return `Promise` which resolves with server response.
If some error was occurred then method rejects with status code and error description.

For more details check [api reference](https://trade.cex.io/docs).

```js
const { RestClient } = require('@cex-io/cexio-spot-trading')

const client = new RestClient()

try {
  const res = await client.callPublic('get_ticker')
  console.log(res)
} catch (err) {
  console.log(err)
}
```

```js
{ error: 'Bad Request', statusCode: 400 }
{ error: 'Unexpected error', statusCode: 500 }
```

### Private actions

To make private api calls use `async callPrivate(action, params)`. It's similar to public method but requires `apiKey` and `apiSecret` arguments to client initialization. Each private request is signed with `HMAC sha256` so if key is incorrect or signature is wrong client will return rejected promise with error like this `{ error: 'Authorization Failed', statusCode: 401 }`

```js
const { RestClient } = require('@cex-io/cexio-spot-trading')

const key = '_account_api_key_'
const secret = '_account_api_secret_'
const action = 'get_my_orders'
const params = {
  pair: 'BTC-USD'
}

const client = new RestClient(key, secret)

try {
  const res = await client.callPrivate(action, params)
  console.log(res)
} catch (err) {
  console.error(err)
}
```

Success response example:

```js
{ ok: 'ok', data: { ... } }
```

## WebSocket client

```js
const { WebsocketClient } = require('@cex-io/cexio-spot-trading')
const ws = new WebsocketClient(apiKey, apiSecret, options)
```

To init the WebsocketClient you must pass `apiKey` and `apiSecret` arguments. You can generate them in UI terminal.

- `apiKey` _string_ - Api key for specific account.
- `apiSecret` _string_ - Api secret for specific account.
- `options` _object_ - Additional settings for client.

Available client options described below, they all are optional:

- `wsReplyTimeout` _integer_ - Request timeout in milliseconds. Default is 30000.
- `rejectUnauthorized` _boolean_ - This option useful when you test demo env. Default is true.
- `host` _string_ - Can be changed to test your bot on demo environment. Default is 'wss://trade.cex.io/api/spot/'
- `apiUrlPublic` _string_ - Use a concrete url for public WS calls. This option overrides `host` value. Default is 'wss://trade.cex.io/api/spot/ws-public/'
- `apiUrl` _string_ - Use a concrete url for private WS calls. This option overrides `host` value. Default is 'wss://trade.cex.io/api/spot/ws/'


### Call Private actions
To send request to the server you need to connect and auth first. Everything is under the hood and all you need is call `async ws.connect()` method. After that you can invoke `async ws.callPrivate(action, params)` method which returns `Promise` with server response.
If some error was occurred then method rejects with status code and error description.

```js
  const { WebsocketClient } = require('@cex-io/cexio-spot-trading')
  const ws = new WebsocketClient(apiKey, apiSecret, options)

  await ws.connect() // connect and auth on the server

  const res = await ws.callPrivate(action, params)
  console.log('result:', res)

  ws.disconnect() // close connection
```

### Subscribe to updates
The WebsocketClient allows you to receive updates. The following types of updates are available: `account_update`, `executionReport`, `order_book_increment`, `tradeUpdate`, etc. You can get more details about them in [documentation](https://trade.cex.io/docs#websocket-private-api-calls-account-events).

```js
const { WebsocketClient } = require('@cex-io/cexio-spot-trading')
const ws = new WebsocketClient(apiKey, apiSecret)

try {
  await ws.connect()

  ws.subscribe('executionReport', msg => {
    console.log('executionReport:', msg)
  })

  ws.subscribe('account_update', msg => {
    console.log('account_update:', msg)
  })
} catch (err) {
  console.error(err)
}
```