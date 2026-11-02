---
name: implement
description: >-
  Implémente fidèlement un plan d'implémentation (`.claude/plans/`), dernière étape du
  cycle spec → plan → implémentation. Exécute le plan sans rien ajouter hors
  périmètre, s'intègre à la stack et aux conventions du dépôt, écrit les tests
  des critères d'acceptation. À invoquer uniquement via `/implement`, avec le
  plan ou les identifiants (1-1, 2-1…) à réaliser.
argument-hint: <plan> (ou vide si le plan est déjà dans le contexte)
disable-model-invocation: true
---

# /implement — Réalisation du plan

## Rôle dans le cycle spec-driven

Dernière étape de **spec → plan → implémentation**. Écrit le code qui réalise un
plan (`.claude/plans/`).

## Invocation

- Déclenchement **uniquement** via `/implement`.
- Argument : le plan à réaliser, ou des identifiants précis (`1-1`, `2-1`…).
- Hérite du modèle d'agent courant.
- **Emplacements** : specs dans `.claude/specs/`, plans dans `.claude/plans/`. Si
  `_specs/` ou `_plans/` existent encore à la racine alors que le dossier
  correspondant sous `.claude/` n'existe pas, le signaler et proposer de les
  déplacer (`git mv`, avec confirmation) avant de poursuivre.

## Sources d'information (par priorité)

1. Le **code existant** du dépôt.
2. Le **plan** (`.claude/plans/<slug>.md`) — référence principale.
3. La **spec** (`.claude/specs/<slug>.md`) — besoin et critères d'acceptation.
4. Les **conventions** du projet (`CLAUDE.md` ou `AGENTS.md`, patterns du dépôt).
5. Les **bonnes pratiques** de la stack utilisée.

En cas de conflit : le code existant prime sur les préférences personnelles ; le
plan prime sur les suppositions ; toute incohérence est **signalée avant**
d'agir.

## Principes

- **Fidélité au plan.** N'implémenter que ce que le plan décrit ; chaque
  modification a une justification directe dans le plan.
- **Respect du périmètre.** Ne pas ajouter de fonctionnalité, anticiper un
  besoin futur, modifier un comportement existant sans justification, ni corriger
  du code hors périmètre (sauf blocage — alors le signaler avant). Un
  refactoring que le plan ne contient pas n'est pas fait, même s'il paraît évident :
  le signaler pour un prochain `/plan`.
- **Aucun effet de bord non voulu.** Modifier un élément partagé ne doit rien
  changer ailleurs : avant de toucher à du code, des styles, un contrat d'API ou
  une donnée existants, retrouver leurs utilisateurs dans le dépôt, s'appuyer sur
  les « Effets de bord » du plan et préférer le changement additif qui conserve
  l'ancien comportement par défaut. Un effet imprévu n'est pas absorbé en silence :
  le signaler (voir « Effets de bord »).
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
   identifiants de spec. Les refactorings du plan (`R1`, `R2`…) passent en
   premier : comportement inchangé, tests verts avant et après chaque étape,
   sans élargir ce que le plan a validé. Les constats listés en « Refactoring
   différé » ne sont pas traités ici.
4. **Tests** — couvrir les critères d'acceptation, les validations, les cas
   d'erreur et les cas limites du plan. Pas de tests redondants.
5. **Vérification** — build/compilation, tests verts, conformité aux conventions,
   couverture de toutes les étapes du plan, et contrôle des effets de bord (voir
   ci-dessous).
6. **Suite** — une fois l'implémentation vérifiée, proposer de lancer
   `/recette <spec> [ids]` pour valider le résultat côté utilisateur (parcours
   réels dans l'application locale), avant `/review-workflow`.

## Effets de bord

- **Avant de modifier** un élément existant : le rechercher dans tout le dépôt
  (usages, imports, sélecteurs, consommateurs) et confronter le résultat aux
  « Effets de bord » du plan. Un utilisateur que le plan ne mentionne pas = une
  analyse incomplète : le signaler avant de continuer.
- **En modifiant** : garder le comportement existant pour tous les usages non
  concernés (paramètre optionnel à valeur par défaut, nouvelle variante plutôt
  qu'altération de l'existant, style scopé plutôt que règle globale).
- **Après** : lancer les tests des modules touchés **et** de leurs utilisateurs,
  pas seulement les nouveaux ; contrôler type-check/lint/build ; vérifier les
  pages ou parcours qui partagent le composant ou les styles modifiés.
- **Un test existant qui casse** est un indice d'effet de bord : comprendre
  pourquoi avant de le modifier. Ne l'adapter que si le changement de comportement
  est voulu par le plan ; sinon corriger l'implémentation.
- **Un effet hors plan** (comportement ailleurs modifié, usage inattendu) :
  décrire le problème et demander à l'utilisateur comment le traiter, sans
  élargir le périmètre de sa propre initiative.

## Ambiguïtés

Si le plan est ambigu : décrire le problème, proposer des options, demander une
clarification avant d'implémenter. Ne jamais inventer un comportement métier.

## Critères de succès

- Toutes les exigences du plan sont réalisées, aucune hors périmètre.
- Les tests couvrent les critères d'acceptation et passent.
- Aucun comportement existant hors périmètre n'a changé : les utilisateurs des
  éléments modifiés ont été vérifiés.
- Le code respecte les conventions et s'intègre naturellement au dépôt : un
  développeur du projet ne distingue pas les nouvelles modifications du reste.
