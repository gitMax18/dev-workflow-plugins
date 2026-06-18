# dev-workflow — Marketplace de plugins Claude Code

Marketplace de plugins [Claude Code](https://code.claude.com/docs/en/plugins) pour le workflow de développement : un cycle **spec-driven** complet et la génération de **commits conventionnels**.

## Installation

Ajoutez la marketplace, puis installez les plugins voulus :

```shell
/plugin marketplace add maxime/dev-workflow-plugins
/plugin install spec-driven@dev-workflow
/plugin install conventional-commit@dev-workflow
```

> Remplacez `maxime/dev-workflow-plugins` par votre `owner/repo` GitHub réel.

Pour mettre à jour la marketplace après une nouvelle version :

```shell
/plugin marketplace update dev-workflow
```

## Plugins disponibles

| Plugin | Description | Commandes |
| :----- | :---------- | :-------- |
| [`spec-driven`](plugins/spec-driven) | Cycle spec → plan → implement | `/spec-driven:spec`, `/spec-driven:plan`, `/spec-driven:implement` |
| [`conventional-commit`](plugins/conventional-commit) | Messages de commit Conventional Commits | `/conventional-commit:conventional-commit` |

> Les skills de plugin sont **namespacés** par le nom du plugin pour éviter les collisions.

## Développement local

Tester un plugin sans l'installer depuis la marketplace :

```shell
claude --plugin-dir ./plugins/spec-driven
```

Valider la structure et les manifests :

```shell
claude plugin validate .
```

Après modification d'un plugin chargé, recharger sans redémarrer :

```shell
/reload-plugins
```

## Structure

```
.
├── .claude-plugin/
│   └── marketplace.json          # Catalogue de la marketplace
└── plugins/
    ├── spec-driven/
    │   ├── .claude-plugin/plugin.json
    │   └── skills/{spec,plan,implement}/SKILL.md
    └── conventional-commit/
        ├── .claude-plugin/plugin.json
        └── skills/conventional-commit/SKILL.md
```

## Versionnage

Chaque plugin est épinglé via le champ `version` de son `plugin.json` (et du catalogue). Les utilisateurs ne reçoivent une mise à jour que lorsque cette valeur change. Suivez [SemVer](https://semver.org/lang/fr/).

## Licence

[MIT](LICENSE)
