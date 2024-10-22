---
summary: Apprenez comment AdonisJS démarre le serveur HTTP, gère les requêtes entrantes et les modules disponibles au niveau de la couche HTTP.
---

# Vue d'ensemble HTTP

AdonisJS est principalement un framework web pour créer des applications qui répondent aux requêtes HTTP. Dans ce guide, nous allons apprendre comment AdonisJS démarre le serveur HTTP, gère les requêtes entrantes, et les modules disponibles au niveau de la couche HTTP.

## La couche HTTP

La couche HTTP dans une application AdonisJS se compose des modules suivants. Il est important de mentionner que la couche HTTP d'AdonisJS est construite à partir de zéro et n'utilise aucun microframework sous le capot.


<dl>

<dt>

[Routeur](../basics/routing.md)

</dt>

<dd>

Le [module router](https://github.com/adonisjs/http-server/blob/main/src/router/main.ts) est responsable de la définition des points d'entrée de votre application, connus sous le nom de routes. Une route doit définir un gestionnaire responsable du traitement de la requête. Le gestionnaire peut être une fonction anonyme ou une référence à un contrôleur.

</dd>

<dt>

[Contrôleurs](../basics/controllers.md)

</dt>

<dd>

Les contrôleurs sont des classes JavaScript que vous liez à une route pour gérer les requêtes HTTP. Les contrôleurs agissent comme une couche d'organisation et vous aident à diviser la logique métier de votre application dans différents fichiers/classes.

</dd>

<dt>

[Contexte HTTP](./http_context.md)

</dt>


<dd>

AdonisJS crée une instance de la classe [HttpContext](https://github.com/adonisjs/http-server/blob/main/src/http_context/main.ts) pour chaque requête HTTP entrante. Le contexte HTTP (aussi appelé `ctx`) contient des informations comme le corps de la requête, les en-têtes, l'utilisateur authentifié, etc., pour une requête donnée.

</dd>

<dt>

[Middlewares](../basics/middleware.md)

</dt>

<dd>

Le pipeline de middlewares dans AdonisJS est une implémentation du modèle de conception [Chain of Responsibility](https://refactoring.guru/design-patterns/chain-of-responsibility). Vous pouvez utiliser les middlewares pour intercepter les requêtes HTTP et y répondre avant qu'elles n'atteignent le gestionnaire de route.

</dd>

<dt>

[Gestionnaire d'exceptions global](../basics/exception_handling.md)

</dt>

<dd>

Le gestionnaire d'exceptions global gère les exceptions levées pendant une requête HTTP à un emplacement central. Vous pouvez utiliser le gestionnaire d'exceptions global pour convertir les exceptions en une réponse HTTP ou les signaler à un service de journalisation externe.

</dd>

<dt>

Serveur

</dt>

<dd>

Le [module server](https://github.com/adonisjs/http-server/blob/main/src/server/main.ts) relie le routeur, les middlewares, le gestionnaire d'exceptions global et exporte [une fonction `handle`](https://github.com/adonisjs/http-server/blob/main/src/server/main.ts#L330) que vous pouvez lier au serveur HTTP de Node.js pour gérer les requêtes.

</dd>

</dl>

## Comment AdonisJS démarre le serveur HTTP

Le serveur HTTP est démarré une fois que vous appelez [la méthode `boot`](https://github.com/adonisjs/http-server/blob/main/src/server/main.ts#L252) sur la classe Server. En interne, cette méthode effectue les actions suivantes :

- Crée le pipeline de middlewares.
- Compile les routes.
- Importe et instancie le gestionnaire d'exceptions global.

Dans une application AdonisJS typique, la méthode `boot` est appelée par le module [Ignitor](https://github.com/adonisjs/core/blob/main/src/ignitor/http.ts) dans le fichier `bin/server.ts`.

Il est également essentiel de définir les routes, les middlewares et le gestionnaire d'exceptions global avant que la méthode `boot` ne soit appelée, et AdonisJS y parvient en utilisant les [fichiers à précharger](./adonisrc_file.md#preloads) `start/routes.ts` et `start/kernel.ts`.

![](./server_boot_lifecycle.png)

## Cycle de vie d'une requête HTTP

Maintenant que nous avons un serveur HTTP à l'écoute des requêtes entrantes, voyons comment AdonisJS gère une requête HTTP donnée.

:::note
**Voir aussi :**\
[Flux d'exécution du middleware](../basics/middleware.md#middleware-execution-flow)\
[Middleware et gestion des exceptions](../basics/middleware.md#middleware-and-exception-handling)
:::


<dl>

<dt> Création du contexte HTTP </dt>


<dd>

Dans un premier temps, le module server crée une instance de la classe [HttpContext](./http_context.md) et la passe en référence aux middlewares, aux gestionnaires de route et au gestionnaire d'exceptions global.

Si vous avez activé l'[AsyncLocalStorage](./async_local_storage.md#usage), alors la même instance est partagée comme état de stockage local.

</dd>

<dt> Exécution de la pile de middlewares du serveur </dt>

<dd>

Ensuite, les middlewares de la [pile des middlewares du serveur](../basics/middleware.md#server-middleware-stack) sont exécutés. Ces middlewares peuvent intercepter et répondre à la requête avant qu'elle n'atteigne le gestionnaire de route.

De plus, chaque requête HTTP passe par la pile des middlewares du serveur, même si vous n'avez pas défini de route pour le point d'entrée donné. Cela permet aux middlewares du serveur d'ajouter des fonctionnalités à une application sans dépendre du système de routage.

</dd>

<dt> Recherche de la route correspondante </dt>

<dd>

Si un middleware du serveur ne termine pas la requête, nous cherchons une route correspondante pour la propriété `req.url`. La requête est interrompue avec une exception `404 - Not found` lorsqu'aucune route correspondante n'existe. Sinon, nous continuons avec la requête.

</dd>

<dt> Exécution des middlewares de route </dt>

<dd>

Une fois qu'il y a une route correspondante, nous exécutons les [middlewares globaux du routeur](../basics/middleware.md#router-middleware-stack) et la [pile de middlewares nommés](../basics/middleware.md#named-middleware-collection). Encore une fois, les middlewares peuvent intercepter la requête avant qu'elle n'atteigne le gestionnaire de route.

</dd>

<dt> Exécution du gestionnaire de route </dt>

<dd>

Comme étape finale, la requête atteint le gestionnaire de route et retourne au client avec une réponse.

Si une exception est levée pendant n'importe quelle étape du processus, la requête sera transmise au gestionnaire d'exceptions global, qui est responsable de la conversion de l'exception en une réponse.

</dd>

<dt> Sérialisation de la réponse </dt>

<dd>

Une fois que vous définissez le corps de la réponse en utilisant la méthode `response.send` ou en retournant une valeur depuis le gestionnaire de route, nous commençons le processus de sérialisation de la réponse et définissons les en-têtes appropriés.

En savoir plus sur la [sérialisation du corps de la réponse](../basics/response.md#response-body-serialization)

</dd>

</dl>
