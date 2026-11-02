# spec-driven

Cycle de développement **spec-driven** pour Claude Code : du besoin fonctionnel au code, en étapes traçables.

```
/spec   →   /plan   →   /implement   →   /recette   →   /review-workflow
(QUOI)    (COMMENT)    (le code)       (la recette)     (la clôture)
   │          │                            │                 │
   ▼          ▼                            ▼                 ▼
.claude/   .claude/    code + tests    .claude/test/    specs à jour,
specs/     plans/                                       plans supprimés
                              ↓
                       /front-contract
                   (contrat d'intégration)
                        .claude/front/
```

## Commandes

| Commande | Rôle |
| :------- | :--- |
| `/spec-driven:spec <fonctionnalité>` | Crée/maintient une spec fonctionnelle versionnée dans `.claude/specs/`. Commence par une revue rapide du code existant (section « Existant en rapport »). Décrit le besoin, jamais le code. Découpe en tâches à identifiants stables (`1`, `1-1`…). |
| `/spec-driven:plan <ids>` | Traduit une ou plusieurs tâches de spec en plan technique détaillé dans `.claude/plans/`. Décrit ce qui doit exister, sans écrire de code. Examine l'existant au-delà de la spec pour repérer le refactoring/la simplification utiles : le petit et lié entre dans le plan, le gros est soumis à l'utilisateur (maintenant, avant, plus tard). S'exécute en mode plan. |
| `/spec-driven:implement <ids\|plan>` | Réalise le code d'après le plan, écrit les tests des critères d'acceptation, puis met à jour les cases de la spec. |
| `/spec-driven:recette <spec> [ids…]` | Génère un plan de recette `recette-<périmètre>.md` dans `.claude/test/<spec>/` : éléments à tester, puis scénarios (compte issu de `.claude/test/comptes-test`, procédure via l'interface, vérifications, contrôles visuels si le front a changé). Propose ensuite de les jouer avec Playwright, **en local uniquement**, avec un script de retour à l'état de base. |
| `/spec-driven:review-workflow [plans…]` | Clôture le cycle : valide et coche les tâches réalisées, génère un résumé d'implémentation par section de spec, puis supprime les plans après confirmation. Sans argument, traite tous les `.claude/plans/*.md`. |
| `/spec-driven:front-contract <spec> [ids…]` | Génère un contrat d'intégration frontend dans `.claude/front/` à partir d'une spec et du code backend : endpoints, formats requête/réponse, auth, erreurs et règles métier. |

## Principes

- **Source de vérité fonctionnelle** : la spec décrit le QUOI et le POURQUOI ; le plan le COMMENT ; l'implémentation le code.
- **Traçabilité** : les identifiants de spec (`1-1`, `2-1`…) relient spec → plan → code, sans réinterprétation.
- **Documents vivants** : l'état d'avancement se lit dans les cases à cocher de la spec ; `/review-workflow` les consolide en fin de cycle.
- **Ancrage dans le dépôt** : stack et conventions déduites du code existant et de `CLAUDE.md`/`AGENTS.md`.
- **S'insérer dans l'existant** : `/spec` relève ce qui existe déjà, `/plan` cherche refactoring et simplification (en regardant au-delà du périmètre de la spec) et arbitre avec l'utilisateur ce qui entre dans le plan ; les constats reportés sont gardés dans `.claude/refactoring.md`.
- **Pas d'effet de bord** : `/plan` recense les utilisateurs de chaque élément existant modifié (appelants, styles partagés, contrats d'API, données) et la parade prévue ; `/implement` les vérifie avant et après modification, et signale tout effet hors plan au lieu de l'absorber.
- **Recette en conditions réelles** : `/recette` rejoue les critères d'acceptation comme un utilisateur (UI d'abord, scripts en dernier recours), uniquement sur environnement local, et ne laisse aucune donnée de test derrière elle.

## Emplacements des fichiers

| Dossier | Contenu | À versionner |
| :------ | :------ | :----------- |
| `.claude/specs/` | Specs (`<slug>.md`) | oui |
| `.claude/plans/` | Plans en cours, supprimés par `/review-workflow` | selon l'équipe |
| `.claude/front/` | Contrats d'intégration frontend | oui |
| `.claude/refactoring.md` | Refactorings reportés lors des plans | oui |
| `.claude/test/` | Recettes, captures, scripts ; **`comptes-test` contient des secrets** | recettes et scripts oui, `comptes-test` **non** (à ignorer dans git) |

Projet créé avec une version antérieure (`_specs/`, `_plans/`, `_front/` à la racine) : chaque skill détecte les anciens dossiers et propose de les déplacer avec `git mv`.

## Installation

```shell
/plugin install spec-driven@dev-workflow
```
