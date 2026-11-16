---
title: "01 - Préparer la tâche commune"
description: "Ajouter un test qui échoue, sur la branche de départ des deux harness"
---

Pour comparer deux harness, il faut leur donner exactement la même tâche, sur le même point de départ. La tâche sera de faire passer un test qui échoue. Cette étape est un **support** : vous ajoutez ce test sur la branche `agent-config`, d'où partiront les deux essais.

## 01.1 - Ajouter le test countRemaining

Le test vérifie une méthode qui n'existe pas encore, `countRemaining()`, qui doit compter les tâches non terminées. On se place sur la branche `agent-config`, puis on écrit le fichier de test :

```exec
cd ~/byte-layers/ai/todo-api && git switch agent-config && cat > test/count.test.js <<'FIN'
import { test } from 'node:test';
import assert from 'node:assert/strict';
import { createStore } from '../src/todos.js';

test('countRemaining counts the todos that are not done', () => {
  const store = createStore();
  assert.equal(store.countRemaining(), 0);
  const first = store.add('Lire');
  store.add('Écrire');
  store.complete(first.id);
  assert.equal(store.countRemaining(), 1);
});
FIN
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `cd ~/byte-layers/ai/todo-api && git switch agent-config` | se place dans le projet, sur la branche de configuration ; `&&` n'enchaîne que si l'étape précédente a réussi |
> | `cat > test/count.test.js <<'FIN'` … `FIN` | écrit les lignes entre les deux `FIN` dans le fichier de test |

Lancez les tests. Le nouveau test échoue, puisque la méthode n'existe pas :

```exec
cd ~/byte-layers/ai/todo-api && npm test
```

La fin de l'affichage compte un échec, avec la cause :

```text
✖ countRemaining counts the todos that are not done
  TypeError: store.countRemaining is not a function
ℹ tests 8
ℹ pass 7
ℹ fail 1
```

Commitez le test, pour que les deux branches d'essai partent de ce même état :

```exec
cd ~/byte-layers/ai/todo-api && git add test/count.test.js && git commit -m "Ajoute le test de countRemaining"
```

```verify
D=~/byte-layers/ai/todo-api
cd "$D" 2>/dev/null || { echo "Le dossier $D n'existe pas : faites d'abord le lab Une configuration d'agent portable d'abord."; exit 1; }
if ! git show agent-config:test/count.test.js >/dev/null 2>&1; then
  echo "Le test test/count.test.js n'est pas commité sur la branche agent-config. Lancez les commandes de cette section."
  exit 1
fi
if git show agent-config:src/todos.js | grep -q countRemaining; then
  echo "La branche agent-config contient déjà countRemaining : elle doit rester le point de départ commun, sans la solution."
  exit 1
fi
```
