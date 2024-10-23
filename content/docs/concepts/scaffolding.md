---
summary: Générer des fichiers sources à partir de templates et mettre à jour le code source TypeScript à l'aide de codemods dans AdonisJS
---

# Scaffolding et codemods

Le scaffolding (échaffaudage) fait référence au processus de génération de fichiers sources à partir de modèles statiques (aussi appelés stubs), et les codemods font référence à la mise à jour du code source TypeScript en analysant l'AST.

AdonisJS utilise les deux pour accélérer les tâches répétitives de création de nouveaux fichiers et de configuration des packages. Dans ce guide, nous allons passer en revue les éléments de base du scaffolding et couvrir l'API des codemods que vous pouvez utiliser dans les commandes Ace.

## Éléments de base

### Stubs

Les stubs font référence aux templates qui sont utilisés pour créer des fichiers sources lors d'une action spécifique. Par exemple, la commande `make:controller` utilise le [stub de contrôleur](https://github.com/adonisjs/core/blob/main/stubs/make/controller/main.stub) pour créer un fichier de contrôleur dans le projet hôte.

### Générateurs

Les générateurs imposent une convention de nommage et génèrent des noms de fichiers, de classes ou de méthodes basés sur des conventions prédéfinies.

Par exemple, les stubs de contrôleur utilisent les générateurs [controllerName](https://github.com/adonisjs/application/blob/main/src/generators.ts#L122) et [controllerFileName](https://github.com/adonisjs/application/blob/main/src/generators.ts#L139) pour créer un contrôleur.

Comme les générateurs sont définis comme un objet, vous pouvez remplacer les méthodes existantes pour ajuster les conventions. Nous en apprendrons plus à ce sujet plus tard dans ce guide.

### Codemods

L'API des codemods provient du package [@adonisjs/assembler](https://github.com/adonisjs/assembler/blob/main/src/code_transformer/main.ts) et utilise [ts-morph](https://github.com/dsherret/ts-morph) en interne.

Étant donné que `@adonisjs/assembler` est une dépendance de développement, `ts-morph` n'alourdit pas les dépendances de votre projet en production. Cela signifie également que les API de codemods ne sont pas disponibles en production.

Les API de codemods exposées par AdonisJS sont très spécifiques pour accomplir des tâches de haut niveau comme **ajouter un fournisseur au fichier `.adonisrc.ts`**, ou **enregistrer un middleware dans le fichier `start/kernel.ts`**. De plus, ces API reposent sur les conventions de nommage par défaut, donc si vous apportez des modifications drastiques à votre projet, vous ne pourrez pas exécuter les codemods.

### Commande configure

La commande configure est utilisée pour configurer un package AdonisJS. Sous le capot, cette commande importe le fichier d'entrée principal et exécute la méthode `configure` exportée par le package mentionné.

La méthode `configure` du package reçoit une instance de la commande [Configure](https://github.com/adonisjs/core/blob/main/commands/configure.ts), et peut donc accéder directement aux API de stubs et de codemods à partir de l'instance de la commande.

## Utilisation des stubs

La plupart du temps, vous utiliserez les stubs dans une commande Ace ou à l'intérieur de la méthode `configure` d'un package que vous avez créé. Vous pouvez initialiser le module codemods dans les deux cas via la méthode `createCodemods` de la commande Ace.

La méthode `codemods.makeUsingStub` crée un fichier source à partir d'un template de stub. Elle accepte les arguments suivants :

- L'URL vers la racine du répertoire où les stubs sont stockés.
- Le chemin relatif depuis le répertoire `STUBS_ROOT` vers le fichier stub (y compris l'extension).
- Et l'objet de données à partager avec le stub.

```ts
// title: Dans une commande
import { BaseCommand } from '@adonisjs/core/ace'

const STUBS_ROOT = new URL('./stubs', import.meta.url)

export default class MakeApiResource extends BaseCommand {
  async run() {
    // highlight-start
    const codemods = await this.createCodemods()
    await codemods.makeUsingStub(STUBS_ROOT, 'api_resource.stub', {})
    // highlight-end
  }
}
```

### Templating des stubs

Nous utilisons le moteur de template [Tempura](https://github.com/lukeed/tempura) pour traiter les stubs avec des données d'exécution. Tempura est un moteur de template très léger comme handlebars pour JavaScript.

:::tip
Comme la syntaxe de Tempura est compatible avec handlebars, vous pouvez configurer vos éditeurs de code pour utiliser la coloration syntaxique de handlebars avec les fichiers `.stub`.
:::

Dans l'exemple suivant, nous créons un stub qui génère une classe JavaScript. Il utilise les doubles accolades pour évaluer les valeurs d'exécution.

```handlebars
export default class {{ modelName }}Resource {
  serialize({{ modelReference }}: {{ modelName }}) {
    return {{ modelReference }}.toJSON()
  }
}
```

### Utilisation des générateurs

Si vous exécutez le stub ci-dessus maintenant, il échouera car nous n'avons pas fourni les propriétés des données `modelName` et `modelReference`.

Nous recommandons de calculer ces propriétés dans le stub en utilisant des variables inline. De cette façon, l'application hôte peut [éjecter le stub](#ejecting-stubs) et modifier les variables.

```js
// insert-start
{{#var entity = generators.createEntity('user')}}
{{#var modelName = generators.modelName(entity.name)}}
{{#var modelReference = string.toCamelCase(modelName)}}
// insert-end

export default class {{ modelName }}Resource {
  serialize({{ modelReference }}: {{ modelName }}) {
    return {{ modelReference }}.toJSON()
  }
}
```

### Destination du résultat

Enfin, nous devons spécifier le chemin de destination du fichier qui sera créé en utilisant le stub. Une fois de plus, nous spécifions le chemin de destination dans le fichier stub, car cela permet à l'application hôte d'[éjecter le stub](#ejecting-stubs) et de personnaliser la destination du résultat.

Le chemin de destination est défini en utilisant la fonction `exports`. La fonction accepte un objet et l'exporte comme état de sortie du stub. Plus tard, l'API codemods utilise cet objet pour créer le fichier à l'emplacement mentionné.

```js
{{#var entity = generators.createEntity('user')}}
{{#var modelName = generators.modelName(entity.name)}}
{{#var modelReference = string.toCamelCase(modelName)}}
// insert-start
{{#var resourceFileName = string(modelName).snakeCase().suffix('_resource').ext('.ts').toString()}}
{{{
  exports({
    to: app.makePath('app/api_resources', entity.path, resourceFileName)
  })
}}}
// insert-end
export default class {{ modelName }}Resource {
  serialize({{ modelReference }}: {{ modelName }}) {
    return {{ modelReference }}.toJSON()
  }
}
```

### Accepter le nom de l'entité via la commande

Pour l'instant, nous avons codé en dur le nom de l'entité comme `user` dans le stub. Cependant, vous devriez l'accepter comme argument de commande et le partager avec le stub comme état du template.

```ts
import { BaseCommand, args } from '@adonisjs/core/ace'

export default class MakeApiResource extends BaseCommand {
  // insert-start
  @args.string({
    description: 'The name of the resource'
  })
  declare name: string
  // insert-end

  async run() {
    const codemods = await this.createCodemods()
    await codemods.makeUsingStub(STUBS_ROOT, 'api_resource.stub', {
      // insert-start
      name: this.name,
      // insert-end
    })
  }
}
```

```js
// delete-start
{{#var entity = generators.createEntity('user')}}
// delete-end
// insert-start
{{#var entity = generators.createEntity(name)}}
// insert-end
{{#var modelName = generators.modelName(entity.name)}}
{{#var modelReference = string.toCamelCase(modelName)}}
{{#var resourceFileName = string(modelName).snakeCase().suffix('_resource').ext('.ts').toString()}}
{{{
  exports({
    to: app.makePath('app/api_resources', entity.path, resourceFileName)
  })
}}}
export default class {{ modelName }}Resource {
  serialize({{ modelReference }}: {{ modelName }}) {
    return {{ modelReference }}.toJSON()
  }
}
```

### Variables globales

Les variables globales suivantes sont toujours partagées avec un stub.

| Variable       | Description                                                                                                                                                         |
|----------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `app`          | Référence à une instance de la [classe Application](./application.md).                                                                                          |
| `generators`   | Référence au [module des générateurs](https://github.com/adonisjs/application/blob/main/src/generators.ts).                                                          |
| `randomString` | Référence à la fonction helper [randomString](../references/helpers.md#random).                                                                       |
| `string`       | Une fonction pour créer une instance de [string builder](../references/helpers.md#string-builder). Vous pouvez utiliser le string builder pour appliquer des transformations sur une chaîne de caractères. |
| `flags`        | Les flags de ligne de commande sont définis lors de l'exécution de la commande ace.                                                                                        |


## Éjecter les stubs

Vous pouvez éjecter/copier des stubs dans une application AdonisJS en utilisant la commande `node ace eject`. La commande eject accepte un chemin vers le fichier stub original ou son répertoire parent et copie les templates dans le répertoire `stubs` à la racine de votre projet.

Dans l'exemple suivant, nous allons copier le fichier `make/controller/main.stub` du package `@adonisjs/core`.

```sh
node ace eject make/controller/main.stub
```

Si vous ouvrez le fichier stub, il aura le contenu suivant.

```js
{{#var controllerName = generators.controllerName(entity.name)}}
{{#var controllerFileName = generators.controllerFileName(entity.name)}}
{{{
  exports({
    to: app.httpControllersPath(entity.path, controllerFileName)
  })
}}}
// import type { HttpContext } from '@adonisjs/core/http'

export default class {{ controllerName }} {
}
```

- Dans les deux premières lignes, nous utilisons le [module generators](https://github.com/adonisjs/application/blob/main/src/generators.ts) pour générer le nom de la classe du contrôleur et le nom du fichier du contrôleur.
- Des lignes 3 à 7, nous [définissons le chemin de destination](#utiliser-les-flags-cli-pour-personnaliser-la-destination-de-sortie-du-stub) pour le fichier du contrôleur en utilisant la fonction `exports`.
- Enfin, nous définissons le contenu du contrôleur généré.

N'hésitez pas à modifier le stub. La prochaine fois, les changements seront pris en compte lorsque vous exécuterez la commande `make:controller`.

### Éjecter des répertoires

Vous pouvez éjecter un répertoire entier de stubs en utilisant la commande `eject`. Passez le chemin vers le répertoire, et la commande copiera le répertoire entier.

```sh
# Copie tous les stubs make
node ace eject make

# Copie tous les stubs make:controller
node ace eject make/controller
```

### Utiliser les flags CLI pour personnaliser la destination de sortie du stub

Toutes les commandes de scaffolding partagent les flags CLI (y compris ceux non pris en charge) avec les templates de stub. Par conséquent, vous pouvez les utiliser pour créer des workflows personnalisés ou changer la destination du résultat.

Dans l'exemple suivant, nous utilisons le flag `--feature` pour créer un contrôleur dans le répertoire de fonctionnalités mentionné.

```sh
node ace make:controller invoice --feature=billing
```

```js
// title: Stub du contrôleur
{{#var controllerName = generators.controllerName(entity.name)}}
// insert-start
{{#var featureDirectoryName = generators.makePath('features', flags.feature)}}
// insert-end
{{#var controllerFileName = generators.controllerFileName(entity.name)}}
{{{
  exports({
    // delete-start
    to: app.httpControllersPath(entity.path, controllerFileName)
    // delete-end
    // insert-start
    to: app.makePath(featureDirectoryName, entity.path, controllerFileName)
    // insert-end
  })
}}}
// import type { HttpContext } from '@adonisjs/core/http'

export default class {{ controllerName }} {
}
```

### Éjecter des stubs d'autres packages

Par défaut, la commande eject copie les templates du package `@adonisjs/core`. Cependant, vous pouvez copier des stubs d'autres packages en utilisant le flag `--pkg`.

```sh
node ace eject make/migration/main.stub --pkg=@adonisjs/lucid
```

### Comment trouver quels stubs copier ?

Vous pouvez trouver les stubs d'un package en visitant son dépôt GitHub. Nous stockons tous les stubs au niveau racine du package dans le répertoire `stubs`.

## Flux d'exécution des stubs

Voici une représentation visuelle de la façon dont nous trouvons et exécutons les stubs via la méthode `makeUsingStub`.

![](./scaffolding_workflow.png)

## API Codemods

L'API codemods est alimentée par [ts-morph](https://github.com/dsherret/ts-morph) et n'est disponible que pendant le développement. Vous pouvez instancier à la demande le module codemods en utilisant la méthode `command.createCodemods`. La méthode `createCodemods` retourne une instance de la classe [Codemods](https://github.com/adonisjs/core/blob/main/modules/ace/codemods.ts).

```ts
import type Configure from '@adonisjs/core/commands/configure'

export async function configure(command: ConfigureCommand) {
  const codemods = await command.createCodemods()
}
```

### defineEnvValidations

Définit des règles de validation pour les variables d'environnement. La méthode accepte une paire clé-valeur de variables. La `clé` est le nom de la variable env, et la `valeur` est l'expression de validation sous forme de chaîne de caractères.

:::note
Ce codemod s'attend à ce que le fichier `start/env.ts` existe et doit avoir l'appel de méthode `export default await Env.create`.

De plus, le codemod ne remplace pas la règle de validation existante pour une variable d'environnement donnée. Cela permet de respecter les modifications apportées dans l'application.
:::

```ts
const codemods = await command.createCodemods()

try {
  await codemods.defineEnvValidations({
    leadingComment: 'App environment variables',
    variables: {
      PORT: 'Env.schema.number()',
      HOST: 'Env.schema.string()',
    }
  })
} catch (error) {
  console.error('Unable to define env validations')
  console.error(error)
}
```

```ts
// title: Résultat
import { Env } from '@adonisjs/core/env'

export default await Env.create(new URL('../', import.meta.url), {
  /**
   * Variables d'environnement de l'application
   */
  PORT: Env.schema.number(),
  HOST: Env.schema.string(),
})
```

### defineEnvVariables

Ajoute une ou plusieurs nouvelles variables d'environnement aux fichiers `.env` et `.env.example`. La méthode accepte une paire clé-valeur de variables.

```ts
const codemods = await command.createCodemods()

try {
  await codemods.defineEnvVariables({
    MY_NEW_VARIABLE: 'some-value',
    MY_OTHER_VARIABLE: 'other-value'
  })
} catch (error) {
  console.error('Unable to define env variables')
  console.error(error)
}
```

Parfois, vous pouvez vouloir **ne pas** insérer la valeur de la variable dans le fichier `.env.example`. Vous pouvez le faire en utilisant l'option `omitFromExample`.

```ts
const codemods = await command.createCodemods()

await codemods.defineEnvVariables({
  MY_NEW_VARIABLE: 'SOME_VALUE',
}, {
  omitFromExample: ['MY_NEW_VARIABLE']
})
```

Le code ci-dessus insérera `MY_NEW_VARIABLE=SOME_VALUE` dans le fichier `.env` et `MY_NEW_VARIABLE=` dans le fichier `.env.example`.

### registerMiddleware

Enregistre un middleware AdonisJS dans l'une des piles de middlewares connues. La méthode accepte la pile de middlewares et un tableau de middlewares à enregistrer.

La pile de middlewares peut être l'une des suivantes : `server | router | named`.

:::note
Ce codemod s'attend à ce que le fichier `start/kernel.ts` existe et doit avoir un appel de fonction pour la pile de middlewares pour laquelle vous essayez d'enregistrer un middleware.
:::

```ts
const codemods = await command.createCodemods()

try {
  await codemods.registerMiddleware('router', [
    {
      path: '@adonisjs/core/bodyparser_middleware'
    }
  ])
} catch (error) {
  console.error('Unable to register middleware')
  console.error(error)
}
```

```ts
// title: Résultat
import router from '@adonisjs/core/services/router'

router.use([
  () => import('@adonisjs/core/bodyparser_middleware')
])
```

Vous pouvez définir des middlewares nommés comme ceci.

```ts
const codemods = await command.createCodemods()

try {
  await codemods.registerMiddleware('named', [
    {
      name: 'auth',
      path: '@adonisjs/auth/auth_middleware'
    }
  ])
} catch (error) {
  console.error('Unable to register middleware')
  console.error(error)
}
```

### updateRcFile

Enregistre des `fournisseurs`, des `commandes`, définit des `metaFiles` et des `alias de commande` dans le fichier `adonisrc.ts`.

:::note
Ce codemod s'attend à ce que le fichier `adonisrc.ts` existe et doit avoir un appel de fonction `export default defineConfig`.
:::

```ts
const codemods = await command.createCodemods()

try {
  await codemods.updateRcFile((rcFile) => {
    rcFile
      .addProvider('@adonisjs/lucid/db_provider')
      .addCommand('@adonisjs/lucid/commands'),
      .setCommandAlias('migrate', 'migration:run')
  })
} catch (error) {
  console.error('Unable to update adonisrc.ts file')
  console.error(error)  
}
```

```ts
// title: Résultat
import { defineConfig } from '@adonisjs/core/app'

export default defineConfig({
  commands: [
    () => import('@adonisjs/lucid/commands')
  ],
  providers: [
    () => import('@adonisjs/lucid/db_provider')
  ],
  commandAliases: {
    migrate: 'migration:run'
  }
})
```

### registerJapaPlugin

Enregistre un plugin Japa dans le fichier `tests/bootstrap.ts`.

:::note
Ce codemod s'attend à ce que le fichier `tests/bootstrap.ts` existe et doit avoir l'export `export const plugins: Config['plugins']`.
:::

```ts
const codemods = await command.createCodemods()

const imports = [
  {
    isNamed: false,
    module: '@adonisjs/core/services/app',
    identifier: 'app'
  },
  {
    isNamed: true,
    module: '@adonisjs/session/plugins/api_client',
    identifier: 'sessionApiClient'
  }
]
const pluginUsage = 'sessionApiClient(app)'

try {
  await codemods.registerJapaPlugin(pluginUsage, imports)
} catch (error) {
  console.error('Unable to register japa plugin')
  console.error(error)
}
```

```ts
// title: Résultat
import app from '@adonisjs/core/services/app'
import { sessionApiClient } from '@adonisjs/session/plugins/api_client'

export const plugins: Config['plugins'] = [
  sessionApiClient(app)
]
```

### registerPolicies

Enregistre des politiques d'accès d'AdonisJS dans l'objet `policies` exporté depuis le fichier `app/policies/main.ts`.

:::note
Ce codemod s'attend à ce que le fichier `app/policies/main.ts` existe et doit exporter un objet `policies`.
:::

```ts
const codemods = await command.createCodemods()

try {
  await codemods.registerPolicies([
    {
      name: 'PostPolicy',
      path: '#policies/post_policy'
    }
  ])
} catch (error) {
  console.error('Unable to register policy')
  console.error(error)
}
```

```ts
// title: Résultat
export const policies = {
  PostPolicy: () => import('#policies/post_policy')
}
```

### registerVitePlugin

Enregistre un plugin Vite dans le fichier `vite.config.ts`.

:::note
Ce codemod s'attend à ce que le fichier `vite.config.ts` existe et doit avoir l'appel de fonction `export default defineConfig`.
:::

```ts
const transformer = new CodeTransformer(appRoot)
const imports = [
  {
    isNamed: false,
    module: '@vitejs/plugin-vue',
    identifier: 'vue'
  },
]
const pluginUsage = 'vue({ jsx: true })'

try {
  await transformer.addVitePlugin(pluginUsage, imports)
} catch (error) {
  console.error('Unable to register vite plugin')
  console.error(error)
}
```

```ts
// title: Résultat
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [
    vue({ jsx: true })
  ]
})
```

### installPackages

Installe un ou plusieurs packages en utilisant le gestionnaire de packages détecté dans le projet de l'utilisateur.

```ts
const codemods = await command.createCodemods()

try {
  await codemods.installPackages([
    { name: 'vinejs', isDevDependency: false },
    { name: 'edge', isDevDependency: false }
  ])
} catch (error) {
  console.error('Unable to install packages')
  console.error(error)
}
```

### getTsMorphProject

La méthode `getTsMorphProject` retourne une instance de `ts-morph`. Cela peut être utile lorsque vous voulez effectuer des transformations de fichiers personnalisées qui ne sont pas couvertes par l'API Codemods.

```ts
const project = await codemods.getTsMorphProject()

project.getSourceFileOrThrow('start/routes.ts')
```

Assurez-vous de lire la [documentation de ts-morph](https://ts-morph.com/) pour en savoir plus sur les API disponibles.
