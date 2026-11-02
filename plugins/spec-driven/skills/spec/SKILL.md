---
name: spec
description: >-
  Crée et maintient une spécification fonctionnelle versionnée dans `.claude/specs/`,
  point de départ du cycle spec-driven (spec → plan → implémentation). Décrit le
  besoin (le QUOI et le POURQUOI), jamais le code, après une revue rapide du
  code existant pour tenir compte de ce qui est déjà là. Découpe la fonctionnalité en
  tâches cochables à identifiants stables (1, 1-1, 1-2…) qui servent de fil
  conducteur jusqu'à l'implémentation. À invoquer uniquement via la commande
  `/spec`, avec la fonctionnalité à spécifier en argument.
argument-hint: <fonctionnalité à spécifier> (obligatoire)
disable-model-invocation: true
---

# /spec — Spécification fonctionnelle

## Rôle dans le cycle spec-driven

Première étape de **spec → plan → implémentation**. La spec est la **source de
vérité fonctionnelle** : `/plan` la traduit en plan technique, `/implement`
réalise le code. Elle décrit le besoin, jamais la solution technique ni du code.

## Invocation

- Déclenchement **uniquement** via `/spec`. Ne pas s'activer autrement.
- Argument : la fonctionnalité à spécifier (description libre), **attendue et
  obligatoire**. Si l'argument est absent ou vide, demander la description à
  l'utilisateur avant de poursuivre — ne jamais démarrer une spec sans elle.
- Hérite du modèle d'agent courant (aucun override).
- **Emplacements** : specs dans `.claude/specs/`, plans dans `.claude/plans/`. Si
  `_specs/` ou `_plans/` existent encore à la racine alors que le dossier
  correspondant sous `.claude/` n'existe pas, le signaler et proposer de les
  déplacer (`git mv`, avec confirmation) avant de poursuivre.

## Principes

- **Le QUOI et le POURQUOI, pas le COMMENT.** Aucune classe, fonction, schéma de
  base de données, techno ni bibliothèque : le découpage technique appartient à
  `/plan`.
- **Ne jamais inventer une règle métier.** Toute zone d'ombre devient une
  question ouverte adressée à l'utilisateur.
- **Document vivant.** La spec reflète en permanence l'état réel du code pour la
  fonctionnalité concernée (voir « Cycle de vie »).

## Processus

1. **Analyser** la demande et le contexte : specs existantes dans
   `.claude/specs/`. Les nouvelles exigences ne doivent pas contredire ce qui y est
   déjà défini.
2. **Revue rapide de l'existant** — avant toute rédaction, parcourir le code pour
   savoir ce qui existe déjà en rapport avec la demande (voir « Revue de
   l'existant »).
3. **Clarifier** : poser des questions tant que des exigences ou des règles
   restent ambiguës, en s'appuyant sur ce que la revue a révélé. Ne pas produire
   de spec fondée sur des suppositions.
4. **Rédiger** `.claude/specs/<slug>.md` (slug en kebab-case) selon le gabarit.
5. **Itérer** : l'utilisateur relit (commentaires dans le fichier ou via le
   prompt), tu ajustes, jusqu'à validation explicite.

> Prochaine étape : `/plan <ids>` pour transformer les tâches de cette spec en
> plan d'implémentation technique.

## Revue de l'existant

Une fonctionnalité s'insère dans un produit déjà là : sans cette revue, la spec
risque de redemander ce qui existe, de contredire un comportement établi ou de
passer à côté d'un cas déjà géré.

- **Rapide et ciblée** : chercher dans le code les termes, écrans, parcours,
  modèles ou règles proches de la demande, et lire les quelques modules les plus
  pertinents. Ce n'est pas une cartographie complète — c'est le rôle de `/plan`.
- **Chercher** : ce qui existe déjà et couvre tout ou partie du besoin (la
  demande est peut-être une évolution, pas une création), les comportements
  voisins que la nouveauté doit respecter ou ne pas casser, les règles métier
  déjà appliquées, ce qui pourrait être réutilisé tel quel.
- **Restituer à l'utilisateur** avant de rédiger : résumé court de ce qui a été
  trouvé et des écarts avec la demande (doublon, contradiction, partie déjà
  faite). Ces écarts alimentent les questions de clarification.
- **Consigner côté fonctionnel uniquement** dans la section « Existant en
  rapport » du gabarit : des comportements décrits du point de vue de l'usager,
  sans chemin de fichier, nom de classe ni code. Les détails techniques de
  l'existant appartiennent à `/plan`.
- Si rien de pertinent n'existe (projet neuf, domaine inédit), écrire « Aucun
  existant en rapport » plutôt que de combler la section.

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
- « Existant en rapport » est mise à jour à chaque évolution de la spec : une
  fois la fonctionnalité livrée, le nouveau comportement en fait partie.
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

## Gabarit de sortie (`.claude/specs/<slug>.md`)

```markdown
# Spec — <Titre>

## Contexte et objectif
<le besoin global et le pourquoi>

## Existant en rapport
- <comportement déjà présent, du point de vue usager : ce qui est réutilisé, étendu
  ou à ne pas casser — ou « Aucun existant en rapport »>

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
