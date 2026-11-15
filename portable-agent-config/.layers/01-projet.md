---
title: "01 - Récupérer le projet todo-api"
description: "Cloner le projet fil rouge, lancer ses tests et créer son fichier .env"
---

Le projet de ce lab est **todo-api**, une petite API de tâches en Node.js, sans aucune dépendance à installer. Cette étape est un **support** : elle prépare le projet sur lequel vous écrirez la configuration de l'agent.

## 01.1 - Cloner le projet et lancer ses tests

On récupère le projet dans le dossier de travail. L'option `--branch v1` prend la version du projet écrite pour ce lab, pour que tout le monde parte du même état :

```exec
git clone --branch v1 https://github.com/byte-layers/todo-api.git ~/byte-layers/ai/todo-api
```

Si Git répond `already exists and is not an empty directory`, le projet est déjà là, d'une tentative précédente : passez à la commande suivante.

Git signale `You are in 'detached HEAD' state` : c'est normal quand on clone une étiquette (*tag*) plutôt qu'une branche. On crée donc une branche de travail, `agent-config`, en se plaçant dans le projet :

```exec
cd ~/byte-layers/ai/todo-api && git switch -c agent-config
```

Lancez les tests du projet. Ils utilisent `node --test`, l'outil de test livré avec Node.js :

```exec
cd ~/byte-layers/ai/todo-api && npm test
```

La fin de l'affichage compte sept tests réussis :

```text
ℹ tests 7
ℹ pass 7
ℹ fail 0
```

Le projet est rangé en deux dossiers : `src/` pour le code (`todos.js` stocke les tâches, `server.js` expose l'API HTTP) et `test/` pour les tests.

```verify
D=~/byte-layers/ai/todo-api
if [ ! -f "$D/package.json" ]; then
  echo "Le projet n'est pas dans $D. Lancez la commande git clone de cette section."
  exit 1
fi
if ! (cd "$D" && npm test >/dev/null 2>&1); then
  echo "npm test échoue dans $D. Vérifiez que node -v affiche v22.9 ou plus récent."
  exit 1
fi
```

## 01.2 - Créer le fichier .env

Comme beaucoup de projets, todo-api lit sa configuration dans un fichier `.env` quand il existe (`npm start` le charge), un fichier qui contient des secrets et n'est jamais commité : `.gitignore` l'exclut. Le dépôt fournit un modèle, `.env.example`. Copiez-le en `.env`, depuis le dossier du projet :

```exec
cd ~/byte-layers/ai/todo-api && cp .env.example .env
```

Ce fichier contient une valeur sensible, `API_TOKEN`. Plus loin dans le lab, vous interdirez à l'agent de le lire.

```verify
if [ ! -f ~/byte-layers/ai/todo-api/.env ]; then
  echo "Le fichier ~/byte-layers/ai/todo-api/.env n'existe pas. Lancez la commande de cette section."
  exit 1
fi
```
