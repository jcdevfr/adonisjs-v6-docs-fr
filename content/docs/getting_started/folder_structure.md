---
summary: Faites le tour des fichiers et dossiers importants créés par AdonisJS lors du processus d'installation.
---

# Structure des dossiers

Dans ce guide, nous allons explorer les fichiers et dossiers importants créés par AdonisJS pendant le processus d'installation.

Nous fournissons une structure de dossiers par défaut bien pensée qui vous aide à garder vos projets ordonnés et faciles à refactoriser. Cependant, vous avez toute la liberté de diverger et d'avoir une structure de dossiers qui fonctionne bien pour votre équipe et votre projet.

## Le fichier `adonisrc.ts`

Le fichier `adonisrc.ts` est utilisé pour configurer l'espace de travail et certains paramètres d'exécution de votre application.

Dans ce fichier, vous pouvez enregistrer des fournisseurs de services, définir des alias de commandes ou spécifier les fichiers à copier dans la version de production.

Voir aussi : [Guide de référence du fichier AdonisRC](../concepts/adonisrc_file.md)

## Le fichier `tsconfig.json`

Le fichier `tsconfig.json` contient la configuration TypeScript de votre application. N'hésitez pas à apporter des modifications à ce fichier selon les besoins de votre projet ou de votre équipe.

Les options de configuration suivantes sont requises pour que les composants internes d'AdonisJS fonctionnent correctement.

```json
{
  "compilerOptions": {
    "module": "NodeNext",
    "isolatedModules": true,
    "declaration": false,
    "outDir": "./build",
    "esModuleInterop": true,
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,
    "skipLibCheck": true
  }
}
``` 

## Les importations de sous-chemins

AdonisJS utilise les [importations de sous-chemins](https://nodejs.org/dist/latest-v19.x/docs/api/packages.html#subpath-imports) de Node.js pour définir les alias d'importation.

Les alias d'importation suivants sont pré-configurés dans le fichier `package.json`. N'hésitez pas à ajouter de nouveaux alias ou à modifier ceux déjà existants.

```json
// title: package.json
{
  "imports": {
    "#controllers/*": "./app/controllers/*.js",
    "#exceptions/*": "./app/exceptions/*.js",
    "#models/*": "./app/models/*.js",
    "#mails/*": "./app/mails/*.js",
    "#services/*": "./app/services/*.js",
    "#listeners/*": "./app/listeners/*.js",
    "#events/*": "./app/events/*.js",
    "#middleware/*": "./app/middleware/*.js",
    "#validators/*": "./app/validators/*.js",
    "#providers/*": "./app/providers/*.js",
    "#policies/*": "./app/policies/*.js",
    "#abilities/*": "./app/abilities/*.js",
    "#database/*": "./database/*.js",
    "#tests/*": "./tests/*.js",
    "#start/*": "./start/*.js",
    "#config/*": "./config/*.js"
  }
}
```

## Le répertoire`bin`

Le répertoire `bin` contient les fichiers de point d'entrée pour charger votre application dans un environnement spécifique. Par exemple :

- Le fichier `bin/server.ts` démarre l'application dans l'environnement web pour écouter les requêtes HTTP. 
- Le fichier `bin/console.ts` démarre la ligne de commande Ace et exécute les commandes.
- Le fichier `bin/test.ts` démarre l'application pour exécuter les tests.

## Le fichier `ace.js`

Le fichier `ace` démarre le framework de ligne de commande qui est local à votre application. Ainsi, chaque fois que vous exécutez une commande ace, elle passe par ce fichier.

Si vous remarquez, le fichier ace se termine par une extension `.js`. C'est parce que nous voulons exécuter ce fichier en utilisant node sans le compiler.

## Le répertoire `app`

Le répertoire `app` organise le code pour la logique métier de votre application. Par exemple, les contrôleurs, les modèles, les services, etc., se trouvent tous dans le répertoire `app`.

N'hésitez pas à créer des répertoires supplémentaires pour mieux organiser le code de votre application.

```
├── app
│  └── controllers
│  └── exceptions
│  └── middleware
│  └── models
│  └── validators
```


## Le répertoire `resources`

Le répertoire `resources` contient les templates Edge, ainsi que les fichiers source de votre code frontend. En d'autres termes, le code de la couche de présentation de votre application se trouve dans le répertoire `resources`.

```
├── resources
│  └── views
│  └── js
│  └── css
│  └── fonts
│  └── images
```

## Le répertoire `start`

Le répertoire `start` contient les fichiers que vous voulez importer pendant le cycle de démarrage de l'application. Par exemple, les fichiers pour enregistrer les routes et définir les écouteurs d'événements devraient se trouver dans le répertoire `start`.

```
├── start
│  ├── env.ts
│  ├── kernel.ts
│  ├── routes.ts
│  ├── validator.ts
│  ├── events.ts
```

AdonisJS n'importe pas automatiquement les fichiers du répertoire `start`. Il est simplement utilisé comme une convention pour regrouper des fichiers similaires.

Nous vous recommandons de lire la documentation sur les [fichiers de préchargement](../concepts/adonisrc_file.md#preloads) et le [cycle de démarrage de l'application](../concepts/application_lifecycle.md) pour mieux comprendre quels fichiers garder dans le répertoire start.

## Le répertoire `public`

Le répertoire `public` héberge les ressources statiques comme les fichiers CSS, les images, les polices ou le JavaScript frontend.

Ne confondez pas le répertoire `public` avec le répertoire `resources`. Le répertoire resources contient le code source de votre application frontend, et le répertoire public contient le résultat compilé.

Lorsque vous utilisez Vite, vous devez stocker les ressources frontend dans les répertoires `resources/<SUB_DIR>` et laisser le compilateur Vite créer les fichiers finaux dans le répertoire `public`.

D'autre part, si vous n'utilisez pas Vite, vous pouvez créer des fichiers directement dans le répertoire `public` et y accéder en utilisant le nom du fichier. Par exemple, vous pouvez accéder au fichier `./public/style.css` à partir de l'URL `http://localhost:3333/style.css`.

## Le répertoire `database`

Le répertoire `database` contient les fichiers pour les migrations de base de données et les seeders.

```
├── database
│  └── migrations
│  └── seeders
```


## Le répertoire `commands`

Les [commandes ace](../ace/introduction.md) sont stockées dans le répertoire `commands`. Vous pouvez créer des commandes dans ce répertoire en exécutant `node ace make:command`.

## Le répertoire `config`

Le répertoire `config` contient les fichiers de configuration d'exécution pour votre application.

Le noyau du framework et les autres packages installés lisent les fichiers de configuration à partir de ce répertoire. Vous pouvez également stocker la configuration locale à votre application dans ce répertoire.

En savoir plus sur la [gestion de la configuration](./configuration.md).

```
├── config
│  ├── app.ts
│  ├── bodyparser.ts
│  ├── cors.ts
│  ├── database.ts
│  ├── drive.ts
│  ├── hash.ts
│  ├── logger.ts
│  ├── session.ts
│  ├── static.ts
```


## Le répertoire `types`

Le répertoire `types` contient les interfaces ou types TypeScript utilisés dans votre application.

Le répertoire est vide par défaut, cependant, vous pouvez créer des fichiers et des dossiers dans le répertoire `types` pour définir des types et des interfaces personnalisés.

```
├── types
│  ├── events.ts
│  ├── container.ts
```

## Le répertoire `providers`

Le répertoire `providers` est utilisé pour stocker les [fournisseurs de services](../concepts/service_providers.md) utilisés par votre application. Vous pouvez créer de nouveaux fournisseurs en utilisant la commande `node ace make:provider`.

En savoir plus sur les [fournisseurs de services](../concepts/service_providers.md)

```
├── providers
│  └── app_provider.ts
│  └── http_server_provider.ts
```

## Le répertoire `tmp`

Les fichiers temporaires générés par votre application sont stockés dans le répertoire `tmp`. Par exemple, il peut s'agir de fichiers téléchargés par les utilisateurs (générés pendant le développement) ou de logs écrits sur le disque.

Le répertoire `tmp` doit être ignoré par les règles du fichier `.gitignore`, et vous ne devez pas non plus le copier sur le serveur de production.

## Le répertoire `tests`

Le répertoire `tests` organise les tests de votre application. De plus, des sous-répertoires sont créés pour les tests `unitaires` et `fonctionnels`.

Voir aussi : [Tests](../testing/introduction.md)

```
├── tests
│  ├── bootstrap.ts
│  └── functional
│  └── regression
│  └── unit
```
