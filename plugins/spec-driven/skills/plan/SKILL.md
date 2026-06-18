---
name: plan
description: >-
  Transforme une ou plusieurs fonctionnalités d'une spec (`_specs/`) en plan
  d'implémentation technique détaillé dans `_plans/`, directement exploitable par
  `/implement` sans réanalyser la spec. Décrit ce qui doit exister (modèles,
  flux, validations, tests) sans écrire de code, en s'ancrant dans la stack et
  les conventions réelles du dépôt. À invoquer uniquement via `/plan`, avec les
  identifiants de spec à planifier (1, 1-1, 2-1…). S'exécute en mode plan.
disable-model-invocation: true
---

# /plan — Plan d'implémentation technique

## Rôle dans le cycle spec-driven

Étape intermédiaire de **spec → plan → implémentation**. Traduit le besoin
fonctionnel (la spec) en **référence technique** que `/implement` exécute sans
avoir à réinterpréter la spec.

## Invocation

- Déclenchement **uniquement** via `/plan`, en **mode plan** de Claude Code.
- Argument : les identifiants de spec à couvrir (`1`, `1-1`, `2-1`…). Une spec
  peut donner lieu à plusieurs plans ciblés.
- **Rester strictement dans le périmètre demandé** : ni plus, ni moins.
- Hérite du modèle d'agent courant.

## Principes

- **Décrire, ne pas coder.** Le plan dit ce qui doit exister (responsabilités,
  contrats, données, flux) sans fournir l'implémentation : aucune classe,
  fonction, requête ou migration rédigée.
- **S'ancrer dans le dépôt réel.** Déduire la stack, l'architecture et les
  conventions du **code existant** et du fichier de conventions (`CLAUDE.md` ou
  `AGENTS.md` selon ce qui est présent). Ne jamais présumer une techno :
  réutiliser les patterns en place.
- **Tracer.** Chaque élément du plan référence l'identifiant de spec qu'il
  couvre (`1-1`, `2-1`…), pour garder la traçabilité jusqu'au code.
- **Ne pas redéfinir le périmètre** fixé par la spec.

## Processus d'analyse

1. **Comprendre le besoin** — relire les fonctionnalités/tâches de spec ciblées,
   leurs règles métier et leurs critères d'acceptation.
2. **Cartographier l'existant** — inspecter le code : architecture, modules,
   conventions, mécanismes de persistance/test/communication. Ne jamais supposer
   qu'un élément existe sans vérifier.
3. **Identifier les concepts métier** — pour chacun : existe / à créer / à
   modifier.
4. **Définir les flux** — nominal, cas d'erreur, cas limites, validations.
5. **Préparer l'implémentation** — modèle de données, logique applicative,
   interfaces, persistance, intégrations externes, traitements asynchrones.
6. **Découper et ordonner** — étapes cohérentes, vérifiables, à valeur
   fonctionnelle, limitant les régressions.

## Décisions et incertitudes

- Toute techno ou dépendance **nouvelle** est proposée à l'utilisateur, jamais
  imposée.
- Toute ambiguïté va dans **Questions ouvertes** ; ne jamais inventer un
  comportement métier.
- Toute décision absente de la spec est explicitée dans **Hypothèses**.

## Critère de qualité

Le plan doit être complet, cohérent, non ambigu, fidèle à la spec, ancré dans la
stack du dépôt, et assez détaillé pour qu'`/implement` n'ait **pas** à réanalyser
la spec.

## Gabarit de sortie (`_plans/<slug>.md`)

```markdown
# Plan — <Titre>  (spec : <ids couverts, ex. 1-1, 1-2>)

## Résumé
<ce que ce plan réalise, en quelques lignes>

## Périmètre (spec couverte)
- **1-1** — <rappel court de la tâche de spec>
- **1-2** — <…>

## Analyse fonctionnelle
- **Objectif** :
- **Acteurs** :
- **Comportements attendus** :
- **Critères d'acceptation** (repris de la spec) :

## Concepts métier
- **Existants** :
- **À créer** :
- **À modifier** :

## Flux
- **Nominal** : <étapes>
- **Cas d'erreur** :
- **Cas limites** :

## Éléments à implémenter
> Décrire responsabilités et contrats, jamais le code.
- **Modèle de données / persistance** :
- **Logique applicative** :
- **Interfaces (UI / API)** :
- **Intégrations externes** :

## Validations
- <liste exhaustive des règles à valider>

## Impacts
- **Nouveaux éléments** :
- **Éléments modifiés** :
- **Éléments supprimés** :

## Dépendances
- <internes / externes>

## Ordre d'implémentation
1. <étape cohérente et vérifiable>
2. …

## Stratégie de tests
- **Scénarios fonctionnels** (couvrant les critères d'acceptation) :
- **Cas d'erreur** :
- **Cas limites** :
- **Régressions à surveiller** :

## Hypothèses
- <décisions prises hors spec, explicitées comme telles>

## Questions ouvertes
- [ ] <ambiguïté à lever avec l'utilisateur>
```
