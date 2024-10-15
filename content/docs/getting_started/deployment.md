---
summary: Découvrez les directives générales pour déployer une application AdonisJS en production.
---

# Déploiement

Le déploiement d'une application AdonisJS n'est pas différent du déploiement d'une application Node.js standard. Vous avez besoin d'un serveur exécutant `Node.js >= 20.6` ainsi que `npm` pour installer les dépendances de production.

Ce guide couvrira les directives générales pour déployer et exécuter une application AdonisJS en production.

## Création de la version de production

Comme première étape, vous devez créer la version de production de votre application AdonisJS en exécutant la commande de `construction`.

Voir aussi : [Processus de construction TypeScript](../concepts/typescript_build_process.md)

```sh
node ace build
```

La résultat de la compilation est placé dans le répertoire `./build`. Si vous utilisez Vite, les fichiers générés seront placés dans le répertoire `./build/public`.

Une fois que vous avez créé la version de production, vous pouvez copier le dossier `./build` sur votre serveur de production. **À partir de maintenant, le dossier build sera la racine de votre application**.

### Création d'une image Docker

Si vous utilisez Docker pour déployer votre application, vous pouvez créer une image Docker en utilisant le `Dockerfile` suivant.

```dockerfile
FROM node:20.12.2-alpine3.18 AS base

# Étape de toutes les dépendances
FROM base AS deps
WORKDIR /app
ADD package.json package-lock.json ./
RUN npm ci

# Étape des dépendances de production uniquement
FROM base AS production-deps
WORKDIR /app
ADD package.json package-lock.json ./
RUN npm ci --omit=dev

# Étape de construction
FROM base AS build
WORKDIR /app
COPY --from=deps /app/node_modules /app/node_modules
ADD . .
RUN node ace build

# Étape de production
FROM base
ENV NODE_ENV=production
WORKDIR /app
COPY --from=production-deps /app/node_modules /app/node_modules
COPY --from=build /app/build /app
EXPOSE 8080
CMD ["node", "./bin/server.js"]
```

N'hésitez pas à modifier le Dockerfile pour l'adapter à vos besoins.

## Configuration d'un proxy inverse

Les applications Node.js sont généralement déployées [derrière un serveur proxy inverse](https://medium.com/intrinsic-blog/why-should-i-use-a-reverse-proxy-if-node-js-is-production-ready-5a079408b2ca) comme Nginx. Ainsi, le trafic entrant sur les ports `80` et `443` sera d'abord géré par Nginx, puis transmis à votre application Node.js.

Voici un exemple de fichier de configuration Nginx que vous pouvez utiliser comme point de départ.

:::warning
Assurez-vous de remplacer les valeurs entre crochets angulaires `<>`.
:::

```nginx
server {
  listen 80;
  listen [::]:80;

  server_name <APP_DOMAIN.COM>;

  location / {
    proxy_pass http://localhost:<ADONIS_PORT>;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_cache_bypass $http_upgrade;
  }
}
```

## Définition des variables d'environnement

Si vous déployez votre application sur un serveur minimaliste, comme un Droplet DigitalOcean ou une instance EC2, vous pouvez utiliser un fichier `.env` pour définir les variables d'environnement. Assurez-vous que le fichier est stocké de manière sécurisée et que seuls les utilisateurs autorisés peuvent y accéder.

:::note
Si vous utilisez une plateforme de déploiement comme Heroku ou Cleavr, vous pouvez utiliser leur panneau de contrôle pour définir les variables d'environnement.
:::

En supposant que vous ayez créé le fichier `.env` dans un répertoire `/etc/secrets`, vous devez démarrer votre serveur de production de la manière suivante :

```sh
ENV_PATH=/etc/secrets node build/bin/server.js
```

La variable d'environnement `ENV_PATH` indique à AdonisJS de chercher le fichier `.env` dans le répertoire mentionné.

## Démarrage du serveur de production

Vous pouvez démarrer le serveur de production en exécutant le fichier `node server.js`. Cependant, il est recommandé d'utiliser un gestionnaire de processus comme [pm2](https://pm2.keymetrics.io/docs/usage/quick-start).

- PM2 exécutera votre application en arrière-plan sans bloquer la session de terminal en cours.
- Il redémarrera l'application si elle plante pendant le traitement des requêtes.
- De plus, PM2 facilite grandement l'exécution de votre application en [mode cluster](https://nodejs.org/api/cluster.html#cluster).

Voici un exemple de [fichier ecosystem pm2](https://pm2.keymetrics.io/docs/usage/application-declaration) que vous pouvez utiliser comme point de départ.

```js
// title: ecosystem.config.js
module.exports = {
  apps: [
    {
      name: 'web-app',
      script: './server.js',
      instances: 'max',
      exec_mode: 'cluster',
      autorestart: true,
    },
  ],
}
```

```sh
// title: Démarrer le serveur
pm2 start ecosystem.config.js
```

## Migration de la base de données

Si vous utilisez une base de données SQL, vous devez exécuter les migrations de la base de données sur le serveur de production pour créer les tables requises.

Si vous utilisez Lucid, vous pouvez exécuter la commande suivante :

```sh
node ace migration:run --force
```

Le flag `--force` est requis lors de l'exécution des migrations dans l'environnement de production.

### Quand exécuter les migrations

Il est préférable d'exécuter toujours les migrations avant de redémarrer le serveur. Si la migration échoue, ne redémarrez pas le serveur.

En utilisant un service géré comme Cleavr ou Heroku, ils peuvent gérer automatiquement ce cas d'utilisation. Sinon, vous devrez exécuter le script de migration dans un pipeline CI/CD ou l'exécuter manuellement via SSH.

### Ne pas effectuer de rollback en production

Effectuer un rollback des migrations en production est une opération risquée. La méthode `down` dans vos fichiers de migration contient généralement des actions destructrices comme **la suppression de table** ou **la suppression de colonne**, etc.

Il est recommandé de désactiver les rollbacks en production dans le fichier `config/database.ts` et à la place, créer une nouvelle migration pour corriger le problème et l'exécuter sur le serveur de production.

La désactivation des rollbacks en production garantira que la commande `node ace migration:rollback` génère une erreur.

```js
{
  pg: {
    client: 'pg',
    migrations: {
      disableRollbacksInProduction: true,
    }
  }
}
```

### Migrations concurrentes

Si vous exécutez des migrations sur un serveur avec plusieurs instances, vous devez vous assurer qu'une seule instance exécute les migrations.

Pour MySQL et PostgreSQL, Lucid obtiendra des verrous consultatifs pour s'assurer que la migration concurrente n'est pas autorisée. Cependant, il est préférable d'éviter d'exécuter des migrations à partir de plusieurs serveurs dès le départ.

## Stockage persistant pour les téléchargements de fichiers

Les environnements comme Amazon EKS, Google Kubernetes, Heroku, DigitalOcean Apps, etc., exécutent votre code d'application dans [un système de fichiers éphémère](https://devcenter.heroku.com/articles/dynos#ephemeral-filesystem), ce qui signifie que chaque déploiement, par défaut, supprimera le système de fichiers existant et en créera un nouveau.

Si votre application permet aux utilisateurs de télécharger des fichiers, vous devez utiliser un service de stockage persistant comme Amazon S3, Google Cloud Storage ou DigitalOcean Spaces au lieu de vous fier au système de fichiers local.

## Écriture des logs

AdonisJS utilise par défaut le [logger pino](../digging_deeper/logger.md), qui écrit les logs dans la console au format JSON. Vous pouvez soit configurer un service de logging externe pour lire les logs depuis stdout/stderr, soit les transférer vers un fichier local sur le même serveur.

## Servir des ressources statiques

Servir efficacement des ressources statiques est essentiel pour les performances de votre application. Quelle que soit la rapidité de vos applications AdonisJS, la distribution des ressources statiques joue un rôle majeur dans l'amélioration de l'expérience utilisateur.

### Utilisation d'un CDN

La meilleure approche consiste à utiliser un CDN (Content Delivery Network) pour délivrer les ressources statiques de votre application AdonisJS.

Les ressources front-end compilées à l'aide de [Vite](../basics/vite.md) sont dotées d'une empreinte unique (fingerprinted) par défaut, ce qui signifie que les noms de fichiers sont hachés en fonction de leur contenu. Cela vous permet de mettre en cache les ressources indéfiniment et de les servir depuis un CDN.

Selon le service CDN que vous utilisez et votre technique de déploiement, vous devrez peut-être ajouter une étape à votre processus de déploiement pour déplacer les fichiers statiques vers le serveur CDN. Pour faire simple, voici comment cela devrait fonctionner :

1. Mettez à jour la configuration `vite.config.js` et `config/vite.ts` pour [utiliser l'URL du CDN](../basics/vite.md#deploying-assets-to-a-cdn).

2. Exécutez la commande de `construction` pour compiler l'application et les ressources.

3. Copiez le contenu de `public/assets` vers votre serveur CDN. Par exemple, [voici une commande](https://github.com/adonisjs-community/polls-app/blob/main/commands/PublishAssets.ts) que nous utilisons pour publier les ressources vers un bucket Amazon S3.

### Utilisation de Nginx pour délivrer des ressources statiques

Une autre option consiste à déléguer la tâche de servir les ressources à Nginx. Si vous utilisez Vite pour compiler les ressources front-end, vous devez mettre en cache de manière agressive tous les fichiers statiques car ils possèdent une empreinte unique.

Ajoutez le bloc suivant à votre fichier de configuration Nginx. **Assurez-vous de remplacer les valeurs entre crochets angulaires `<>`**.

```nginx
location ~ \.(jpg|png|css|js|gif|ico|woff|woff2) {
  root <PATH_TO_ADONISJS_APP_PUBLIC_DIRECTORY>;
  sendfile on;
  sendfile_max_chunk 2m;
  add_header Cache-Control "public";
  expires 365d;
}
```

### Utilisation du serveur de fichiers statiques d'AdonisJS

Vous pouvez également vous fier au [serveur de fichiers statiques intégré d'AdonisJS](../basics/static_file_server.md) pour servir les ressources statiques depuis le répertoire `public` afin de simplifier les choses.

Aucune configuration supplémentaire n'est requise. Déployez simplement votre application AdonisJS comme d'habitude, et les requêtes pour les ressources statiques seront automatiquement servies.

:::warning
Le serveur de fichiers statiques n'est pas recommandé pour une utilisation en production. Il est préférable d'utiliser un CDN ou Nginx pour servir les ressources statiques.
:::
