# REST API Design — Reference Documentation

> Sources compilées et synthétisées depuis :
> - [Microsoft Azure — Web API Design Best Practices](https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design)
> - [Zalando RESTful API Guidelines](https://opensource.zalando.com/restful-api-guidelines/)
> - [Postman Blog](https://blog.postman.com)
> - [Nordic APIs Blog](https://nordicapis.com/blog/)
> - [API Evangelist](https://apievangelist.com)

---

## Table of Contents

1. [Principes fondamentaux REST](#1-principes-fondamentaux-rest)
2. [Conception des URIs](#2-conception-des-uris)
3. [Méthodes HTTP](#3-méthodes-http)
4. [Codes de statut HTTP](#4-codes-de-statut-http)
5. [Payload et formats de données](#5-payload-et-formats-de-données)
6. [Headers HTTP](#6-headers-http)
7. [Versionnage](#7-versionnage)
8. [Pagination](#8-pagination)
9. [Gestion des erreurs](#9-gestion-des-erreurs)
10. [Sécurité](#10-sécurité)
11. [Performance et cache](#11-performance-et-cache)
12. [HATEOAS](#12-hateoas)
13. [OpenAPI / Documentation](#13-openapi--documentation)
14. [Conventions de nommage (synthèse)](#14-conventions-de-nommage-synthèse)

---

## 1. Principes fondamentaux REST

**Source: Microsoft Azure + Zalando**

- **Indépendance de la plateforme** : les clients appellent l'API sans connaître l'implémentation interne. Utiliser HTTP standard, JSON/XML, documentation claire.
- **Couplage faible** : client et service évoluent indépendamment.
- **Modèle sans état (Stateless)** : chaque requête HTTP est indépendante. Pas d'état transient entre requêtes. L'état est dans les ressources elles-mêmes.
- **Interface uniforme** : verbes HTTP standard (GET, POST, PUT, PATCH, DELETE).
- **API First** *(Zalando — MUST)* : concevoir l'API avant l'implémentation. L'API est un contrat.
- **API as a Product** *(Zalando)* : l'API est un produit livré à des clients (internes ou externes). Penser expérience développeur.

---

## 2. Conception des URIs

**Source: Microsoft Azure + Zalando**

### Règles fondamentales

| Règle | Exemple correct | À éviter |
|-------|-----------------|----------|
| Noms (nouns), pas de verbes | `/orders` | `/create-order` |
| Pluriel pour les collections | `/customers` | `/customer` |
| kebab-case pour les segments *(Zalando — MUST)* | `/order-items` | `/orderItems`, `/order_items` |
| snake_case pour les query params *(Zalando — MUST)* | `?page_size=10` | `?pageSize=10` |
| Pas de `/api` comme base path *(Zalando — SHOULD)* | `/v1/orders` | `/api/orders` |
| Pas de slash final | `/orders/1` | `/orders/1/` |
| URIs en minuscules | `/orders` | `/Orders` |
| Identifiants URL-friendly *(Zalando — MUST)* | `/orders/abc-123` | `/orders/abc 123` |

### Hiérarchie recommandée

```
/resources                        → collection
/resources/{id}                   → item
/resources/{id}/sub-resources     → sous-collection
/resources/{id}/sub-resources/{id} → sous-item
```

> ⚠️ **Éviter** plus de 2 niveaux de sous-ressources *(Zalando — SHOULD)*.

### Cas particuliers

- **Opérations non-ressources** : utiliser des pseudo-ressources avec query params sparingly.  
  Exemple : `GET /calculations/sum?a=99&b=1`
- **Clés composées** : peuvent être exposées comme identifiant *(Zalando — MAY)*.
- **Relations** : préférer les liens HATEOAS plutôt que des URIs profondément imbriquées.

---

## 3. Méthodes HTTP

**Source: Microsoft Azure + Zalando**

### Tableau de correspondance standard

| Méthode | Collection `/resources` | Item `/resources/{id}` |
|---------|------------------------|------------------------|
| **GET** | Liste tous les items | Retourne l'item |
| **POST** | Crée un nouvel item | ❌ (éviter) |
| **PUT** | Remplace toute la collection (rare) | Remplace l'item entier |
| **PATCH** | ❌ (rare) | Mise à jour partielle |
| **DELETE** | Supprime toute la collection (rare) | Supprime l'item |

### Propriétés des méthodes *(Zalando — MUST)*

| Méthode | Safe | Idempotent | Cacheable |
|---------|------|------------|-----------|
| GET | ✅ | ✅ | ✅ |
| HEAD | ✅ | ✅ | ✅ |
| OPTIONS | ✅ | ✅ | ❌ |
| POST | ❌ | ❌ | ❌ |
| PUT | ❌ | ✅ | ❌ |
| PATCH | ❌ | ❌ | ❌ |
| DELETE | ❌ | ✅ | ❌ |

### Bonnes pratiques

- **POST idempotent** *(Zalando — SHOULD)* : utiliser une clé secondaire (ex: `Idempotency-Key` header) pour rendre POST idempotent.
- **PUT** : remplace la ressource entière. Si l'objet n'existe pas, peut le créer (upsert).
- **PATCH** : format recommandé → JSON Merge Patch (RFC 7396) ou JSON Patch (RFC 6902).
- **Traitement asynchrone** *(Zalando — MAY)* : retourner `202 Accepted` + `Location` header pour les opérations longues.

---

## 4. Codes de statut HTTP

**Source: Microsoft Azure + Zalando**

### Codes à utiliser *(Zalando — SHOULD use most common)*

#### 2xx — Succès
| Code | Signification | Usage |
|------|--------------|-------|
| 200 OK | Succès standard | GET, PUT, PATCH, DELETE avec body |
| 201 Created | Ressource créée | POST avec `Location` header |
| 202 Accepted | Traitement asynchrone accepté | Long processing |
| 204 No Content | Succès sans body | DELETE, PUT/PATCH sans retour |
| 207 Multi-Status | Batch/bulk *(Zalando — MUST)* | Résultats mixtes |

#### 3xx — Redirection
| Code | Signification | Usage |
|------|--------------|-------|
| 301 Moved Permanently | Dépréciation d'URI | Migration |
| 304 Not Modified | Cache valide | ETag/If-None-Match |

> *(Zalando — SHOULD NOT)* : ne pas abuser des codes de redirection.

#### 4xx — Erreur client
| Code | Signification | Usage |
|------|--------------|-------|
| 400 Bad Request | Requête malformée | Validation échouée |
| 401 Unauthorized | Non authentifié | Token manquant/invalide |
| 403 Forbidden | Non autorisé | Droits insuffisants |
| 404 Not Found | Ressource introuvable | ID inexistant |
| 405 Method Not Allowed | Méthode non supportée | — |
| 406 Not Acceptable | Format non supporté | Content-Type |
| 409 Conflict | Conflit de ressource | Doublon, état incompatible |
| 410 Gone | Ressource supprimée définitivement | Dépréciation |
| 412 Precondition Failed | ETag mismatch | Optimistic locking |
| 415 Unsupported Media Type | Content-Type non supporté | — |
| 422 Unprocessable Entity | Sémantiquement invalide | Business rules |
| 429 Too Many Requests *(Zalando — MUST)* | Rate limit | Avec `Retry-After` header |

#### 5xx — Erreur serveur
| Code | Signification | Usage |
|------|--------------|-------|
| 500 Internal Server Error | Erreur générique | Non-prévu |
| 503 Service Unavailable | Service indisponible | Maintenance, overload |

---

## 5. Payload et formats de données

**Source: Zalando + Microsoft Azure**

### Format général *(Zalando — MUST)*

- Utiliser **JSON** comme format principal d'échange.
- Property names en **snake_case** (`order_id`, pas `orderId`).
- Enum values en **UPPER_SNAKE_CASE** (`ORDER_PLACED`, `PAYMENT_FAILED`).
- Arrays : noms au **pluriel** (`items`, pas `item`).

### Types de données standards *(Zalando — MUST)*

| Type | Format recommandé |
|------|-------------------|
| Date | `"2024-01-15"` (ISO 8601, date only) |
| DateTime | `"2024-01-15T10:30:00Z"` (ISO 8601 UTC) |
| Duration | `"PT1H30M"` (ISO 8601 duration) |
| Money | `{"amount": 9.99, "currency": "EUR"}` |
| Country | ISO 3166-1 alpha-2 (`"FR"`, `"DE"`) |
| Language | BCP 47 (`"fr"`, `"en-US"`) |
| UUID | RFC 4122 (`"550e8400-e29b-41d4-a716-446655440000"`) |
| Integer | Préciser le format (`int32`, `int64`) |
| Decimal | `number` avec précision documentée |

### Règles sur `null` *(Zalando — MUST/SHOULD)*

- `null` et propriété absente ont la **même sémantique** (MUST).
- Ne **pas** utiliser `null` pour les booléens — utiliser `true`/`false`.
- Ne **pas** utiliser `null` pour les arrays vides — utiliser `[]`.

### Champs communs *(Zalando — MUST)*

```json
{
  "id": "string (UUID)",
  "created": "2024-01-15T10:30:00Z",
  "modified": "2024-01-15T11:00:00Z",
  "type": "ENUM_VALUE"
}
```

---

## 6. Headers HTTP

**Source: Zalando + Microsoft Azure**

### Headers standards courants

| Header | Usage |
|--------|-------|
| `Content-Type` | Type du body de la requête (`application/json`) |
| `Accept` | Format attendu en réponse |
| `Authorization` | Token d'authentification (`Bearer ...`) |
| `Location` | URI de la ressource créée (201) ou asynchrone (202) |
| `ETag` | Version/fingerprint de la ressource |
| `If-Match` | Optimistic locking (PUT/PATCH conditionnels) |
| `If-None-Match` | Cache conditionnel (GET) |
| `Cache-Control` | Directives de cache |
| `Retry-After` | Délai avant retry (429, 503) |
| `X-Request-ID` / `X-Flow-ID` | Tracing distribué |

### Conventions *(Zalando)*

- Headers propriétaires : **kebab-case avec mots en majuscule** (`X-Flow-Id`, `X-Request-Id`).
- `Location` plutôt que `Content-Location` pour les redirections *(Zalando — SHOULD)*.
- Propagation obligatoire des headers de tracing *(Zalando — MUST)*.

---

## 7. Versionnage

**Source: Microsoft Azure + Zalando + Nordic APIs**

### Stratégies de versionnage

| Stratégie | Exemple | Avantages | Inconvénients |
|-----------|---------|-----------|---------------|
| **URI path** *(recommandé)* | `/v1/orders` | Simple, visible, cacheable | "pollue" l'URI |
| **Query parameter** | `/orders?version=1` | Non intrusif | Moins conventionnel |
| **Header** | `API-Version: 1` | URI propre | Moins visible |
| **Content negotiation** | `Accept: application/vnd.api+json;version=1` | Standard HTTP | Complexe |

### Règles *(Zalando — MUST)*

- Utiliser le **versionnage sémantique** (MAJOR.MINOR.PATCH).
- Incrémenter MAJOR pour les **breaking changes**.
- Les changements **backward-compatible** n'incrémentent pas MAJOR.
- Inclure la version dans les métadonnées de l'API (`info.version` OpenAPI).

### Breaking changes à éviter

- Supprimer ou renommer des endpoints
- Changer le type d'un champ existant
- Rendre obligatoire un champ précédemment optionnel
- Changer le comportement d'un endpoint existant

---

## 8. Pagination

**Source: Zalando + Microsoft Azure**

### Obligation *(Zalando — MUST)*

Toute collection doit supporter la pagination.

### Stratégies

#### Offset-based (simple mais limité)
```
GET /orders?offset=20&limit=10
```
```json
{
  "items": [...],
  "total": 150,
  "offset": 20,
  "limit": 10
}
```

#### Cursor-based *(Zalando — SHOULD prefer)*
```
GET /orders?cursor=eyJpZCI6MTB9&limit=10
```
```json
{
  "items": [...],
  "next": "/orders?cursor=eyJpZCI6MjB9&limit=10",
  "prev": "/orders?cursor=eyJpZCI6MX0=&limit=10"
}
```

> Cursor-based est préféré pour les grandes collections (stable, performant).

### Response page object *(Zalando — SHOULD)*

```json
{
  "items": [...],
  "self":  "/orders?cursor=current",
  "first": "/orders?cursor=first",
  "next":  "/orders?cursor=next",
  "prev":  "/orders?cursor=prev"
}
```

### Query params de pagination *(Zalando — MUST use conventional)*

| Param | Usage |
|-------|-------|
| `limit` | Nombre d'items par page |
| `offset` | Position de départ (offset-based) |
| `cursor` | Token opaque (cursor-based) |
| `sort` | Champ de tri |
| `q` | Recherche textuelle |

---

## 9. Gestion des erreurs

**Source: Zalando + Microsoft Azure + Postman Blog**

### Problem JSON *(Zalando — MUST support)*

RFC 7807 — Format standard de réponse d'erreur :

```json
{
  "type": "https://api.example.com/errors/insufficient-funds",
  "title": "Insufficient Funds",
  "status": 422,
  "detail": "Your account balance is too low to complete this transaction.",
  "instance": "/accounts/abc123/transactions/xyz789"
}
```

| Champ | Obligatoire | Description |
|-------|-------------|-------------|
| `type` | SHOULD | URI identifiant le type d'erreur |
| `title` | SHOULD | Description courte lisible |
| `status` | MUST | Code HTTP |
| `detail` | SHOULD | Explication spécifique |
| `instance` | MAY | URI de l'occurrence |

### Règles importantes *(Zalando — MUST)*

- **Ne jamais exposer les stack traces** en production.
- Spécifier les réponses success ET error dans la documentation.
- Utiliser `207 Multi-Status` pour les opérations batch avec résultats mixtes.

### Erreurs de validation (extension suggérée)

```json
{
  "type": "https://api.example.com/errors/validation",
  "title": "Validation Error",
  "status": 400,
  "detail": "Request validation failed",
  "errors": [
    { "field": "email", "message": "Invalid email format" },
    { "field": "age", "message": "Must be >= 18" }
  ]
}
```

---

## 10. Sécurité

**Source: Zalando + Microsoft Azure + Nordic APIs**

### Authentification & Autorisation *(Zalando — MUST)*

- Tous les endpoints doivent être **sécurisés** (pas d'endpoint public non intentionnel).
- Utiliser **OAuth 2.0** avec scopes pour l'autorisation.
- Documenter les **permissions (scopes)** pour chaque endpoint.
- Convention de nommage des scopes *(Zalando)* : `uid.resource.action` (ex: `orders.read`, `orders.write`).

### Bonnes pratiques

- Utiliser **HTTPS uniquement** (TLS 1.2+).
- Implémenter le **rate limiting** (429 + `Retry-After`).
- Ne jamais exposer des données sensibles dans les URIs (tokens, passwords).
- Valider et **sanitiser** toutes les entrées côté serveur.
- Implémenter la **pagination** pour éviter les dumps massifs de données.
- Penser au **principe du moindre privilège** pour les scopes OAuth.
- Gérer les erreurs sans **information disclosure** (stack traces, détails internes).

---

## 11. Performance et cache

**Source: Zalando + Microsoft Azure**

### Réduction de la bande passante *(Zalando — SHOULD)*

- Supporter le **filtrage partiel** : `GET /orders?fields=id,status,total`
- Supporter l'**embedding optionnel** de sous-ressources : `GET /orders?embed=customer`
- Activer la **compression gzip** pour les réponses JSON.
- Documenter quels endpoints GET/HEAD/POST sont **cacheables** *(Zalando — MUST)*.

### Cache HTTP

```http
Cache-Control: max-age=3600, must-revalidate
ETag: "v1-abc123"
Last-Modified: Mon, 15 Jan 2024 10:30:00 GMT
```

**Requête conditionnelle :**
```http
GET /orders/1
If-None-Match: "v1-abc123"
→ 304 Not Modified (si inchangé)
```

### Optimistic Locking (concurrence)

```http
GET /orders/1
← ETag: "v1-abc123"

PUT /orders/1
If-Match: "v1-abc123"
→ 200 OK (si version correcte)
→ 412 Precondition Failed (si conflict)
```

### Éviter les anti-patterns *(Microsoft Azure)*

- **Chatty APIs** : éviter trop de petites ressources nécessitant de multiples appels.
- **Extraneous Fetching** : ne pas retourner plus de données que nécessaire.
- **Mirroring DB structure** : l'API doit abstraire le modèle de données interne.

---

## 12. HATEOAS

**Source: Microsoft Azure + Zalando**

HATEOAS (Hypermedia as the Engine of Application State) = niveau de maturité REST 3.

*(Zalando)* : Niveau 2 est **MUST**, Niveau 3 (HATEOAS) est **MAY**.

### Exemple

```json
{
  "order_id": "123",
  "status": "PENDING",
  "total": 99.90,
  "_links": {
    "self":    { "href": "/orders/123" },
    "cancel":  { "href": "/orders/123/cancel", "method": "POST" },
    "payment": { "href": "/orders/123/payment" },
    "customer":{ "href": "/customers/456" }
  }
}
```

### Contrôles hypertext communs *(Zalando — MUST)*

| Relation | Description |
|----------|-------------|
| `self` | URI canonique de la ressource |
| `first` / `last` | Navigation pagination |
| `prev` / `next` | Pages adjacentes |
| `related` | Ressource liée |

> *(Zalando — MUST)* : utiliser des URIs absolus pour identifier les ressources.

---

## 13. OpenAPI / Documentation

**Source: Zalando + Postman Blog**

### Obligations *(Zalando — MUST)*

- Fournir une **spécification OpenAPI** (v3.x recommandé).
- Inclure les métadonnées : `title`, `description`, `version`, `contact`, `license`.
- Documenter **tous** les codes de réponse success ET error.
- Utiliser **l'anglais américain** pour toutes les descriptions.

### Structure OpenAPI minimale

```yaml
openapi: 3.1.0
info:
  title: Orders API
  version: 1.2.0
  description: Manage customer orders
  contact:
    name: API Support
    email: api-support@example.com

servers:
  - url: https://api.example.com/v1

paths:
  /orders:
    get:
      summary: List orders
      parameters:
        - name: limit
          in: query
          schema:
            type: integer
            default: 20
      responses:
        '200':
          description: List of orders
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/OrderList'
        '400':
          $ref: '#/components/responses/BadRequest'
        '401':
          $ref: '#/components/responses/Unauthorized'
```

---

## 14. Conventions de nommage (synthèse)

**Source: Zalando**

| Élément | Convention | Exemple |
|---------|-----------|---------|
| Path segments | kebab-case | `/order-items` |
| Query parameters | snake_case | `?page_size=10` |
| JSON property names | snake_case | `"order_id"` |
| Enum values | UPPER_SNAKE_CASE | `"ORDER_PLACED"` |
| HTTP Headers | Kebab-Case-Capitalized | `X-Flow-Id` |
| Hostnames | kebab-case | `order-service.api.example.com` |
| API version | Semantic versioning | `v1`, `v1.2.0` |
| Scopes OAuth | dot.notation | `orders.read` |

---

*Dernière mise à jour : référence compilée depuis les sources officielles. Consulter les liens originaux pour les versions les plus récentes.*
