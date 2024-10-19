---
summary: Mettez à jour votre application AdonisJS sans redémarrer le processus en utilisant le remplacement de module à chaud (HMR).
---

# HMR

Le remplacement de module à chaud (Hot Module Replacement ou HMR) fait référence au processus de rechargement des modules JavaScript après modification sans redémarrer l'ensemble du processus. Le HMR se traduit généralement par une boucle de rétroaction plus rapide car, après un changement de fichier, vous n'avez pas à attendre le redémarrage complet du processus.

Le terme HMR est utilisé depuis de nombreuses années dans l'écosystème frontend, où des outils comme Vite peuvent recharger à chaud les modules et appliquer des modifications à une page web tout en maintenant son état existant.

Cependant, le HMR effectué par AdonisJS est beaucoup plus simple et diffère considérablement des outils comme Vite ou Webpack. Notre objectif avec le HMR est d'offrir des rechargements plus rapides, c'est tout.

## Concepts clés

### Aucune mise à jour n'est propagée au navigateur

Étant donné qu'AdonisJS est un framework backend, nous ne sommes pas chargés de maintenir l'état d'une application frontend ou d'appliquer du CSS à une page web. Par conséquent, notre intégration HMR ne peut pas communiquer avec votre application frontend et réconcilier son état.

En fait, toutes les applications AdonisJS ne sont pas des applications web rendues par le navigateur. Beaucoup utilisent AdonisJS pour créer des API JSON pures, et elles peuvent également bénéficier de notre intégration HMR.

### Fonctionne uniquement avec les importations dynamiques

La plupart des outils HMR utilisent des transformations de code pour injecter du code supplémentaire dans le résultat compilé. Chez AdonisJS, nous ne sommes pas de grands fans des transpileurs et nous nous efforçons toujours d'adopter la plateforme telle qu'elle est. Par conséquent, notre approche du HMR utilise les [hooks de chargement de Node.js](https://nodejs.org/api/module.html#customization-hooks) et fonctionne uniquement avec les importations dynamiques.

**La bonne nouvelle est que toutes les parties critiques de votre application AdonisJS sont importées dynamiquement par défaut**. Par exemple, les contrôleurs, les middlewares et les écouteurs d'événements sont tous importés dynamiquement, et vous pouvez donc tirer parti du HMR dès aujourd'hui sans changer une seule ligne de code dans votre application.

Il est important de mentionner que les importations d'un module importé dynamiquement peuvent être au niveau supérieur. Par exemple, un contrôleur (qui est importé dynamiquement dans le fichier de routes) peut avoir des importations de niveau supérieur pour les validateurs, les fichiers TSX, les modèles et les services, et ils bénéficient tous du HMR.

## Utilisation

Tous les kits de démarrage officiels ont été mis à jour pour utiliser le HMR par défaut. Cependant, si vous avez une application existante, vous pouvez configurer le HMR de la manière suivante.

Installez le package npm [hot-hook](https://github.com/Julien-R44/hot-hook) en tant que dépendance de développement. L'équipe principale d'AdonisJS a créé ce package, qui peut également être utilisé en dehors d'une application AdonisJS.

```sh
npm i -D hot-hook
```

Ensuite, copiez-collez la configuration suivante dans le fichier `package.json`. La propriété `boundaries` accepte un tableau de motifs glob qui doivent être pris en compte pour le HMR.

```json
{
  "hotHook": {
    "boundaries": [
      "./app/controllers/**/*.ts",
      "./app/middleware/*.ts"
    ]
  }
}
```

Après la configuration, vous pouvez démarrer le serveur de développement avec le flag `--hmr`.

```sh
node ace serve --hmr
```

Vous voudrez peut-être également mettre à jour le script `dev` dans le fichier `package.json` pour utiliser ce nouveau flag.

```json
{
  "scripts": {
    "dev": "node ace serve --hmr"
  }
}
```

## Rechargements complets vs HMR

:::note
Cette section explique le fonctionnement de `hot-hook`. N'hésitez pas à la sauter si vous n'êtes pas d'humeur à lire une théorie technique approfondie 🤓

Ou, consultez le [fichier README](https://github.com/Julien-R44/hot-hook) du package si vous voulez une explication encore plus détaillée.
:::

Comprenons quand AdonisJS effectuera un rechargement complet (redémarrage du processus) et quand il rechargera à chaud le module.

### Création d'un arbre de dépendances

Lors de l'utilisation d'un flag `--hmr`, AdonisJS utilisera `hot-hook` pour créer un arbre de dépendances de votre application à partir du fichier `bin/server.ts` et surveillera tous les fichiers qui font partie de cet arbre de dépendances.

Cela signifie que si vous créez un fichier TypeScript dans le code source de votre application mais ne l'importez jamais nulle part dans votre application, ce fichier ne déclenchera aucun rechargement. Il sera ignoré comme si le fichier n'existait pas.

### Identification des limites

Ensuite, `hot-hook` utilisera le tableau `boundaries` de la configuration pour identifier les fichiers qui sont éligibles pour le HMR.

En règle générale, vous ne devriez jamais enregistrer les fichiers de configuration, les fournisseurs de services ou les fichiers à précharger comme limites. Ces fichiers entraînent généralement des effets secondaires qui se reproduiront si nous les rechargeons sans effacer les effets secondaires. Voici quelques exemples :

- Le fichier `config/database.ts` établit une connexion avec la base de données. Recharger à chaud ce fichier signifie fermer la connexion existante et la recréer. Le même résultat peut être obtenu en redémarrant l'ensemble du processus sans ajouter de complexité supplémentaire.

- Le fichier `start/routes.ts` est utilisé pour enregistrer les routes. Recharger à chaud ce fichier signifie supprimer les routes existantes enregistrées avec le framework et les réenregistrer. Là encore, redémarrer le processus est simple.

En d'autres termes, nous pouvons dire que les modules importés/exécutés pendant une requête HTTP devraient faire partie des limites du HMR, et les modules nécessaires au démarrage de l'application ne devraient pas en faire partie.

### Exécution des rechargements

Une fois que `hot-hook` a identifié les limites, il effectuera le HMR pour les modules importés dynamiquement qui font partie de la limite et redémarrera le processus pour le reste des fichiers.

