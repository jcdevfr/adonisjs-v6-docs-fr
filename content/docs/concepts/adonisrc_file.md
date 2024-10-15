---
summary: Le fichier adonisrc.ts est utilisé pour configurer les paramètres de l'espace de travail de votre application.
---

# Fichier AdonisRC

Le fichier `adonisrc.ts` est utilisé pour configurer les paramètres de l'espace de travail de votre application. Dans ce fichier, vous pouvez [enregistrer des fournisseurs](#providers), définir des [alias de commandes](#commandsaliases), spécifier les [fichiers à copier](#metafiles) dans la version de production, et bien plus encore.

:::warning
Le fichier `adonisrc.ts` est importé par d'autres outils que votre application AdonisJS. Par conséquent, vous ne devez pas y écrire de code spécifique à l'application ou de conditions spécifiques à un environnement.
:::

Le fichier contient la configuration minimale requise pour exécuter votre application. Cependant, vous pouvez voir le contenu complet du fichier en exécutant la commande `node ace inspect:rcfile`.

```sh
node ace inspect:rcfile
```

Vous pouvez accéder au contenu analysé du RCFile en utilisant le service `app`.

```ts
import app from '@adonisjs/core/services/app'

console.log(app.rcFile)
```

## typescript

La propriété `typescript` informe le framework et les commandes Ace que votre application utilise TypeScript. Actuellement, cette valeur est toujours définie sur `true`. Cependant, nous permettrons plus tard aux applications d'être écrites en JavaScript.

## directories

Un ensemble de répertoires et leurs chemins utilisés par les commandes de scaffolding (échaffaudage). Si vous décidez de renommer des répertoires spécifiques, mettez à jour leur nouveau chemin dans l'objet `directories` pour en informer les commandes de scaffolding.

```ts
{
  directories: {
    config: 'config',
    commands: 'commands',
    contracts: 'contracts',
    public: 'public',
    providers: 'providers',
    languageFiles: 'resources/lang',
    migrations: 'database/migrations',
    seeders: 'database/seeders',
    factories: 'database/factories',
    views: 'resources/views',
    start: 'start',
    tmp: 'tmp',
    tests: 'tests',
    httpControllers: 'app/controllers',
    models: 'app/models',
    services: 'app/services',
    exceptions: 'app/exceptions',
    mails: 'app/mails',
    middleware: 'app/middleware',
    policies: 'app/policies',
    validators: 'app/validators',
    events: 'app/events',
    listeners: 'app/listeners',
    stubs: 'stubs',
  }
}
```

## preloads

Un tableau de fichiers à importer au moment du démarrage de l'application. Les fichiers sont importés immédiatement après le démarrage des fournisseurs de services.

Vous pouvez définir l'environnement dans lequel importer le fichier. Les options valides sont :

- L'environnement `web` fait référence au processus démarré pour le serveur HTTP.
- L'environnement `console` fait référence aux commandes Ace, à l'exception de la commande `repl`.
- L'environnement `repl` fait référence au processus démarré avec la commande `node ace repl`.
- Enfin, l'environnement `test` fait référence au processus démarré pour l'exécution des tests.


:::note
Vous pouvez créer et enregistrer un fichier à précharger en utilisant la commande `node ace make:preload`.
:::


```ts
{
  preloads: [
    () => import('./start/view.js')
  ]
}
```

```ts
{
  preloads: [
    {
      file: () => import('./start/view.js'),
      environment: [
        'web',
        'console',
        'test'
      ]
    },
  ]
}
```

## metaFiles

Le tableau `metaFiles` est une collection de fichiers que vous souhaitez copier dans le dossier de `build` lors de la création de la version de production.

Ce sont des fichiers non-TypeScript/JavaScript qui doivent exister dans la version de production pour que votre application fonctionne. Par exemple, les templates Edge, les fichiers de langue i18n, etc.

- `pattern`: Le [motif glob](https://github.com/sindresorhus/globby#globbing-patterns) pour trouver les fichiers correspondants. 
- `reloadServer`: Recharge le serveur de développement lorsque les fichiers correspondants changent.

```ts
{
  metaFiles: [
    {
      pattern: 'public/**',
      reloadServer: false
    },
    {
      pattern: 'resources/views/**/*.edge',
      reloadServer: false
    }
  ]
}
```

## commands

Un tableau de fonctions pour importer à la demande les commandes ace des packages installés. Les commandes de vos applications seront importées automatiquement et vous n'avez donc pas besoin de les enregistrer explicitement.

Voir aussi : [Créer des commandes ace](../ace/creating_commands.md)

```ts
{
  commands: [
    () => import('@adonisjs/core/commands'),
    () => import('@adonisjs/lucid/commands')
  ]
}
```

## commandsAliases

Une paire clé-valeur d'alias de commandes. C'est généralement pour vous aider à créer des alias faciles à mémoriser pour les commandes qui sont plus difficiles à taper ou à retenir.

Voir aussi : [Créer des alias de commandes](../ace/introduction.md#creating-command-aliases)

```ts
{
  commandsAliases: {
    migrate: 'migration:run'
  }
}
```

Vous pouvez également définir plusieurs alias pour la même commande.

```ts
{
  commandsAliases: {
    migrate: 'migration:run',
    up: 'migration:run'
  }
}
```

## tests

L'objet `tests` enregistre les suites de tests et certains paramètres globaux pour l'exécuteur de tests.

Voir aussi : [Introduction aux tests](../testing/introduction.md)

```ts
{
  tests: {
    timeout: 2000,
    forceExit: false,
    suites: [
      {
        name: 'functional',
        files: [
          'tests/functional/**/*.spec.ts'
        ],
        timeout: 30000
      }
    ]
  }
}
```

- `timeout`: Définit le délai d'attente par défaut pour tous les tests.
- `forceExit`: Termine le processus de l'application de manière forcée dès que les tests sont terminés. Il est généralement recommandé de procéder à une fermeture en douceur.
- `suite.name`: Un nom unique pour la suite de tests.
- `suite.files`: Un tableau de motifs glob pour importer les fichiers de test.
- `suite.timeout`: Le délai d'attente par défaut pour tous les tests à l'intérieur de la suite.

## providers

Un tableau de fournisseurs de services à charger pendant la phase de démarrage de l'application.

Par défaut, les fournisseurs sont chargés dans tous les environnements. Cependant, vous pouvez également définir un tableau explicite d'environnements pour importer le fournisseur.

- L'environnement `web` fait référence au processus démarré pour le serveur HTTP.
- L'environnement `console` fait référence aux commandes Ace, à l'exception de la commande `repl`.
- L'environnement `repl` fait référence au processus démarré avec la commande `node ace repl`.
- Enfin, l'environnement `test` fait référence au processus démarré pour l'exécution des tests.

:::note
Les fournisseurs sont chargés dans le même ordre que celui enregistré dans le tableau `providers`.
:::

Voir aussi : [Fournisseurs de services](./service_providers.md)

```ts
{
  providers: [
    () => import('@adonisjs/core/providers/app_provider'),
    () => import('@adonisjs/core/providers/http_provider'),
    () => import('@adonisjs/core/providers/hash_provider'),
    () => import('./providers/app_provider.js'),
  ]
}
```

```ts
{
  providers: [
    {
      file: () => import('./providers/app_provider.js'),
      environment: [
        'web',
        'console',
        'test'
      ]
    },
    {
      file: () => import('@adonisjs/core/providers/http_provider'),
      environment: [
        'web'
      ]
    },
    () => import('@adonisjs/core/providers/hash_provider'),
    () => import('@adonisjs/core/providers/app_provider')
  ]
}
```

## assetsBundler

Les commandes `serve` et `build` tentent de détecter les ressources utilisées par votre application pour compiler les ressources front-end.

La détection est effectuée pour [Vite](https://vitejs.dev) en recherchant le fichier `vite.config.js` et pour [Webpack encore](https://github.com/symfony/webpack-encore) en recherchant le fichier `webpack.config.js`.

Cependant, si vous utilisez un bundler de ressources différent, vous pouvez le configurer dans le fichier `adonisrc.ts` de la manière suivante :

```ts
{
  assetsBundler: {
    name: 'vite',
    devServer: {
      command: 'vite',
      args: []
    },
    build: {
      command: 'vite',
      args: ["build"]
    },
  }
}
```

- `name` - Le nom du bundler de ressources que vous utilisez. Il est requis à des fins d'affichage.
- `devServer.*` - La commande et ses arguments pour démarrer le serveur de développement.
- `build.*` - La commande et ses arguments pour créer la version de production.
