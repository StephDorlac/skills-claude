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
