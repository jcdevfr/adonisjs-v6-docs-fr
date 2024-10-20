---
summary: Les fournisseurs de services sont de simples classes JavaScript avec des méthodes de cycle de vie pour effectuer des actions pendant différentes phases de l'application.
---

# Fournisseurs de services 

Les fournisseurs de services sont de simples classes JavaScript avec des méthodes de cycle de vie pour effectuer des actions pendant différentes phases de l'application.

Un fournisseur de services peut enregistrer des [liaisons dans le conteneur](../concepts/dependency_injection.md#liaisons-du-conteneur), [étendre des liaisons existantes](../concepts/dependency_injection.md#evenements-du-conteneur) ou exécuter des actions après le démarrage du serveur HTTP.

Les fournisseurs de services sont le point d'entrée d'une application AdonisJS avec la capacité de modifier l'état de l'application avant qu'elle ne soit considérée comme prête.

Les fournisseurs sont enregistrés dans le fichier `adonisrc.ts` sous le tableau `providers`. La valeur est une fonction pour importer à la demande le fournisseur de services.

```ts
{
  providers: [
    () => import('@adonisjs/core/providers/app_provider'),
    () => import('./providers/app_provider.js'),
  ]
}
```

Par défaut, un fournisseur est chargé dans tous les environnements d'exécution. Cependant, vous pouvez limiter le fournisseur à s'exécuter dans des environnements spécifiques.

```ts
{
  providers: [
    () => import('@adonisjs/core/providers/app_provider'),
    {
      file: () => import('./providers/app_provider.js'),
      environment: ['web', 'repl']
    }
  ]
}
```

## Écrire un fournisseur de services

Les fournisseurs de services sont stockés dans le répertoire `providers` de votre application. Vous pouvez également utiliser la commande `node ace make:provider app`.

Le module du fournisseur doit avoir une instruction `export default` retournant la classe du fournisseur. Le constructeur de la classe reçoit une instance de la classe [Application](./application.md).

Voir aussi : [Commande pour créer un fournisseur](../references/commands.md#makeprovider)

```ts
import { ApplicationService } from '@adonisjs/core/types'

export default class AppProvider {
  constructor(protected app: ApplicationService) {
  }
}
```

Voici les méthodes de cycle de vie que vous pouvez implémenter pour effectuer différentes actions.

```ts
export default class AppProvider {
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

### register

La méthode `register` est appelée après la création d'une instance de la classe du fournisseur. La méthode `register` peut enregistrer des liaisons dans le conteneur IoC.

La méthode `register` est synchrone, vous ne pouvez donc pas utiliser de promesses dans cette méthode.

```ts
export default class AppProvider {
  register() {
    this.app.container.bind('db', () => {
      return new Database()
    })
  }
}
```

### boot

La méthode `boot` est appelée après que toutes les liaisons aient été enregistrées avec le conteneur IoC. Dans cette méthode, vous pouvez résoudre les liaisons du conteneur pour les étendre/modifier.

```ts
export default class AppProvider {
  async boot() {
   const validator = await this.app.container.make('validator')
    
   // Ajouter des règles de validation personnalisées
   validator.rule('foo', () => {})
  }
}
```

C'est une bonne pratique d'étendre les liaisons lorsqu'elles sont résolues à partir du conteneur. Par exemple, vous pouvez utiliser le hook `resolving` pour ajouter des règles personnalisées au validateur.

```ts
async boot() {
  this.app.container.resolving('validator', (validator) => {
    validator.rule('foo', () => {})
  })
}
```

### start

La méthode `start` est appelée après la méthode `boot` et avant la méthode `ready`. Elle vous permet d'effectuer des actions dont les actions du hook `ready` pourraient avoir besoin.

### ready

La méthode `ready` est appelée à différentes étapes selon l'environnement de l'application.

<table>
    <tr>
        <td width="100"><code> web </code></td>
        <td>La méthode <code>ready</code> est appelée après que le serveur HTTP ait été démarré et soit prêt à accepter des requêtes.</td>
    </tr>
    <tr>
        <td width="100"><code>console</code></td>
        <td>La méthode <code> ready</code> est appelée juste avant la méthode <code>run</code> de la commande principale.</td>
    </tr>
    <tr>
        <td width="100"><code>test</code></td>
        <td>La méthode <code>ready</code> est appelée juste avant l'exécution de tous les tests. Cependant, les fichiers de test sont importés avant la méthode <code>ready</code>.</td>
    </tr>
    <tr>
        <td width="100"><code>repl</code></td>
        <td>La méthode <code>ready</code> est appelée avant que l'invite REPL ne s'affiche sur le terminal.</td>
    </tr>
</table>

```ts
export default class AppProvider {
  async start() {
    if (this.app.getEnvironment() === 'web') {
    }

    if (this.app.getEnvironment() === 'console') {
    }

    if (this.app.getEnvironment() === 'test') {
    }

    if (this.app.getEnvironment() === 'repl') {
    }
  }
}
```

### shutdown

La méthode `shutdown` est appelée lorsqu'AdonisJS est en train de quitter en douceur l'application.

L'arrêt de l'application dépend de l'environnement dans lequel l'application s'exécute et de la façon dont le processus de l'application a été démarré. Veuillez lire le [guide du cycle de vie de l'application](./application_lifecycle.md) pour en savoir plus à ce sujet.

```ts
export default class AppProvider {
  async shutdown() {
    // effectuer le nettoyage
  }
}
```
