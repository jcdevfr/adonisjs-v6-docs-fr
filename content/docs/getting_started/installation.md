---
summary: Comment créer et configurer une nouvelle application AdonisJS.
---

# Installation

Avant de créer une nouvelle application, assurez-vous d'avoir Node.js et npm installés sur votre ordinateur. AdonisJS nécessite `Node.js >= 20.6`.

Vous pouvez installer Node.js en utilisant soit les [ installateurs officiels](https://nodejs.org/en/download/), soit  [Volta](https://docs.volta.sh/guide/getting-started). Volta est un gestionnaire de packages multiplateforme qui permet d'installer et d'exécuter plusieurs versions de Node.js sur votre ordinateur.

```sh
// title: Vérifier la version de Node.js
node -v
# v22.0.0
```

:::tip
**Vous apprenez mieux visuellement ?** - Découvrez la série gratuite de screencasts [Let's Learn AdonisJS 6](https://adocasts.com/series/lets-learn-adonisjs-6) de nos amis chez Adocasts.
:::


## Créer une nouvelle application

Vous pouvez créer un nouveau projet en utilisant [npm init](https://docs.npmjs.com/cli/v7/commands/npm-init). Ces commandes téléchargeront le package d'initialisation [create-adonisjs](http://npmjs.com/create-adonisjs) et démarreront le processus d'installation.

Vous pouvez personnaliser la sortie initiale du projet en utilisant l'un des flags du CLI suivants :

- `--kit`: Sélectionnez le [kit de démarrage](#starter-kits) pour le projet. Vous pouvez choisir entre **web**, **api**, **slim** ou **inertia**.

- `--db`: Spécifiez le dialecte de base de données de votre choix. Vous pouvez choisir entre **sqlite**, **postgres**, **mysql**, ou **mssql**.

- `--git-init`: Initialisez le dépôt git. Par défaut à `false`.

- `--auth-guard`: Spécifiez le garde d'authentification de votre choix. Vous pouvez choisir entre **session**, **access_tokens**, ou **basic_auth**.

:::codegroup

```sh
// title: npm
npm init adonisjs@latest hello-world
```

:::

Lorsque vous passez des flags au CLI en utilisant la commande `npm init`, assurez-vous d'utiliser deux fois les [doubles tirets](https://stackoverflow.com/questions/43046885/what-does-do-when-running-an-npm-command). Sinon,`npm init` ne transmettra pas les flags au package d'initialisation `create-adonisjs`. Par exemple :

```sh
# Créer un projet et être invité à configurer toutes les options
npm init adonisjs@latest hello-world

# Créer un projet avec MySQL
npm init adonisjs@latest hello-world -- --db=mysql

# Créer un projet avec PostgreSQL et le kit de démarrage API
npm init adonisjs@latest hello-world -- --db=postgres --kit=api

# Créer un projet avec le kit de démarrage API et le garde des jetons d'accès
npm init adonisjs@latest hello-world -- --kit=api --auth-guard=access_tokens
```

## Kits de démarrage

Les kits de démarrage servent de point de départ pour créer des applications avec AdonisJS. Ils sont fournis avec une [structure de dossiers définie](./folder_structure.md), des packages AdonisJS pré-configurés et les outils nécessaires pendant le développement.


:::note
Les kits de démarrage officiels utilisent les modules ES et TypeScript. Cette combinaison vous permet d'utiliser des structures JavaScript modernes et de bénéficier de la sécurité des types statiques.
:::

### Kit de démarrage Web

Le kit de démarrage Web est conçu pour créer des applications web traditionnelles avec rendu côté serveur. Ne vous laissez pas décourager par le mot **"traditionnel"**. Nous recommandons ce kit de démarrage si vous créez une application web avec une interactivité frontend limitée.

La simplicité du rendu HTML sur le serveur avec [Edge.js](https://edgejs.dev) augmentera votre productivité car vous n'aurez pas à gérer des systèmes de construction complexes pour rendre du HTML.

Plus tard, vous pourrez utiliser [Hotwire](https://hotwired.dev), [HTMX](http://htmx.org) ou [Unpoly](http://unpoly.com) pour que vos applications se comportent comme une SPA et utiliser [Alpine.js](http://alpinejs.dev) pour créer des widgets interactifs comme un menu déroulant ou une modale.

```sh
npm init adonisjs@latest -- -K=web

# Change le dialecte de la base de données.
npm init adonisjs@latest -- -K=web --db=mysql
```

Le kit de démarrage web est composé des packages suivants :

<table>
<thead>
<tr>
<th width="180px">Package</th>
<th>Description</th>
</tr>
</thead>
<tbody><tr>
<td><code>@adonisjs/core</code></td>
<td>Le noyau du framework contenant les fonctionnalités de base dont vous pourriez avoir besoin lors de la création d'applications backend.</td>
</tr>
<tr>
<td><code>edge.js</code></td>
<td>Le moteur de template <a href="https://edgejs.dev">edge</a> pour composer des pages HTML.</td>
</tr>
<tr>
<td><code>@vinejs/vine</code></td>
<td><a href="https://vinejs.dev">VineJS</a> est l'une des bibliothèques de validation les plus rapides de l'écosystème Node.js.</td>
</tr>
<tr>
<td><code>@adonisjs/lucid</code></td>
<td>Lucid est un ORM SQL maintenu par l'équipe principale d'AdonisJS.</td>
</tr>
<tr>
<td><code>@adonisjs/auth</code></td>
<td>La couche d'authentification du framework. Elle est configurée pour utiliser les sessions.</td>
</tr>
<tr>
<td><code>@adonisjs/shield</code></td>

<td>Un ensemble de mécanismes de sécurité pour protéger vos applications web contre les attaques telles que <strong>CSRF</strong> et <strong>‌XSS</strong>.</td>
</tr>
<tr>
<td><code>@adonisjs/static</code></td>
<td>Middleware pour servir des ressources statiques depuis le répertoire <code>/public</code> de votre application.</td>
</tr>
<tr>
<td><code>vite</code></td>
<td><a href="https://vitejs.dev/">Vite</a> est utilisé pour compiler les ressources frontend.
</td>
</tr>
</tbody></table>

---

### Kit de démarrage API

Le kit de démarrage API est conçu pour créer des API JSON. C'est une version allégée du kit de démarrage `web`. Si vous prévoyez de construire votre application frontend avec React ou Vue, vous pouvez créer votre backend AdonisJS en utilisant le kit de démarrage API.

```sh
npm init adonisjs@latest -- -K=api

# Change le dialecte de la base de données.
npm init adonisjs@latest -- -K=api --db=mysql
```

Dans ce kit de démarrage :

- Nous supprimons le support pour servir des fichiers statiques.
- Nous ne configurons pas la couche des vues et vite.
- Nous désactivons la protection XSS et CSRF et activons la protection CORS.
- Nous utilisons le middleware ContentNegotiation pour envoyer des réponses HTTP en JSON.

Le kit de démarrage API est configuré avec une authentification basée sur les sessions. Cependant, si vous souhaitez utiliser une authentification basée sur les jetons, vous pouvez utiliser le flag `--auth-guard`.

Voir aussi : [Quel garde d'authentification dois-je utiliser ?](../authentication/introduction.md#choosing-an-auth-guard)

```sh
npm init adonisjs@latest -- -K=api --auth-guard=access_tokens
```

---

### Kit de démarrage Slim

Pour les minimalistes, nous avons créé un kit de démarrage `slim`. Il ne contient que le noyau du framework et la structure de dossiers par défaut. Vous pouvez l'utiliser quand vous ne voulez pas toutes les fonctionnalités d'AdonisJS.

```sh
npm init adonisjs@latest -- -K=slim

# Change le dialecte de la base de données.
npm init adonisjs@latest -- -K=slim --db=mysql
```

---

### Kit de démarrage Inertia

[Inertia](https://inertiajs.com/) est une façon de construire des applications monopages pilotées par le serveur. Vous pouvez utiliser votre framework frontend préféré (React, Vue, Solid, Svelte) pour construire le frontend de votre application.

Vous pouvez utiliser le flag `--adapter` pour choisir le framework frontend que vous voulez utiliser. Les options disponibles sont `react`, `vue`, `solid` et `svelte`.

Vous pouvez également utiliser les flags `--ssr` et `--no-ssr` pour activer ou désactiver le rendu côté serveur.

```sh
# React avec rendu côté serveur
npm init adonisjs@latest -- -K=inertia --adapter=react --ssr

# Vue sans rendu côté serveur
npm init adonisjs@latest -- -K=inertia --adapter=vue --no-ssr
```

---

### Apportez votre kit de démarrage

Les kits de démarrage sont des projets pré-construits hébergés avec un hébergeur de dépôt Git comme GitHub, Bitbucket ou GitLab. Vous pouvez également créer vos propres kits de démarrage et les télécharger de la manière suivante :

```sh
npm init adonisjs@latest -- -K="github_user/repo"

# Télécharge depuis GitLab
npm init adonisjs@latest -- -K="gitlab:user/repo"

# Télécharge depuis BitBucket
npm init adonisjs@latest -- -K="bitbucket:user/repo"
```

Vous pouvez télécharger des dépôts privés en utilisant l'authentification Git+SSH en mode `git` :

```sh
npm init adonisjs@latest -- -K="user/repo" --mode=git
```

Enfin, vous pouvez spécifier un tag, une branche ou un commit :

```sh
# Branche
npm init adonisjs@latest -- -K="user/repo#develop"

# Tag
npm init adonisjs@latest -- -K="user/repo#v2.1.0"
```

## Démarrer le serveur de développement

Une fois que vous avez créé une application AdonisJS, vous pouvez démarrer le serveur de développement en exécutant la commande `node ace serve`.

Ace est un framework en ligne de commande intégré dans le noyau du framework. Le flag `--hmr` surveille le système de fichiers et effectue le [remplacement de module à chaud (HMR)](../concepts/hmr.md) pour certaines sections de votre code.

```sh
node ace serve --hmr
```

Une fois le serveur de développement lancé, vous pouvez visiter [http://localhost:3333](http://localhost:3333) pour voir votre application dans un navigateur.

## Compiler pour la production

Les applications AdonisJS étant écrites en TypeScript, elles doivent être compilées en JavaScript avant d'être exécutées en production.

Vous pouvez créer la sortie JavaScript en utilisant la commande `node ace build`. La sortie JavaScript est écrite dans le répertoire `build`.

Lorsque Vite est configuré, cette commande compile également les ressources frontend en utilisant Vite et écrit la sortie dans le répertoire `build/public`.

Voir aussi : [Processus de construction TypeScript](../concepts/typescript_build_process.md).

```sh
node ace build
```

## Configurer l'environnement de développement

Bien qu'AdonisJS prenne en charge la construction des applications destinées aux utilisateurs finaux, vous pourriez avoir besoin d'outils supplémentaires pour apprécier le processus de développement et avoir une cohérence dans votre style de codage.

Nous recommandons fortement d'utiliser **[ESLint](https://eslint.org/)** pour analyser votre code et **[Prettier](https://prettier.io)** pour reformater votre code par souci de cohérence.

Les kits de démarrage officiels sont pré-configurés avec ESLint et Prettier et utilisent les préréglages recommandés par l'équipe principale d'AdonisJS. Vous pouvez en apprendre davantage à leur sujet dans la section [Configuration des outils](../concepts/tooling_config.md) de la documentation.

Enfin, nous vous recommandons d'installer les plugins ESLint et Prettier pour votre éditeur de code afin d'avoir un feedback pendant le développement de l'application. Vous pouvez également utiliser les commandes suivantes pour `analyser` et `formater` votre code depuis la ligne de commande.

```sh
# Exécute ESLint
npm run lint

# Exécute ESLint et corrige automatiquement les problèmes
npm run lint -- --fix

# Exécute prettier
npm run format
```

## Extensions VSCode

Vous pouvez développer une application AdonisJS sur n'importe quel éditeur de code prenant en charge TypeScript. Cependant, nous avons développé plusieurs extensions pour VSCode afin d'améliorer davantage l'expérience de développement.

- [**AdonisJS**](https://marketplace.visualstudio.com/items?itemName=jripouteau.adonis-vscode-extension) - Visualisez les routes de l'application, exécutez des commandes ace, migrez la base de données et lisez la documentation directement depuis votre éditeur de code.

- [**Edge**](https://marketplace.visualstudio.com/items?itemName=AdonisJS.vscode-edge) - Boostez votre worflow avec la prise en charge de la coloration syntaxique, l'autocomplétion et les snippets de code.

- [**Japa**](https://marketplace.visualstudio.com/items?itemName=jripouteau.japa-vscode) - Exécutez des tests sans quitter votre éditeur de code en utilisant des raccourcis clavier ou exécutez-les directement depuis la barre d'activité.
