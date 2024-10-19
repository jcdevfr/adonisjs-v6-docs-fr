---
summary: Apprenez à étendre le framework AdonisJS en utilisant des macros et des accesseurs (getters).
---

# Étendre le framework

L'architecture d'AdonisJS rend très facile l'extension du framework. Nous utilisons les API principales du framework pour créer un écosystème de packages officiels.

Dans ce guide, nous explorerons différentes API que vous pouvez utiliser pour étendre le framework à travers un package ou au sein du code de votre application.

## Macros et accesseurs

Les macros et les accesseurs (getters) offrent une API pour ajouter des propriétés au prototype d'une classe. Vous pouvez les considérer comme du sucre syntaxique pour `Object.defineProperty`. En interne, nous utilisons le package [macroable](https://github.com/poppinss/macroable), et vous pouvez vous référer à son README pour une explication technique approfondie.

Comme les macros et les accesseurs sont ajoutés à l'exécution, vous devrez informer TypeScript des informations de type pour la propriété ajoutée en utilisant la [fusion de déclarations](https://www.typescriptlang.org/docs/handbook/declaration-merging.html).

Vous pouvez écrire le code pour ajouter des macros dans un fichier dédié (comme `extensions.ts`) et l'importer dans la méthode `boot` du fournisseur de services.

```ts
// title: providers/app_provider.ts
export default class AppProvider {
  async boot() {
    await import('../src/extensions.js')
  }
}
```

Dans l'exemple suivant, nous ajoutons la méthode `wantsJSON` à la classe [Request](../basics/request.md) et définissons ses types simultanément.

```ts
// title: src/extensions.ts
import { Request } from '@adonisjs/core/http'

Request.macro('wantsJSON', function (this: Request) {
  const firstType = this.types()[0]
  if (!firstType) {
    return false
  }
  
  return firstType.includes('/json') || firstType.includes('+json')
})
```

```ts
// title: src/extensions.ts
declare module '@adonisjs/core/http' {
  interface Request {
    wantsJSON(): boolean
  }
}
```
- Le chemin du module lors de l'appel `declare module` doit être le même que le chemin que vous utilisez pour importer la classe.
- Le nom de l'`interface` doit être le même que le nom de la classe à laquelle vous ajoutez la macro ou l'accesseur.

### Accesseurs

Les accesseurs sont des propriétés évaluées à la demande ajoutées à une classe. Vous pouvez ajouter un accesseur en utilisant la méthode `Class.getter`. Le premier argument est le nom de l'accesseur, et le second argument est la fonction de rappel pour calculer la valeur de la propriété.

Les fonctions de rappel des accesseurs ne peuvent pas être asynchrones car les [accesseurs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get) en JavaScript ne peuvent pas être asynchrones.

```ts
import { Request } from '@adonisjs/core/http'

Request.getter('hasRequestId', function (this: Request) {
  return this.header('x-request-id')
})

// vous pouvez utiliser la propriété comme ceci
if (ctx.request.hasRequestId) {
}
```

Les accesseurs peuvent être des singletons, ce qui signifie que la fonction pour calculer la valeur de l'accesseur sera appelée une fois, et la valeur de retour sera mise en cache pour une instance de la classe.

```ts
const isSingleton = true

Request.getter('hasRequestId', function (this: Request) {
  return this.header('x-request-id')
}, isSingleton)
```

### Classes extensibles

Voici la liste des classes qui peuvent être étendues en utilisant des macros et des accesseurs.

| Classe                                                                                          | Chemin d'importation                 |
|------------------------------------------------------------------------------------------------|-----------------------------|
| [Application](https://github.com/adonisjs/application/blob/main/src/application.ts)            | `@adonisjs/core/app`        |
| [Request](https://github.com/adonisjs/http-server/blob/main/src/request.ts)                    | `@adonisjs/core/http`       |
| [Response](https://github.com/adonisjs/http-server/blob/main/src/response.ts)                  | `@adonisjs/core/http`       |
| [HttpContext](https://github.com/adonisjs/http-server/blob/main/src/http_context/main.ts)      | `@adonisjs/core/http`       |
| [Route](https://github.com/adonisjs/http-server/blob/main/src/router/route.ts)                 | `@adonisjs/core/http`       |
| [RouteGroup](https://github.com/adonisjs/http-server/blob/main/src/router/group.ts)            | `@adonisjs/core/http`       |
| [RouteResource](https://github.com/adonisjs/http-server/blob/main/src/router/resource.ts)      | `@adonisjs/core/http`       |
| [BriskRoute](https://github.com/adonisjs/http-server/blob/main/src/router/brisk.ts)            | `@adonisjs/core/http`       |
| [ExceptionHandler](https://github.com/adonisjs/http-server/blob/main/src/exception_handler.ts) | `@adonisjs/core/http`       |
| [MultipartFile](https://github.com/adonisjs/bodyparser/blob/main/src/multipart/file.ts)        | `@adonisjs/core/bodyparser` |


## Étendre les modules

La plupart des modules AdonisJS fournissent des API extensibles pour enregistrer des implémentations personnalisées. En voici une liste récapitulative :

- [Création d'un driver de hachage](../security/hashing.md#creating-a-custom-hash-driver)
- [Création d'un driver de session](../basics/session.md#creating-a-custom-session-store)
- [Création d'un driver d'authentification sociale](../authentication/social_authentication.md#creating-a-custom-social-driver)
- [Extension du REPL](../digging_deeper/repl.md#adding-custom-methods-to-repl)
- [Création d'un chargeur de traductions i18n](../digging_deeper/i18n.md#creating-a-custom-translation-loader)
- [Création d'un formateur de traductions i18n](../digging_deeper/i18n.md#creating-a-custom-translation-formatter)
