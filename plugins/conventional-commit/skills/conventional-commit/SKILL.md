---
name: conventional-commit
description: Analyse les fichiers stagés et génère un message de commit qui suit la spécification Conventional Commits (type, scope, description, breaking change). Utiliser ce skill chaque fois que l'utilisateur demande de faire un commit, de "committer", de "git commit", ou de sauvegarder des changements dans git. Ne jamais faire un commit sans utiliser ce skill.
---

# Conventional Commit

Quand l'utilisateur demande de faire un commit, suis ce processus en entier sans sauter d'étape.

## 1. Analyser les changements stagés

```bash
git diff --staged
git status --short
```

**Périmètre strict : seuls les fichiers déjà stagés sont pris en compte.**
- Ne jamais ajouter de fichiers au staging toi-même (`git add`, `git add -A`, `git add .`, etc.) : c'est l'utilisateur qui décide de ce qui entre dans le commit.
- Si **aucun fichier n'est stagé**, ne commite pas : signale-le à l'utilisateur et arrête-toi là.
- Le commit ne portera donc que sur ce qui apparaît dans `git diff --staged`.

Lis attentivement le diff complet. Comprends :
- Quels fichiers ont changé et pourquoi
- Si c'est une nouvelle fonctionnalité, une correction, de la configuration, des tests, etc.
- Si un changement casse la compatibilité ascendante (breaking change)

## 2. Choisir le type

| Type | Utiliser quand |
|---|---|
| `feat` | Nouvelle fonctionnalité visible par l'utilisateur |
| `fix` | Correction d'un bug |
| `docs` | Uniquement de la documentation |
| `style` | Formatage, espaces, ponctuation — zéro impact logique |
| `refactor` | Restructuration sans ajout de feature ni correction de bug |
| `perf` | Amélioration mesurable de performance |
| `test` | Ajout ou correction de tests |
| `build` | Système de build, dépendances (npm, maven, gradle…) |
| `ci` | Fichiers et scripts de CI/CD |
| `chore` | Maintenance diverse sans impact sur le code source |
| `revert` | Annulation d'un commit précédent |

## 3. Choisir le scope (optionnel)

Le scope décrit la section du code impactée. Utilise le nom du module, composant ou domaine fonctionnel (ex : `auth`, `api`, `ui`, `db`, `user`). Omet-le si le changement est transversal ou si sa valeur ajoutée est faible.

## 4. Détecter un breaking change

Il y a un breaking change si le changement :
- Modifie une API publique de façon incompatible
- Supprime ou renomme une fonctionnalité existante
- Change un comportement dont des consommateurs dépendent

Dans ce cas : ajouter `!` après le type/scope ET un footer `BREAKING CHANGE: <explication>`.

## 5. Rédiger le message

Format :
```
<type>[(<scope>)][!]: <description>

[corps optionnel]

[BREAKING CHANGE: <description> si applicable]
```

Règles de rédaction :
- Description en minuscules, à l'impératif présent, sans point final ("add login page" et non "added" ni "adding")
- Ligne de titre ≤ 72 caractères
- Corps uniquement si le contexte n'est pas évident depuis le diff
- Pas de `Co-Authored-By` ni d'autres footers non demandés

## 6. Faire le commit

```bash
git commit -m "$(cat <<'EOF'
<message généré>
EOF
)"
```

Si le staging est vide, signale-le à l'utilisateur avant de continuer.
