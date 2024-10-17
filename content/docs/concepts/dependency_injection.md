---
summary: Découvrez l'injection de dépendances dans AdonisJS et comment utiliser le conteneur IoC pour résoudre les dépendances.
---

# Injection de dépendances

Au coeur de chaque application AdonisJS se trouve un conteneur IoC (Inversion of Control) capable de construire des classes et de résoudre les dépendances avec presque aucune configuration.

Le conteneur IoC répond aux deux principaux cas d'utilisation suivants :

- Exposer une API permettant aux packages internes et tiers d'enregistrer et de résoudre des liaisons (bindings) à partir du conteneur (plus de détails sur [les liaisons plus tard](#liaisons-du-conteneur)).
- Résoudre et injecter automatiquement les dépendances dans le constructeur d'une classe ou dans les méthodes d'une classe.

Commençons par l'injection de dépendances dans une classe.

## Exemple basique

L'injection automatique de dépendances repose sur [l'implémentation des décorateurs de TypeScript](https://www.typescriptlang.org/docs/handbook/decorators.html) et sur l'API [Metadata Reflection](https://www.npmjs.com/package/reflect-metadata).

Dans l'exemple suivant, nous créons une classe `EchoService` et injectons une instance de celle-ci dans la classe `HomeController`. Vous pouvez suivre en copiant-collant les exemples de code.

### Étape 1. Créer la classe Service

Commencez par créer la classe `EchoService` dans le répertoire `app/services`.

```ts
// title: app/services/echo_service.ts
export default class EchoService {
  respond() {
    return 'hello'
  }
}
```

### Étape 2. Injecter le service dans le contrôleur

Créez un nouveau contrôleur HTTP dans le répertoire `app/controllers`. Vous pouvez également utiliser la commande `node ace make:controller home`.

Importez `EchoService` dans le fichier du contrôleur et acceptez-le comme dépendance du constructeur.

```ts
// title: app/controllers/home_controller.ts
import EchoService from '#services/echo_service'

export default class HomeController {
  constructor(protected echo: EchoService) {
  }
  
  handle() {
    return this.echo.respond()
  }
}
```

### Étape 3. Utilisation du décorateur inject

Pour que la résolution automatique des dépendances fonctionne, nous devons utiliser le décorateur `@inject` sur la classe `HomeController`.

```ts
import EchoService from '#services/echo_service'
// insert-start
import { inject } from '@adonisjs/core'
// insert-end

// insert-start
@inject()
// insert-end
export default class HomeController {
  constructor(protected echo: EchoService) {
  }
  
  handle() {
    return this.echo.respond()
  }
}
```

C'est tout ! Vous pouvez maintenant lier la classe `HomeController` à une route et elle recevra automatiquement une instance de la classe `EchoService`.

### Conclusion

Vous pouvez considérer le décorateur `@inject` comme un espion qui observe les dépendances du constructeur ou des méthodes de la classe et en informe le conteneur.

Lorsque le routeur d'AdonisJS demande au conteneur de construire `HomeController`, le conteneur connaît déjà les dépendances du contrôleur.

## Construction d'un arbre de dépendances

Actuellement, la classe `EchoService` n'a pas de dépendances, et l'utilisation du conteneur pour  en créer une instance peut sembler excessive.

Mettons à jour le constructeur de la classe et faisons-lui accepter une instance de la classe `HttpContext`.

```ts
// title: app/services/echo_service.ts
// insert-start
import { inject } from '@adonisjs/core'
import { HttpContext } from '@adonisjs/core/http'
// insert-end

// insert-start
@inject()
// insert-end
export default class EchoService {
  // insert-start
  constructor(protected ctx: HttpContext) {
  }
  // insert-end

  respond() {
    return `Hello from ${this.ctx.request.url()}`
  }
}
```

Une fois de plus, nous devons placer notre espion (le décorateur `@inject`) sur la classe `EchoService` pour inspecter ses dépendances.

Et voilà, c'est tout ce que nous avons à faire. Sans changer une seule ligne de code dans le contrôleur, vous pouvez ré-exécuter le code, et la classe `EchoService` recevra une instance de la classe `HttpContext`.


:::note
Le grand avantage de l'utilisation du conteneur est que vous pouvez avoir des dépendances profondément imbriquées, et le conteneur peut résoudre l'arbre entier pour vous. La seule condition est d'utiliser le décorateur `@inject`.
:::

## Utilisation de l'injection de méthode

L'injection de méthode est utilisée pour injecter des dépendances à l'intérieur d'une méthode de classe. Pour que l'injection de méthode fonctionne, vous devez placer le décorateur `@inject` avant la signature de la méthode.

Continuons avec notre exemple précédent et déplaçons la dépendance `EchoService` du constructeur de `HomeController` vers la méthode `handle`.

:::note
Lorsque vous utilisez l'injection de méthode dans un contrôleur, rappelez-vous que le premier paramètre reçoit une valeur fixe (c'est-à-dire le contexte HTTP), et le reste des paramètres est résolu en utilisant le conteneur.
:::

```ts
// title: app/controllers/home_controller.ts
import EchoService from '#services/echo_service'
import { inject } from '@adonisjs/core'

// delete-start
@inject()
// delete-end
export default class HomeController {
  // delete-start
  constructor(private echo: EchoService) {
  }
  // delete-end
  
  // insert-start
  @inject()
  handle(ctx, echo: EchoService) {
    return echo.respond()
  }
  // insert-end
}
```

C'est tout ! Cette fois, l'instance de la classe `EchoService` sera injectée dans la méthode `handle`.

## Quand utiliser l'injection de dépendances

Il est recommandé d'utiliser l'injection de dépendances dans vos projets car elle crée un couplage souple entre les différentes parties de votre application. En conséquence, le code devient plus facile à tester et à refactoriser.

Cependant, vous devez être prudent et ne pas pousser l'idée de l'injection de dépendances à l'extrême au point d'en perdre les avantages. Par exemple :

- Vous ne devriez pas injecter des bibliothèques d'aide comme `lodash` en tant que dépendance de votre classe. Importez-les et utilisez-les directement.
- Votre code pourrait ne pas avoir besoin de couplage pour des composants qui ne sont pas susceptibles d'être échangés ou remplacés. Par exemple, vous pourriez préférer importer le service `logger` plutôt que d'injecter la classe `Logger` en tant que dépendance.

## Utilisation directe du conteneur

La plupart des classes dans votre application AdonisJS, comme les **contrôleurs**, les **middlewares**, les **écouteurs d'événements**, les **validateurs** et les **mailers**, sont construites en utilisant le conteneur. Par conséquent, vous pouvez utiliser le décorateur `@inject` pour l'injection automatique de dépendances.

Pour les situations où vous voulez construire vous-même une instance de classe en utilisant le conteneur, vous pouvez utiliser la méthode `container.make`.

La méthode `container.make` accepte un constructeur de classe et retourne une instance de celle-ci après avoir résolu toutes ses dépendances.

```ts
import { inject } from '@adonisjs/core'
import app from '@adonisjs/core/services/app'

class EchoService {}

@inject()
class SomeService {
  constructor(public echo: EchoService) {}
}

/**
 * Équivalent à la création d'une nouvelle instance de la classe, mais
 * avec l'avantage de l'injection automatique de dépendances
 */
const service = await app.container.make(SomeService)

console.log(service instanceof SomeService)
console.log(service.echo instanceof EchoService)
```

Vous pouvez utiliser la méthode `container.call` pour injecter des dépendances dans une méthode. La méthode `container.call` accepte les arguments suivants :

1. Une instance de la classe.
2. Le nom de la méthode à exécuter sur l'instance de la classe. Le conteneur résoudra les dépendances et les passera à la méthode.
3. Un tableau optionnel de paramètres fixes à passer à la méthode.

```ts
class EchoService {}

class SomeService {
  @inject()
  run(echo: EchoService) {
  }
}

const service = await app.container.make(SomeService)

/**
 * Une instance de la classe Echo sera passée
 * à la méthode run
 */
await app.container.call(service, 'run')
```

## Liaisons du conteneur

Les liaisons du conteneur sont l'une des principales raisons de l'existence du conteneur IoC dans AdonisJS. Les liaisons agissent comme un pont entre les packages que vous installez et votre application.

Les liaisons sont essentiellement une paire clé-valeur, la clé étant l'identifiant unique pour la liaison, et la valeur étant une fonction factory qui renvoie la valeur.

- Le nom de la liaison peut être une `chaîne de caractères`, un `symbole` ou un constructeur de classe.
- La fonction factory peut être asynchrone et doit renvoyer une valeur.

Vous pouvez utiliser la méthode `container.bind` pour enregistrer une liaison du conteneur. Voici un exemple simple d'enregistrement et de résolution de liaisons à partir du conteneur :

```ts
import app from '@adonisjs/core/services/app'

class MyFakeCache {
  get(key: string) {
    return `${key}!`
  }
}

app.container.bind('cache', function () {
  return new MyCache()
})

const cache = await app.container.make('cache')
console.log(cache.get('foo')) // renvoie foo!
```

### Quand utiliser les liaisons du conteneur ?

Les liaisons du conteneur sont utilisées pour des cas d'utilisation spécifiques, comme l'enregistrement de services singleton exportés par un package ou la construction manuelle d'instances de classe lorsque l'injection automatique de dépendances est insuffisante.

Nous vous recommandons de ne pas rendre vos applications inutilement complexes en enregistrant tout dans le conteneur. Au lieu de cela, recherchez des cas d'utilisation spécifiques dans votre code d'application avant de recourir aux liaisons du conteneur.

Voici quelques exemples qui utilisent des liaisons du conteneur dans les packages du framework :

- [Enregistrement de BodyParserMiddleware dans le conteneur](https://github.com/adonisjs/core/blob/main/providers/app_provider.ts#L134-L139) : Comme la classe middleware nécessite une configuration stockée dans le fichier `config/bodyparser.ts`, il n'y a aucun moyen pour que l'injection automatique de dépendances fonctionne. Dans ce cas, nous construisons manuellement l'instance de la classe middleware en l'enregistrant comme une liaison.
- [Enregistrement du service Encryption comme singleton](https://github.com/adonisjs/core/blob/main/providers/app_provider.ts#L97-L100) : La classe Encryption nécessite l'`appKey` stockée dans le fichier `config/app.ts`, nous utilisons donc la liaison du conteneur comme un pont pour lire l'`appKey` de l'application utilisateur et configurer une instance singleton de la classe Encryption.

:::important
Le concept de liaisons du conteneur n'est pas couramment utilisé dans l'écosystème JavaScript. N'hésitez donc pas à [rejoindre notre communauté Discord](https://discord.gg/vDcEjq6) pour clarifier vos doutes.
:::

### Résolution des liaisons à l'intérieur de la fonction factory

Vous pouvez résoudre d'autres liaisons à partir du conteneur dans la fonction factory de liaison. Par exemple, si la classe `MyFakeCache` a besoin de la configuration du fichier `config/cache.ts`, vous pouvez y accéder comme ceci :

```ts
this.app.container.bind('cache', async (resolver) => {
  const configService = await resolver.make('config')
  const cacheConfig = configService.get<any>('cache')

  return new MyFakeCache(cacheConfig)
})
```

### Singletons

Les singletons sont des liaisons pour lesquelles la fonction factory est appelée une seule fois, et la valeur retournée est mise en cache pour la durée de vie de l'application.

Vous pouvez enregistrer une liaison singleton en utilisant la méthode `container.singleton`.

```ts
this.app.container.singleton('cache', async (resolver) => {
  const configService = await resolver.make('config')
  const cacheConfig = configService.get<any>('cache')

  return new MyFakeCache(cacheConfig)
})
```

### Liaison de valeurs

Vous pouvez lier des valeurs directement au conteneur en utilisant la méthode `container.bindValue`.

```ts
this.app.container.bindValue('cache', new MyFakeCache())
```

### Alias

Vous pouvez définir des alias pour les liaisons en utilisant la méthode `alias`. La méthode accepte le nom de l'alias comme premier paramètre et une référence à une liaison existante ou un constructeur de classe comme valeur de l'alias.

```ts
this.app.container.singleton(MyFakeCache, async () => {
  return new MyFakeCache()
})

this.app.container.alias('cache', MyFakeCache)
```

### Définition de types statiques pour les liaisons

Vous pouvez définir les informations de type statique pour la liaison en utilisant la [fusion de déclarations TypeScript](https://www.typescriptlang.org/docs/handbook/declaration-merging.html).

Les types sont définis sur l'interface `ContainerBindings` comme une paire clé-valeur.

```ts
declare module '@adonisjs/core/types' {
  interface ContainerBindings {
    cache: MyFakeCache
  }
}
```

Si vous créez un package, vous pouvez écrire le bloc de code ci-dessus à l'intérieur du fichier du fournisseur de services.

Dans votre application AdonisJS, vous pouvez écrire le bloc de code ci-dessus à l'intérieur du fichier `types/container.ts`.

## Création d'une couche d'abstraction

Le conteneur vous permet de créer une couche d'abstraction pour votre application. Vous pouvez définir une liaison pour une interface et la résoudre en une implémentation concrète.

:::note
Cette méthode est utile lorsque vous voulez appliquer l'architecture hexagonale, également connue sous le nom ports et adaptateurs, à votre application.
:::

Comme les interfaces TypeScript n'existent pas au moment de l'exécution, vous devez utiliser un constructeur de classe abstraite pour votre interface.

```ts
export abstract class PaymentService {
  abstract charge(amount: number): Promise<void>
  abstract refund(amount: number): Promise<void>
}
```

Ensuite, vous pouvez créer une implémentation concrète de l'interface `PaymentService`.

```ts
import { PaymentService } from '#contracts/payment_service'

export class StripePaymentService implements PaymentService {
  async charge(amount: number) {
    // Facturer le montant en utilisant Stripe
  }

  async refund(amount: number) {
    // Rembourser le montant en utilisant Stripe
  }
}
```

Maintenant, vous pouvez enregistrer l'interface `PaymentService` et l'implémentation concrète `StripePaymentService` à l'intérieur du conteneur dans votre `AppProvider`.

```ts
// title: providers/app_provider.ts
import { PaymentService } from '#contracts/payment_service'

export default class AppProvider {
  async boot() {
    const { StripePaymentService } = await import('#services/stripe_payment_service')
    
    this.app.container.bind(PaymentService, () => {
      return this.app.container.make(StripePaymentService)
    })
  }
}
```

Enfin, vous pouvez résoudre l'interface `PaymentService` à partir du conteneur et l'utiliser dans votre application.

```ts
import { PaymentService } from '#contracts/payment_service'

@inject()
export default class PaymentController {
  constructor(private paymentService: PaymentService) {
  }

  async charge() {
    await this.paymentService.charge(100)
    
    // ...
  }
}
```

## Échange d'implémentations pendant les tests

Lorsque vous vous appuyez sur le conteneur pour résoudre un arbre de dépendances, vous avez moins/pas de contrôle sur les classes de cet arbre. Par conséquent, la simulation/falsification de ces classes peut devenir plus difficile.

Dans l'exemple suivant, la méthode `UsersController.index` accepte une instance de la classe `UserService`, et nous utilisons le décorateur `@inject` pour résoudre la dépendance et la donner à la méthode `index`.

```ts
import UserService from '#services/user_service'
import { inject } from '@adonisjs/core'

export default class UsersController {
  @inject()
  index(service: UserService) {}
}
```

Disons que pendant les tests, vous ne voulez pas utiliser le véritable `UserService` car il fait des requêtes HTTP externes. Au lieu de cela, vous voulez utiliser une fausse implémentation.

Mais d'abord, regardez le code que vous pourriez écrire pour tester le `UsersController`.

```ts
import UserService from '#services/user_service'

test('get all users', async ({ client }) => {
  const response = await client.get('/users')

  response.assertBody({
    data: [{ id: 1, username: 'virk' }]
  })
})
```

Dans le test ci-dessus, nous interagissons avec le `UsersController` via une requête HTTP et n'avons pas de contrôle direct sur celui-ci.

Le conteneur fournit une API simple pour échanger des classes avec de fausses implémentations. Vous pouvez définir un échange en utilisant la méthode `container.swap`.

La méthode `container.swap` accepte le constructeur de classe que vous voulez échanger, suivi d'une fonction factory pour retourner une implémentation alternative.

```ts
import UserService from '#services/user_service'
// insert-start
import app from '@adonisjs/core/services/app'
// insert-end

test('get all users', async ({ client }) => {
  // insert-start
  class FakeService extends UserService {
    all() {
      return [{ id: 1, username: 'virk' }]
    }
  }
    
  app.container.swap(UserService, () => {
    return new FakeService()
  })
  // insert-end
  
  const response = await client.get('users')
  response.assertBody({
    data: [{ id: 1, username: 'virk' }]
  })
})
```

Une fois qu'un échange a été défini, le conteneur l'utilisera au lieu de la classe réelle. Vous pouvez restaurer l'implémentation originale en utilisant la méthode `container.restore`.

```ts
app.container.restore(UserService)

// Restaurer UserService et PostService
app.container.restoreAll([UserService, PostService])

// Tout restaurer
app.container.restoreAll()
```

## Dépendances contextuelles

Les dépendances contextuelles vous permettent de définir comment une dépendance doit être résolue pour une classe donnée. Par exemple, vous avez deux services dépendant de la classe Drive Disk.

```ts
import { Disk } from '@adonisjs/drive'

export default class UserService {
  constructor(protected disk: Disk) {}
}
``` 

```ts
import { Disk } from '@adonisjs/drive'

export default class PostService {
  constructor(protected disk: Disk) {}
}
```

Vous voulez que `UserService` reçoive une instance de disk avec le driver GCS et que `PostService` reçoive une instance de disk avec le driver S3. Vous pouvez le faire en utilisant des dépendances contextuelles.

Le code suivant doit être écrit à l'intérieur de la méthode `register` d'un fournisseur de service.

```ts
import { Disk } from '@adonisjs/drive'
import UserService from '#services/user_service'
import PostService from '#services/post_service'
import { ApplicationService } from '@adonisjs/core/types'

export default class AppProvider {
  constructor(protected app: ApplicationService) {}

  register() {
    this.app.container
      .when(UserService)
      .asksFor(Disk)
      .provide(async (resolver) => {
        const driveManager = await resolver.make('drive')
        return drive.use('gcs')
      })

    this.app.container
      .when(PostService)
      .asksFor(Disk)
      .provide(async (resolver) => {
        const driveManager = await resolver.make('drive')
        return drive.use('s3')
      })
  }
}
```

## Hooks du conteneur

Vous pouvez utiliser le hook `resolving` du conteneur pour modifier/étendre la valeur de retour de la méthode `container.make`.

Généralement, vous utiliserez les hooks dans un fournisseur de services lorsque vous essayez d'étendre une liaison particulière. Par exemple, le fournisseur Database utilise le hook `resolving` pour enregistrer des règles de validation supplémentaires basées sur la base de données.

```ts
import { ApplicationService } from '@adonisjs/core/types'

export default class DatabaseProvider {
  constructor(protected app: ApplicationService) {
  }

  async boot() {
    this.app.container.resolving('validator', (validator) => {
      validator.rule('unique', implementation)
      validator.rule('exists', implementation)
    })
  }
}
```

## Événements du conteneur

Le conteneur émet l'événement `container_binding:resolved` après avoir résolu une liaison ou construit une instance de classe. La propriété `event.binding` sera une chaîne de caractères (nom de la liaison) ou un constructeur de classe, et la propriété `event.value` est la valeur résolue.

```ts
import emitter from '@adonisjs/core/services/emitter'

emitter.on('container_binding:resolved', (event) => {
  console.log(event.binding)
  console.log(event.value)
})
```

## Voir aussi

- [Le fichier README du conteneur](https://github.com/adonisjs/fold/blob/develop/README.md) couvre l'API du conteneur de manière indépendante du framework.
- [Pourquoi avez-vous besoin d'un conteneur IoC ?](https://github.com/thetutlage/meta/discussions/4) Dans cet article, le créateur du framework partage son raisonnement pour l'utilisation du conteneur IoC.
