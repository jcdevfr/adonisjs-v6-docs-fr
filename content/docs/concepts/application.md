---
summary: Découvrez la classe Application et apprenez à accéder à l'environnement, à l'état et à créer des URL et des chemins d'accès aux fichiers du projet.
---

# Application

La classe [Application](https://github.com/adonisjs/application/blob/main/src/application.ts) effectue l'essentiel du travail pour connecter les différents éléments d'une application AdonisJS. Vous pouvez utiliser cette classe pour connaître l'environnement dans lequel votre application s'exécute, obtenir l'état actuel de l'application ou créer des chemins vers des répertoires spécifiques.

Voir aussi : [Cycle de vie de l'application](./application_lifecycle.md)

## Environnement

L'environnement fait référence à l'environnement d'exécution de l'application. L'application est toujours démarrée dans l'un des environnements connus suivants.

- L'environnement `web` fait référence au processus démarré pour le serveur HTTP.

- L'environnement `console` fait référence aux commandes Ace, à l'exception de la commande REPL.

- L'environnement `repl` fait référence au processus démarré à l'aide de la commande `node ace repl`.

- Enfin, l'environnement de `test` fait référence au processus démarré à l'aide de la commande `node ace test`.

Vous pouvez accéder à l'environnement de l'application en utilisant la méthode `getEnvironment`.

```ts
import app from '@adonisjs/core/services/app'

console.log(app.getEnvironment())
```

Vous pouvez également changer l'environnement de l'application avant qu'elle ne soit démarrée. Un excellent exemple de cela est la commande REPL.

La commande `node ace repl` démarre l'application dans l'environnement `console`, mais en interne, la commande bascule l'environnement vers `repl` avant d'afficher l'invite REPL.

```ts
if (!app.isBooted) {
	app.setEnvironment('repl')
}
```

## Environnement Node

Vous pouvez accéder à l'environnement `Node.js` en utilisant la propriété `nodeEnvironment`. La valeur est une référence à la variable d'environnement `NODE_ENV`. Cependant, la valeur est normalisée pour être cohérente.

```ts
import app from '@adonisjs/core/services/app'

console.log(app.nodeEnvironment)
```

| NODE_ENV | Normalized to |
|----------|---------------|
| dev      | development   |
| develop  | development   |
| stage    | staging       |
| prod     | production    |
| testing  | test          |

De plus, vous pouvez utiliser les propriétés suivantes comme raccourci pour connaître l'environnement actuel.

- `inProduction` : Vérifie si l'application s'exécute dans l'environnement de production.
- `inDev` : Vérifie si l'application s'exécute dans l'environnement de développement.
- `inTest` : Vérifie si l'application s'exécute dans l'environnement de test.

```ts
import app from '@adonisjs/core/services/app'

// En production
app.inProduction
app.nodeEnvironment === 'production'

// En développement
app.inDev
app.nodeEnvironment === 'development'

// En test
app.inTest
app.nodeEnvironment === 'test'
```

## État

L'état fait référence à l'état actuel de l'application. Les fonctionnalités du framework auxquelles vous pouvez accéder dépendent considérablement de l'état actuel de l'application. Par exemple, vous ne pouvez pas accéder aux [liaisons du conteneur](./dependency_injection.md#liaisons-du-conteneur) ou aux [services du conteneur](./container_services.md) tant que l'application n'est pas dans un état `booted`.

L'application est toujours dans l'un des états connus suivants.

- `created` : C'est l'état par défaut de l'application.

- `initiated` : Dans cet état, nous analysons/validons les variables d'environnement et traitons le fichier `adonisrc.ts`.

- `booted` : Les fournisseurs de services de l'application sont enregistrés et démarrés dans cet état.

- `ready` : L'état ready varie selon les différents environnements. Par exemple, dans l'environnement `web`, l'état ready signifie que l'application est prête à accepter de nouvelles requêtes HTTP.

- `terminated` : L'application a été arrêtée et le processus va bientôt se terminer. L'application n'acceptera pas de nouvelles requêtes HTTP dans l'environnement `web`.

```ts
import app from '@adonisjs/core/services/app'

console.log(app.getState())
```

Vous pouvez également utiliser les propriétés raccourcies suivantes pour savoir si l'application est dans un état donné.

```ts
import app from '@adonisjs/core/services/app'

// L'application est démarrée
app.isBooted
app.getState() !== 'created' && app.getState() !== 'initiated'

// L'application est prête
app.isReady
app.getState() === 'ready'

// Tentative d'arrêt en douceur de l'application
app.isTerminating

// L'application a été arrêtée
app.isTerminated
app.getState() === 'terminated'
```

## Écouter des signaux de processus

Vous pouvez écouter les [signaux POSIX](https://man7.org/linux/man-pages/man7/signal.7.html) en utilisant les méthodes `app.listen` ou `app.listenOnce`. En interne, nous enregistrons l'écouteur avec l'objet `process` de Node.js.

```ts
import app from '@adonisjs/core/services/app'

// Écouter un signal SIGTERM
app.listen('SIGTERM', () => {
})

// Écouter une fois un signal SIGTERM
app.listenOnce('SIGTERM', () => {
})
```

Parfois, vous voudrez peut-être enregistrer les écouteurs de manière conditionnelle. Par exemple, écouter le signal `SIGINT` lors de l'exécution dans l'environnement pm2.

Vous pouvez utiliser les méthodes `listenIf` ou `listenOnceIf` pour enregistrer un écouteur de manière conditionnelle. L'écouteur n'est enregistré que lorsque la valeur du premier argument est vraie.

```ts
import app from '@adonisjs/core/services/app'

app.listenIf(app.managedByPm2, 'SIGTERM', () => {
})

app.listenOnceIf(app.managedByPm2, 'SIGTERM', () => {
})
```

## Notification du processus parent

Si votre application démarre en tant que processus enfant, vous pouvez envoyer des messages au processus parent en utilisant la méthode `app.notify`. En interne, nous utilisons la méthode `process.send`.

```ts
import app from '@adonisjs/core/services/app'

app.notify('ready')

app.notify({
  isReady: true,
  port: 3333,
  host: 'localhost'
})
```

## Création d'URL et de chemins vers les fichiers du projet

Au lieu de construire vous-même des URL absolues ou des chemins vers les fichiers du projet, nous vous recommandons vivement d'utiliser les helpers suivants.

### makeURL

La méthode `makeURL` renvoie une URL de fichier vers un fichier ou un répertoire spécifique du projet. Par exemple, vous pouvez générer une URL lors de l'importation d'un fichier.

```ts
import app from '@adonisjs/core/services/app'

const files = [
  './tests/welcome.spec.ts',
  './tests/maths.spec.ts'
]

await Promise.all(files.map((file) => {
  return import(app.makeURL(file).href)
}))
```

### makePath

La méthode `makePath` renvoie un chemin absolu vers un fichier ou un répertoire spécifique du projet.

```ts
import app from '@adonisjs/core/services/app'

app.makePath('app/middleware/auth.ts')
```

### configPath

Renvoie le chemin vers un fichier dans le répertoire config du projet.

```ts
app.configPath('shield.ts')
// /project_root/config/shield.ts

app.configPath()
// /project_root/config
```

### publicPath

Renvoie le chemin vers un fichier dans le répertoire public du projet.

```ts
app.publicPath('style.css')
// /project_root/public/style.css

app.publicPath()
// /project_root/public
```

### providersPath

Renvoie le chemin vers un fichier dans le répertoire des fournisseurs.

```ts
app.providersPath('app_provider')
// /project_root/providers/app_provider.ts

app.providersPath()
// /project_root/providers
```

### factoriesPath

Renvoie le chemin vers un fichier dans le répertoire des factories de la base de données.

```ts
app.factoriesPath('user.ts')
// /project_root/database/factories/user.ts

app.factoriesPath()
// /project_root/database/factories
```

### migrationsPath

Renvoie le chemin vers un fichier dans le répertoire des migrations de la base de données.

```ts
app.migrationsPath('user.ts')
// /project_root/database/migrations/user.ts

app.migrationsPath()
// /project_root/database/migrations
```

### seedersPath

Renvoie le chemin vers un fichier dans le répertoire des seeders de la base de données.

```ts
app.seedersPath('user.ts')
// /project_root/database/seeders/users.ts

app.seedersPath()
// /project_root/database/seeders
```

### languageFilesPath

Renvoie le chemin vers un fichier dans le répertoire des langues.

```ts
app.languageFilesPath('en/messages.json')
// /project_root/resources/lang/en/messages.json

app.languageFilesPath()
// /project_root/resources/lang
```

### viewsPath

Renvoie le chemin vers un fichier dans le répertoire des vues.

```ts
app.viewsPath('welcome.edge')
// /project_root/resources/views/welcome.edge

app.viewsPath()
// /project_root/resources/views
```

### startPath

Renvoie le chemin vers un fichier dans le répertoire start.

```ts
app.startPath('routes.ts')
// /project_root/start/routes.ts

app.startPath()
// /project_root/start
```

### tmpPath

Renvoie le chemin vers un fichier dans le répertoire `tmp` à la racine du projet.

```ts
app.tmpPath('logs/mail.txt')
// /project_root/tmp/logs/mail.txt

app.tmpPath()
// /project_root/tmp
```

### httpControllersPath

Renvoie le chemin vers un fichier dans le répertoire des contrôleurs HTTP.

```ts
app.httpControllersPath('users_controller.ts')
// /project_root/app/controllers/users_controller.ts

app.httpControllersPath()
// /project_root/app/controllers
```

### modelsPath

Renvoie le chemin vers un fichier dans le répertoire des modèles.

```ts
app.modelsPath('user.ts')
// /project_root/app/models/user.ts

app.modelsPath()
// /project_root/app/models
```

### servicesPath

Renvoie le chemin vers un fichier dans le répertoire des services.

```ts
app.servicesPath('user.ts')
// /project_root/app/services/user.ts

app.servicesPath()
// /project_root/app/services
```

### exceptionsPath

Renvoie le chemin vers un fichier dans le répertoire des exceptions.

```ts
app.exceptionsPath('handler.ts')
// /project_root/app/exceptions/handler.ts

app.exceptionsPath()
// /project_root/app/exceptions
```

### mailsPath

Renvoie le chemin vers un fichier dans le répertoire des mails.

```ts
app.mailsPath('verify_email.ts')
// /project_root/app/mails/verify_email.ts

app.mailsPath()
// /project_root/app/mails
```

### middlewarePath

Renvoie le chemin vers un fichier dans le répertoire des middlewares.

```ts
app.middlewarePath('auth.ts')
// /project_root/app/middleware/auth.ts

app.middlewarePath()
// /project_root/app/middleware
```

### policiesPath

Renvoie le chemin vers un fichier dans le répertoire des policies.

```ts
app.policiesPath('posts.ts')
// /project_root/app/policies/posts.ts

app.policiesPath()
// /project_root/app/policies
```

### validatorsPath

Renvoie le chemin vers un fichier dans le répertoire des validateurs.

```ts
app.validatorsPath('create_user.ts')
// /project_root/app/validators/create_user.ts

app.validatorsPath()
// /project_root/app/validators/create_user.ts
```

### commandsPath

Renvoie le chemin vers un fichier dans le répertoire des commandes.

```ts
app.commandsPath('greet.ts')
// /project_root/commands/greet.ts

app.commandsPath()
// /project_root/commands
```

### eventsPath

Renvoie le chemin vers un fichier dans le répertoire des événements.

```ts
app.eventsPath('user_created.ts')
// /project_root/app/events/user_created.ts

app.eventsPath()
// /project_root/app/events
```

### listenersPath

Renvoie le chemin vers un fichier dans le répertoire des écouteurs.

```ts
app.listenersPath('send_invoice.ts')
// /project_root/app/listeners/send_invoice.ts

app.listenersPath()
// /project_root/app/listeners
```

## Générateurs

Les générateurs sont utilisés pour créer des noms de classes et de fichiers pour différentes entités. Par exemple, vous pouvez utiliser la méthode `generators.controllerFileName` pour générer le nom de fichier d'un contrôleur.

```ts
import app from '@adonisjs/core/services/app'

app.generators.controllerFileName('user')
// Résultat - users_controller.ts

app.generators.controllerName('user')
// Résultat - UsersController
```

Veuillez vous [référer au code source generators.ts](https://github.com/adonisjs/application/blob/main/src/generators.ts) pour voir la liste des générateurs disponibles.
