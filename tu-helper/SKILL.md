# Skill : Tests unitaires Java avec Mockito

## Objectif

Lors de la génération de tests unitaires pour des projets Java, utiliser **Mockito par défaut comme framework de mock**.

Cette règle doit être appliquée **automatiquement**, même si l'utilisateur ne demande pas explicitement l'utilisation de Mockito.

## Stack de test par défaut

Sauf indication contraire explicite, toujours utiliser :

- JUnit 5 (JUnit Jupiter)
- Mockito
- AssertJ (préféré) ou assertions standards JUnit

## Règles obligatoires

### 1. Stratégie de mock

- Toujours mocker les dépendances externes :
  - Repository
  - Services
  - Clients HTTP
  - Gateways
  - Accès base de données
  - Système de fichiers
  - Messaging

- Ne jamais appeler d’infrastructure réelle dans un test unitaire.
- Utiliser :
  - `@Mock`
  - `@InjectMocks`
  - ou `Mockito.mock()`

### 2. Structure des tests

Les tests doivent suivre le pattern **AAA** :

- Arrange
- Act
- Assert

Chaque test doit :

- Tester un seul comportement
- Avoir un nom clair et explicite
- Avoir une préparation minimale

### 3. Utilisation de Mockito

Toujours préférer :

```java
@ExtendWith(MockitoExtension.class)
