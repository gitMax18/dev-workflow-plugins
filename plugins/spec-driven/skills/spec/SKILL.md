---
name: spec
description: >-
  Crée et maintient une spécification fonctionnelle versionnée dans `_specs/`,
  point de départ du cycle spec-driven (spec → plan → implémentation). Décrit le
  besoin (le QUOI et le POURQUOI), jamais le code. Découpe la fonctionnalité en
  tâches cochables à identifiants stables (1, 1-1, 1-2…) qui servent de fil
  conducteur jusqu'à l'implémentation. À invoquer uniquement via la commande
  `/spec`, avec la fonctionnalité à spécifier en argument.
disable-model-invocation: true
---

# /spec — Spécification fonctionnelle

## Rôle dans le cycle spec-driven

Première étape de **spec → plan → implémentation**. La spec est la **source de
vérité fonctionnelle** : `/plan` la traduit en plan technique, `/implement`
réalise le code. Elle décrit le besoin, jamais la solution technique ni du code.

## Invocation

- Déclenchement **uniquement** via `/spec`. Ne pas s'activer autrement.
- Argument : la fonctionnalité à spécifier (description libre).
- Hérite du modèle d'agent courant (aucun override).

## Principes

- **Le QUOI et le POURQUOI, pas le COMMENT.** Aucune classe, fonction, schéma de
  base de données, techno ni bibliothèque : le découpage technique appartient à
  `/plan`.
- **Ne jamais inventer une règle métier.** Toute zone d'ombre devient une
  question ouverte adressée à l'utilisateur.
- **Document vivant.** La spec reflète en permanence l'état réel du code pour la
  fonctionnalité concernée (voir « Cycle de vie »).

## Processus

1. **Analyser** la demande et le contexte : specs existantes, code en place, et
   `_specs/common.md` s'il existe (éléments transversaux déjà documentés — types
   partagés, règles d'erreur communes, comportements établis). Les nouvelles
   exigences ne doivent pas contredire ce qui y est déjà défini.
2. **Clarifier** : poser des questions tant que des exigences ou des règles
   restent ambiguës. Ne pas produire de spec fondée sur des suppositions.
3. **Rédiger** `_specs/<slug>.md` (slug en kebab-case) selon le gabarit.
4. **Itérer** : l'utilisateur relit (commentaires dans le fichier ou via le
   prompt), tu ajustes, jusqu'à validation explicite.

> Prochaine étape : `/plan <ids>` pour transformer les tâches de cette spec en
> plan d'implémentation technique.

## Identifiants et traçabilité

- Chaque fonctionnalité et chaque tâche reçoit un **identifiant stable** :
  `1`, `2` pour les fonctionnalités ; `1-1`, `1-2` pour leurs tâches ; `1-1-1`
  pour une sous-tâche.
- Les identifiants ne sont **jamais réutilisés ni renumérotés**, même après
  suppression d'une tâche : ils sont la clé de liaison utilisée par `/plan` et
  `/implement`.
- Découper en tâches **petites, ordonnées logiquement et livrables/vérifiables
  indépendamment**, pour que chaque implémentation reste facile à relire.

## Critères d'acceptation

Chaque fonctionnalité/tâche définit des critères d'acceptation **testables et
observables** — vérifiables sans connaître l'implémentation, formulés de
préférence en _Étant donné / Quand / Alors_. Ils constituent la définition du
« fait » et servent de base aux tests dans `/plan` puis `/implement`.

## Cycle de vie (document vivant)

- L'avancement se lit dans les **cases à cocher** : `[ ]` à faire, `[x]` fait
  (implémenté **et** testé dans le code). C'est la source de vérité de l'état.
- **Évolution d'une fonctionnalité existante** : la spec intègre ce qu'il faut
  ajouter, modifier ou retirer. Une fois la modification livrée, la spec ne
  garde **aucune trace de l'ancien comportement** : elle ne décrit que ce qui
  existe ou ce qui reste à faire.
- Toute modification du fichier par l'agent se fait **avec confirmation** de
  l'utilisateur.

## Contenu d'une fonctionnalité / tâche

- **Contexte et objectif** — le besoin et le pourquoi.
- **Exigences fonctionnelles** — comportements attendus, du point de vue usager.
- **Règles métier** — contraintes, invariants, autorisations.
- **Cas limites** — situations aux bords, erreurs, états vides.
- **Critères d'acceptation** — testables (cf. ci-dessus).
- **Hors périmètre** — ce qui est explicitement exclu.
- **Questions ouvertes** — clarifications attendues de l'utilisateur.

La spec **ne contient ni code ni choix d'implémentation**.

## Gabarit de sortie (`_specs/<slug>.md`)

```markdown
# Spec — <Titre>

## Contexte et objectif
<le besoin global et le pourquoi>

## Questions ouvertes
- [ ] <clarification attendue de l'utilisateur>

## Fonctionnalités

### [ ] 1 — <Nom de la fonctionnalité>
- **Contexte et objectif** :
- **Exigences fonctionnelles** :
- **Règles métier** :
- **Cas limites** :
- **Critères d'acceptation** :
- **Hors périmètre** :
- **Questions ouvertes** :

#### Tâches
- [ ] **1-1** — <tâche, livrable et vérifiable>
- [ ] **1-2** — <tâche>

### [ ] 2 — <Nom de la fonctionnalité>
…
```

> Cocher `[x]` une fonctionnalité/tâche **uniquement** lorsqu'elle est
> implémentée et testée dans le code.
