---
summary: Apprenez à lire et à mettre à jour les valeurs de la configuration dans AdonisJS.
---

# Configuration

Les fichiers de configuration de votre application AdonisJS sont stockés dans le répertoire `config`. Une nouvelle application AdonisJS est accompagnée de quelques fichiers préexistants utilisés par le noyau du framework et les packages installés.

N'hésitez pas à créer des fichiers supplémentaires dont votre application a besoin dans le répertoire `config`.

:::note
Nous recommandons d'utiliser des [variables d'environnement](./environment_variables.md) pour stocker les données sensibles et la configuration spécifique à un environnement.
:::

## Importer les fichiers de configuration

Vous pouvez importer les fichiers de configuration dans le code de votre application en utilisant l'instruction JavaScript standard `import`. Par exemple :

```ts
import { appKey } from '#config/app'
```

```ts
import databaseConfig from '#config/database'
```

## Utiliser le service config

Le service config offre une API alternative pour lire les valeurs de la configuration. Dans l'exemple suivant, nous utilisons le service config pour lire la valeur `appKey` stockée dans le fichier `config/app.ts`.

```ts
import config from '@adonisjs/core/services/config'

config.get('app.appKey')
config.get('app.http.cookie') // Lire des valeurs imbriquées
```
La méthode `config.get` accepte une clé séparée par des points et l'interprète de la manière suivante :

- La première partie correspond au nom du fichier à partir duquel vous souhaitez lire les valeurs, par exemple le fichier `app.ts`.
- Le reste de la chaîne représente la clé que vous souhaitez accéder parmi les valeurs exportées, par exemple, `appKey` dans ce cas.

## Service config vs. importation directe des fichiers de configuration

L'utilisation du service config plutôt que l'importation directe des fichiers de configuration n'a pas d'avantages directs. Cependant, le service config est le seul choix pour lire la configuration dans les packages externes et les templates Edge.

### Lecture de la configuration dans les packages externes

Si vous créez un package tiers, vous ne devez pas importer directement les fichiers de configuration depuis l'application utilisateur car cela rendrait votre package étroitement couplé à la structure des dossiers de l'application hôte.

À la place, vous devez utiliser le service config pour accéder aux valeurs de la configuration à l'intérieur d'un fournisseur de services. Par exemple :

```ts
import { ApplicationService } from '@adonisjs/core/types'

export default class DriveServiceProvider {
  constructor(protected app: ApplicationService) {}
  
  register() {
    this.app.container.singleton('drive', () => {
      // highlight-start
      const driveConfig = this.app.config.get('drive')
      return new DriveManager(driveConfig)
      // highlight-end
    })
  }
}
```

### Lecture de la configuration dans les templates Edge

Vous pouvez accéder aux valeurs de la configuration dans les templates Edge en utilisant la méthode globale `config`.

```edge
<a href="{{ config('app.appUrl') }}"> Home </a>
```

Vous pouvez utiliser la méthode `config.has` pour vérifier si une valeur de la configuration existe pour une clé donnée. La méthode renvoie `false` si la valeur est `undefined`.

```edge
@if(config.has('app.appUrl'))
  <a href="{{ config('app.appUrl') }}"> Home </a>
@else
  <a href="/"> Home </a>
@end
```

## Modification de l'emplacement de la configuration

Vous pouvez mettre à jour l'emplacement du répertoire de configuration en modifiant le fichier `adonisrc.ts`. Après cette modification, les fichiers de configuration seront importés depuis le nouvel emplacement.

```ts
directories: {
  config: './configurations'
}
```

Assurez-vous de mettre à jour l'alias d'importation dans le fichier `package.json`.

```json
{
  "imports": {
    "#config/*": "./configurations/*.js"
  }
}
```

## Limitations des fichiers de configuration

Les fichiers de configuration stockés dans le répertoire `config` sont importés pendant la phase de démarrage de l'application. Par conséquent, les fichiers de configuration ne peuvent pas dépendre du code de l'application.

Par exemple, si vous essayez d'importer et d'utiliser le service router dans le fichier `config/app.ts`, l'application ne démarrera pas. C'est parce que le service router n'est pas configuré tant que l'application n'est pas dans un état `booted`.

Fondamentalement, cette limitation a un impact positif sur votre code car le code de l'application devrait dépendre de la configuration, et non l'inverse.

## Mise à jour de la configuration pendant l'exécution

Vous pouvez modifier les valeurs de la configuration pendant l'exécution en utilisant le service config. La méthode `config.set` met à jour la valeur en mémoire, et aucun changement n'est apporté aux fichiers sur le disque.

:::note
La configuration est modifiée pour toute l'application, pas seulement pour une seule requête HTTP. C'est parce que Node.js n'est pas un environnement d'exécution multithread, et la mémoire dans Node.js est partagée entre plusieurs requêtes HTTP.
:::

```ts
import env from '#start/env'
import config from '@adonisjs/core/services/config'

const HOST = env.get('HOST')
const PORT = env.get('PORT')

config.set('app.appUrl', `http://${HOST}:${PORT}`)
```
