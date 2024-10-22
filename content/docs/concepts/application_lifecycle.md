---
summary: Découvrez comment AdonisJS démarre votre application et quels sont les hooks du cycle de vie que vous pouvez utiliser pour modifier l'état de l'application avant qu'elle ne soit considérée comme prête.
---

# Cycle de vie de l'application

Dans ce guide, nous allons apprendre comment AdonisJS démarre votre application et quels hooks du cycle de vie vous pouvez utiliser pour modifier l'état de l'application avant qu'elle ne soit considérée comme prête.

Le cycle de vie d'une application dépend de l'environnement dans lequel elle s'exécute. Par exemple, un processus de longue durée pour servir des requêtes HTTP est géré différemment d'une commande ace de courte durée.

Voyons donc le cycle de vie de l'application pour chaque environnement pris en charge.

## Démarrage d'une application AdonisJS

Une application AdonisJS a plusieurs points d'entrée, et chaque point d'entrée démarre l'application dans un environnement spécifique. Les fichiers de point d'entrée suivants sont stockés dans le répertoire `bin`.

- Le point d'entrée `bin/server.ts` démarre l'application AdonisJS pour gérer les requêtes HTTP. Lorsque vous exécutez la commande `node ace serve`, en interne nous exécutons ce fichier en tant que processus enfant.
- Le point d'entrée `bin/console.ts` démarre l'application AdonisJS pour gérer les commandes CLI. Ce fichier utilise [Ace](../ace/introduction.md) en interne.
- Le point d'entrée `bin/test.ts` démarre l'application AdonisJS pour exécuter des tests en utilisant Japa.

Si vous ouvrez l'un de ces fichiers, vous verrez que nous utilisons le module [Ignitor](https://github.com/adonisjs/core/blob/main/src/ignitor/main.ts#L23) pour configurer les choses puis démarrer l'application.

Le module Ignitor encapsule la logique de démarrage d'une application AdonisJS. En interne, il effectue les actions suivantes :

- Crée une instance de la classe [Application](https://github.com/adonisjs/application/blob/main/src/application.ts).
- Initialise/démarre l'application.
- Effectue l'action principale pour lancer l'application. Par exemple, dans le cas d'un serveur HTTP, l'action `principale` consiste à démarrer le serveur HTTP. Dans le cas des tests, l'action `principale` consiste à exécuter les tests.

Le [code source d'Ignitor](https://github.com/adonisjs/core/tree/main/src/ignitor) est relativement simple, donc parcourez-le pour mieux le comprendre.

## La phase de démarrage

La phase de démarrage reste la même pour tous les environnements sauf l'environnement `console`. Dans l'environnement `console`, la commande exécutée décide si l'application doit être démarrée.

Vous ne pouvez utiliser les liaisons du conteneur et les services qu'une fois l'application démarrée.

![](./boot_phase_flow_chart.png)

## La phase de lancement

La phase de lancement varie entre tous les environnements. De plus, le flux d'exécution est divisé en sous-phases suivantes :

- La phase de `pré-lancement` fait référence aux actions effectuées avant le lancement de l'application.

- La phase de `post-lancement` fait référence aux actions effectuées après le lancement de l'application. Dans le cas d'un serveur HTTP, les actions seront exécutées après que le serveur HTTP soit prêt à accepter de nouvelles connexions.

![](./start_phase_flow_chart.png)

### Pendant l'environnement web

Dans l'environnement web, une connexion HTTP de longue durée est créée pour écouter les requêtes entrantes, et l'application reste dans l'état `ready` jusqu'à ce que le serveur plante ou que le processus reçoive un signal pour s'arrêter.

### Pendant l'environnement de test

Les actions de **pré-lancement** et de **post-lancement** sont exécutées dans l'environnement de test. Après cela, nous importons les fichiers de test et exécutons les tests.

### Pendant l'environnement console

Dans l'environnement `console`, la commande exécutée décide si l'application doit être lancée.

Une commande peut lancer l'application en activant l'option `options.startApp`. En conséquence, les actions de **pré-lancement** et de **post-lancement** s'exécuteront avant la méthode `run` de la commande.

```ts
import { BaseCommand } from '@adonisjs/core/ace'

export default class GreetCommand extends BaseCommand {
  static options = {
    startApp: true
  }
  
  async run() {
    console.log(this.app.isReady) // true
  }
}
```

## La phase d'arrêt

L'arrêt de l'application varie grandement entre les processus de courte durée et de longue durée.

Une commande de courte durée ou le processus de test commence l'arrêt après la fin de l'opération principale.

Un processus de serveur HTTP de longue durée attend les signaux de sortie comme `SIGTERM` pour commencer le processus d'arrêt.

![](./termination_phase_flow_chart.png)

### Répondre aux signaux du processus

Dans tous les environnements, nous commençons un processus d'arrêt en douceur lorsque l'application reçoit un signal `SIGTERM`. Si vous avez démarré votre application en utilisant [pm2](https://pm2.keymetrics.io/docs/usage/signals-clean-restart/), l'arrêt en douceur se produira après avoir reçu l'événement `SIGINT`.

### Pendant l'environnement web

Dans l'environnement web, l'application continue de s'exécuter jusqu'à ce que le serveur HTTP plante avec une erreur. Dans ce cas, nous commençons l'arrêt de l'application.

### Pendant l'environnement de test

L'arrêt en douceur commence après l'exécution de tous les tests.

### Pendant l'environnement console

Dans l'environnement `console`, l'arrêt de l'application dépend de la commande exécutée.

L'application se terminera dès que la commande sera exécutée, sauf si l'option `options.staysAlive` est activé, et dans ce cas, la commande doit explicitement arrêter l'application.

```ts
import { BaseCommand } from '@adonisjs/core/ace'

export default class GreetCommand extends BaseCommand {
  static options = {
    startApp: true,
    staysAlive: true,
  }
  
  async run() {
    await runSomeProcess()
    
    // Déclenche l'arrêt du processus
    await this.terminate()
  }
}
```

## Hooks du cycle de vie

Les hooks du cycle de vie vous permettent d'intervenir dans le processus de démarrage de l'application et d'effectuer des actions lorsque l'application passe par différents états.

Vous pouvez écouter les hooks en utilisant les classes de fournisseurs de services ou les définir directement sur la classe Application.

### Fonctions de rappel

Vous devez enregistrer les hooks du cycle de vie dès qu'une instance d'application est créée.

Les fichiers de point d'entrée `bin/server.ts`, `bin/console.ts` et `bin/test.ts` créent une nouvelle instance d'application pour différents environnements, et vous pouvez enregistrer des fonctions de rappel dans ces fichiers.

```ts
const app = new Application(new URL('../', import.meta.url))

new Ignitor(APP_ROOT, { importer: IMPORTER })
  .tap((app) => {
    // highlight-start
    app.booted(() => {
      console.log("invoqué après le démarrage de l'application")
    })
    
    app.ready(() => {
      console.log("invoqué après que l'application soit prête")
    })
    
    app.terminating(() => {
      console.log("invoqué avant le début de l'arrêt")
    })
    // highlight-end
  })
```

- `initiating` : Les actions du hook sont appelées avant que l'application ne passe à l'état initiated. Le fichier `adonisrc.ts` est analysé après l'exécution des hooks `initiating`.

- `booting` : Les actions du hook sont appelées avant le démarrage de l'application. Les fichiers de configuration sont importés après l'exécution des hooks `booting`.

- `booted` : Les actions du hook sont invoquées après que tous les fournisseurs de services aient été enregistrés et démarrés.

- `starting` : Les actions du hook sont invoquées avant l'importation des fichiers à précharger.

- `ready` : Les actions du hook sont invoquées après que l'application soit prête.

- `terminating` : Les actions du hook sont invoquées une fois que le processus d'arrêt en douceur commence. Par exemple, ce hook peut fermer les connexions à la base de données ou terminer les streams ouverts.

### Utilisation des fournisseurs de services

Les fournisseurs de services définissent les hooks du cycle de vie comme des méthodes dans la classe du fournisseur. Nous recommandons d'utiliser des fournisseurs de services plutôt que des callbacks inline, car ils permettent de tout organiser proprement.

Voici la liste des méthodes du cycle de vie disponibles :

```ts
import { ApplicationService } from '@adonisjs/core/types'

export default class AppProvider {
  constructor(protected app: ApplicationService) {}
  
  register() {
  }
  
  async boot() {
  }
  
  async start() {
  }
  
  async ready() {
  }
  
  async shutdown() {
  }
}
```

- `register` : La méthode register enregistre les liaisons dans le conteneur. Cette méthode est synchrone par conception.

- `boot` : La méthode boot est utilisée pour démarrer ou initialiser les liaisons que vous avez enregistrées dans le conteneur.

- `start` : La méthode start s'exécute juste avant la méthode `ready`. Elle vous permet d'effectuer des actions dont les actions du hook `ready` pourraient avoir besoin.

- `ready` : La méthode ready s'exécute après que l'application soit considérée comme prête.

- `shutdown` : La méthode shutdown est invoquée lorsque l'application commence l'arrêt en douceur. Vous pouvez utiliser cette méthode pour fermer les connexions à la base de données ou terminer les streams ouverts.
