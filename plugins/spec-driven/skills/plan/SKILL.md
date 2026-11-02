---
name: plan
description: >-
  Transforme une ou plusieurs fonctionnalités d'une spec (`.claude/specs/`) en plan
  d'implémentation technique détaillé dans `.claude/plans/`, directement exploitable par
  `/implement` sans réanalyser la spec. Décrit ce qui doit exister (modèles,
  flux, validations, tests) sans écrire de code, en s'ancrant dans la stack et
  les conventions réelles du dépôt, et en repérant le refactoring/la
  simplification nécessaires à une insertion propre (arbitrés avec l'utilisateur
  quand ils débordent de la spec). À invoquer uniquement via `/plan`, avec les
  identifiants de spec à planifier (1, 1-1, 2-1…). S'exécute en mode plan.
argument-hint: <spec> <ids à planifier> (ou juste <ids> si la spec est déjà dans le contexte)
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
- **Rester strictement dans le périmètre fonctionnel demandé** : ni plus, ni
  moins. L'*analyse* de l'existant, elle, regarde au-delà de ce périmètre (voir
  « Refactoring et simplification »).
- Hérite du modèle d'agent courant.
- **Emplacements** : specs dans `.claude/specs/`, plans dans `.claude/plans/`. Si
  `_specs/` ou `_plans/` existent encore à la racine alors que le dossier
  correspondant sous `.claude/` n'existe pas, le signaler et proposer de les
  déplacer (`git mv`, avec confirmation) avant de poursuivre.

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
- **S'insérer, pas se superposer.** La fonctionnalité s'ajoute à du code qui
  vit : le plan doit dire où et comment elle s'y loge, et si le code autour doit
  être préparé ou simplifié pour que l'insertion soit propre.
- **Aucun effet de bord non voulu.** Tout élément existant que le plan modifie
  peut être utilisé ailleurs : repérer ces usages et dire comment le changement
  restera sans conséquence hors du périmètre (voir « Effets de bord »).

## Processus d'analyse

1. **Comprendre le besoin** — relire les fonctionnalités/tâches de spec ciblées,
   leurs règles métier et leurs critères d'acceptation.
2. **Cartographier l'existant** — inspecter le code : architecture, modules,
   conventions, mécanismes de persistance/test/communication, et éléments partagés
   (types, middlewares, classes d'erreur, utilitaires…) à réutiliser plutôt
   qu'à recréer. Ne jamais supposer qu'un élément existe sans vérifier.
3. **Identifier les concepts métier** — pour chacun : existe / à créer / à
   modifier.
4. **Chercher refactoring et simplification** — en s'appuyant sur la
   cartographie, examiner le code que la fonctionnalité va toucher, ses
   voisins, et plus largement tout ce qui partage ses responsabilités (voir
   « Refactoring et simplification »). Arbitrer, avec l'utilisateur si besoin,
   ce qui entre dans le plan.
5. **Définir les flux** — nominal, cas d'erreur, cas limites, validations.
6. **Préparer l'implémentation** — modèle de données, logique applicative,
   interfaces, persistance, intégrations externes, traitements asynchrones.
7. **Mesurer les effets de bord** — pour chaque élément existant modifié ou
   supprimé, retrouver ses utilisateurs dans le dépôt et évaluer l'impact (voir
   « Effets de bord »).
8. **Découper et ordonner** — étapes cohérentes, vérifiables, à valeur
   fonctionnelle, limitant les régressions. Les étapes de refactoring passent
   **avant** celles de la fonctionnalité qu'elles préparent.

## Refactoring et simplification

Objectif : que le code livré soit mieux construit qu'avant, pas seulement
augmenté. Ajouter une fonctionnalité sur une base qui la gêne produit des
contournements et de la dette ; mieux vaut le voir au moment de planifier.

**Où regarder** — sans se limiter aux fichiers listés par la spec : le code à
modifier, les modules qui l'appellent ou qu'il appelle, et les autres endroits du
dépôt qui font déjà la même chose ou presque.

**Quoi chercher :**
- duplication que la nouvelle fonctionnalité aggraverait (ou qu'elle pourrait
  absorber en factorisant) ;
- logique trop longue, imbriquée ou mêlant plusieurs responsabilités, là où il
  faut en ajouter ;
- abstraction, paramètre ou branche devenus inutiles (code mort, cas jamais
  atteint, couche de passage sans valeur) ;
- patterns incohérents entre eux pour un même besoin, alors que la
  fonctionnalité devra choisir lequel suivre ;
- point d'insertion naturel absent : la fonctionnalité ne trouve pas où se loger
  sans contourner l'existant.

**Cadre.** Tout refactoring proposé préserve le comportement observable, a une
justification liée à la fonctionnalité ou à la lisibilité du code touché, et
reste vérifiable par des tests (existants, ou à ajouter avant de modifier).
Ne pas rechercher la perfection du dépôt : seules les améliorations dont le gain
est net comptent.

**Arbitrage par taille** — classer chaque constat :

| Constat | Traitement |
| :------ | :--------- |
| **Petit et lié** : dans les fichiers touchés, sert directement l'insertion ou la simplification de ce qui est modifié | Intégré au plan (identifiant `R1`, `R2`…), étapes dédiées en tête de l'ordre d'implémentation |
| **Gros ou hors périmètre** : étendu sur plusieurs modules, change des structures partagées, ou sans lien direct avec la spec | **Ne pas l'intégrer d'office.** En informer l'utilisateur (nature, fichiers concernés, bénéfice, effort estimé, risque) et lui demander son avis : le faire **maintenant** dans ce plan, **avant** dans un plan dédié qui précède celui-ci, ou **plus tard** |

Le plan ne contient que ce que l'utilisateur a validé. Les constats reportés sont
listés dans la section « Refactoring différé » du plan, puis — une fois le plan
accepté — ajoutés à `.claude/refactoring.md` (créé au besoin : constat, fichiers,
bénéfice, spec d'origine, date) pour survivre à la suppression du plan par
`/review-workflow`. Si rien ne mérite refactoring, l'écrire explicitement.

## Effets de bord

Modifier du code partagé change aussi ce qui en dépend, parfois très loin de la
fonctionnalité. Mieux vaut le voir au moment de planifier que le découvrir en
recette.

**Pour chaque élément existant modifié ou supprimé** (fonction, composant, style,
route, modèle, colonne, événement, configuration…), **chercher réellement** ses
utilisateurs dans le dépôt (références, imports, appels, sélecteurs, abonnements)
sans se fier à sa seule localisation. Ne pas supposer qu'un élément n'est utilisé
qu'ici.

**Où les effets se cachent :**
- appelants et composants/pages qui partagent la fonction, le composant ou les
  styles modifiés (une classe SCSS commune change toutes les pages qui l'emploient) ;
- contrats exposés : champs d'API, formats de réponse, événements, messages — et
  leurs consommateurs (front, autres services, jobs) ;
- données : schéma, valeurs par défaut, enregistrements existants, migrations,
  requêtes qui supposent l'ancien état ;
- règles transverses : autorisations, validations, caches, tris/filtres,
  traductions, configuration, comportements par défaut ;
- tests existants qui encodent l'ancien comportement : un test qui casse signale
  un effet de bord à examiner, pas à « corriger » par réflexe.

**Traitement** — pour chaque effet identifié, le plan indique : l'élément
touché, ce qui en dépend, le risque, et la parade (changement additif qui conserve
l'ancien comportement par défaut, adaptation explicite des utilisateurs concernés,
étape de migration) avec le test ou la vérification qui la prouve. Un effet de
bord voulu par la spec est documenté comme tel ; un effet qui change un
comportement hors périmètre est soumis à l'utilisateur avant d'entrer dans le
plan.

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

## Gabarit de sortie (`.claude/plans/<slug>.md`)

> Le plan doit être enregistré dans ce fichier.

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

## Insertion dans l'existant
- **Points d'insertion** : <où la fonctionnalité se branche, modules et patterns suivis>
- **Éléments réutilisés** : <ce qui existe et sera réutilisé tel quel>

## Refactoring et simplification
> Comportement observable inchangé. Uniquement ce que l'utilisateur a validé.
- **R1** — <constat> → <ce qui change, en termes de responsabilités> · sert : <ids spec> · non-régression : <tests existants ou à ajouter avant>
- **R2** — <…>
- **Refactoring différé** : <constats gros ou hors périmètre non retenus, avec la décision de l'utilisateur — ou « aucun »>
- Si rien à signaler : « Aucun refactoring nécessaire ».

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

## Effets de bord
> Un par élément existant modifié ou supprimé et utilisé ailleurs. Si aucun, l'écrire explicitement.
- **<élément>** — utilisé par : <appelants, pages, consommateurs> · risque : <changement non voulu> · parade : <approche + test/vérification>

## Dépendances
- <internes / externes>

## Ordre d'implémentation
> Les étapes de refactoring (R1, R2…) d'abord, chacune suivie d'une vérification
> que les tests passent.
1. <étape cohérente et vérifiable>
2. …

## Stratégie de tests
- **Scénarios fonctionnels** (couvrant les critères d'acceptation) :
- **Cas d'erreur** :
- **Cas limites** :
- **Régressions à surveiller** : <inclure les comportements couverts par les refactorings R1, R2… et les utilisateurs listés dans « Effets de bord »>

## Hypothèses
- <décisions prises hors spec, explicitées comme telles>

## Questions ouvertes
- [ ] <ambiguïté à lever avec l'utilisateur>
```
