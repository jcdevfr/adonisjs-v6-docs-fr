---
summary: Les hooks de l'assembleur sont un moyen d'exécuter du code à des moments spécifiques du cycle de vie de l'assembleur. 
---

# Hooks de l'assembleur

Les hooks de l'assembleur sont un moyen d'exécuter du code à des moments spécifiques du cycle de vie de l'assembleur. Pour rappel, l'assembleur est une partie d'AdonisJS qui vous permet de lancer votre serveur de développement, de construire votre application et d'exécuter vos tests.

Ces hooks peuvent être utiles pour des tâches telles que la génération de fichiers, la compilation de code ou l'injection d'étapes de construction personnalisées.

Par exemple, le package `@adonisjs/vite` utilise le hook `onBuildStarting` pour injecter une étape où les ressources front-end sont construits. Ainsi, lorsque vous exécutez `node ace build`, le package `@adonisjs/vite` construira vos ressources front-end avant le reste du processus de construction. C'est un bon exemple de la façon dont les hooks peuvent être utilisés pour personnaliser le processus de construction.

## Ajouter un hook

Les hooks de l'assembleur sont définis dans le fichier `adonisrc.ts`, dans la clé `hooks` :

```ts
import { defineConfig } from '@adonisjs/core/app'

export default defineConfig({
  hooks: {
    onBuildCompleted: [
      () => import('my-package/hooks/on_build_completed')
    ],
    onBuildStarting: [
      () => import('my-package/hooks/on_build_starting')
    ],
    onDevServerStarted: [
      () => import('my-package/hooks/on_dev_server_started')
    ],
    onSourceFileChanged: [
      () => import('my-package/hooks/on_source_file_changed')
    ],
  },
})
```

Plusieurs hooks peuvent être définis pour chaque étape du cycle de vie de l'assemblage. Chaque hook est un tableau de fonctions à exécuter.

Nous recommandons d'utiliser des importations dynamiques pour charger les hooks. Cela garantit que les hooks ne sont pas chargés inutilement mais seulement lorsque c'est nécessaire. Si vous écrivez directement votre code de hook dans le fichier `adonisrc.ts`, cela peut ralentir le démarrage de votre application.

## Créer un hook

Un hook n'est qu'une simple fonction. Prenons l'exemple d'un hook censé exécuter une tâche de construction personnalisée.

```ts
// title: hooks/on_build_starting.ts
import type { AssemblerHookHandler } from '@adonisjs/core/types/app'

const buildHook: AssemblerHookHandler = async ({ logger }) => {
  logger.info('Génération de certains fichiers...')

  await myCustomLogic()
}

export default buildHook
```

Notez que le hook doit être exporté par défaut.

Une fois ce hook défini, il ne vous reste plus qu'à l'ajouter au fichier `adonisrc.ts` comme ceci :

```ts
// title: adonisrc.ts
import { defineConfig } from '@adonisjs/core/app'

export default defineConfig({
  hooks: {
    onBuildStarting: [
      () => import('./hooks/on_build_starting')
    ],
  },
})
```

Et maintenant, chaque fois que vous exécuterez `node ace build`, le hook `onBuildStarting` sera exécuté avec la logique personnalisée que vous avez définie.

## Liste des hooks

Voici la liste des hooks disponibles :

### onBuildStarting

Ce hook est exécuté avant le début de la construction. Il est utile pour des tâches telles que la génération de fichiers ou pour injecter des étapes de construction personnalisées.

### onBuildCompleted

Ce hook est exécuté une fois la construction terminée. Il peut également être utilisé pour personnaliser le processus de construction.

### onDevServerStarted

Ce hook est exécuté une fois que le serveur de développement Adonis est démarré.

### onSourceFileChanged

Ce hook est exécuté chaque fois qu'un fichier source (inclus par votre `tsconfig.json`) est modifié. Votre hook recevra le chemin du fichier modifié comme argument.
