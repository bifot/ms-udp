# ms-udp

Solution for communication between services using udp protocol with built-in auto-retry & reconnect. 🔬

## Install

```sh
$ npm i ms-udp -S
```

## Examples

[There are some simple examples](examples).

## API

### Server

#### .constructor()

```js
const server = new udp.Server();
```

#### .on(action, ...middlewares)

* `event` <[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#String_type)> Action name
* `...middlewares` <[function[]](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function)> Action middlewares

This method creates action.

```js
const { Balances } = require('./db');

server.on('get', async (ctx) => {
  const { amount } = await Balances.findOne({
    userId: ctx.payload.userId,
  });
  
  ctx.reply(amount);
});
```

#### .use(...middlewares)

* `...middlewares` <[function[]](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function)> Common middlewares

This method creates common middlewares.

#### .listen(port[, host, callback])

* `port` <[number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Number_type)>
* `host` <[?string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#String_type)>
* `callback` <[?function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function)>

This method starts listening.

### Client

#### .constructor(options)

* `options` <[Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)>
  * `services` <[Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)> Available services
    * `[key]` - <[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#String_type)> Service name
    * `[value]` - <[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#String_type) / [Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)<[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#String_type)>> Service's address

```js
const client = new udp.Client({
  services: {
    balances: '127.0.0.1:3000',
  },
});
```

#### .ask(name[, payload, options])

* `event` <[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#String_type)> Event name in format `<service_name>.<action>`
* `data` <[?Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)> Event data
* `options` <[?Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)> Options
  * `attempts` <[?number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Number_type)> Maximum number of attempts *(default: 5)*
  * `timeout` <[?number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Number_type)> Maximum timeout in ms *(default: 5000)*

This method asks other service for something.

```js
app.use(async (ctx, next) => {
  const isAuth = await ctx.udp.ask('users.checkAuth', {
    login: ctx.query.login,
    password: ctx.query.password,
  });

  ctx.assert(isAuth, 403);
  
  await next();
});
```

#### .middleware()

This method returns middleware for Koa or Express.

```js
const Koa = require('koa');
const { Client } = require('udp');

const app = new Koa();
const client = new Client();

app.use(client.middleware());

app.use((ctx) => {
  ctx.body = 'Hello, world!';
});

app.listen(3000);
```
