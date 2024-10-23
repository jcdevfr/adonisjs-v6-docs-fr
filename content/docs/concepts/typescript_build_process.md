---
summary: Découvrez le processus de construction TypeScript dans AdonisJS.
---

# Processus de construction TypeScript

Les applications écrites en TypeScript doivent être compilées en JavaScript avant de pouvoir être exécutées en production.

La compilation des fichiers source TypeScript peut être effectuée à l'aide de différents outils de construction. Cependant, avec AdonisJS, nous nous en tenons à l'approche la plus simple en utilisant les outils éprouvés suivants.


:::note
Tous les outils mentionnés ci-dessous sont pré-installés comme dépendances de développement avec les kits de démarrage officiels.
:::

- **[TSC](https://www.typescriptlang.org/docs/handbook/compiler-options.html)** est le compilateur officiel de TypeScript. Nous utilisons TSC pour effectuer la vérification des types et créer la version de production.

- **[TS Node](https://typestrong.org/ts-node/)** est un compilateur à la volée pour TypeScript. Il vous permet d'exécuter des fichiers TypeScript sans les compiler en JavaScript et s'avère être un excellent outil pour le développement.

- **[SWC](https://swc.rs/)** est un compilateur TypeScript écrit en Rust. Nous l'utilisons pendant le développement avec TS Node pour rendre le processus de compilation à la volée extrêmement rapide.

| Outil     | Utilisé pour              | Vérification des types |
|-----------|---------------------------|---------------|
| `TSC`     | Création version de production | Oui           |
| `TS Node` | Développement               | Non            |
| `SWC`     | Développement               | Non            |

## Exécution de fichiers TypeScript sans compilation

Vous pouvez exécuter les fichiers TypeScript sans les compiler en utilisant le loader `ts-node/esm`. Par exemple, vous pouvez démarrer le serveur HTTP en exécutant la commande suivante.

```sh
node --loader="ts-node/esm" bin/server.js
```

- `--loader`: Le flag loader enregistre les hooks du loader avec le système de modules ES. Les hooks du loader font partie de l'[API Node.js](https://nodejs.org/dist/latest-v21.x/docs/api/esm.html#loaders).

- `ts-node/esm`: Le chemin vers le script `ts-node/esm` qui enregistre les hooks de cycle de vie pour effectuer la compilation à la volée de la source TypeScript vers JavaScript.

- `bin/server.js`: Le chemin vers le fichier d'entrée du serveur HTTP AdonisJS. **Voir aussi : [Une note sur les extensions de fichiers](#une-note-sur-les-extensions-de-fichiers)**.

Vous pouvez répéter ce processus pour d'autres fichiers TypeScript. Par exemple :

```sh
// title: Exécuter les tests
node --loader ts-node/esm bin/test.js
```


```sh
// title: Exécuter les commandes ace
node --loader ts-node/esm bin/console.js
```

```sh
// title: Exécuter un autre fichier TypeScript
node --loader ts-node/esm path/to/file.js
```

### Une note sur les extensions de fichiers

Vous avez peut-être remarqué que nous utilisons l'extension `.js` partout, même si le fichier sur le disque est enregistré avec l'extension `.ts`.

C'est parce qu'avec les modules ES, TypeScript vous oblige à utiliser l'extension `.js` dans les importations et lors de l'exécution des scripts. Vous pouvez en apprendre davantage sur la thèse derrière ce choix dans la [documentation TypeScript](https://www.typescriptlang.org/docs/handbook/modules/theory.html#typescript-imitates-the-hosts-module-resolution-but-with-types).

## Exécution du serveur de développement

Au lieu d'exécuter directement le fichier `bin/server.js`, nous recommandons d'utiliser la commande `serve` pour les raisons suivantes :

- La commande inclut un observateur de fichiers et redémarre le serveur de développement lors des modifications de fichiers.
- La commande `serve` détecte le bundler de ressources frontend que votre application utilise et démarre son serveur de développement. Par exemple, si vous avez un fichier `vite.config.js` à la racine de votre projet, la commande `serve` démarrera le serveur de développement `vite`.

```sh
node ace serve --watch
```

Vous pouvez passer des arguments au serveur de développement Vite en utilisant le flag `--assets-args` en ligne de commande.

```sh
node ace serve --watch --assets-args="--debug --base=/public"
```

Vous pouvez utiliser le flag `--no-assets` pour désactiver le serveur de développement Vite.

```sh
node ace serve --watch --no-assets
```

### Passage d'options à la ligne de commande Node.js

La commande `serve` démarre le serveur de développement `(fichier bin/server.ts)` en tant que processus enfant. Si vous voulez passer des [arguments node](https://nodejs.org/api/cli.html#options) au processus enfant, vous pouvez les définir avant le nom de la commande.

```sh
node ace --no-warnings --inspect serve --watch
```

## Création de la version de production

Le version de production de votre application AdonisJS est créée en utilisant la commande `node ace build`. La commande `build` effectue les opérations suivantes pour créer une [**application JavaScript autonome**](#what-is-a-standalone-build) dans le répertoire `./build` :

- Supprime le dossier `./build` existant (s'il y en a un).
- Réécrit le fichier `ace.js` à **partir de zéro** pour supprimer le loader `ts-node/esm`.
- Compile les ressources frontend en utilisant Vite (si configuré).
- Compile le code source TypeScript en JavaScript en utilisant [`tsc`](https://www.typescriptlang.org/docs/handbook/compiler-options.html).
- Copie les fichiers non-TypeScript enregistrés dans le tableau [`metaFiles`](../concepts/adonisrc_file.md#metafiles) vers le dossier `./build`.
- Copie les fichiers `package.json` et `package-lock.json/yarn.lock` vers le dossier `./build`.

:::warning
Toute modification du fichier `ace.js` sera perdue pendant le processus de construction puisque le fichier est réécrit à partir de zéro. Si vous voulez avoir du code supplémentaire qui s'exécute avant qu'Ace ne démarre, vous devriez plutôt le faire dans le fichier `bin/console.ts`.
:::

Et c'est tout !

```sh
node ace build
```

Une fois que la version de production a été créée, vous pouvez `cd` dans le dossier `build`, installer les dépendances de production et exécuter votre application.

```sh
cd build

# Installer les dépendances de production
npm i --omit=dev

# Exécuter le serveur
node bin/server.js
```

Vous pouvez passer des arguments à la commande de construction Vite en utilisant le flag `--assets-args` en ligne de commande.

```sh
node ace build --assets-args="--debug --base=/public"
```

Vous pouvez utiliser le flag `--no-assets` pour éviter de compiler les ressources frontend.

```sh
node ace build --no-assets
```

### Qu'est-ce qu'une version autonome ?

Une version autonome fait référence au JavaScript de votre application que vous pouvez exécuter sans la source TypeScript originale.

La création d'une version autonome aide à réduire la taille du code que vous déployez sur votre serveur de production, car vous n'avez pas à copier à la fois les fichiers source et  JavaScript.

Après avoir créé la version de production, vous pouvez copier le `./build` sur votre serveur de production, installer les dépendances, définir les variables d'environnement et exécuter l'application.
