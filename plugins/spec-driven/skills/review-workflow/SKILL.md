---
name: review-workflow
description: >-
  Clôture le cycle spec-driven : lit les plans (`_plans/`), valide et coche les
  tâches réalisées dans les specs, génère un résumé d'implémentation par section,
  extrait les éléments transversaux vers `_specs/common.md`, puis supprime les
  plans après confirmation. À invoquer via `/review-workflow`, avec en argument les
  fichiers de plan à traiter (sinon, tous les `_plans/*.md`).
disable-model-invocation: true
---

# /review-workflow — Clôture du cycle spec-driven

## Rôle dans le cycle spec-driven

Quatrième et dernière étape du cycle **spec → plan → implémentation → review-workflow**.
Consolide ce qui a été livré dans la spec, extrait les éléments transversaux, nettoie
les plans consommés.

## Invocation

- Déclenchement **uniquement** via `/review-workflow`.
- Argument optionnel : un ou plusieurs fichiers de plan à traiter (chemins ou noms,
  ex. `_plans/auth-login.md _plans/user-profile.md`). Si aucun argument, traiter
  **tous** les fichiers `_plans/*.md` présents.
- Hérite du modèle d'agent courant.

## Processus

### 1. Identifier les plans à traiter

- Si des arguments sont fournis : utiliser uniquement ces plans.
- Sinon : lister tous les fichiers `_plans/*.md`.
- Si aucun plan n'existe : informer l'utilisateur et s'arrêter.

### 2. Identifier la spec correspondante à chaque plan

- Chercher `_specs/<même-slug>.md` (ex. `_plans/auth-login.md` → `_specs/auth-login.md`).
- Si le slug ne correspond à aucune spec, déduire la spec depuis le contenu du plan
  (section « Périmètre » et en-tête du fichier).
- Si la correspondance reste ambiguë, demander à l'utilisateur avant de continuer.
- Plusieurs plans peuvent pointer vers la même spec : agréger leurs contributions.

### 3. Analyse croisée plan ↔ spec ↔ code

Pour chaque fonctionnalité/tâche référencée dans le plan :

**a. Tâche déjà `[x]`** dans la spec : aucune modification des cases, inclure dans le
résumé.

**b. Tâche encore `[ ]` dans la spec, mais couverte par le plan :**
- Inspecter le code (fichiers concernés selon le plan) pour déterminer l'état réel.
- *Totalement réalisée et testée* → cocher `[x]`.
- *Partiellement réalisée* → scinder la tâche (voir « Gestion des tâches partielles »).
- *Non réalisée* → laisser `[ ]` sans modification.

**c. Élément présent dans la spec mais absent du plan :** ne pas toucher, ne pas cocher.

**d. Fonctionnalité parente :** cocher `[x]` uniquement si **toutes** ses tâches sont
cochées.

### 4. Gestion des tâches partiellement réalisées

Scinder la tâche en deux lignes suffixées `a` / `b` (identifiants stables) :

```markdown
- [x] **1-2a** — <ce qui a effectivement été réalisé>
- [ ] **1-2b** — <ce qui reste à faire>
```

L'identifiant original (`1-2`) ne doit plus apparaître après scission : il est remplacé
par `1-2a` et `1-2b`, qui ne seront jamais renumérotés.

### 5. Générer le résumé d'implémentation

Ajouter une section `## Résumé d'implémentation` **à la fin de chaque bloc de
fonctionnalité** (`### N — <Nom>`) concerné par au moins un plan traité, après les
tâches :

```markdown
## Résumé d'implémentation

<Courte intro en une phrase décrivant ce qui a été livré.>

**Créations** : `chemin/fichier.ts`, `chemin/fichier.test.ts`
**Modifications** : `chemin/autre.ts` — <raison courte>
```

Règles :
- Se baser sur les sections « Impacts » et « Ordre d'implémentation » du plan, croisées
  avec l'état réel du code.
- Omettre les catégories vides (pas de `**Modifications**` s'il n'y en a aucune).
- Ne pas mentionner les éléments transversaux dans ce résumé : ils sont traités
  exclusivement dans `_specs/common.md`.
- Si plusieurs plans couvrent la même fonctionnalité, fusionner leurs contributions dans
  un seul résumé.
- Si un résumé existe déjà dans la section, le remplacer plutôt que le dupliquer.

### 6. Extraire les éléments transversaux vers `_specs/common.md`

Identifier dans les plans (section « Impacts ») et dans le code les éléments **non
spécifiques à une seule fonctionnalité** : classes d'erreur partagées, utilitaires
génériques, middlewares globaux, types/interfaces transversaux, configuration commune, etc.

Pour chaque élément identifié, ajouter une entrée dans `_specs/common.md` sous la
section thématique appropriée :

```markdown
# Éléments communs

## <Thème> (ex. Gestion des erreurs, Utilitaires, Modèles partagés)
- `<chemin/fichier>` — <rôle> — ajouté lors de <fonctionnalité source> (<YYYY-MM-DD>)
```

Règles :
- Créer `_specs/common.md` s'il n'existe pas.
- Si la section thématique existe déjà, ajouter l'entrée sans dupliquer.
- Si un élément déjà référencé a évolué, mettre à jour son entrée plutôt que d'en
  ajouter une nouvelle.
- Si aucun élément transversal n'est identifié, ne pas créer ni modifier `_specs/common.md`.

### 7. Présenter le bilan à l'utilisateur

Afficher un récapitulatif structuré **avant toute modification** :

```
Bilan de révision

Specs mises à jour :
  _specs/auth-login.md
    [x] 1-1 : cochée
    [x] 1-2a : cochée (scindée depuis 1-2)
    [ ] 1-2b : non réalisée, laissée telle quelle
    + Résumé d'implémentation ajouté à la section 1

Éléments transversaux → _specs/common.md :
  - AuthError (src/errors/auth.ts) — Gestion des erreurs

Plans à supprimer après confirmation :
  - _plans/auth-login.md
```

Attendre la **confirmation explicite** de l'utilisateur avant d'appliquer quoi que ce soit.

Si des questions ouvertes non résolues subsistent dans le plan, les signaler dans le
bilan sans bloquer la confirmation.

### 8. Appliquer les modifications (après confirmation)

Dans cet ordre :
1. Mettre à jour chaque spec concernée (cases à cocher, scissions de tâches, résumés).
2. Mettre à jour `_specs/common.md` si des éléments transversaux ont été identifiés.
3. Supprimer les fichiers de plan traités.

### 9. Fin

Confirmer les modifications appliquées (fichiers mis à jour, plans supprimés).

## Invariants

- Ne jamais repasser une case `[x]` à `[ ]`.
- Ne jamais toucher aux specs dont aucun plan traité ne référence les tâches.
- Ne jamais inventer un contenu non présent dans le plan ou le code.
- Toute modification est soumise à confirmation via le bilan (étape 7).
- Les identifiants de tâches (`1-1`, `1-2`…) sont stables : après scission, les
  sous-identifiants (`1-2a`, `1-2b`) ne sont jamais renumérotés ni réutilisés.
- La date insérée dans `_specs/common.md` est la date du jour au format `YYYY-MM-DD`.
