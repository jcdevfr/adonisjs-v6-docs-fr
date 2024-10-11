---
summary: Apprenez à utiliser les variables d'environnement dans une application AdonisJS.
---

# Variables d'environnement

Les variables d'environnement servent à stocker des données sensibles comme le mot de passe de la base de données, la clé secrète de l'application, ou une clé API en dehors du code source de votre application.

De plus, les variables d'environnement peuvent être utilisées pour avoir différentes configurations pour différents environnements. Par exemple, vous pouvez utiliser un gestionnaire de mail en mémoire pendant les tests, un gestionnaire SMTP pendant le développement, et un service tiers en production.

Comme les variables d'environnement sont prises en charge par tous les systèmes d'exploitation, les plateformes de déploiement et les pipelines CI/CD, elles sont devenues un standard de facto pour stocker les données sensibles et la configuration spécifique à un environnement.

Dans ce guide, nous allons apprendre comment exploiter les variables d'environnement dans une application AdonisJS.

## Lecture des variables d'environnement

Node.js expose nativement toutes les variables d'environnement sous forme d'objet via la [propriété globale `process.env`](https://nodejs.org/dist/latest-v8.x/docs/api/process.html#process_process_env), et vous pouvez y accéder de cette façon :

```dotenv
process.env.NODE_ENV
process.env.HOST
process.env.PORT
```

## Utilisation du module env d'AdonisJS

La lecture des variables d'environnement via l'objet `process.env` ne nécessite aucune configuration du côté d'AdonisJS, car l'environnement d'exécution Node.js la prend en charge. Cependant, dans le reste de ce guide, nous utiliserons le module env d'AdonisJS pour les raisons suivantes :

- Capacité à stocker et analyser les variables d'environnement à partir de plusieurs fichiers `.env`.
- Valider les variables d'environnement dès le démarrage de l'application.
- Avoir une sécurité de typage statique pour les variables d'environnement validées.

Le module env est instancié dans le fichier`start/env.ts`, et vous pouvez y accéder ailleurs dans votre application de la manière suivante :

```ts
import env from '#start/env'

env.get('NODE_ENV')
env.get('HOST')
env.get('PORT')

// Renvoie 3333 quand PORT n'est pas défini
env.get('PORT', 3333)
```

### Partage du module env avec les templates Edge

Si vous souhaitez accéder aux variables d'environnement dans les templates Edge, vous devez partager le module `env` comme variable globale avec les templates Edge.

Vous pouvez [créer `view.ts` comme fichier de préchargement](../concepts/adonisrc_file.md#preloads) dans le répertoire `start` et y écrire les lignes de code suivantes :

```ts
// title: start/view.ts
import env from '#start/env'
import edge from 'edge.js'

edge.global('env', env)
```

## Validation des variables d'environnement

Les règles de validation pour les variables d'environnement sont définies dans le fichier `start/env.ts` à l'aide de la méthode `Env.create`. 

La validation est effectuée automatiquement lors de la première importation de ce fichier. Généralement, le fichier `start/env.ts` est importé par l'un des fichiers de configuration de votre projet. Sinon, AdonisJS importera ce fichier implicitement [avant de démarrer l'application](https://github.com/adonisjs/slim-starter-kit/blob/main/bin/server.ts#L34-L36).

La méthode `Env.create` accepte le schéma de validation sous forme de paires clé-valeur.

- La clé est le nom de la variable d'environnement.
- La valeur est la fonction qui effectue la validation. Il peut s'agir d'une fonction personnalisée inline ou d'une référence aux méthodes de schéma prédéfinies comme `schema.string` ou `schema.number`.

```ts
import Env from '@adonisjs/core/env'

/**
 * La racine de l'application est utilisée pour localiser 
 * les fichiers .env à la racine du projet.
 */
const APP_ROOT = new URL('../', import.meta.url)

export default await Env.create(APP_ROOT, {
  HOST: Env.schema.string({ format: 'host' }),
  PORT: Env.schema.number(),
  APP_KEY: Env.schema.string(),
  APP_NAME: Env.schema.string(),
  CACHE_VIEWS: Env.schema.boolean(),
  SESSION_DRIVER: Env.schema.string(),
  NODE_ENV: Env.schema.enum([
    'development',
    'production',
    'test'
  ] as const),
})
```

### Informations de typage statique

Les mêmes règles de validation sont utilisées pour déduire les informations de typage statique. Ces informations de type sont disponibles lors de l'utilisation du module env.

![](./env_intellisense.jpeg)

## API du schéma de validation

### schema.string

La méthode `schema.string` s'assure que la valeur est une chaîne de caractères valide. Les chaînes vides échouent à la validation, et vous devez utiliser la variante optionnelle pour autoriser les chaînes vides.

```ts
{
  APP_KEY: Env.schema.string()
}

// Marquer APP_KEY comme optionnel
{
  APP_KEY: Env.schema.string.optional()
}
```

La valeur de la chaîne peut être validée pour son format. Voici la liste des formats disponibles.

#### host

Valide que la valeur est une URL valide ou une adresse IP.

```ts
{
  HOST: Env.schema.string({ format: 'host' })
}
```

#### url

Valide que la valeur est une URL valide. Optionnellement, vous pouvez rendre la validation moins stricte en permettant aux URLs de ne pas avoir de `protocol` ou de `tld`.

```ts
{
  S3_ENDPOINT: Env.schema.string({ format: 'url' })

  // Autoriser les URLs sans protocole
  S3_ENDPOINT: Env.schema.string({ format: 'url', protocol: false })

  // Autoriser les URLs sans tld
  S3_ENDPOINT: Env.schema.string({ format: 'url', tld: false })
}
```
  
#### email

Valide que la valeur est une adresse email valide.

```ts
{
  SENDER_EMAIL: Env.schema.string({ format: 'email' })
}
```

### schema.boolean

La méthode `schema.boolean` s'assure que la valeur est un booléen valide. Les valeurs vides échouent à la validation, et vous devez utiliser la variante optionnelle pour autoriser les valeurs vides.

Les représentations sous forme de chaîne `'true'`, `'1'`, `'false'` et `'0'` sont converties en type de données booléen.

```ts
{
  CACHE_VIEWS: Env.schema.boolean()
}

// Le marquer comme optionnel
{
  CACHE_VIEWS: Env.schema.boolean.optional()
}
```

### schema.number

La méthode `schema.number` s'assure que la valeur est un nombre valide. La représentation sous forme de chaîne d'une valeur numérique est convertie en type de données nombre.

```ts
{
  PORT: Env.schema.number()
}

// Le marquer comme optionnel
{
  PORT: Env.schema.number.optional()
}
```

### schema.enum

La méthode `schema.enum` valide la variable d'environnement par rapport à l'une des valeurs prédéfinies. Les options d'énumération peuvent être spécifiées comme un tableau de valeurs ou un type d'énumération natif TypeScript.

```ts
{
  NODE_ENV: Env
    .schema
    .enum(['development', 'production'] as const)
}

// Le marquer comme optionnel
{
  NODE_ENV: Env
    .schema
    .enum
    .optional(['development', 'production'] as const)
}

// Utilisation des énumérations natives
enum NODE_ENV {
  development = 'development',
  production = 'production'
}

{
  NODE_ENV: Env.schema.enum(NODE_ENV)
}
```

### Fonctions personnalisées

Les fonctions personnalisées peuvent effectuer des validations non couvertes par l'API du schéma.

La fonction reçoit le nom de la variable d'environnement comme premier argument et la valeur comme second argument. Elle doit renvoyer la valeur finale après validation.

```ts
{
  PORT: (name, value) => {
    if (!value) {
      throw new Error('Value for PORT is required')
    }
    
    if (isNaN(Number(value))) {
      throw new Error('Value for PORT must be a valid number')
    }

    return Number(value)
  }
}
```

## Définition des variables d'environnement

### En développement

Les variables d'environnement sont définies dans le fichier `.env` pendant le développement. Le module env recherche ce fichier à la racine du projet et l'analyse automatiquement (s'il existe).

```dotenv
// title: .env
PORT=3333
HOST=0.0.0.0
NODE_ENV=development
APP_KEY=sH2k88gojcp3PdAJiGDxof54kjtTXa3g
SESSION_DRIVER=cookie
CACHE_VIEWS=false
```

### En production

Il est recommandé d'utiliser votre plateforme de déploiement pour définir les variables d'environnement en production. La plupart des plateformes de déploiement modernes offrent un support natif pour définir les variables d'environnement depuis leur interface web.

Si votre plateforme de déploiement ne permet pas de définir des variables d'environnement, vous pouvez créer un fichier `.env` à la racine du projet ou à un autre emplacement sur votre serveur de production.

AdonisJS lira automatiquement le fichier `.env` à la racine du projet. Cependant, vous devez définir la variable `ENV_PATH` lorsque le fichier `.env` est stocké à un autre emplacement.

```sh
# Tente de lire le fichier .env à la racine du projet
node server.js

# Lit le fichier .env depuis le répertoire "/etc/secrets"
ENV_PATH=/etc/secrets node server.js
```

### Pendant les tests

Les variables d'environnement spécifiques à l'environnement de test doivent être définies dans le fichier `.env.test`. Les valeurs de ce fichier remplacent les valeurs du fichier `.env`.

```dotenv
// title: .env
NODE_ENV=development
SESSION_DRIVER=cookie
ASSETS_DRIVER=vite
```

```dotenv
// title: .env.test
NODE_ENV=test
SESSION_DRIVER=memory
ASSETS_DRIVER=fake
```

```ts
// Pendant les tests
import env from '#start/env'

env.get('SESSION_DRIVER') // memory
```

## Tous les autres fichiers dot-env

En plus du fichier `.env`, AdonisJS traite les variables d'environnement des fichiers dot-env suivants. Vous pouvez donc créer ces fichiers de manière optionnelle (si nécessaire).

Le fichier avec le rang le plus élevé remplace les valeurs des fichiers de rang inférieur.

<table>
    <thead>
        <tr>
            <th width="40px">Rang</th>
            <th width="220px">Nom du fichier</th>
            <th>Notes</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>1er</td>
            <td><code>.env.[NODE_ENV].local</code></td>
            <td>
            Chargé pour le <code>NODE_ENV</code> actuel. Par exemple, si le <code>NODE_ENV</code> est défini sur <code>development</code>, alors le fichier <code>.env.development.local</code> sera chargé.
            </td>
        </tr>
        <tr>
            <td>2ème</td>
            <td><code>.env.local</code></td>
            <td>Chargé dans tous les environnements sauf les environnements <code>test</code> et <code>testing</code>.</td>
        </tr>
        <tr>
            <td>3ème</td>
            <td><code>.env.[NODE_ENV]</code></td>
            <td>
            Chargé pour le <code>NODE_ENV</code> actuel. Par exemple, si le <code>NODE_ENV</code> est défini sur <code>development</code>, alors le fichier <code>.env.development</code> sera chargé.
            </td>
        </tr>
        <tr>
            <td>4ème</td>
            <td><code>.env</code></td>
            <td>Chargé dans tous les environnements. Vous devez ajouter ce fichier à <code>.gitignore</code> lorsque vous y stockez des données sensibles.</td>
        </tr>
    </tbody>
</table>

## Utilisation des variables dans les fichiers dot-env

Dans les fichiers dot-env, vous pouvez faire référence à d'autres variables d'environnement en utilisant la syntaxe de substitution de variable.

Dans l'exemple suivant, nous calculons `APP_URL` à partir des propriétés `HOST` et `PORT`.

```dotenv
HOST=localhost
PORT=3333
// highlight-start
URL=$HOST:$PORT
// highlight-end
```

Tous les **lettres**, **chiffres** et le **souligné (_)** après le signe `$` sont utilisés pour former un nom de variable. Vous devez entourer le nom de la variable par des accolades `{}` si le nom contient des caractères spéciaux autres qu'un souligné.

```dotenv
REDIS-USER=admin
REDIS-URL=localhost@${REDIS-USER}
```

### Échapper le signe `$`

Pour utiliser le signe `$` comme valeur, vous devez l'échapper pour empêcher la substitution de variable.

```dotenv
PASSWORD=pa\$\$word
```
