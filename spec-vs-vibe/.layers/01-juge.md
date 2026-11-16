---
title: "01 - Préparer le juge"
description: "Écrire les tests d'acceptation de l'export, hors des branches qu'ils vont départager"
---

Pour comparer deux versions d'une fonctionnalité, il faut un critère que ni l'une ni l'autre n'a choisi. Ce critère, ce sont des **tests d'acceptation** : des tests qui couvrent les exigences de `requirements.md`, écrits avant le code. Cette étape est un **support** : le fichier est fourni complet.

## 01.1 - Les tests d'acceptation

Les tests vivent dans un dossier à part, `~/byte-layers/ai/acceptance`, en dehors du projet : aucune des deux branches ne pourra les modifier, ni l'agent les adapter à son code. L'extension `.mjs` dit à Node que le fichier est un module JavaScript moderne (avec `import`), même sans `package.json` à côté.

```exec
cd ~ && mkdir -p ~/byte-layers/ai/acceptance && cat > ~/byte-layers/ai/acceptance/export.test.mjs <<'FIN'
// Tests d'acceptation de l'export, écrits d'après docs/specs/export/requirements.md.
// À lancer depuis la racine du projet : node --test ~/byte-layers/ai/acceptance/export.test.mjs
import { test } from 'node:test';
import assert from 'node:assert/strict';
import { join } from 'node:path';
import { pathToFileURL } from 'node:url';

const { createServer } = await import(pathToFileURL(join(process.cwd(), 'src/server.js')).href);

async function start(t) {
  const server = createServer().listen(0);
  t.after(() => server.close());
  return `http://localhost:${server.address().port}`;
}

async function addTodos(base) {
  for (const title of ['Lire la spécification', 'Écrire les tests']) {
    await fetch(`${base}/todos`, { method: 'POST', body: JSON.stringify({ title }) });
  }
  await fetch(`${base}/todos/1`, { method: 'PATCH' });
}

test('exigences 1 et 2 : GET /todos/export répond 200 avec un document JSON', async (t) => {
  const base = await start(t);
  const res = await fetch(`${base}/todos/export`);
  assert.equal(res.status, 200);
  assert.match(res.headers.get('content-type') ?? '', /^application\/json/);
  JSON.parse(await res.text());
});

test('exigences 3 et 4 : toutes les tâches, avec id, title et done', async (t) => {
  const base = await start(t);
  await addTodos(base);
  const body = await (await fetch(`${base}/todos/export`)).json();
  assert.ok(Array.isArray(body.todos), 'le document doit avoir un champ todos, une liste');
  assert.equal(body.todos.length, 2);
  for (const todo of body.todos) assert.deepEqual(Object.keys(todo).sort(), ['done', 'id', 'title']);
  assert.equal(body.todos.filter((todo) => todo.done).length, 1);
});

test('exigence 5 : exportedAt au format ISO 8601', async (t) => {
  const base = await start(t);
  const body = await (await fetch(`${base}/todos/export`)).json();
  assert.equal(typeof body.exportedAt, 'string');
  assert.equal(new Date(body.exportedAt).toISOString(), body.exportedAt);
});

test('exigence 6 : téléchargement sous le nom todos.json', async (t) => {
  const base = await start(t);
  const res = await fetch(`${base}/todos/export`);
  const disposition = res.headers.get('content-disposition') ?? '';
  assert.match(disposition, /^attachment/);
  assert.match(disposition, /filename="?todos\.json"?/);
});

test('exigence 7 : sans tâche, un export valide avec une liste vide', async (t) => {
  const base = await start(t);
  const res = await fetch(`${base}/todos/export`);
  assert.equal(res.status, 200);
  assert.deepEqual((await res.json()).todos, []);
});
FIN
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `cd ~ &&` | part de votre dossier personnel |
> | `mkdir -p ~/byte-layers/ai/acceptance` | crée le dossier des tests d'acceptation, hors du projet |
> | `&&` | n'enchaîne la suite que si la création a réussi |
> | `cat > …/export.test.js <<'FIN'` … `FIN` | écrit les lignes entre les deux `FIN` dans le fichier |

Le fichier se lit ainsi :

| Partie | Rôle |
|---|---|
| `await import(pathToFileURL(join(process.cwd(), 'src/server.js')).href)` | charge le serveur **du dossier courant** : lancé depuis le projet, le test juge la branche sur laquelle vous êtes |
| `start(t)` | démarre le serveur sur un port libre (`listen(0)`), et l'arrête à la fin du test |
| `addTodos(base)` | crée deux tâches et termine la première, par l'API elle-même |
| cinq `test(…)` | chaque nom cite les exigences qu'il vérifie, numérotées comme dans la solution du lab précédent : cinq tests couvrent les sept exigences |
| `new Date(x).toISOString() === x` | exige le format exact de `toISOString()`, celui que fixe `design.md` ; une date ISO 8601 avec un fuseau `+01:00` serait refusée |

## 01.2 - Voir le juge échouer

Lancez les tests depuis le projet, sur la branche `agent-config`, qui n'a pas encore d'export :

```exec
cd ~/byte-layers/ai/todo-api && git switch agent-config && node --test ~/byte-layers/ai/acceptance/export.test.mjs
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `cd ~/byte-layers/ai/todo-api &&` | se place dans le projet : le juge charge le `src/server.js` du dossier courant |
> | `git switch agent-config &&` | se place sur la branche sans export |
> | `node --test <fichier>` | lance les tests de ce fichier avec le lanceur intégré à Node |

Les cinq tests échouent, ce qui est attendu : la fin de l'affichage compte `pass 0` et `fail 5`. Un test d'acceptation qui passerait avant le code ne prouverait rien.

> 📖 [Le lanceur de tests de Node.js](https://nodejs.org/api/test.html) · [L'en-tête Content-Disposition (MDN)](https://developer.mozilla.org/fr/docs/Web/HTTP/Reference/Headers/Content-Disposition)

```verify
F=~/byte-layers/ai/acceptance/export.test.mjs
[ -f "$F" ] || { echo "Le fichier ~/byte-layers/ai/acceptance/export.test.mjs n'existe pas. Lancez la commande de la section 01.1."; exit 1; }
N=$(grep -c "^test('exigence" "$F")
[ "$N" -eq 5 ] || { echo "Le fichier contient $N tests au lieu de 5 : réécrivez-le avec la commande de la section 01.1."; exit 1; }
git -C ~/byte-layers/ai/todo-api rev-parse --verify -q spec/export >/dev/null || { echo "La branche spec/export n'existe pas : faites d'abord le lab De l'issue floue à la spécification."; exit 1; }
```
