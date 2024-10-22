---
summary: Découvrez AsyncLocalStorage et comment l'utiliser dans AdonisJS.
---

# Stockage local asynchrone

Selon la [documentation officielle de Node.js](https://nodejs.org/docs/latest-v21.x/api/async_context.html#class-asynclocalstorage) : “AsyncLocalStorage est utilisé pour créer un état asynchrone au sein des fonctions de rappel et des chaînes de promesses. **Il permet de stocker des données tout au long de la durée de vie d'une requête web ou de toute autre durée asynchrone. C'est similaire au stockage local de thread dans d'autres langages**.“

Pour simplifier davantage l'explication, AsyncLocalStorage vous permet de stocker un état lors de l'exécution d'une fonction asynchrone et de le rendre disponible pour tous les flux d'exécution au sein de cette fonction.

## Exemple basique

Voyons cela en action. Tout d'abord, nous allons créer un nouveau projet Node.js (sans dépendances) et utiliser `AsyncLocalStorage` pour partager l'état entre les modules sans le passer par référence.

:::note
Vous pouvez trouver le code final de cet exemple sur le dépôt GitHub [als-basic-example](https://github.com/thetutlage/als-basic-example).
:::

### Étape 1. Créer un nouveau projet

```sh
npm init --yes
```
Ouvrez le fichier `package.json` et définissez le système de modules sur ESM.

```json
{
  "type": "module"
}
```

### Étape 2. Créer une instance d'`AsyncLocalStorage`

Créez un fichier nommé `storage.js`, qui crée et exporte une instance d'`AsyncLocalStorage`.

```ts
// title: storage.js
import { AsyncLocalStorage } from 'async_hooks'
export const storage = new AsyncLocalStorage()
```

### Étape 3. Exécuter du code à l'intérieur de `storage.run`

Créez un fichier d'entrée nommé `main.js`. Dans ce fichier, importez l'instance d'`AsyncLocalStorage` créée dans le fichier `./storage.js`.

La méthode `storage.run` accepte l'état que nous voulons partager comme premier argument et une fonction de rappel comme second argument. Tous les flux d'exécution à l'intérieur de cette fonction de rappel (y compris les modules importés) auront accès au même état.

```ts
// title: main.js
import { storage } from './storage.js'
import UserService from './user_service.js'
import { setTimeout } from 'node:timers/promises'

async function run(user) {
  const state = { user }

  return storage.run(state, async () => {
    await setTimeout(100)
    const userService = new UserService()
    await userService.get()
  })
}
```

Pour la démonstration, nous exécuterons la méthode `run` trois fois sans l'attendre. Collez le code suivant à la fin du fichier `main.js`.

```ts
// title: main.js
run({ id: 1 })
run({ id: 2 })
run({ id: 3 })
```

### Étape 4. Accéder à l'état depuis le module `user_service`

Enfin, importons l'instance de storage dans le module `user_service` et accédons à l'état actuel.

```ts
// title: user_service.js
import { storage } from './storage.js'

export default class UserService {
  async get() {
    const state = storage.getStore()
    console.log(`The user id is ${state.user.id}`)
  }
}
```

### Étape 5. Exécuter le fichier `main.js`

Exécutons le fichier `main.js` pour voir si le `UserService` peut accéder à l'état.

```sh
node main.js
```

## Quel est le besoin d'un stockage local asynchrone ?

Contrairement à d'autres langages comme PHP, Node.js n'est pas un langage threadé. Dans PHP, chaque requête HTTP crée un nouveau thread, et chaque thread a sa propre mémoire. Cela vous permet de stocker l'état dans la mémoire globale et d'y accéder n'importe où dans votre code.

Dans Node.js, vous ne pouvez pas avoir un état global isolé entre les requêtes HTTP car Node.js fonctionne sur un seul thread et a une mémoire partagée. Par conséquent, toutes les applications Node.js partagent des données en les passant comme paramètres.

Passer des données par référence n'a pas d'inconvénients techniques. Mais cela rend le code verbeux, surtout lorsque vous configurez des outils APM et que vous devez leur fournir manuellement les données de requête.

## Utilisation

AdonisJS utilise `AsyncLocalStorage` pendant les requêtes HTTP et partage le [contexte HTTP](./http_context.md) comme état. Par conséquent, vous pouvez accéder au contexte HTTP dans votre application globalement.

Tout d'abord, vous devez activer le flag `useAsyncLocalStorage` dans le fichier `config/app.ts`.

```ts
// title: config/app.ts
export const http = defineConfig({
  useAsyncLocalStorage: true,
})
```

Une fois activé, vous pouvez utiliser les méthodes `HttpContext.get` ou `HttpContext.getOrFail` pour obtenir une instance du contexte HTTP pour la requête en cours.

Dans l'exemple suivant, nous obtenons le contexte à l'intérieur d'un modèle Lucid.

```ts
import { HttpContext } from '@adonisjs/core/http'
import { BaseModel } from '@adonisjs/lucid'

export default class Post extends BaseModel {
  get isLiked() {
    const ctx = HttpContext.getOrFail()
    const authUserId = ctx.auth.user.id
    
    return !!this.likes.find((like) => {
      return like.userId === authUserId
    })
  }
}
```

## Mises en garde

Vous pouvez utiliser ALS si cela rend votre code plus simple et si vous préférez l'accès global plutôt que de passer le contexte HTTP par référence.

Cependant, soyez conscient des situations suivantes qui peuvent facilement conduire à des fuites de mémoire ou à un comportement instable du programme.

### Accès au niveau supérieur

N'accédez pas à l'ALS au niveau supérieur d'un module car les modules dans Node.js sont mis en cache.

:::caption{for="error"}
**Utilisation incorrecte**\
Assigner le résultat de la méthode `HttpContext.getOrFail()` à une variable au niveau supérieur conservera la référence à la requête qui a importé le module en premier.
:::


```ts
import { HttpContext } from '@adonisjs/core/http'
const ctx = HttpContext.getOrFail()

export default class UsersController {
  async index() {
    ctx.request
  }
}
```

:::caption[]{for="success"}
**Utilisation correcte**\
Au lieu de cela, vous devriez déplacer l'appel de la méthode `getOrFail` à l'intérieur de la méthode `index`.
:::

```ts
import { HttpContext } from '@adonisjs/core/http'

export default class UsersController {
  async index() {
    const ctx = HttpContext.getOrFail()
  }
}
```

### À l'intérieur des propriétés statiques

Les propriétés statiques (pas les méthodes) de toute classe sont évaluées dès que le module est importé ; par conséquent, vous ne devez pas accéder au contexte HTTP dans les propriétés statiques.

:::caption{for="error"}
**Utilisation incorrecte**
:::

```ts
import { HttpContext } from '@adonisjs/core/http'
import { BaseModel } from '@adonisjs/lucid'

export default class User extends BaseModel {
  static connection = HttpContext.getOrFail().tenant.name
}
```

:::caption[]{for="success"}
**Utilisation correcte**\
Au lieu de cela, vous devriez déplacer l'appel de `HttpContext.get` à l'intérieur d'une méthode ou convertir la propriété en getter.
:::

```ts
import { HttpContext } from '@adonisjs/core/http'
import { BaseModel } from '@adonisjs/lucid'

export default class User extends BaseModel {
  static query() {
    const ctx = HttpContext.getOrFail()
    return super.query({ connection: tenant.connection })
  }
}
```

### Gestionnaires d'événements

Les gestionnaires d'événements sont exécutés après la fin de la requête HTTP. Par conséquent, vous devez vous abstenir d'essayer d'accéder au contexte HTTP à l'intérieur de ceux-ci.

```ts
import emitter from '@adonisjs/core/services/emitter'

export default class UsersController {
  async index() {
    const user = await User.create({})
    emitter.emit('new:user', user)
  }
}
```

:::caption[]{for="error"}
**Évitez l'utilisation à l'intérieur des écouteurs d'événements**
:::

```ts
import { HttpContext } from '@adonisjs/core/http'
import emitter from '@adonisjs/core/services/emitter'

emitter.on('new:user', () => {
  const ctx = HttpContext.getOrFail()
})
```
