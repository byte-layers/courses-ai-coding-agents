---
title: "02 - Premier essai : DeepSeek Harness"
description: "Sur une branche try-dsh, confier la tâche à dsh avec le modèle local, puis vérifier et commiter"
---

Le premier essai utilise dsh, configuré dans le lab de préparation du poste pour parler à Ollama. Chaque essai travaille sur sa propre branche : on pourra comparer les deux résultats sans qu'ils se mélangent.

## 02.1 - Créer la branche et lancer dsh

Créez la branche `try-dsh` à partir de `agent-config`, puis lancez l'interface web de dsh **depuis le projet**, pour que son espace de travail soit `todo-api` :

```exec
cd ~/byte-layers/ai/todo-api && git switch -c try-dsh agent-config && OLLAMA_API_KEY=ollama dsh web
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `git switch -c try-dsh agent-config` | crée la branche `try-dsh` à partir de `agent-config` et s'y place |
> | `OLLAMA_API_KEY=ollama dsh web` | lance l'interface web de dsh avec la clé quelconque qu'il réclame, comme dans le lab de préparation |

Dans le navigateur, vérifiez que le modèle est `qwen3.5:4b` et que l'espace de travail est `todo-api` (sinon, cliquez sur le nom de l'espace de travail et ouvrez `byte-layers/ai/todo-api`).

## 02.2 - Confier la tâche

Envoyez cette demande à l'agent, mot pour mot, pour que les deux harness reçoivent la même :

```text
Le test test/count.test.js échoue. Ajoute la méthode countRemaining() au store de src/todos.js pour qu'il passe, puis lance npm test.
```

L'agent lit les fichiers, modifie `src/todos.js`, puis lance les tests. Notez pendant qu'il travaille : le temps total, le nombre d'actions, et s'il vous demande une autorisation. Si dsh vous demande d'autoriser une modification ou une commande, acceptez.

> ⚠️ **Piège : l'agent tourne longtemps sans rien produire, ou s'arrête sur une erreur.**
> - *Symptôme :* au bout de dix minutes, `src/todos.js` n'a pas changé, ou l'agent affiche une erreur `500`.
> - *Cause :* sans carte graphique ni puce Apple, un modèle de 4 milliards de paramètres traite lentement les milliers de tokens de consignes qu'un harness envoie à chaque tour ; une erreur `500` signale en général un manque de mémoire.
> - *Correctif :* fermez les applications gourmandes et relancez la demande une fois. Si l'échec revient, écrivez la méthode vous-même avec la solution ci-dessous : le lab continue, et la comparaison portera sur le comportement de chaque harness plutôt que sur leur code.

````solution
Dans `src/todos.js`, ajoutez cette fonction avant `return { add, list, complete, remove };`, puis ajoutez `countRemaining` à l'objet renvoyé :

```javascript
  function countRemaining() {
    return list().filter((todo) => !todo.done).length;
  }
```
````

Quand l'agent a fini, arrêtez dsh (`Ctrl+C` dans le terminal) et lancez les tests vous-même. Ne croyez pas l'agent sur parole :

```exec
cd ~/byte-layers/ai/todo-api && npm test
```

Les huit tests passent :

```text
ℹ tests 8
ℹ pass 8
ℹ fail 0
```

Commitez le résultat sur la branche `try-dsh` :

```exec
cd ~/byte-layers/ai/todo-api && git add src/todos.js && git commit -m "countRemaining, écrit avec dsh"
```

La vérification contrôle que dsh parle bien à Ollama, puis rejoue les tests de la branche `try-dsh` dans un dossier temporaire :

```verify
D=~/byte-layers/ai/todo-api
if ! grep -q ':11434' ~/.dsh/profiles/web/cordis.patch.yml 2>/dev/null; then
  echo "La configuration de dsh ne désigne pas Ollama (port 11434) : reprenez l'étape 06 du lab Préparer son poste."
  exit 1
fi
cd "$D" 2>/dev/null || { echo "Le dossier $D n'existe pas."; exit 1; }
git rev-parse --verify -q try-dsh >/dev/null || { echo "La branche try-dsh n'existe pas. Lancez la commande git switch de cette section."; exit 1; }
git show try-dsh:src/todos.js | grep -q countRemaining || { echo "La méthode countRemaining n'est pas commitée sur try-dsh. Commitez src/todos.js."; exit 1; }
T=$(mktemp -d)
git archive try-dsh | tar -x -C "$T"
if ! (cd "$T" && npm test >/dev/null 2>&1); then
  rm -rf "$T"
  echo "Les tests échouent sur la branche try-dsh. Lancez npm test, corrigez src/todos.js, puis commitez."
  exit 1
fi
rm -rf "$T"
```
