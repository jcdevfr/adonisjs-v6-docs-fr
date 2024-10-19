---
summary: Découvrez les services du conteneur et comment ils contribuent à garder votre code propre et testable.
---

# Services du conteneur

Comme nous l'avons vu dans le [guide sur le conteneur IoC](./dependency_injection.md#container-bindings), les liaisons du conteneur sont l'une des principales raisons d'être du conteneur IoC dans AdonisJS.

Les liaisons du conteneur permettent de garder votre code propre en évitant le code répétitif nécessaire pour construire des objets avant de pouvoir les utiliser.

Dans l'exemple suivant, avant de pouvoir utiliser la classe `Database`, vous devrez en créer une instance. Selon la classe que vous construisez, vous devrez peut-être écrire beaucoup de code répétitif pour obtenir toutes ses dépendances.

```ts
import { Database } from '@adonisjs/lucid'
export const db = new Database(
  // injecter la configuration et d'autres dépendances
)
```

Cependant, en utilisant un conteneur IoC, vous pouvez déléguer la tâche de construction d'une classe au conteneur et récupérer une instance pré-construite.

```ts
import app from '@adonisjs/core/services/app'
const db = await app.container.make('lucid.db')
```

## La nécessité des services du conteneur

Utiliser le conteneur pour résoudre des objets pré-configurés est excellent. Cependant, l'utilisation de la méthode `container.make` présente ses propres inconvénients.

- Les éditeurs sont bons avec les importations automatiques. Si vous essayez d'utiliser une variable et que l'éditeur peut deviner le chemin d'importation de la variable, il écrira alors la déclaration d'importation pour vous. **Cependant, cela ne peut pas fonctionner avec les appels à `container.make`**.

- Utiliser un mélange de déclarations d'importation et d'appels à `container.make` semble peu intuitif par rapport à une syntaxe unifiée pour importer/utiliser des modules.

Pour surmonter ces inconvénients, nous enveloppons les appels à `container.make` dans un module JavaScript ordinaire, afin que vous puissiez les récupérer à l'aide de l'instruction `import`.

Par exemple, le package `@adonisjs/lucid` a un fichier appelé `services/db.ts` et ce fichier contient à peu près le contenu suivant.

```ts
const db = await app.container.make('lucid.db')
export { db as default }
```

Dans votre application, vous pouvez remplacer l'appel à `container.make` par une importation pointant vers le fichier `services/db.ts`.

```ts
// delete-start
import app from '@adonisjs/core/services/app'
const db = await app.container.make('lucid.db')
// delete-end
// insert-start
import db from '@adonisjs/lucid/services/db'
// insert-end
```

Comme vous pouvez le voir, nous nous appuyons toujours sur le conteneur pour résoudre une instance de la classe Database pour nous. Cependant, avec une couche d'indirection, nous pouvons remplacer l'appel à `container.make` par une instruction `import` ordinaire.

**Le module JavaScript enveloppant les appels à `container.make` est connu sous le nom de service du conteneur**. Presque tous les packages qui interagissent avec le conteneur sont livrés avec un ou plusieurs services du conteneur.

## Services du conteneur vs. Injection de dépendances

Les services du conteneur sont une alternative à l'injection de dépendances. Par exemple, au lieu d'accepter la classe `Disk` comme dépendance, vous demandez au service `drive` de vous donner une instance de disk. Examinons quelques exemples de code.

Dans l'exemple suivant, nous utilisons le décorateur `@inject` pour injecter une instance de la classe `Disk`.

```ts
import { Disk } from '@adonisjs/drive'
import { inject } from '@adonisjs/core'

  // highlight-start
@inject()
export default class PostService {
  constructor(protected disk: Disk) {
  }
  // highlight-end  

  async save(post: Post, coverImage: File) {
    const coverImageName = 'random_name.jpg'

    // highlight-start
    await this.disk.put(coverImageName, coverImage)
    // highlight-end
    
    post.coverImage = coverImageName
    await post.save()
  }
}
```

Lorsque nous utilisons le service `drive`, nous appelons la méthode `drive.use` pour obtenir une instance de `Disk` avec le driver `s3`.

```ts
import drive from '@adonisjs/drive/services/main'

export default class PostService {
  async save(post: Post, coverImage: File) {
    const coverImageName = 'random_name.jpg'

    // highlight-start
    const disk = drive.use('s3')
    await disk.put(coverImageName, coverImage)
    // highlight-end
    
    post.coverImage = coverImageName
    await post.save()
  }
}
```

Les services du conteneur sont excellents pour garder votre code concis. D'autre part, l'injection de dépendances crée un couplage entre les différentes parties de l'application.

Choisir l'un plutôt que l'autre dépend de votre choix personnel et de l'approche que vous souhaitez adopter pour structurer votre code.

## Tests avec les services du conteneur

L'avantage évident de l'injection de dépendances est la possibilité d'échanger les dépendances au moment d'écrire les tests.

Pour offrir une expérience de test similaire avec les services du conteneur, AdonisJS fournit des API de première classe pour simuler des implémentations lors de l'écriture de tests.

Dans l'exemple suivant, nous appelons la méthode `drive.fake` pour remplacer les disques drive par un driver en mémoire. Après la création d'une simulation, tout appel à la méthode `drive.use` recevra l'implémentation factice.

```ts
import drive from '@adonisjs/drive/services/main'
import PostService from '#services/post_service'

test('save post', async ({ assert }) => {
  /**
   * Simuler le disque s3
   */
  drive.fake('s3')
 
  const postService = new PostService()
  await postService.save(post, coverImage)
  
  /**
   * Écrire les assertions
   */
  assert.isTrue(await drive.use('s3').exists(coverImage.name))
  
  /**
   * Restaurer le disque
   */
  drive.restore('s3')
})
```

## Liaisons du conteneur et services

Le tableau suivant présente une liste des liaisons du conteneur et de leurs services associés exportés par le noyau du framework et les packages internes.

<table>
  <thead>
    <tr>
      <th width="100px">Liaison</th>
      <th width="140px">Classe</th>
      <th>Service</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <code>app</code>
      </td>
      <td>
        <a href="https://github.com/adonisjs/application/blob/main/src/application.ts">Application</a>
      </td>
      <td>
        <code>@adonisjs/core/services/app</code>
      </td>
    </tr>
    <tr>
      <td>
        <code>ace</code>
      </td>
      <td>
        <a href="https://github.com/adonisjs/core/blob/main/modules/ace/kernel.ts">Kernel</a>
      </td>
      <td>
        <code>@adonisjs/core/services/kernel</code>
      </td>
    </tr>
    <tr>
      <td>
        <code>config</code>
      </td>
      <td>
        <a href="https://github.com/adonisjs/config/blob/main/src/config.ts">Config</a>
      </td>
      <td>
        <code>@adonisjs/core/services/config</code>
      </td>
    </tr>
    <tr>
      <td>
        <code>encryption</code>
      </td>
      <td>
        <a href="https://github.com/adonisjs/encryption/blob/main/src/encryption.ts">Encryption</a>
      </td>
      <td>
        <code>@adonisjs/core/services/encryption</code>
      </td>
    </tr>
    <tr>
      <td>
        <code>emitter</code>
      </td>
      <td>
        <a href="https://github.com/adonisjs/events/blob/main/src/emitter.ts">Emitter</a>
      </td>
      <td>
        <code>@adonisjs/core/services/emitter</code>
      </td>
    </tr>
    <tr>
      <td>
        <code>hash</code>
      </td>
      <td>
        <a href="https://github.com/adonisjs/hash/blob/main/src/hash_manager.ts">HashManager</a>
      </td>
      <td>
        <code>@adonisjs/core/services/hash</code>
      </td>
    </tr>
    <tr>
      <td>
        <code>logger</code>
      </td>
      <td>
        <a href="https://github.com/adonisjs/logger/blob/main/src/logger_manager.ts">LoggerManager</a>
      </td>
      <td>
        <code>@adonisjs/core/services/logger</code>
      </td>
    </tr>
    <tr>
      <td>
        <code>repl</code>
      </td>
      <td>
        <a href="https://github.com/adonisjs/repl/blob/main/src/repl.ts">Repl</a>
      </td>
      <td>
        <code>@adonisjs/core/services/repl</code>
      </td>
    </tr>
    <tr>
      <td>
        <code>router</code>
      </td>
      <td>
        <a href="https://github.com/adonisjs/http-server/blob/main/src/router/main.ts">Router</a>
      </td>
      <td>
        <code>@adonisjs/core/services/router</code>
      </td>
    </tr>
    <tr>
      <td>
        <code>server</code>
      </td>
      <td>
        <a href="https://github.com/adonisjs/http-server/blob/main/src/server/main.ts">Server</a>
      </td>
      <td>
        <code>@adonisjs/core/services/server</code>
      </td>
    </tr>
    <tr>
      <td>
        <code> testUtils</code>
      </td>
      <td>
        <a href="https://github.com/adonisjs/core/blob/main/src/test_utils/main.ts">TestUtils</a>
      </td>
      <td>
        <code>@adonisjs/core/services/test_utils</code>
      </td>
    </tr>
  </tbody>
</table>
