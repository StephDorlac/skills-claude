---
name: commit-helper
description: >
  Génère des messages de commit et de stash clairs, structurés et conventionnels.
  Utilise ce skill dès que l'utilisateur mentionne vouloir faire un commit, un stash,
  committer des changements, enregistrer son travail dans git, ou demande un message
  de commit/stash — même implicitement (ex. j'ai fini ma feature, je veux sauvegarder
  mes modifs, c'est quoi mon message de commit). Produit une ligne de résumé, une
  description détaillée et la liste des éléments affectés au format Conventional Commits.
---

# Commit Helper

Génère des messages de commit (et de stash) au format **Conventional Commits**, structurés en trois parties : résumé, description et éléments affectés.

---

## Format Conventional Commits

### Structure de base

```
<type>(<scope>): <résumé court>

<description détaillée>

Affects:
- <élément 1>
- <élément 2>
```

### Types disponibles

| Type       | Usage |
|------------|-------|
| `feat`     | Nouvelle fonctionnalité |
| `fix`      | Correction de bug |
| `refactor` | Refactoring sans changement de comportement |
| `perf`     | Amélioration de performance |
| `test`     | Ajout ou modification de tests |
| `docs`     | Documentation uniquement |
| `style`    | Formatage, espaces, point-virgules (pas de logique) |
| `chore`    | Maintenance, dépendances, config CI/CD |
| `build`    | Système de build, outils externes |
| `ci`       | Pipelines CI/CD |
| `revert`   | Annulation d'un commit précédent |

### Règles de rédaction

- **Résumé** : impératif présent, minuscule, max 72 caractères, pas de point final
- **Scope** : optionnel, entre parenthèses — module, package, couche concernée
- **Description** : explique le *pourquoi*, pas le *comment* (le code dit déjà le comment)
- **Breaking change** : ajoute `!` après le type/scope, et un footer `BREAKING CHANGE: <explication>`
- **Langue** : respecter la langue de l'utilisateur (FR ou EN selon le contexte)

---

## Workflow à suivre

### 1. Collecter le contexte

Si l'utilisateur ne fournit pas les changements, demande-lui :
- Quels fichiers / modules ont été modifiés ?
- Quel était l'objectif de ces changements ?
- Y a-t-il des breaking changes ?

Si l'utilisateur a collé un `git diff` ou `git status`, analyse-le directement.

### 2. Générer le message

Produis **toujours** les trois sections :

#### Ligne de résumé
```
<type>(<scope>): <résumé impératif, ≤72 caractères>
```

#### Description
2 à 5 lignes expliquant le contexte et la motivation. Commence par un verbe d'action.

#### Éléments affectés
Liste des fichiers, classes, méthodes, endpoints ou modules impactés.

---

## Cas particulier : git stash

Pour un stash, génère un message descriptif (utilisé avec `git stash push -m "..."`).

Format recommandé :
```
WIP(<scope>): <description courte de l'état en cours>
```

Exemple :
```
WIP(auth): implémentation partielle du refresh token JWT
```

---

## Exemple complet

**Contexte fourni par l'utilisateur** : *"J'ai ajouté la validation des emails dans le service d'inscription, modifié UserService.cs et ajouté des tests unitaires."*

**Sortie générée** :

```
feat(auth): ajouter la validation d'email à l'inscription

Intègre une vérification du format et de l'unicité de l'email
lors de l'enregistrement d'un nouvel utilisateur.
Lève une BadRequestException si l'email est invalide ou déjà utilisé.

Affects:
- Services/UserService.cs
- Models/RegisterRequest.cs
- Tests/UserServiceTests.cs
```

---

## Conseils additionnels (à mentionner si pertinent)

- Pour lier un ticket : ajouter en footer `Refs: #123` ou `Closes: #456`
- Pour un revert : `revert: feat(auth): ajouter la validation d'email` + footer `Reverts: <hash>`
- Éviter les messages vagues : `fix: bug`, `update`, `wip` (sauf pour stash)
- Un commit = une intention logique unique

---

## Pour les projets .NET / Java Spring (contexte utilisateur)

- **Scope courants .NET** : `api`, `domain`, `infrastructure`, `migrations`, `dto`, `auth`, `config`
- **Scope courants Spring** : `controller`, `service`, `repository`, `entity`, `config`, `security`, `dto`
- Mentionner la couche architecturale dans le scope aide à la lisibilité (ex: `feat(service/user)`)
