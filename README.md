# skills-claude

Collection de skills pour [Claude Code](https://github.com/anthropics/claude-code), le CLI officiel d'Anthropic.

Chaque skill est un fichier `SKILL.md` qui étend les capacités de Claude Code avec des comportements spécialisés, activés automatiquement selon le contexte ou sur demande explicite.

## Installation

Cloner ce dépôt dans le répertoire de skills de Claude Code :

```bash
git clone https://github.com/StephDorlac/skills-claude.git ~/.claude/skills
```

## Skills disponibles

### `commit-helper`

Génère des messages de commit et de stash clairs, structurés et conventionnels au format [Conventional Commits](https://www.conventionalcommits.org/).

**Déclenchement** : dès que tu mentionnes vouloir faire un commit, un stash, ou demandes un message de commit — même implicitement.

**Fonctionnalités :**
- Ligne de résumé (`type(scope): résumé`)
- Description détaillée orientée *pourquoi*
- Liste des éléments affectés
- Support des breaking changes, reverts et références de tickets
- Conseils adaptés aux projets .NET et Java Spring

### `rest-api-design`

Expert en conception et implémentation d'APIs REST, tous langages et frameworks (C# .NET, Java Spring, Node.js, Python FastAPI, Go, PHP…).

**Déclenchement** : dès que tu veux créer, concevoir, scaffolder, refactorer ou reviewer une API REST, ou que tu poses une question sur les bonnes pratiques REST.

**Fonctionnalités :**
- Checklist complète (URIs, méthodes HTTP, codes de statut, payload, pagination, sécurité)
- Patterns courants : pagination cursor-based, erreurs Problem JSON (RFC 7807), opérations asynchrones
- Adaptation automatique au langage/framework demandé
- Génération de specs OpenAPI/Swagger
- Basé sur les références Microsoft Azure, Zalando et autres sources reconnues

### `tu-helper`

Génère des règles de test unitaire claires et structurées pour tout langage et framework.

**Déclenchement** : dès que tu veux créer des tests unitaires, écrire des règles de test, ou refactorer des tests existants.

**Fonctionnalités :**
- Génération de règles de test basées sur les bonnes pratiques (AAA, Given/When/Then)
- Adaptation automatique au langage/framework demandé
- Support des patterns de test courants (mocks, stubs, fixtures)
- Conseils pour la couverture de test et la maintenabilité
