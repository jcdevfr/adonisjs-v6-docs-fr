---
summary: Découvrez les préréglages de configuration des outils utilisés par AdonisJS pour TypeScript, Prettier et ESLint.
---

# Configuration des outils

AdonisJS s'appuie fortement sur TypeScript, Prettier et ESLint pour assurer la cohérence du code, détecter les erreurs lors de la compilation et, plus important encore, offrir une expérience de développement agréable.

Nous avons regroupé tous nos choix dans des préréglages prêts à l'emploi, utilisés par tous les packages officiels et les kits de démarrage officiels.

Continuez la lecture de ce guide si vous souhaitez utiliser les mêmes préréglages de configuration dans vos applications Node.js écrites en TypeScript.

## TSConfig

Le package [`@adonisjs/tsconfig`](https://github.com/adonisjs/tooling-config/tree/main/packages/typescript-config) contient la configuration de base pour les projets TypeScript. Nous configurons le système de modules TypeScript sur `NodeNext` et utilisons `TS Node + SWC` pour la compilation à la volée.

N'hésitez pas à explorer les options dans le [fichier de configuration de base](https://github.com/adonisjs/tooling-config/blob/main/packages/typescript-config/tsconfig.base.json), le [fichier de configuration d'application](https://github.com/adonisjs/tooling-config/blob/main/packages/typescript-config/tsconfig.app.json) et le [fichier de configuration de développement de package](https://github.com/adonisjs/tooling-config/blob/main/packages/typescript-config/tsconfig.package.json).

Vous pouvez installer le package et l'utiliser comme ceci :

```sh
npm i -D @adonisjs/tsconfig

# Assurez-vous également d'installer les packages suivants
npm i -D typescript ts-node @swc/core
```

Étendez le fichier `tsconfig.app.json` lors de la création d'une application AdonisJS (Pré-configuré avec les kits de démarrage).

```jsonc
{
  "extends": "@adonisjs/tsconfig/tsconfig.app.json",
  "compilerOptions": {
    "rootDir": "./",
    "outDir": "./build"
  }
}
```

Étendez le fichier `tsconfig.package.json` lors de la création d'un package pour l'écosystème AdonisJS.

```jsonc
{
  "extends": "@adonisjs/tsconfig/tsconfig.package.json",
  "compilerOptions": {
    "rootDir": "./",
    "outDir": "./build"
  }
}
```

## Configuration Prettier

Le package [`@adonisjs/prettier-config`](https://github.com/adonisjs/tooling-config/tree/main/packages/prettier-config) contient la configuration de base pour formater automatiquement le code source avec un style cohérent. N'hésitez pas à explorer les options de configuration dans le [fichier index.json](https://github.com/adonisjs/tooling-config/blob/main/packages/prettier-config/index.json).

Vous pouvez installer le package et l'utiliser comme ceci :

```sh
npm i -D @adonisjs/prettier-config

# Assurez-vous également d'installer prettier
npm i -D prettier
```

Définissez la propriété suivante dans le fichier `package.json`.

```jsonc
{
  "prettier": "@adonisjs/prettier-config"
}
```

Créez également un fichier `.prettierignore` pour ignorer certains fichiers et répertoires§.

```
// title: .prettierignore
build
node_modules
```

## Configuration ESLint

Le package [`@adonisjs/eslint-config`](https://github.com/adonisjs/tooling-config/tree/main/packages/eslint-config) contient la configuration de base pour appliquer les règles d'analyse. N'hésitez pas à explorer les options dans le [fichier de configuration de base](https://github.com/adonisjs/tooling-config/blob/main/packages/eslint-config/presets/ts_base.js), le [fichier de configuration d'application](https://github.com/adonisjs/tooling-config/blob/main/packages/eslint-config/presets/ts_app.js) et le [fichier de configuration de développement de package](https://github.com/adonisjs/tooling-config/blob/main/packages/eslint-config/presets/ts_package.js).

Vous pouvez installer le package et l'utiliser comme ceci. 

:::note
Notre préréglage utilise [eslint-plugin-prettier](https://github.com/prettier/eslint-plugin-prettier) pour s'assurer qu'ESLint et Prettier peuvent fonctionner ensemble sans se gêner mutuellement.
:::

```sh
npm i -D @adonisjs/eslint-config

# Assurez-vous également d'installer eslint
npm i -D eslint
```
Étendez le fichier `eslint-config/app` lors de la création d'une application AdonisJS (Pré-configuré avec les kits de démarrage).

```json
// title: package.json
{
  "eslintConfig": {
    "extends": "@adonisjs/eslint-config/app"
  }
}
```

Étendez le fichier `eslint-config/package` lors de la création d'un package pour l'écosystème AdonisJS.

```json
// title: package.json
{
  "eslintConfig": {
    "extends": "@adonisjs/eslint-config/package"
  }
}
```
