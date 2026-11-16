---
title: "03 - L'essai avec la spécification"
description: "Faire réaliser les tâches de tasks.md par l'agent, sur la branche export-spec, et noter son score brut"
---

Second essai : le même agent, le même modèle, mais cette fois une demande qui renvoie à la spécification. L'agent n'a plus rien à choisir : `requirements.md` dit quoi, `design.md` comment, `tasks.md` dans quel ordre.

## 03.1 - Faire réaliser les tâches

Créez la branche `export-spec` à partir de `spec/export`, la branche qui contient la spécification :

```exec
cd ~/byte-layers/ai/todo-api && git switch -c export-spec spec/export
```

Si la branche existe déjà, lancez seulement `git switch export-spec`. Lancez ensuite l'agent, avec la même précision sur `test/count.test.js` qu'au premier essai :

```exec
cd ~/byte-layers/ai/todo-api && opencode run "Réalise les tâches de docs/specs/export/tasks.md, dans l'ordre, en respectant docs/specs/export/requirements.md et docs/specs/export/design.md. Lance npm test à la fin. Le test test/count.test.js échoue déjà : ignore-le et ne le modifie pas."
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `cd ~/byte-layers/ai/todo-api &&` | se place dans le projet, puis lance la suite si le `cd` a réussi |
> | `opencode run "…"` | donne la demande à OpenCode ; elle désigne les trois fichiers au lieu de décrire la fonctionnalité |

La demande est plus courte que la précédente, mais elle pointe vers trois fichiers qui disent tout. Si votre `tasks.md` commence par l'écriture des tests, regardez si l'agent respecte cet ordre. Comme au premier essai, interrompez-le avec `Ctrl+C` s'il tourne trop longtemps.

## 03.2 - Noter le score brut

Passez le travail de l'agent au juge, avant toute correction :

```exec
cd ~/byte-layers/ai/todo-api && node --test ~/byte-layers/ai/acceptance/export.test.mjs
```

**À vous** : ajoutez deux lignes à `~/byte-layers/ai/acceptance/scores.txt`, sur le même modèle qu'à l'étape précédente : `spec = …`, le nombre de tests passés, et `spec_auteur = …`, qui vaut `agent` si vous gardez le code de l'agent tel quel, `corrige` si vous le corrigez vous-même d'après la spécification, ou `secours` si vous appliquez le résultat de secours.

```verify
F=~/byte-layers/ai/acceptance/scores.txt
[ -f "$F" ] || { echo "Le fichier ~/byte-layers/ai/acceptance/scores.txt n'existe pas : il a été créé à l'étape 02."; exit 1; }
v() { tr -d '\r' < "$F" | sed -n "s/^[[:space:]]*$1[[:space:]]*=[[:space:]]*//p" | tail -1 | sed 's/[[:space:]]*$//'; }
case "$(v spec)" in [0-5]) ;; *) echo "La ligne spec = … doit donner le nombre de tests passés, de 0 à 5 (lu : « $(v spec) »)."; exit 1;; esac
case "$(v spec_auteur)" in agent|corrige|corrigé|secours) ;; *) echo "La ligne spec_auteur = … doit valoir agent, corrige ou secours (lu : « $(v spec_auteur) »)."; exit 1;; esac
```

## 03.3 - Commiter l'essai

Ici, contrairement au premier essai, vous avez une référence pour corriger : si le score brut n'est pas de 5, vous pouvez compléter le code de l'agent en relisant la spécification. C'est ce que ferait une équipe, et c'est permis, puisque votre score brut est déjà noté.

**Si l'agent n'a rien produit d'utilisable**, effacez ce qu'il a laissé avec `git restore . && git clean -fd src test`, la commande expliquée à l'étape précédente, puis appliquez le résultat de secours : une réalisation fidèle à la spécification. Dans `src/server.js`, juste avant la ligne `const match = req.url.match(…)`, ajoutez :

````solution
```js
    if (req.method === 'GET' && req.url === '/todos/export') {
      res.writeHead(200, {
        'Content-Type': 'application/json',
        'Content-Disposition': 'attachment; filename="todos.json"',
      });
      return res.end(JSON.stringify({ exportedAt: new Date().toISOString(), todos: store.list() }));
    }
```
````

Le secours a aussi besoin de tests dans le projet. Cette commande les tire des tests d'acceptation, en remplaçant le chargement du serveur du dossier courant par un `import` du serveur du projet :

```exec
cd ~/byte-layers/ai/todo-api && sed -e '1,2d' -e '/^import { join }/d' -e '/^import { pathToFileURL }/d' -e "s#^const { createServer } = await import.*#import { createServer } from '../src/server.js';#" ~/byte-layers/ai/acceptance/export.test.mjs > test/export.test.js
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `cd ~/byte-layers/ai/todo-api &&` | se place dans le projet, puis lance la suite si le `cd` a réussi |
> | `sed -e '1,2d'` | supprime les deux lignes de commentaire du début, qui parlent du dossier `acceptance` |
> | `-e '/^import { join }/d' -e '/^import { pathToFileURL }/d'` | supprime les deux `import` qui servaient à trouver le serveur du dossier courant |
> | `-e "s#^const { createServer } = await import.*#…#"` | remplace le chargement dynamique par `import { createServer } from '../src/server.js';` |
> | `> test/export.test.js` | écrit le résultat dans le dossier des tests du projet |

C'est un raccourci du lab : les tests du projet deviennent une copie du juge, qui ne peut alors plus rien départager. Dans une vraie équipe, les tests du projet s'écrivent avec le code, et les tests d'acceptation restent un regard extérieur.

Lancez les tests du projet. Comme au premier essai, seul `test/count.test.js` peut échouer :

```exec
cd ~/byte-layers/ai/todo-api && npm test
```

Commitez tout ce qui est sur la branche, code de l'agent, corrections ou secours :

```exec
cd ~/byte-layers/ai/todo-api && git add -A && git commit -m "Export des tâches, d'après la spécification"
```

Pour repartir de zéro après un commit raté, `git reset --hard spec/export` ramène la branche `export-spec` à la spécification.

```verify
cd ~/byte-layers/ai/todo-api 2>/dev/null || { echo "Le dossier ~/byte-layers/ai/todo-api n'existe pas."; exit 1; }
git rev-parse --verify -q export-spec >/dev/null || { echo "La branche export-spec n'existe pas. Lancez la commande de la section 03.1."; exit 1; }
git cat-file -e export-spec:docs/specs/export/requirements.md 2>/dev/null || { echo "La branche export-spec ne contient pas la spécification : elle doit partir de spec/export."; exit 1; }
git diff --quiet spec/export export-spec -- src && { echo "La branche export-spec ne modifie rien dans src/ : commitez le travail de l'agent, ou appliquez le résultat de secours puis commitez."; exit 1; }
T=$(mktemp -d)
git archive export-spec | tar -x -C "$T"
rm -f "$T/test/count.test.js"
if ! (cd "$T" && node --test >/dev/null 2>&1); then
  rm -rf "$T"; echo "Des tests du projet échouent sur la branche export-spec (hors test/count.test.js). Lancez npm test et corrigez, puis commitez."; exit 1
fi
rm -rf "$T"
```
