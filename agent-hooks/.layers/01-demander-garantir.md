---
title: "01 - Pourquoi une règle ask ne suffit pas"
description: "Voir ce qu'une règle ask laisse passer, ce qu'un hook ajoute, et créer la branche du lab"
---

Au lab précédent, `opencode.json` a reçu deux sortes de règles. La règle `"read": { "*.env": "deny" }` **garantit** une chose précise : l'outil de lecture d'OpenCode refuse le fichier `.env`. La règle `"*": "ask"`, elle, **demande** : toute autre commande du shell, `cat .env` compris, passe par votre accord. Cette seconde protection repose sur vous.

## 01.1 - Ce que la règle ask laisse passer

Une règle `ask` cède dans trois cas courants :

| Cas | Ce qui se passe |
|---|---|
| vous acceptez trop vite | après vingt demandes légitimes, la vingt et unième, `grep TOKEN .env`, passe sans être lue |
| la règle change | quelqu'un passe à `"*": "allow"` pour ne plus être dérangé, et toute commande passe sans question |
| la commande prend un détour | `node -e "console.log(require('fs').readFileSync('.env','utf8'))"` échappe à la règle `read`, et la demande `ask` qu'elle déclenche ne dit pas clairement qu'elle lit `.env` |

Un **hook** est un programme que le harness lance à un moment précis de la boucle de l'agent, par exemple juste avant chaque appel d'outil. Il reçoit l'appel complet : le nom de l'outil et ses arguments. S'il refuse, l'outil n'est pas exécuté, et le modèle reçoit le message du refus. Le hook ne dépend ni de votre vigilance, ni de la forme de la commande, tant que le texte de l'appel contient ce qu'il cherche.

| | Règle `deny` | Règle `ask` | Hook |
|---|---|---|---|
| Qui décide | le harness, d'après un motif | vous, à chaque demande | votre code, à chaque appel |
| Ce qu'il voit | un outil et un motif (`*.env`, `rm *`) | la commande, que vous lisez | tout l'appel : l'outil et tous ses arguments |
| Peut agir après l'outil | non | non | oui : vérifier, compléter le résultat |

Un hook se modifie ou se supprime comme une règle : c'est un fichier du projet, qu'on relit en revue de code comme le reste.

## 01.2 - Créer la branche du lab

Les hooks s'ajoutent à la configuration du lab précédent. Créez une branche `hooks` à partir de `agent-config` :

```exec
cd ~/byte-layers/ai/todo-api && git switch -c hooks agent-config
```

Git répond `Switched to a new branch 'hooks'`. Affichez le fichier `opencode.json` du lab précédent, pour avoir ses règles sous les yeux :

```exec
cd ~/byte-layers/ai/todo-api && cat opencode.json
```

Vous y retrouvez la règle `"*.env": "deny"` sous `read`, puis `"*": "ask"`, les deux exceptions de `npm test` et votre règle `"rm *": "deny"` sous `bash`.

```verify
D=~/byte-layers/ai/todo-api
cd "$D" 2>/dev/null || { echo "Le dossier $D n'existe pas : faites d'abord le lab Une configuration d'agent portable d'abord."; exit 1; }
if [ "$(git branch --show-current)" != "hooks" ]; then
  echo "Vous n'êtes pas sur la branche hooks. Lancez : git switch -c hooks agent-config (ou git switch hooks si elle existe déjà)."
  exit 1
fi
if [ ! -f opencode.json ]; then
  echo "opencode.json est absent de la branche : la branche agent-config du lab précédent doit le contenir."
  exit 1
fi
```
