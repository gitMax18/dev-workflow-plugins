# versionning

Génère des messages de commit conformes à la spécification [Conventional Commits](https://www.conventionalcommits.org/fr/) à partir des fichiers **déjà stagés**.

## Commande

```shell
/versionning:conventional-commit
```

Ce skill est aussi **invocable automatiquement** : il se déclenche dès que vous demandez à Claude de « faire un commit » ou de « committer ».

## Comportement

1. Analyse `git diff --staged` (périmètre strict : aucun `git add` automatique).
2. Choisit le `type` (`feat`, `fix`, `docs`, `refactor`, …) et un `scope` optionnel.
3. Détecte les *breaking changes* (`!` + footer `BREAKING CHANGE:`).
4. Rédige un titre impératif ≤ 72 caractères, corps optionnel.
5. Crée le commit.

Si rien n'est stagé, il s'arrête et le signale.

## Installation

```shell
/plugin install versionning@dev-workflow
```
