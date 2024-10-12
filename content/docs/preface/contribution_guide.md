---
summary: Contribuer aux projets AdonisJS est un excellent moyen de rendre service à la communauté. Ce guide fournit une vue d'ensemble de la manière dont vous pouvez contribuer à un projet AdonisJS.
---

# Contribuer

Ceci est un guide général de contribution pour tous les dépôts [AdonisJS](https://github.com/adonisjs). Veuillez lire attentivement ce guide avant de contribuer à l'un des dépôts 🙏

Le code n'est pas la seule façon de contribuer. Voici d'autres moyens de contribuer et de faire partie de la communauté :

- Corriger des fautes de frappe dans la documentation.
- Améliorer la documentation existante.
- Écrire des guides pratiques ou des articles de blog pour former les autres membres de la communauté.
- Traitement des problèmes.
- Partager votre opinion sur les problèmes existants.
- Aider la communauté sur Discord et le forum de discussions.

## Signaler des bugs

De nombreux problèmes signalés sur les projets open source sont généralement des questions ou des erreurs de configuration du côté de la personne qui les signale. Par conséquent, nous vous recommandons vivement de bien résoudre vos problèmes avant de les signaler.

Si vous signalez un bug, incluez autant d'informations que possible avec les exemples de code que vous avez écrits. L'échelle des bons et mauvais problèmes (issues) se présente de la façon suivante :

- **PARFAIT** : Vous isolez le bug sous-jacent. Créez un test qui échoue dans le dépôt et ouvrez une issue Github à ce sujet.
- **BON** : Vous isolez le bug sous-jacent et fournissez une reproduction minimale sous forme de dépôt Github. Antfu a écrit un excellent article sur la [raison pour laquelle les reproductions sont nécessaires](https://antfu.me/posts/why-reproductions-are-required).
- **CORRECT** : Vous exposez correctement votre problème. Partagez le code qui produit le problème. Incluez également les fichiers de configuration associés et la version du package que vous utilisez.
Enfin, formatez correctement chaque bloc de code en suivant le [guide de syntaxe markdown de Github](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax).
- **MAUVAIS** : Vous posez simplement votre question en espérant que l'autre personne posera les questions pertinentes et vous aidera. Ce genre de problèmes est automatiquement fermé sans explication.

## Avoir une discussion

Vous souhaitez souvent discuter d'un sujet ou partager des idées. Dans ce cas, créez une discussion dans le forum de discussions sous la catégorie **💡Ideas**.

## Former les autres

Former les autres est l'une des meilleures façons de contribuer à une communauté et de gagner en reconnaissance.

Vous pouvez utiliser la catégorie **📚 Cookbooks** sur notre forum de discussions pour partager un article avec les autres. La section cookbooks n'est PAS strictement modérée, sauf que les connaissances partagées doivent être pertinentes pour le projet.

## Créer des pull requests

Ce n'est jamais une bonne expérience de voir sa pull request (PR) refusée après avoir investi beaucoup de temps et d'efforts dans l'écriture du code. Par conséquent, nous vous recommandons vivement de [lancer une discussion](https://github.com/orgs/adonisjs/discussions) avant de commencer tout nouveau travail de votre côté.

Démarrez simplement une discussion et expliquez ce que vous prévoyez de faire.

- **Vous essayez de créer une PR pour corriger un bug** : Les PR pour les bugs sont généralement acceptées une fois que le bug a été confirmé.
- **Vous prévoyez d'ajouter une nouvelle fonctionnalité** : Veuillez expliquer en détail pourquoi cette fonctionnalité est nécessaire et partagez des liens vers du matériel pédagogique que nous pouvons lire pour nous informer.

Par exemple : Si vous ajoutez la prise en charge des tests snapshot pour Japa ou AdonisJS, partagez les liens que je peux utiliser pour en apprendre davantage sur les tests snapshot en général.

> Note : Vous devez également être disponible pour ouvrir des PR supplémentaires pour documenter la fonctionnalité ou l'amélioration apportée.

## Configuration du dépôt

1. Commencez par cloner le dépôt sur votre machine locale.

    ```sh
    git clone <REPO_URL>
    ```

2. Installez les dépendances sur votre machine locale. Veuillez ne pas mettre à jour les dépendances en même temps qu'une demande de fonctionnalité. Si vous trouvez des dépendances obsolètes, créez une PR séparée pour les mettre à jour.

   Nous utilisons `npm` pour gérer les dépendances, donc n'utilisez pas `yarn` ou tout autre outil.

    ```sh
    npm install
    ```

3. Exécutez les tests avec la commande suivante.

    ```sh
    npm test
    ```

## Outils utilisés

Voici la liste des outils utilisés.

| Outil                   | Utilisation                                                                                                                                                                                                                                                                  |
|------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| TypeScript             | Tous les dépôts sont écrits en TypeScript. Le JavaScript compilé et les définitions de types sont publiés sur npm.                                                                                                                                                     |
| TS Node                | Nous utilisons [ts-node](https://typestrong.org/ts-node/) pour exécuter les tests ou les scripts sans compiler TypeScript. L'objectif principal de ts-node est d'avoir un feedback plus rapide pendant le développement.                                                                                  |
| SWC                    | [SWC](https://swc.rs/) est un compilateur TypeScript basé sur Rust. TS Node est livré avec une prise en charge native pour utiliser SWC plutôt que le compilateur officiel TypeScript. La raison principale de l'utilisation de SWC est le gain de rapidité.                                                               |
| Release-It             | Nous utilisons [release-it](https://github.com/release-it/release-it) pour publier nos packages sur npm. Il gère tout le travail de création d'une version et la publie sur npm et Github. Sa configuration est définie dans le fichier `package.json`.                            |
| ESLint                 | ESLint nous aide à imposer un style de codage cohérent dans tous les dépôts avec plusieurs contributeurs. Toutes nos règles ESLint sont publiées dans le package [eslint-plugin-adonis](https://github.com/adonisjs-community/eslint-plugin-adonis).                            |
| Prettier               | Nous utilisons prettier pour formater le code source pour une présentation visuelle cohérente. Si vous vous demandez pourquoi nous utilisons à la fois ESLint et Prettier, lisez la documentation [Prettier vs. Linters](https://prettier.io/docs/en/comparison.html) sur le site web de Prettier.           |
| EditorConfig           | Le fichier `.editorconfig` à la racine de chaque projet configure votre éditeur de code pour utiliser un ensemble de règles pour l'indentation et la gestion des espaces. Encore une fois, Prettier est utilisé pour le formatage post-écriture de votre code, et Editorconfig est utilisé pour configurer l'éditeur à l'avance. |
| Conventional Changelog | Tous les commits dans tous les dépôts utilisent [commitlint](https://github.com/conventional-changelog/commitlint/#what-is-commitlint) pour imposer des messages de commit cohérents.                                                                                             |
| Husky                  | Nous utilisons [husky](https://typicode.github.io/husky/#/) pour appliquer les conventions de commit lors de l'envoi du code. Husky est un système de hooks git écrit en Node.                                                                                                                |

## Commandes

| Commande | Description |
|-------|--------|
| `npm run test` | Exécute les tests du projet avec `ts-node`. |
| `npm run compile` | Compile le projet TypeScript en JavaScript. Le résultat est écrit dans le répertoire `build`. |
| `npm run release` | Démarre le processus de publication en utilisant `np`. |
| `npm run lint` | Analyse le code source avec ESLint. |
| `npm run format` | Formate le code source avec Prettier. | 
| `npm run sync-labels` | Synchronise les labels définis dans le fichier `.github/labels.json` avec GitHub. (Commande réservée à l'administrateur du projet). |

## Style de codage

Tous nos projets sont écrits en TypeScript et évoluent vers le modèle ESM pur.

- Vous pouvez en apprendre davantage sur [mon style de codage ici](https://github.com/thetutlage/meta/discussions/3)
- Consultez la configuration que je suis pour [ESM et TypeScript ici](https://github.com/thetutlage/meta/discussions/2)

Assurez-vous également d'exécuter les commandes suivantes avant de pousser votre code :

```sh
# Formater avec Prettier
npm run format

# Analyser avec ESLint
npm run lint
```

## Être reconnu comme contributeur

Nous nous appuyons sur GitHub pour lister tous les contributeurs du dépôt dans le panneau latéral situé à droite. Voici un exemple.

De plus, nous utilisons la fonctionnalité de [génération automatique des notes de version](https://docs.github.com/en/repositories/releasing-projects-on-github/automatically-generated-release-notes#about-automatically-generated-release-notes) de Github, qui ajoute une référence au profil du contributeur dans les notes de version.
