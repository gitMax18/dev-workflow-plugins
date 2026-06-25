---
name: front-contract
description: >-
  Génère un fichier Markdown dans `_front/` décrivant tout ce qu'un développeur
  frontend doit savoir pour intégrer un backend : endpoints, formats de
  requête/réponse, authentification, codes d'erreur et règles métier côté
  serveur. À appeler sur un repo backend, idéalement après `/review-workflow`
  (cycle spec-driven terminé) ou dès que la spec est stabilisée. Se base sur la
  spec (générée par `/spec`) et explore le code backend pour valider et compléter
  les informations. À invoquer uniquement via `/front-contract`, avec en argument
  le fichier de spec (ou la spec elle-même) et optionnellement les identifiants
  de sections à traiter (ex. `1`, `1-1`).
disable-model-invocation: true
---

# /front-contract — Contrat Frontend

## Rôle dans le cycle spec-driven

Complément de **spec → plan → implémentation**, orienté équipe frontend. Là où
`/spec` décrit le besoin fonctionnel, `/front-contract` traduit ce besoin en
**contrat d'intégration** : ce que le backend expose concrètement et ce que le
frontend doit faire pour en tirer parti. S'appuie sur la spec pour cadrer le
périmètre, puis plonge dans le code source pour extraire les détails réels.

## Invocation

- Déclenchement **uniquement** via `/front-contract`. Ne pas s'activer autrement.
- Arguments possibles :
  - **`<fichier-spec>`** — chemin vers un fichier de spec (`_specs/auth.md`).
  - **`<identifiants>`** — un ou plusieurs identifiants de sections de la spec,
    séparés par des espaces (`1`, `1-1 1-2`, `2`). Limite le périmètre aux
    sections indiquées.
  - Sans argument : la spec peut être fournie directement dans le contexte de la
    conversation.

**Exemples d'appel :**
```
/front-contract _specs/auth.md
/front-contract _specs/auth.md 1-1
/front-contract _specs/auth.md 1-1 1-2 2
```

## Processus

### 1. Lire la spec

- Si un chemin de fichier est fourni, lire le fichier.
- Si des identifiants de sections sont fournis, extraire uniquement les sections
  correspondantes (ex. `### [ ] 1 — …` et ses sous-sections pour `1`, ou
  `**1-1**` pour `1-1`).
- Si aucun argument n'est fourni, utiliser la spec présente dans le contexte de
  la conversation.

### 2. Explorer le code backend

Pour chaque fonctionnalité ou tâche dans le périmètre, partir des **exigences
fonctionnelles et règles métier** de la spec pour retrouver, dans le code :

- **Routes/endpoints** : fichiers de routage, contrôleurs, handlers.
- **Modèles de données** : DTOs, schemas de validation, types de requête et
  réponse.
- **Authentification et autorisation** : middlewares, guards, annotations de
  permissions — quelles routes sont protégées et comment.
- **Validation** : règles de validation des champs (obligatoire, format, longueur,
  valeurs autorisées…).
- **Erreurs** : codes HTTP retournés, messages d'erreur, cas particuliers gérés
  côté serveur.
- **Comportements métier notables** : effets de bord, règles de calcul, états
  transitoires, limites de débit, dépendances entre appels.

> Explorer largement : routes, contrôleurs, services, middlewares, schemas,
> enums, constantes. Le but est de ne laisser aucune surprise au développeur
> frontend.

### 3. Générer le fichier de contrat

Créer (ou mettre à jour si le fichier existe déjà) le fichier dans `_front/`.

**Nommage du fichier :**
- Spec complète : `_front/<slug-de-la-spec>.md`
- Sections ciblées : `_front/<slug-de-la-spec>-<identifiants>.md`
  (ex. `_front/auth-1-1.md` pour la section `1-1` de `auth.md`)

## Gabarit de sortie

Le fichier doit être **directement utilisable** par un développeur frontend sans
accès au code backend. Organiser par fonctionnalité (cohérente avec la
numérotation de la spec), puis par endpoint si plusieurs.

```markdown
# Contrat Frontend — <Titre de la spec> [sections X-Y si périmètre limité]

> Généré depuis `_specs/<slug>.md` · sections : <liste ou "toutes">
> Dernière mise à jour : <date ISO>

---

## <Numéro> — <Nom de la fonctionnalité>

### `<MÉTHODE> <chemin>`

**Auth requise :** <oui/non — si oui : type (Bearer JWT, cookie session…) et
rôle/permission minimum>

**Paramètres de chemin :**
| Paramètre | Type | Description |
|-----------|------|-------------|
| `id` | `uuid` | Identifiant de la ressource |

**Paramètres de requête (query) :**
| Paramètre | Type | Obligatoire | Description |
|-----------|------|-------------|-------------|
| `page` | `number` | non | Page (défaut : 1) |

**Corps de la requête (JSON) :**
```json
{
  "email": "string (requis, format email)",
  "password": "string (requis, min 8 chars)"
}
```

**Réponse — succès (`2xx`) :**
```json
{
  "token": "string (JWT)",
  "user": { "id": "uuid", "email": "string" }
}
```

**Réponses d'erreur :**
| Code | Condition | Corps |
|------|-----------|-------|
| `400` | Champ manquant ou invalide | `{ "error": "validation_error", "fields": {...} }` |
| `401` | Identifiants incorrects | `{ "error": "invalid_credentials" }` |
| `429` | Trop de tentatives | `{ "error": "rate_limited", "retryAfter": number }` |

**Règles métier à connaître :**
- <Comportement non évident que le frontend doit gérer>
- <Ex : le token expire après 15 min, un refresh token est retourné séparément>

---
```

## Règles de rédaction

- **Ne jamais inventer.** Si une information est absente du code et non déductible
  de la spec, la noter explicitement comme `⚠️ À confirmer avec le backend`.
- **Pas de code interne.** Le contrat décrit l'API publique, pas les détails
  d'implémentation (noms de variables internes, SQL, etc.).
- **Exhaustivité sur les erreurs.** Tous les cas d'erreur identifiables dans le
  code doivent figurer. Le frontend doit pouvoir gérer chaque cas sans surprise.
- **Format JSON commenté.** Pour les corps de requête/réponse, annoter chaque
  champ avec son type, son caractère obligatoire et les contraintes.
- **Date de génération.** Toujours inclure la date ISO de génération en en-tête
  pour signaler une éventuelle obsolescence.
- Rédiger en français sauf les noms techniques (endpoints, champs JSON, codes
  HTTP).
