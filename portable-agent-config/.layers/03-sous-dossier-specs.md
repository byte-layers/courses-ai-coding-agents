---
title: "03 - Une consigne de sous-dossier et un dossier de spécifications"
description: "Un AGENTS.md dans test/ pour les conventions de test, et docs/specs/ pour les spécifications"
---

Certaines consignes ne concernent qu'une partie du projet : la façon d'écrire un test n'intéresse pas l'agent quand il modifie le serveur. Les mettre dans le fichier racine les chargerait à chaque demande. On les range au plus près des fichiers qu'elles concernent.

## 03.1 - Écrire test/AGENTS.md vous-même

Le format `AGENTS.md` prévoit des fichiers **imbriqués** : un `AGENTS.md` dans un sous-dossier vaut pour les fichiers de ce dossier, et quand deux consignes se contredisent, **le fichier le plus proche du fichier modifié l'emporte**. Le moment où l'outil le lit varie : Kilo, par exemple, charge l'`AGENTS.md` d'un sous-dossier quand il lit un fichier de ce dossier, pas au démarrage ; Claude Code fait de même avec ses propres fichiers `CLAUDE.md` de sous-dossier. C'est la brique « règles ciblées », dans sa forme portable.

Cette fois, écrivez le fichier vous-même. Créez `test/AGENTS.md` dans votre éditeur, avec un titre et trois consignes sur les tests du projet. Inspirez-vous de `test/todos.test.js` pour les trouver.

**Critères :**

- le fichier `test/AGENTS.md` existe ;
- il dit d'écrire les tests avec le module `node:test` ;
- il dit d'utiliser les assertions strictes, `node:assert/strict` ;
- il donne la règle de nommage des fichiers : un fichier `<module>.test.js` par fichier de `src/`.

```hint
Ouvrez `test/todos.test.js` : ses deux premières lignes donnent les deux modules à citer, et son nom suit la règle de nommage.
```

````solution
```markdown
# Tests

- Écrivez les tests avec `node:test` (`import { test } from 'node:test'`).
- Utilisez les assertions strictes : `import assert from 'node:assert/strict'`.
- Un fichier `<module>.test.js` par fichier de `src/` : `src/todos.js` est testé par `test/todos.test.js`.
```
````

```verify
F=~/byte-layers/ai/todo-api/test/AGENTS.md
if [ ! -f "$F" ]; then
  echo "Le fichier $F n'existe pas. Créez-le dans le dossier test du projet."
  exit 1
fi
if ! grep -q 'node:test' "$F"; then
  echo "test/AGENTS.md ne cite pas le module node:test."
  exit 1
fi
if ! grep -q 'node:assert/strict' "$F"; then
  echo "test/AGENTS.md ne cite pas les assertions strictes, node:assert/strict."
  exit 1
fi
if ! grep -q '\.test\.js' "$F"; then
  echo "test/AGENTS.md ne donne pas la règle de nommage des fichiers de test (<module>.test.js)."
  exit 1
fi
```

## 03.2 - Préparer le dossier des spécifications

La rubrique **Spécifications** d'`AGENTS.md` envoie l'agent dans `docs/specs/`. Ce dossier n'existe pas encore : on le crée, avec un court `README.md` qui dit ce qu'on y range. Les spécifications elles-mêmes viendront avec le développement piloté par les spécifications ; l'agent saura déjà où les trouver.

```exec
cd ~/byte-layers/ai/todo-api && mkdir -p docs/specs && printf '# Spécifications\n\nUn dossier par fonctionnalité, avec requirements.md (les exigences, en syntaxe EARS, présentée dans la suite du lab), design.md et tasks.md.\n' > docs/specs/README.md
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `cd ~/byte-layers/ai/todo-api &&` | se place dans le projet, puis lance la suite si le `cd` a réussi |
> | `mkdir -p docs/specs` | crée le dossier `docs/specs` et son parent `docs` ; `-p` ne se plaint pas s'ils existent déjà |
> | `&&` | lance la suite seulement si la création a réussi |
> | `printf '…\n…' > docs/specs/README.md` | écrit le texte dans le fichier ; `\n` est un retour à la ligne |

> 📖 [Les AGENTS.md imbriqués (FAQ du format)](https://agents.md/) · [Kilo et les AGENTS.md de sous-dossier](https://kilo.ai/docs/customize/agents-md)

```verify
if [ ! -f ~/byte-layers/ai/todo-api/docs/specs/README.md ]; then
  echo "Le fichier docs/specs/README.md n'existe pas. Lancez la commande de cette section."
  exit 1
fi
```
