---
summary: Découvrez le contexte HTTP dans AdonisJS et comment y accéder à partir des gestionnaires de routes, des middlewares et des gestionnaires d'exceptions.
---

# Contexte HTTP

Une nouvelle instance de la [classe HttpContext](https://github.com/adonisjs/http-server/blob/main/src/http_context/main.ts) est générée pour chaque requête HTTP et transmise au gestionnaire de route, au middleware et au gestionnaire d'exceptions.

Le contexte HTTP contient toutes les informations dont vous pouvez avoir besoin concernant une requête HTTP. Par exemple :

- Vous pouvez accéder au corps de la requête, aux en-têtes et aux paramètres de requête en utilisant la propriété [ctx.request](../basics/request.md).
- Vous pouvez répondre à la requête HTTP en utilisant la propriété [ctx.response](../basics/response.md).
- Accédez à l'utilisateur connecté en utilisant la propriété [ctx.auth](../authentication/introduction.md).
- Autorisez les actions de l'utilisateur en utilisant la propriété [ctx.bouncer](../security/authorization.md).
- Et ainsi de suite.

En résumé, le contexte est un stockage spécifique à la requête contenant toutes les informations pour la requête en cours.

## Accéder au contexte HTTP

Le contexte HTTP est passé par référence au gestionnaire de route, au middleware et au gestionnaire d'exceptions, et vous pouvez y accéder comme ceci.

### Gestionnaire de route

Le [gestionnaire de route](../basics/routing.md) reçoit le contexte HTTP comme premier paramètre.

```ts
import router from '@adonisjs/core/services/router'

router.get('/', (ctx) => {
  console.log(ctx.inspect())
})
```

```ts
// title: Décomposer les propriétés
import router from '@adonisjs/core/services/router'

router.get('/', ({ request, response }) => {
  console.log(request.url())
  console.log(request.headers())
  console.log(request.qs())
  console.log(request.body())
  
  response.send('hello world')
  response.send({ hello: 'world' })
})
```

### Méthode de contrôleur

La [méthode de contrôleur](../basics/controllers.md) (similaire au gestionnaire de route) reçoit le contexte HTTP comme premier paramètre.

```ts
import { HttpContext } from '@adonisjs/core/http'

export default class HomeController {
  async index({ request, response }: HttpContext) {
  }
}
```

### Classe de middleware

La méthode `handle` d'une [classe de middleware](../basics/middleware.md) reçoit le contexte HTTP comme premier paramètre.

```ts
import { HttpContext } from '@adonisjs/core/http'

export default class AuthMiddleware {
  async handle({ request, response }: HttpContext) {
  }
}
```

### Classe de gestionnaire d'exceptions

Les méthodes `handle` et `report` d'une classe de [gestionnaire d'exceptions global](../basics/exception_handling.md) reçoivent le contexte HTTP comme deuxième paramètre. Le premier paramètre est la propriété `error`.

```ts
import {
  HttpContext,
  HttpExceptionHandler
} from '@adonisjs/core/http'

export default class ExceptionHandler extends HttpExceptionHandler {
  async handle(error: unknown, ctx: HttpContext) {
    return super.handle(error, ctx)
  }

  async report(error: unknown, ctx: HttpContext) {
    return super.report(error, ctx)
  }
}
```

## Injecter le contexte HTTP en utilisant l'injection de dépendances

Si vous utilisez l'injection de dépendances dans votre application, vous pouvez injecter le contexte HTTP dans une classe ou une méthode en utilisant le type `HttpContext`.

:::warning
Assurez-vous que le middleware `#middleware/container_bindings_middleware` est enregistré dans le fichier `kernel/start.ts`. Ce middleware est nécessaire pour résoudre les valeurs spécifiques à la requête (c'est-à-dire la classe HttpContext) à partir du conteneur.
:::

Voir aussi : [Guide du conteneur IoC](../concepts/dependency_injection.md)

```ts
// title: app/services/user_service.ts
import { inject } from '@adonisjs/core'
import { HttpContext } from '@adonisjs/core/http'

@inject()
export default class UserService {
  constructor(protected ctx: HttpContext) {}
  
  all() {
    // implémentation de la méthode
  }
}
```

Pour que la résolution automatique des dépendances fonctionne, vous devez injecter le `UserService` dans votre contrôleur. N'oubliez pas que le premier argument d'une méthode de contrôleur sera toujours le contexte, et le reste sera injecté en utilisant le conteneur IoC.

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

C'est tout ! Le `UserService` recevra maintenant automatiquement une instance de la requête HTTP en cours. Vous pouvez répéter le même processus pour les dépendances imbriquées également.

## Accéder au contexte HTTP depuis n'importe où dans votre application

L'injection de dépendances est une façon d'accepter le contexte HTTP comme dépendance du constructeur de classe ou de méthode, puis de compter sur le conteneur pour le résoudre pour vous.

Cependant, il n'est pas obligatoire de restructurer votre application et d'utiliser l'injection de dépendances partout. Vous pouvez également accéder au contexte HTTP depuis n'importe où dans votre application en utilisant le [stockage local asynchrone](https://nodejs.org/dist/latest-v21.x/docs/api/async_context.html#class-asynclocalstorage) fourni par Node.js.

Nous avons un [guide dédié](./async_local_storage.md) sur le fonctionnement du stockage local asynchrone et comment AdonisJS l'utilise pour fournir un accès global au contexte HTTP.

Dans l'exemple suivant, la classe `UserService` utilise la méthode `HttpContext.getOrFail` pour obtenir l'instance du contexte HTTP pour la requête en cours.

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

Le bloc de code suivant montre l'utilisation de la classe `UserService` dans le `UsersController`.

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

## Propriétés du contexte HTTP

Voici la liste des propriétés auxquelles vous pouvez accéder via le contexte HTTP. Au fur et à mesure que vous installez de nouveaux packages, ils peuvent ajouter des propriétés supplémentaires au contexte.

<dl>
<dt>

ctx.request

</dt>

<dd>

Référence à une instance de la [classe Request](../basics/request.md).

</dd>

<dt>

ctx.response

</dt>

<dd>

Référence à une instance de la [classe Response](../basics/response.md).

</dd>

<dt>

ctx.logger

</dt>

<dd>

Référence à une instance du [logger](../digging_deeper/logger.md) créé pour une requête HTTP spécifique.

</dd>

<dt>

ctx.route

</dt>

<dd>

La route correspondante pour la requête HTTP actuelle. La propriété `route` est un objet de type [StoreRouteNode](https://github.com/adonisjs/http-server/blob/main/src/types/route.ts#L69).

</dd>

<dt>

ctx.params

</dt>

<dd>

Un objet de paramètres d'une route.

</dd>

<dt>

ctx.subdomains

</dt>

<dd>

Un objet de sous-domaines d'une route. N'existe que lorsque la route fait partie d'un sous-domaine dynamique.

</dd>

<dt>

ctx.session

</dt>

<dd>

Référence à une instance de [Session](../basics/session.md) créée pour la requête HTTP actuelle.

</dd>

<dt>

ctx.auth

</dt>

<dd>

Référence à une instance de la [classe Authenticator](https://github.com/adonisjs/auth/blob/main/src/authenticator.ts). En savoir plus sur l'[authentification](../authentication/introduction.md).

</dd>

<dt>

ctx.view

</dt>

<dd>

Référence à une instance du moteur de rendu Edge. En savoir plus sur Edge dans le [guide Vues et templates](../views-and-templates/introduction.md#using-edge).

</dd>

<dt>

ctx\.ally

</dt>

<dd>

Référence à une instance de la [classe AllyManager](https://github.com/adonisjs/ally/blob/main/src/ally_manager.ts) pour implémenter la connexion sociale dans vos applications. En savoir plus sur [Ally](../authentication/social_authentication.md).

</dd>

<dt>

ctx.bouncer

</dt>

<dd>

Référence à une instance de la [classe Bouncer](https://github.com/adonisjs/bouncer/blob/main/src/bouncer.ts). En savoir plus sur l'[autorisation](../security/authorization.md).

</dd>

<dt>

ctx.i18n

</dt>

<dd>

Référence à une instance de la [classe I18n](https://github.com/adonisjs/i18n/blob/main/src/i18n.ts). En savoir plus sur `i18n` dans le guide d'[internationalisation](../digging_deeper/i18n.md).

</dd>

</dl>


## Étendre le contexte HTTP

Vous pouvez ajouter des propriétés personnalisées à la classe du contexte HTTP en utilisant des macros ou des accesseurs. Assurez-vous de lire d'abord le [guide d'extension d'AdonisJS](./extending_the_framework.md) si vous n'êtes pas familier avec le concept de macros.

```ts
import { HttpContext } from '@adonisjs/core/http'

HttpContext.macro('aMethod', function (this: HttpContext) {
  return value
})

HttpContext.getter('aProperty', function (this: HttpContext) {
  return value
})
```

Comme les macros et les accesseurs sont ajoutés au moment de l'exécution, vous devez informer TypeScript de leurs types en utilisant l'augmentation de module.

```ts
import { HttpContext } from '@adonisjs/core/http'

// insert-start
declare module '@adonisjs/core/http' {
  export interface HttpContext {
    aMethod: () => ValueType
    aProperty: ValueType
  }
}
// insert-end

HttpContext.macro('aMethod', function (this: HttpContext) {
  return value
})

HttpContext.getter('aProperty', function (this: HttpContext) {
  return value
})
```

## Créer un contexte fictif pendant les tests

Vous pouvez utiliser le service `testUtils` pour créer un contexte HTTP fictif pendant les tests.

L'instance de contexte n'est attachée à aucune route ; par conséquent, les valeurs `ctx.route` et `ctx.params` seront undefined. Cependant, vous pouvez attribuer manuellement ces propriétés si nécessaire pour le code en cours de test.

```ts
import testUtils from '@adonisjs/core/services/test_utils'

const ctx = testUtils.createHttpContext()
```

Par défaut, la méthode `createHttpContext` utilise des valeurs fictives pour les objets `req` et `res`. Cependant, vous pouvez définir des valeurs personnalisées pour ces propriétés comme indiqué dans l'exemple suivant.

```ts
import { createServer } from 'node:http'
import testUtils from '@adonisjs/core/services/test_utils'

createServer((req, res) => {
  const ctx = testUtils.createHttpContext({
    // highlight-start
    req,
    res
    // highlight-end
  })
})
```

### Utiliser la factory HttpContext

Le service `testUtils` n'est disponible que dans une application AdonisJS ; par conséquent, si vous construisez un package et avez besoin d'accéder à un contexte HTTP fictif, vous pouvez utiliser la classe [HttpContextFactory](https://github.com/adonisjs/http-server/blob/main/factories/http_context.ts#L30).

```ts
import { HttpContextFactory } from '@adonisjs/core/factories/http'
const ctx = new HttpContextFactory().create()
```
