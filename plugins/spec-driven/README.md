# spec-driven

Cycle de développement **spec-driven** pour Claude Code : du besoin fonctionnel au code, en quatre étapes traçables.

```
/spec      →     /plan      →     /implement     →     /review-workflow
(le QUOI)      (le COMMENT)      (le code)              (la clôture)
_specs/          _plans/          code + tests         specs à jour, plans supprimés
                                       ↓
                              /front-contract
                          (contrat d'intégration)
                               _front/
```

## Commandes

| Commande | Rôle |
| :------- | :--- |
| `/spec-driven:spec <fonctionnalité>` | Crée/maintient une spec fonctionnelle versionnée dans `_specs/`. Décrit le besoin, jamais le code. Découpe en tâches à identifiants stables (`1`, `1-1`…). |
| `/spec-driven:plan <ids>` | Traduit une ou plusieurs tâches de spec en plan technique détaillé dans `_plans/`. Décrit ce qui doit exister, sans écrire de code. S'exécute en mode plan. |
| `/spec-driven:implement <ids\|plan>` | Réalise le code d'après le plan, écrit les tests des critères d'acceptation, puis met à jour les cases de la spec. |
| `/spec-driven:review-workflow [plans…]` | Clôture le cycle : valide et coche les tâches réalisées, génère un résumé d'implémentation par section de spec, extrait les éléments transversaux vers `_specs/common.md`, puis supprime les plans après confirmation. Sans argument, traite tous les `_plans/*.md`. |
| `/spec-driven:front-contract <spec> [ids…]` | Génère un contrat d'intégration frontend dans `_front/` à partir d'une spec et du code backend : endpoints, formats requête/réponse, auth, erreurs et règles métier. |

## Principes

- **Source de vérité fonctionnelle** : la spec décrit le QUOI et le POURQUOI ; le plan le COMMENT ; l'implémentation le code.
- **Traçabilité** : les identifiants de spec (`1-1`, `2-1`…) relient spec → plan → code, sans réinterprétation.
- **Documents vivants** : l'état d'avancement se lit dans les cases à cocher de la spec ; `/review-workflow` les consolide en fin de cycle.
- **Ancrage dans le dépôt** : stack et conventions déduites du code existant et de `CLAUDE.md`/`AGENTS.md`.
- **Éléments transversaux** : tout développement partagé entre fonctionnalités (erreurs communes, utilitaires, types…) est documenté dans `_specs/common.md`.

## Installation

```shell
/plugin install spec-driven@dev-workflow
```
