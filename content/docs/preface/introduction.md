---
summary: "AdonisJS est un framework web Node.js écrit en TypeScript. Vous pouvez l'utiliser pour créer une application web full-stack ou une API JSON."
---

# Introduction

::include{template="partials/introduction_cards"}

## Qu'est-ce qu'AdonisJS ?

AdonisJS est un framework web Node.js écrit en TypeScript. Vous pouvez l'utiliser pour créer une application web full-stack ou une API JSON.

Fondamentalement, AdonisJS [fournit une structure à vos applications](../getting_started/folder_structure.md), configure un [environnement de développement TypeScript transparent](../concepts/typescript_build_process.md),  met en place le [HMR](../concepts/hmr.md) (Hot Module Replacement) pour votre code backend,  et offre une vaste collection de packages bien maintenus et largement documentés.

Notre vision est que les équipes utilisant AdonisJS **passent moins de temps** sur des décisions triviales comme la sélection minutieuse de packages npm pour chaque fonctionnalité mineure, l'écriture de code de liaison, les débats sur la structure parfaite des dossiers, et **consacrent plus de temps**  à la réalisation de fonctionnalités réelles critiques pour les besoins de l'entreprise.

### Côté frontend

AdonisJS se concentre sur le backend et vous laisse choisir la stack frontend de votre choix.

Si vous aimez garder les choses simples, associez AdonisJS à un [moteur de template traditionnel](../views-and-templates/introduction.md) pour générer du HTML statique côté serveur, créez une API JSON pour votre application frontend Vue/React ou utilisez [Inertia](../views-and-templates/inertia.md) pour faire fonctionner votre framework frontend préféré en parfaite harmonie.

AdonisJS vise à vous fournir tous les outils nécessaires pour créer une application backend robuste from scratch. Qu'il s'agisse d'envoyer des emails, de valider les entrées utilisateur, d'effectuer des opérations CRUD ou d'authentifier des utilisateurs. Nous nous occupons de tout.

### Moderne et type-safe

AdonisJS est construit sur des primitives JavaScript modernes. Nous utilisons les modules ES, les alias d'importation de sous-chemins Node.js, SWC pour exécuter le code source TypeScript et Vite pour le bundling des assets.

De plus, TypeScript joue un rôle considérable dans la conception des APIs du framework. Par exemple, AdonisJS dispose de :

- [Un émetteur d'événements type-safe](../digging_deeper/emitter.md#making-events-type-safe)
- [Des variables d'environnement type-safe](../getting_started/environment_variables.md)
- [Une bibliothèque de validation type-safe](../basics/validation.md)

### Adoption du MVC

AdonisJS adopte le modèle de conception MVC classique. Vous commencez par définir les routes en utilisant l'API JavaScript fonctionnelle, liez des contrôleurs à ces routes et écrivez la logique pour gérer les requêtes HTTP dans les contrôleurs.

```ts
// title: start/routes.ts
import router from '@adonisjs/core/services/router'
const PostsController = () => import('#controllers/posts_controller')

router.get('posts', [PostsController, 'index'])
```

Les contrôleurs peuvent utiliser des modèles pour récupérer des données de la base de données et rendre une vue (template) comme réponse.

```ts
// title: app/controllers/posts_controller.ts
import Post from '#models/post'
import type { HttpContext } from '@adonisjs/core/http'

export default class PostsController {
  async index({ view }: HttpContext) {
    const posts = await Post.all()
    return view.render('pages/posts/list', { posts })
  }
}
```

Si vous construisez une API, vous pouvez remplacer la couche vue par une réponse JSON. Mais le processus de gestion et de réponse aux requêtes HTTP reste le même.

```ts
// title: app/controllers/posts_controller.ts
import Post from '#models/post'
import type { HttpContext } from '@adonisjs/core/http'

export default class PostsController {
  async index({ view }: HttpContext) {
    const posts = await Post.all()
    // delete-start
    return view.render('pages/posts/list', { posts })
    // delete-end
    // insert-start
    /**
     * Le tableau de posts sera 
     * automatiquement sérialisé en JSON.
     */
    return posts
    // insert-end
  }
}
```

## À propos de ce guide

La documentation d'AdonisJS est rédigée comme un guide de référence, couvrant l'utilisation et l'API de plusieurs packages et modules maintenus par l'équipe principale.

**Le guide n'enseigne pas comment construire une application from scratch**. Si vous cherchez un tutoriel, nous vous recommandons de commencer votre voyage avec [Adocasts](https://adocasts.com/). Tom (le créateur d'Adocasts) a créé des screencasts de haute qualité, vous aidant à faire vos premiers pas avec AdonisJS.

Cela étant dit, la documentation couvre en détail l'utilisation des modules disponibles et le fonctionnement interne du framework.

## Versions récentes
Voici la liste des versions récentes. [Cliquez ici](./releases.md) pour voir toutes les versions.

::include{template="partials/recent_releases"}

## Sponsors

::include{template="partials/sponsors"}
