# spec-driven

Cycle de développement **spec-driven** pour Claude Code : du besoin fonctionnel au code, en trois étapes traçables.

```
/spec      →     /plan      →     /implement
(le QUOI)      (le COMMENT)      (le code)
_specs/          _plans/          code + tests
```

## Commandes

| Commande | Rôle |
| :------- | :--- |
| `/spec-driven:spec <fonctionnalité>` | Crée/maintient une spec fonctionnelle versionnée dans `_specs/`. Décrit le besoin, jamais le code. Découpe en tâches à identifiants stables (`1`, `1-1`…). |
| `/spec-driven:plan <ids>` | Traduit une ou plusieurs tâches de spec en plan technique détaillé dans `_plans/`. Décrit ce qui doit exister, sans écrire de code. S'exécute en mode plan. |
| `/spec-driven:implement <ids\|plan>` | Réalise le code d'après le plan, écrit les tests des critères d'acceptation, puis met à jour les cases de la spec. |

## Principes

- **Source de vérité fonctionnelle** : la spec décrit le QUOI et le POURQUOI ; le plan le COMMENT ; l'implémentation le code.
- **Traçabilité** : les identifiants de spec (`1-1`, `2-1`…) relient spec → plan → code, sans réinterprétation.
- **Documents vivants** : l'état d'avancement se lit dans les cases à cocher de la spec.
- **Ancrage dans le dépôt** : stack et conventions déduites du code existant et de `CLAUDE.md`/`AGENTS.md`.

## Installation

```shell
/plugin install spec-driven@dev-workflow
```
