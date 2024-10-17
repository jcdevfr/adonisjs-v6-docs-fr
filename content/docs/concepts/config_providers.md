---
summary: Découvrez les fournisseurs de configuration et comment ils permettent de générer la configuration à la demande après le démarrage de l'application.
---

# Fournisseurs de configuration

Certains fichiers de configuration comme (`config/hash.ts`) n'exportent pas la configuration sous forme d'objet simple. Au lieu de cela, ils exportent un [fournisseur de configuration](https://github.com/adonisjs/core/blob/main/src/config_provider.ts#L16). Le fournisseur de configuration offre une API transparente permettant aux packages de déterminer à la demande la configuration après le démarrage de l'application.

## Sans fournisseur de configuration

Pour comprendre les fournisseurs de configuration, voyons à quoi ressemblerait le fichier `config/hash.ts` si nous n'utilisions pas de fournisseur de configuration.

```ts
import { Scrypt } from '@adonisjs/core/hash/drivers/scrypt'

export default {
  default: 'scrypt',
  list: {
    scrypt: () => new Scrypt({
      cost: 16384,
      blockSize: 8,
      parallelization: 1,
      maxMemory: 33554432,
    })
  }
}
```

Jusqu'ici, tout va bien. Au lieu de référencer le driver `scrypt` depuis la collection de `drivers`, nous l'importons directement et retournons une instance en utilisant une fonction factory.

Imaginons que le driver `Scrypt` ait besoin d'une instance de la classe Emitter pour émettre un événement chaque fois qu'il hache une valeur.

```ts
import { Scrypt } from '@adonisjs/core/hash/drivers/scrypt'
// insert-start
import emitter from '@adonisjs/core/services/emitter'
// insert-end

export default {
  default: 'scrypt',
  list: {
    scrypt: () => new Scrypt({
      cost: 16384,
      blockSize: 8,
      parallelization: 1,
      maxMemory: 33554432,
    // insert-start
    }, emitter)
    // insert-end
  }
}
```
**🚨 L'exemple ci-dessus échouera** car les [services du conteneur](./container_services.md) AdonisJS ne sont pas disponibles tant que l'application n'a pas été démarrée, et les fichiers de configuration sont importés avant la phase de démarrage de l'application.

### Eh bien, c'est un problème avec l'architecture d'AdonisJS 🤷🏻‍♂️

Pas vraiment. N'utilisons pas le service du conteneur et créons directement une instance de la classe Emitter dans le fichier de configuration.

```ts
import { Scrypt } from '@adonisjs/core/hash/drivers/scrypt'
// delete-start
import emitter from '@adonisjs/core/services/emitter'
// delete-end
// insert-start
import { Emitter } from '@adonisjs/core/events'
// insert-end

// insert-start
const emitter = new Emitter()
// insert-end

export default {
  default: 'scrypt',
  list: {
    scrypt: () => new Scrypt({
      cost: 16384,
      blockSize: 8,
      parallelization: 1,
      maxMemory: 33554432,
    }, emitter)
  }
}
```

Maintenant, nous avons un nouveau problème. L'instance d'`emitter` que nous avons créée pour le driver `Scrypt` n'est pas globalement disponible pour que nous puissions l'importer et écouter les événements émis par le driver.

Par conséquent, vous pourriez vouloir déplacer la construction de la classe `Emitter` dans son propre fichier et exporter une instance. De cette façon, vous pouvez passer l'instance d'emitter au driver et l'utiliser pour écouter les événements.

```ts
// title: start/emitter.ts
import { Emitter } from '@adonisjs/core/events'
export const emitter = new Emitter()
```

```ts
import { Scrypt } from '@adonisjs/core/hash/drivers/scrypt'
// delete-start
import { Emitter } from '@adonisjs/core/events'
// delete-end
// insert-start
import { emitter } from '#start/emitter'
// insert-end

// delete-start
const emitter = new Emitter()
// delete-end

export default {
  default: 'scrypt',
  list: {
    scrypt: () => new Scrypt({
      cost: 16384,
      blockSize: 8,
      parallelization: 1,
      maxMemory: 33554432,
    }, emitter)
  }
}
```

Le code ci-dessus fonctionnera bien. Cependant, cette fois, vous construisez manuellement les dépendances dont votre application a besoin. En conséquence, votre application aura beaucoup de code répétitif pour tout relier ensemble.

Avec AdonisJS, nous nous efforçons d'écrire un minimum de code répétitif et d'utiliser le conteneur IoC pour rechercher les dépendances.

## Avec un fournisseur de configuration

Maintenant, réécrivons le fichier `config/hash.ts` en utilisant cette fois un fournisseur de configuration. Un fournisseur de configuration est un nom élégant pour une fonction qui accepte une instance de la classe [Application](./application.md) et résout ses dépendances en utilisant le conteneur.

```ts
// highlight-start
import { configProvider } from '@adonisjs/core'
// highlight-end
import { Scrypt } from '@adonisjs/core/hash/drivers/scrypt'

export default {
  default: 'scrypt',
  list: {
    // highlight-start
    scrypt: configProvider.create(async (app) => {
      const emitter = await app.container.make('emitter')

      return () => new Scrypt({
        cost: 16384,
        blockSize: 8,
        parallelization: 1,
        maxMemory: 33554432,
      }, emitter)
    })
    // highlight-end
  }
}
```

Une fois que vous utilisez le service [hash](../security/hashing.md), le fournisseur de configuration pour le driver `scrypt` sera exécuté pour résoudre ses dépendances. Par conséquent, nous n'essayons pas de rechercher l'`emitter` tant que nous n'utilisons pas le service hash ailleurs dans notre code.

Comme les fournisseurs de configuration sont asynchrones, vous voudrez peut-être importer le driver `Scrypt` à la demande via une importation dynamique.

```ts
import { configProvider } from '@adonisjs/core'
// delete-start
import { Scrypt } from '@adonisjs/core/hash/drivers/scrypt'
// delete-end

export default {
  default: 'scrypt',
  list: {
    scrypt: configProvider.create(async (app) => {
      // insert-start
      const { Scrypt } = await import('@adonisjs/core/hash/drivers/scrypt')
      // insert-end
      const emitter = await app.container.make('emitter')

      return () => new Scrypt({
        cost: 16384,
        blockSize: 8,
        parallelization: 1,
        maxMemory: 33554432,
      }, emitter)
    })
  }
}
```

## Comment accéder à la configuration résolue ?

Vous pouvez accéder à la configuration résolue directement depuis le service. Par exemple, dans le cas du service hash, vous pouvez obtenir une référence à la configuration résolue de la manière suivante :

```ts
import hash from '@adonisjs/core/services/hash'
console.log(hash.config)
```
