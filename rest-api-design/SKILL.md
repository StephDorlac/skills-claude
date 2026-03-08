---
name: rest-api-design
description: >
  Expert REST API design and implementation skill. Use this skill WHENEVER the user wants to:
  create, design, scaffold, refactor, review, or improve a REST API — regardless of the programming
  language or framework (C# .NET, Java Spring, Node.js, Python FastAPI, Go, PHP, etc.).
  Also trigger when the user asks about: API versioning, endpoint naming, HTTP status codes,
  pagination design, error handling in APIs, OpenAPI/Swagger spec, API security, rate limiting,
  or any question about REST best practices. This skill MUST be consulted before writing any
  API-related code or giving architectural advice on REST endpoints.
---

# REST API Design Skill

## Rôle de ce skill

Ce skill guide la conception, l'implémentation et la révision d'APIs REST en s'appuyant sur les meilleures pratiques issues de sources de référence reconnues dans l'industrie.

## ⚠️ Information obligatoire à communiquer à l'utilisateur

**À chaque activation de ce skill, informer l'utilisateur** :

> 🔍 *Pour cette tâche, je m'appuie sur les références REST suivantes :*
> - *[Microsoft Azure — Web API Design Best Practices](https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design)*
> - *[Zalando RESTful API Guidelines](https://opensource.zalando.com/restful-api-guidelines/)*
> - *[Postman Blog](https://blog.postman.com)*
> - *[Nordic APIs Blog](https://nordicapis.com/blog/)*
> - *[API Evangelist](https://apievangelist.com)*

---

## Workflow d'activation

### Étape 1 — Charger la référence

**Toujours** lire `references/reference.md` avant de répondre.  
Ce fichier contient la documentation consolidée de toutes les sources (URIs, méthodes HTTP, codes de statut, versionnage, pagination, erreurs, sécurité, OpenAPI, conventions de nommage).

### Étape 2 — Analyser la demande

Identifier ce que l'utilisateur souhaite faire :

| Cas | Action principale |
|-----|-------------------|
| Créer une nouvelle API | Proposer architecture + URIs + OpenAPI |
| Ajouter un endpoint | Valider nommage, méthode, codes retour |
| Refactorer une API existante | Identifier les violations, proposer corrections |
| Revue de code API | Checklist des bonnes pratiques |
| Question générale REST | Répondre depuis `reference.md` |
| Générer une spec OpenAPI | Produire le YAML complet |

### Étape 3 — Adapter au langage/framework

Générer le code dans le langage demandé. Voir la section [Adaptations par langage](#adaptations-par-langage) ci-dessous.

---

## Checklist de conception (à appliquer systématiquement)

### URIs
- [ ] Noms (nouns), pas verbes dans les URIs
- [ ] Collections au pluriel (`/orders`, `/customers`)
- [ ] kebab-case pour les segments de path (`/order-items`)
- [ ] snake_case pour les query params (`?page_size=10`)
- [ ] Pas de `/api` en base path (sauf contrainte technique)
- [ ] Maximum 2 niveaux de sous-ressources
- [ ] Identifiants URL-friendly

### Méthodes HTTP
- [ ] GET = lecture (safe + idempotent)
- [ ] POST = création (ni safe, ni idempotent)
- [ ] PUT = remplacement complet (idempotent)
- [ ] PATCH = mise à jour partielle
- [ ] DELETE = suppression (idempotent)

### Codes de statut
- [ ] 201 + Location header pour les créations POST
- [ ] 204 pour DELETE / PUT sans body retour
- [ ] 400 pour erreurs de validation
- [ ] 401 vs 403 correctement distingués
- [ ] 404 pour ressource inexistante
- [ ] 409 pour conflits
- [ ] 422 pour erreurs de règles métier
- [ ] 429 avec Retry-After pour rate limiting

### Payload
- [ ] JSON par défaut
- [ ] snake_case pour les noms de propriétés
- [ ] Dates en ISO 8601 UTC
- [ ] Problem JSON (RFC 7807) pour les erreurs
- [ ] Pas de stack traces exposées

### Pagination
- [ ] Toute collection paginée
- [ ] Cursor-based préféré pour les grandes collections
- [ ] Offset-based acceptable pour les petites collections

### Sécurité
- [ ] HTTPS obligatoire
- [ ] Authentification sur tous les endpoints
- [ ] Rate limiting implémenté
- [ ] Inputs validés et sanitisés

### Documentation
- [ ] Spec OpenAPI fournie ou proposée
- [ ] Tous les codes de réponse documentés (success + error)

---

## Adaptations par langage

### C# .NET (ASP.NET Core)

```csharp
// Conventions REST en .NET
[ApiController]
[Route("v1/[controller]")]  // → /v1/orders
public class OrdersController : ControllerBase
{
    [HttpGet]                          // GET /v1/orders
    [HttpGet("{id}")]                  // GET /v1/orders/{id}
    [HttpPost]                         // POST /v1/orders → 201 Created
    [HttpPut("{id}")]                  // PUT /v1/orders/{id}
    [HttpPatch("{id}")]                // PATCH /v1/orders/{id}
    [HttpDelete("{id}")]               // DELETE /v1/orders/{id} → 204
}
```

- Utiliser `[ProducesResponseType]` pour documenter les codes HTTP
- Utiliser `ProblemDetails` (ASP.NET Core intégré) pour les erreurs → compatible RFC 7807
- NuGet recommandés : `Swashbuckle.AspNetCore` (OpenAPI), `FluentValidation`
- Versionnage : `Microsoft.AspNetCore.Mvc.Versioning`

### Java Spring Boot

```java
// Conventions REST en Spring
@RestController
@RequestMapping("/v1/orders")
public class OrderController {

    @GetMapping               // GET /v1/orders
    @GetMapping("/{id}")      // GET /v1/orders/{id}
    @PostMapping              // POST /v1/orders → 201 Created
    @PutMapping("/{id}")      // PUT /v1/orders/{id}
    @PatchMapping("/{id}")    // PATCH /v1/orders/{id}
    @DeleteMapping("/{id}")   // DELETE /v1/orders/{id} → 204
}
```

- Utiliser `ResponseEntity<>` pour contrôler les codes HTTP et headers
- Spring `@ControllerAdvice` + `@ExceptionHandler` pour la gestion d'erreurs centralisée
- `ProblemDetail` (Spring 6+) pour les erreurs RFC 7807
- Dépendances recommandées : `springdoc-openapi-ui`, `spring-boot-starter-validation`

### Node.js / Express

```javascript
// Conventions REST en Express
router.get('/orders', ...)           // Liste paginée
router.get('/orders/:id', ...)       // Item unique
router.post('/orders', ...)          // Création → 201
router.put('/orders/:id', ...)       // Remplacement
router.patch('/orders/:id', ...)     // Mise à jour partielle
router.delete('/orders/:id', ...)    // Suppression → 204
```

### Python / FastAPI

```python
@router.get("/orders", response_model=OrderListResponse)
@router.get("/orders/{order_id}", response_model=OrderResponse)
@router.post("/orders", response_model=OrderResponse, status_code=201)
@router.put("/orders/{order_id}", response_model=OrderResponse)
@router.patch("/orders/{order_id}", response_model=OrderResponse)
@router.delete("/orders/{order_id}", status_code=204)
```

---

## Patterns courants

### Pattern : Pagination cursor-based

```json
// Response
{
  "items": [...],
  "pagination": {
    "limit": 20,
    "next_cursor": "eyJpZCI6MjB9",
    "prev_cursor": "eyJpZCI6MX0=",
    "has_next": true,
    "has_prev": false
  }
}
```

### Pattern : Erreur Problem JSON (RFC 7807)

```json
{
  "type": "https://api.example.com/errors/validation-error",
  "title": "Validation Error",
  "status": 400,
  "detail": "One or more fields are invalid",
  "instance": "/v1/orders",
  "errors": [
    { "field": "quantity", "message": "Must be greater than 0" }
  ]
}
```

### Pattern : Création avec Location header

```http
POST /v1/orders HTTP/1.1
Content-Type: application/json

{"product_id": "abc", "quantity": 2}

→ HTTP/1.1 201 Created
   Location: /v1/orders/550e8400-e29b-41d4-a716-446655440000
   Content-Type: application/json
```

### Pattern : Opération asynchrone

```http
POST /v1/reports/generate HTTP/1.1

→ HTTP/1.1 202 Accepted
   Location: /v1/reports/jobs/job-123

GET /v1/reports/jobs/job-123
→ { "status": "PROCESSING", "progress": 45 }
→ { "status": "COMPLETED", "result_url": "/v1/reports/report-456" }
```

---

## Référence rapide — Codes HTTP

| Situation | Code |
|-----------|------|
| Lecture réussie | 200 |
| Création réussie | 201 + Location |
| Traitement async | 202 + Location |
| Suppression réussie | 204 |
| Batch avec mix succès/erreur | 207 |
| Données invalides (format) | 400 |
| Non authentifié | 401 |
| Non autorisé | 403 |
| Ressource introuvable | 404 |
| Conflit de données | 409 |
| Erreur règle métier | 422 |
| Rate limit dépassé | 429 + Retry-After |
| Erreur interne | 500 |

---

## Fichiers de référence

- **`references/reference.md`** — Documentation complète consolidée depuis toutes les sources  
  *(URIs, méthodes, codes HTTP, payload, headers, versionnage, pagination, erreurs, sécurité, performance, HATEOAS, OpenAPI, conventions)*

  Table des matières :
  1. Principes fondamentaux REST
  2. Conception des URIs
  3. Méthodes HTTP
  4. Codes de statut HTTP
  5. Payload et formats de données
  6. Headers HTTP
  7. Versionnage
  8. Pagination
  9. Gestion des erreurs
  10. Sécurité
  11. Performance et cache
  12. HATEOAS
  13. OpenAPI / Documentation
  14. Conventions de nommage

> Lire `references/reference.md` en priorité pour tout détail sur un point spécifique.
