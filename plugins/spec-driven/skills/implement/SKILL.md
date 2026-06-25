---
name: implement
description: >-
  Implémente fidèlement un plan d'implémentation (`_plans/`), dernière étape du
  cycle spec → plan → implémentation. Exécute le plan sans rien ajouter hors
  périmètre, s'intègre à la stack et aux conventions du dépôt, écrit les tests
  des critères d'acceptation, puis met à jour les cases de la spec. À invoquer
  uniquement via `/implement`, avec le plan ou les identifiants (1-1, 2-1…) à
  réaliser.
disable-model-invocation: true
---

# /implement — Réalisation du plan

## Rôle dans le cycle spec-driven

Dernière étape de **spec → plan → implémentation**. Écrit le code qui réalise un
plan (`_plans/`), puis met la spec à jour pour qu'elle reflète l'état réel.

## Invocation

- Déclenchement **uniquement** via `/implement`.
- Argument : le plan à réaliser, ou des identifiants précis (`1-1`, `2-1`…).
- Hérite du modèle d'agent courant.

## Sources d'information (par priorité)

1. Le **code existant** du dépôt.
2. Le **plan** (`_plans/<slug>.md`) — référence principale.
3. La **spec** (`_specs/<slug>.md`) — besoin et critères d'acceptation.
4. **`_specs/common.md`** — éléments transversaux déjà livrés (types partagés,
   middlewares, classes d'erreur communes…). Consulter avant de créer quoi que ce
   soit qui pourrait y figurer.
5. Les **conventions** du projet (`CLAUDE.md` ou `AGENTS.md`, patterns du dépôt).
6. Les **bonnes pratiques** de la stack utilisée.

En cas de conflit : le code existant prime sur les préférences personnelles ; le
plan prime sur les suppositions ; toute incohérence est **signalée avant**
d'agir.

## Principes

- **Fidélité au plan.** N'implémenter que ce que le plan décrit ; chaque
  modification a une justification directe dans le plan.
- **Respect du périmètre.** Ne pas ajouter de fonctionnalité, anticiper un
  besoin futur, modifier un comportement existant sans justification, ni corriger
  du code hors périmètre (sauf blocage — alors le signaler avant).
- **Intégration naturelle.** Réutiliser composants, modèles, services et patterns
  existants ; ne pas dupliquer. Le code doit être indiscernable du reste du
  dépôt.
- **Sobriété.** Pas d'abstraction prématurée, de généralisation ni
  d'optimisation spéculative. Préférer la clarté à la sophistication.
- **Stack du dépôt.** Suivre les conventions et bonnes pratiques de la techno en
  place. N'ajouter une dépendance qu'avec l'accord de l'utilisateur.

## Processus

1. **Contexte** — relever architecture, conventions, patterns, outils de
   test/build/qualité. L'implémentation doit s'y fondre.
2. **Plan** — identifier les éléments à créer/modifier, dépendances, validations
   et scénarios. Ne pas réinterpréter le besoin ni redéfinir le périmètre.
3. **Implémentation** — réaliser le plan par étapes cohérentes, traçables aux
   identifiants de spec.
4. **Tests** — couvrir les critères d'acceptation, les validations, les cas
   d'erreur et les cas limites du plan. Pas de tests redondants.
5. **Vérification** — build/compilation, tests verts, conformité aux conventions,
   couverture de toutes les étapes du plan.

## Ambiguïtés

Si le plan est ambigu : décrire le problème, proposer des options, demander une
clarification avant d'implémenter. Ne jamais inventer un comportement métier.

## Critères de succès

- Toutes les exigences du plan sont réalisées, aucune hors périmètre.
- Les tests couvrent les critères d'acceptation et passent.
- Le code respecte les conventions et s'intègre naturellement au dépôt : un
  développeur du projet ne distingue pas les nouvelles modifications du reste.
