# HTTP context

A new instance of [HTTP Context class](https://github.com/adonisjs/http-server/blob/main/src/http_context/main.ts) is generated for every HTTP request and passed along to the route handler, middleware, and exception handler.

HTTP Context holds all the information you may need related to an HTTP request. For example:

- You can access the request body, headers, and query params using the [ctx.request](./request.md) property.
- You can respond to the HTTP request using the [ctx.response](./response.md) property.
- Access the logged-in user using the [ctx.auth]() property.
- Or, authorize user actions using the [ctx.bouncer]() property.

In a nutshell, the context is a request-specific store holding all the information for the ongoing request.

## Getting access to the HTTP context

The HTTP context is passed by reference to the route handler, middleware, and exception handler, and you can access it as follows.

### Route handler

The [router handler](./routing.md) receives the HTTP context as the first parameter.

```ts
Route.get('/', (ctx) => {
  console.log(ctx.inspect())
})

// Destructure properties
Route.get('/', ({ request, response }) => {
  console.log(request.url())
  console.log(request.headers())
  console.log(request.qs())
  console.log(request.body())
  
  response.send('hello world')
  response.send({ hello: 'world' })
})
```

### Controller method

The [controller method](./controllers.md) (similar to the router handler) receives the HTTP context as the first parameter.

```ts
import { HttpContext } from '@adonisjs/core/http'

export default class HomeController {
  async index({ request, response }: HttpContext) {
  }
}
```

### Middleware class

The `handle` method of the [middleware class](./middleware.md) receives HTTP context as the first parameter. 

```ts
import { HttpContext } from '@adonisjs/core/http'

export default class AuthMiddleware {
  async handle({ request, response }: HttpContext) {
  }
}
```

### Exception handler class

The `handle` and the `report` methods of the [global exception handler](./exception_handling.md) class receive HTTP context as the second parameter. The first parameter is the `error` property.

```ts
import {
  HttpContext,
  HttpExceptionHandler
} from '@adonisjs/core/http'

export default class ExceptionHandler extends HttpExceptionHandler {
  async handle(error: any, ctx: HttpContext) {
    return super.handle(error, ctx)
  }

  async report(error: any, ctx: HttpContext) {
    return super.report(error, ctx)
  }
}
```

## Injecting Http Context using Dependency Injection

If you use Dependency injection throughout your application, you can inject the HTTP context to a class or a method by type hinting the `HttpContext` class.


:::warning

Ensure the `#middleware/container_bindings_middleware` middleware is registered inside the `kernel/start.ts` file. This middleware is required to resolve request-specific values (i.e., the HttpContext class) from the container.

:::

See also: [IoC container guide](../fundamentals/ioc_container.md)

```ts
// title: app/services/user_service.ts
import { inject } from '@adonisjs/core'
import { HttpContext } from '@adonisjs/core/http'

@inject()
export default class UserService {
  constructor(protected ctx: HttpContext) {}
  
  all() {
    // method implementation
  }
}
```

For automatic dependency resolution to work, you must inject the `UserService` inside your controller. Remember, the first argument to a controller method will always be the context, and the rest will be injected using the IoC container.

```ts
import { inject } from '@adonisjs/core'
import { HttpContext } from '@adonisjs/core/http'
import UserService from '#services/user_service'

export default class UsersController {
  @inject()
  index(ctx: HttpContext, userService: UserService) {
    return userService.all()
  }
}
```

That's all! The `UserService` will now automatically receive an instance of the ongoing HTTP request. You can repeat the same process for nested dependencies as well.

## Accessing HTTP context from anywhere inside your application

Dependency injection is one way to accept the HTTP context as a class constructor or a method dependency and then rely on the container to resolve it for you.

However, it is not a hard requirement to restructure your application and use Dependency injection everywhere. You can also access the HTTP context from anywhere inside your application using the [Async local storage](https://nodejs.org/dist/latest-v16.x/docs/api/async_context.html#class-asynclocalstorage) provided by Node.js. 

We have a [dedicated guide](../fundamentals/async_local_storage.md) on how Async local storage works and how AdonisJS uses it to provide global access to the HTTP context.

In the following example, the `UserService` class uses the `HttpContext.getOrFail` method to get the HTTP context instance for the ongoing request.

```ts
// title: app/services/user_service.ts
import { HttpContext } from '@adonisjs/core/http'

export default class UserService {
  all() {
    const ctx = HttpContext.getOrFail()
    console.log(ctx.request.url())
  }
}
```

The following code block shows the `UserService` class usage inside the `UsersController`.

```ts
import { HttpContext } from '@adonisjs/core/http'
import UserService from '#services/user_service'

export default class UsersController {
  index(ctx: HttpContext) {
    const userService = new UserService()
    return userService.all()
  }
}
```

## HTTP Context properties

Following is the list of properties may can access through the HTTP context. As you install new packages, they may add additional properties to the context.

<dl>
<dt>

ctx.request

</dt>

<dd>

Reference to an instance of the [HTTP Request class](./request.md).

</dd>

<dt>

ctx.response

</dt>

<dd>

Reference to an instance of the [HTTP Response class](./response.md).

</dd>

<dt>

ctx.logger

</dt>

<dd>

Reference to an instance of [logger](../digging_deeper/logger.md) created for a given HTTP request.

</dd>

<dt>

ctx.route

</dt>

<dd>

The matched route for the current HTTP request. The `route` property is an object of type [StoreRouteNode](https://github.com/adonisjs/http-server/blob/next/src/types/route.ts#L54)

</dd>

<dt>

ctx.params

</dt>

<dd>

An object of route params

</dd>

<dt>

ctx.subdomains

</dt>

<dd>

An object of route subdomains. Only exists when the route is part of a dynamic subdomain

</dd>

<dt>

ctx.session

</dt>

<dd>

Reference to an instance of the [Session class](). The property is condtibuted by the `@adonisjs/session` package.

</dd>

<dt>

ctx.auth

</dt>

<dd>

Reference to an instance of the [Auth class](). The property is condtibuted by the `@adonisjs/auth` package.

</dd>

<dt>

ctx.view

</dt>

<dd>

Reference to an instance of the [View class](). The property is condtibuted by the `@adonisjs/view` package.

</dd>

<dt>

ctx\.ally

</dt>

<dd>

Reference to an instance of the [SocialAuth class](). The property is condtibuted by the `@adonisjs/ally` package.

</dd>

<dt>

ctx.bouncer

</dt>

<dd>

Reference to an instance of the [Bouncer class](). The property is condtibuted by the `@adonisjs/bouncer` package.

</dd>

<dt>

ctx.i18n

</dt>

<dd>

Reference to an instance of the [I18n class](). The property is condtibuted by the `@adonisjs/i18n` package.

</dd>
</dl>


## Extending HTTP context

You may add custom properties to the HTTP context class using macros or getters. Make sure to read the [extending AdonisJS guide](../fundamentals/extending_the_framework.md) first if you are new to the concept of macros.

```ts
import { HttpContext } from '@adonisjs/core/http'

HttpContext.macro('property', function (this: HttpContext) {
  return value
}
HttpContext.getter('property', function (this: HttpContext) {
  return value
})
```

Since the macros and getters are added at runtime, you must inform TypeScript about their types.

```ts
declare module '@adonisjs/core/http' {
  export interface HttpContext {
    property: valueType
  }
}
```

## Creating dummy context during tests

You may use the `testUtils` service to create a dummy HTTP context during tests. 

The context instance is not attached to any route; therefore, the `ctx.route` and `ctx.params` values will be undefined. However, you can manually assign these properties if required by the code under test.

```ts
import testUtils from '@adonisjs/core/services/test_utils'

const ctx = testUtils.createHttpContext()
```

Optionally, you may pass Node.js `req` and `res` objects to create the HttpContext for a specific request.

```ts
import { createServer } from 'node:http'
import testUtils from '@adonisjs/core/services/test_utils'

createServer((req, res) => {
  const ctx = testUtils.createHttpContext({ req, res })
})
```