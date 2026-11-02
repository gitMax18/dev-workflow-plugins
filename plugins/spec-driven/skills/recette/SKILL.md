---
name: recette
description: >-
  Génère un plan de recette (`recette-<périmètre>.md`) à partir de la spec, du
  plan et du code livré par `/implement` : description du périmètre, liste des
  éléments à tester, puis scénarios (compte à utiliser, procédure, vérifications,
  contrôles visuels si le front a changé) jouables comme un vrai utilisateur via
  l'interface graphique. Enregistre le plan dans `.claude/test/<spec>/`, puis
  propose de l'exécuter avec Playwright, uniquement sur environnement local, avec
  un moyen de revenir à l'état de base. À invoquer uniquement via `/recette`,
  après `/implement` et avant `/review-workflow`, avec la spec (et optionnellement
  des identifiants de tâches) en argument.
argument-hint: <spec> [ids de tâches à tester] (optionnel, sinon toute la spec)
disable-model-invocation: true
---

# /recette — Plan de recette et exécution Playwright

## Rôle dans le cycle spec-driven

Étape intercalée entre **/implement** et **/review-workflow**. `/implement` prouve
que le code passe ses tests unitaires ; `/recette` vérifie que la fonctionnalité
marche **du point de vue d'un utilisateur réel**, dans l'application qui tourne.
Ses résultats (OK/KO par scénario) alimentent `/review-workflow`, qui ne doit pas
cocher une tâche dont un scénario est KO.

La recette part des critères d'acceptation de la spec : elle les rejoue en
conditions réelles, elle n'invente aucune règle métier.

## Invocation

- Déclenchement **uniquement** via `/recette`.
- Arguments :
  - **`<fichier-spec>`** — `.claude/specs/auth.md` (ou la spec dans le contexte de la
    conversation).
  - **`<identifiants>`** — tâches/fonctionnalités à couvrir (`1-1 1-2`, `2`).
    Sans identifiants : toute la spec, en ne retenant que les tâches `[x]` ou
    couvertes par un plan présent dans `.claude/plans/`.
- Si aucune spec n'est identifiable, la demander avant de poursuivre.
- **Emplacements** : specs dans `.claude/specs/`, plans dans `.claude/plans/`. Si
  `_specs/` ou `_plans/` existent encore à la racine alors que le dossier
  correspondant sous `.claude/` n'existe pas, le signaler et proposer de les
  déplacer (`git mv`, avec confirmation) avant de poursuivre.
- **Périmètre** (`<perimetre>` dans le nom de fichier) : le slug de la spec pour
  une recette complète, sinon les identifiants joints par `_` (`1-1_1-2`).

```
/recette .claude/specs/auth.md
/recette .claude/specs/auth.md 1-1 1-2
```

## Fichiers manipulés

```
.claude/test/
├── comptes-test                     # comptes de test + comment en créer (lu, jamais modifié)
├── script/                          # scripts de scénarios et de reset
│   └── <perimetre>-<but>.<ext>
└── <slug-spec>/
    ├── recette-<perimetre>.md       # le plan de recette (sortie principale)
    └── captures/                    # preuves Playwright
```

## Processus

### 1. Rassembler le contexte

Dans cet ordre de priorité :

1. La **spec** (périmètre retenu) : exigences, règles métier, cas limites,
   critères d'acceptation — c'est ce qu'on teste.
2. Le **plan** `.claude/plans/<slug>.md` s'il existe : section « Impacts » pour savoir
   quels écrans, endpoints, jobs ou styles ont été touchés.
3. Le **code livré** (`git diff`/`git log` de la branche, fichiers du plan) : pour
   les libellés réels de l'interface, les routes, les rôles et permissions.
4. Le **résumé d'implémentation** de la spec si `/review-workflow` est déjà passé.

Relever aussi : comment lancer l'application en local, quels services tiers elle
appelle, et si un `mise.local.toml` (ou `.mise.local.toml`) existe.

### 2. Lire `.claude/test/comptes-test`

- **Fichier absent : le signaler et s'arrêter.** Dire à l'utilisateur qu'il doit
  le créer (rôle, identifiant, mot de passe, et procédure de création de compte
  pour chaque rôle) ; ne rien générer à sa place — il contient des secrets.
- Fichier présent : identifier, pour chaque rôle dont la spec parle
  (anonyme, utilisateur, admin…), le compte à utiliser et la procédure de
  création si le scénario exige un compte neuf ou dans un état particulier.
- Rôle nécessaire mais absent du fichier, ou procédure de création manquante :
  poser la question à l'utilisateur, ne pas deviner.
- Ne **jamais** recopier un mot de passe dans la recette ni dans la conversation :
  référencer les comptes par leur libellé dans `comptes-test`. Si le fichier
  contient des secrets et n'est pas ignoré par git, le signaler.

### 3. Dégager les éléments à tester

Un **élément** est une unité vérifiable de comportement rattachée à une ou
plusieurs tâches de la spec (un écran, un parcours, une règle d'autorisation, un
état d'erreur, une notification, un job…). Les lister avec un identifiant
`E1`, `E2`… et le lien vers les tâches (`1-1`).

Penser à couvrir pour chaque élément : le nominal, chaque cas limite et erreur de
la spec, les **autorisations** (le rôle qui ne doit pas y accéder), les états
vides, et les effets de bord observables (e-mail envoyé, donnée visible ailleurs).

### 4. Écrire les scénarios

Un ou plusieurs scénarios par élément (`E1-S1`, `E1-S2`…), chacun **autonome** :
il précise son état de départ plutôt que de dépendre d'un scénario précédent.

**Priorité à l'interface graphique, comme une vraie personne** : se connecter
avec le compte, naviguer, cliquer, saisir, lire ce qui s'affiche. Ne pas
raccourcir par l'API ou la base pour la chose testée — sinon on ne teste plus ce
que l'utilisateur vit.

**Script uniquement quand l'UI ne peut pas le faire** (appel d'un webhook, tâche
planifiée, import par lot, lecture d'un e-mail sans interface, préparation ou
vérification d'un état en base). Règles pour les scripts :

- les placer dans `.claude/test/script/`, nommés `<perimetre>-<but>.<ext>`, dans le
  langage de la stack du projet, avec un en-tête qui dit ce qu'ils font ;
- les rendre **idempotents** et **refuser de s'exécuter hors environnement
  local** (vérifier l'hôte/la base visés avant toute action) ;
- les rendre lançables via une tâche **mise** déclarée dans `mise.local.toml`
  (`mise run recette:<nom>`). Si ce fichier n'existe pas, **demander à
  l'utilisateur comment les lancer** plutôt que d'en créer un ;
- ajouter une tâche à `mise.local.toml` seulement après l'avoir annoncé.

**Contrôles visuels** : si le périmètre touche du SCSS/CSS, des templates, des
composants ou tout fichier front (repérable via le plan ou le diff), chaque
scénario concerné contient aussi une vérification d'affichage : rendu conforme
aux maquettes/au design existant, états (survol, focus, désactivé, erreur,
chargement, vide), responsive (au moins mobile et desktop), absence de
régression sur les pages voisines qui partagent les styles modifiés, absence
d'erreur dans la console.

Chaque scénario suit ce gabarit :

```markdown
#### E1-S1 — <titre court>
- **Tâche(s)** : 1-1
- **Compte** : <libellé du compte dans comptes-test> (ou « à créer : <procédure> »)
- **Mode** : Interface graphique | Script (`.claude/test/script/<fichier>`)
- **Prérequis** : <état de départ nécessaire>
- **Procédure** :
  1. <action de l'utilisateur>
  2. <action de l'utilisateur>
- **Vérifications** :
  - [ ] <résultat observable attendu, tiré d'un critère d'acceptation>
  - [ ] <…>
- **Vérifications visuelles** : <… ou « sans objet (aucun fichier front touché) »>
- **Données créées/modifiées** : <ce que le scénario laisse derrière lui, pour le reset>
- **Statut** : À faire
- **Preuve / notes** :
```

Les vérifications sont **observables et précises** (« le message
“Mot de passe incorrect” s'affiche sous le champ », pas « l'erreur est gérée »).

### 5. Prévoir le retour à l'état de base

Agréger les « Données créées/modifiées » de tous les scénarios et générer
`.claude/test/script/reset-<perimetre>.<ext>` qui annule **exactement** ces
changements (suppression des comptes/enregistrements créés, restauration des
valeurs modifiées), en filtrant sur des marqueurs identifiables (préfixe
`recette-` sur les données créées) plutôt que sur des suppressions larges. Le
déclarer comme tâche mise `recette:reset-<perimetre>` (mêmes règles que ci-dessus)
et refuser de s'exécuter hors base locale.

Si une donnée modifiée ne peut pas être restaurée par script, le dire dans la
recette et proposer une alternative (dump local avant/après).

### 6. Enregistrer le plan

Écrire `.claude/test/<slug-spec>/recette-<perimetre>.md` selon le gabarit
ci-dessous. Si le fichier existe déjà, montrer les différences et demander
confirmation avant d'écraser ; conserver les identifiants (`E1`, `E1-S1`) et les
statuts des scénarios inchangés.

### 7. Proposer l'exécution automatique

Annoncer le chemin du fichier et un résumé (nombre d'éléments, de scénarios, dont
combien en script), puis demander : **« Lancer les tests automatiquement avec
Playwright ? »** Si non, s'arrêter là en rappelant que le plan peut être joué à
la main ou relancé plus tard.

## Exécution Playwright (si l'utilisateur accepte)

### Garde-fous d'environnement — avant toute action

L'exécution ne touche **que** des environnements locaux. Jamais de qualification,
préproduction ni production, même « juste pour lire ».

1. Relever l'URL de l'application, la base de données et les services tiers que
   les scénarios vont déclencher (config, `.env`, `docker-compose`, README).
2. L'URL et la base doivent être locales (`localhost`, `127.0.0.1`, `*.localhost`,
   conteneur Docker local). Sinon : **s'arrêter** et le dire.
3. Pour chaque service tiers (paiement, e-mail, SMS, stockage, OAuth, API
   externe…), vérifier qu'il s'agit d'un **service/bac à sable de test** (clés
   `test`, mode sandbox, mailcatcher, émulateur local). En cas de **doute**,
   demander confirmation explicite à l'utilisateur ; en cas de service réel
   certain, ne pas jouer les scénarios concernés (statut `Bloqué`).
4. Afficher un récapitulatif « Environnement vérifié » (URL, base, tiers avec
   leur nature) et obtenir l'accord de l'utilisateur avant de lancer.

### Identifiants inconnus

Si un compte requis n'a pas d'identifiant/mot de passe dans `comptes-test`, ne
pas en inventer et ne pas demander de les taper dans la conversation : demander
à l'utilisateur de **compléter le fichier** puis d'avertir quand c'est fait. Les
scénarios concernés restent `Bloqué` d'ici là.

### Lancement

- Vérifier que l'application locale tourne ; sinon la démarrer avec les moyens du
  projet (tâche mise, script `package.json`, docker compose) ou demander comment.
- Utiliser Playwright via les outils MCP Playwright s'ils sont disponibles, sinon
  via la CLI/le paquet du projet. Si Playwright est absent, le dire et demander
  l'accord avant d'ajouter la moindre dépendance.
- Vérifier que le script de reset existe avant de commencer (le générer sinon).
- Jouer les scénarios un par un, en se comportant comme l'utilisateur : repérer
  les éléments par rôle ARIA/libellé visible plutôt que par sélecteur interne.
  Lancer les scénarios « script » via leur tâche mise.
- Pour chaque scénario, cocher les vérifications obtenues, renseigner le
  **statut** (`OK`, `KO`, `Bloqué`), joindre une capture dans
  `.claude/test/<slug-spec>/captures/<id-scénario>.png` (obligatoire en cas de KO)
  et consigner dans « Preuve / notes » l'écart constaté (attendu vs observé).
- Ne jamais corriger le code pendant la recette : signaler les KO, c'est
  `/implement` qui corrige.

### Après les tests

1. Compléter le tableau de synthèse en tête de recette.
2. Présenter le bilan : OK / KO / Bloqués, avec pour chaque KO l'écart observé.
3. Proposer de **revenir à l'état de base** en lançant `mise run
   recette:reset-<perimetre>` (ou la commande convenue) ; ne l'exécuter qu'avec
   l'accord de l'utilisateur, puis vérifier que les données créées ont disparu.
4. Proposer de rejouer uniquement les scénarios KO/Bloqués après correction.
5. Rappeler la suite : `/review-workflow` (qui tiendra compte des KO).

## Gabarit de sortie (`recette-<perimetre>.md`)

```markdown
# Recette — <Titre de la spec> [— périmètre : <ids>]

> Spec : `.claude/specs/<slug>.md` · tâches : <liste ou "toutes">
> Généré le : <YYYY-MM-DD> · Environnement : local uniquement
> Reset : `mise run recette:reset-<perimetre>`

## Périmètre
<2 à 5 phrases : ce qui est testé, du point de vue utilisateur, et ce qui ne l'est pas.>

## Synthèse
| Élément | Scénarios | OK | KO | Bloqué | À faire |
| :------ | :-------- | :- | :- | :----- | :------ |
| E1 — <nom> | 2 | 0 | 0 | 0 | 2 |

## Éléments à tester
- **E1** — <élément> (tâches 1-1)
- **E2** — <élément> (tâches 1-2)

## Comptes utilisés
| Libellé (dans `comptes-test`) | Rôle | Utilisé par |
| :---------------------------- | :--- | :---------- |

## Scénarios

### E1 — <élément>
#### E1-S1 — <titre>
<gabarit de scénario>

## Données créées et retour à l'état de base
<liste agrégée des données laissées par les scénarios + commande de reset>
```

## Principes

- **Ancrage dans la spec.** Chaque vérification remonte à un critère
  d'acceptation, une règle métier ou un cas limite de la spec ; pas de
  vérification sans source.
- **Comme un utilisateur.** L'UI d'abord ; le script est l'exception justifiée.
- **Local uniquement.** Le doute se résout par une question, jamais par un essai.
- **Aucun secret écrit.** Ni dans la recette, ni dans les scripts, ni dans la
  conversation.
- **Pas de correction de code ici.** La recette constate, elle ne répare pas.
- **Identifiants stables.** `E1`, `E1-S1` ne sont ni renumérotés ni réutilisés
  lors d'une mise à jour de la recette.
- Rédiger en français, sauf les noms techniques (routes, champs, commandes).
