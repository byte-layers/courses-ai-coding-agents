---
title: "02 - L'essai au feeling"
description: "Demander l'export à l'agent avec les seuls mots de l'issue, sur la branche export-vibe, et noter son score brut"
---

Premier essai : vous donnez à l'agent ce qu'un développeur pressé lui donnerait, la demande telle qu'elle est arrivée, sans spécification.

## 02.1 - Demander l'export

Créez la branche `export-vibe` à partir de `agent-config`, la branche qui n'a pas la spécification :

```exec
cd ~/byte-layers/ai/todo-api && git switch -c export-vibe agent-config
```

Si la branche existe déjà, parce que vous reprenez le lab, lancez seulement `git switch export-vibe`.

Lancez ensuite l'agent avec les mots de l'issue. La demande ajoute une seule précision, sur le test `test/count.test.js` laissé en échec par le lab Un modèle, deux harness, pour que l'agent ne perde pas son temps dessus :

```exec
cd ~/byte-layers/ai/todo-api && opencode run "Un client quitte notre service et voudrait récupérer ses tâches. Ajoute un export des tâches à l'API, puis lance npm test. Le test test/count.test.js échoue déjà : ignore-le et ne le modifie pas."
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `cd ~/byte-layers/ai/todo-api &&` | se place dans le projet, puis lance la suite si le `cd` a réussi |
> | `opencode run "…"` | donne la demande à OpenCode, qui travaille dans le projet puis rend la main |

L'agent a carte blanche : la route, le format, les champs, le nom du fichier, il choisit tout. Selon votre poste, il peut travailler plusieurs minutes. Toute commande du shell autre que `npm test` tombe sous la règle `"*": "ask"` de votre `opencode.json` ; `opencode run` ne pose pas de question, et la refuse d'office. L'agent écrit donc le code, mais ne peut pas, par exemple, installer une dépendance.

Si l'agent tourne trop longtemps, interrompez-le avec `Ctrl+C`. Ce qu'il a déjà écrit reste dans vos fichiers.

## 02.2 - Noter le score brut

Avant de toucher à quoi que ce soit, passez le travail de l'agent au juge. Les tests chargent le `src/server.js` du dossier courant, modifications non commitées comprises :

```exec
cd ~/byte-layers/ai/todo-api && node --test ~/byte-layers/ai/acceptance/export.test.mjs
```

La fin de l'affichage compte les tests passés, `pass`, et échoués, `fail`. Ce chiffre est le **score brut** de l'essai : ce que l'agent a produit seul, sans votre aide. Si l'agent n'a rien écrit, il vaut 0.

**À vous** : créez le fichier `~/byte-layers/ai/acceptance/scores.txt` dans votre éditeur, et écrivez-y deux lignes :

| Ligne | Valeur |
|---|---|
| `vibe = …` | le nombre de tests passés, de 0 à 5 |
| `vibe_auteur = …` | `agent` si vous gardez le code de l'agent tel quel, `secours` si vous appliquez le résultat de secours de la section suivante |

```hint
Avec 2 tests passés sur 5 et le code de l'agent gardé tel quel, le fichier contient `vibe = 2` puis `vibe_auteur = agent`.
```

```verify
F=~/byte-layers/ai/acceptance/scores.txt
[ -f "$F" ] || { echo "Le fichier ~/byte-layers/ai/acceptance/scores.txt n'existe pas. Créez-le avec les lignes vibe = … et vibe_auteur = …"; exit 1; }
v() { tr -d '\r' < "$F" | sed -n "s/^[[:space:]]*$1[[:space:]]*=[[:space:]]*//p" | tail -1 | sed 's/[[:space:]]*$//'; }
case "$(v vibe)" in [0-5]) ;; *) echo "La ligne vibe = … doit donner le nombre de tests passés, de 0 à 5 (lu : « $(v vibe) »)."; exit 1;; esac
case "$(v vibe_auteur)" in agent|secours) ;; *) echo "La ligne vibe_auteur = … doit valoir agent ou secours (lu : « $(v vibe_auteur) »)."; exit 1;; esac
```

## 02.3 - Commiter l'essai

**Si l'agent n'a rien produit d'utilisable**, effacez d'abord ce qu'il a laissé, puis appliquez le résultat de secours. Cette commande remet les fichiers suivis dans l'état du dernier commit, et supprime les fichiers nouveaux de `src` et `test` :

```exec
cd ~/byte-layers/ai/todo-api && git restore . && git clean -fd src test
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `cd ~/byte-layers/ai/todo-api &&` | se place dans le projet, puis lance la suite si le `cd` a réussi |
> | `git restore .` | annule les modifications non commitées des fichiers suivis |
> | `git clean -fd src test` | supprime les fichiers et dossiers non suivis de `src` et `test` ; `-f` confirme, `-d` inclut les dossiers |

Le résultat de secours est **un exemple** de ce qu'une demande vague peut donner : une route courte, qui renvoie la liste brute. Un modèle peut tout aussi bien choisir `/todos/export`, et passer alors le premier test.

````solution
Dans `src/server.js`, juste avant la ligne `const match = req.url.match(…)`, ajoutez :

```js
    if (req.method === 'GET' && req.url === '/export') return send(200, store.list());
```
````

Regardez d'abord ce qui a changé : la liste des fichiers modifiés (`M`) ou nouveaux (`??`) :

```exec
cd ~/byte-layers/ai/todo-api && git status --short
```

Lancez ensuite les tests du projet, pour vérifier que l'export n'a rien cassé :

```exec
cd ~/byte-layers/ai/todo-api && npm test
```

Le test `test/count.test.js` échoue encore, sauf si l'agent a écrit `countRemaining` au passage : cet échec ne compte pas ici. Les autres tests doivent passer. Commitez tout, sans retoucher le code de l'agent : c'est lui qu'on veut comparer.

```exec
cd ~/byte-layers/ai/todo-api && git add -A && git commit -m "Export des tâches, demandé au feeling"
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `cd ~/byte-layers/ai/todo-api &&` | se place dans le projet, puis lance la suite si le `cd` a réussi |
> | `git add -A` | ajoute tous les fichiers modifiés ou nouveaux, sauf ceux que `.gitignore` exclut, comme `.env` |
> | `git commit -m "…"` | enregistre le commit sur `export-vibe` |

Si vous avez commité un essai raté et voulez repartir de zéro, `git reset --hard agent-config` ramène la branche `export-vibe` à son point de départ, en effaçant son travail.

```verify
cd ~/byte-layers/ai/todo-api 2>/dev/null || { echo "Le dossier ~/byte-layers/ai/todo-api n'existe pas."; exit 1; }
git rev-parse --verify -q export-vibe >/dev/null || { echo "La branche export-vibe n'existe pas. Lancez la commande de la section 02.1."; exit 1; }
git diff --quiet agent-config export-vibe -- src && { echo "La branche export-vibe ne modifie rien dans src/ : commitez le travail de l'agent, ou appliquez le résultat de secours puis commitez."; exit 1; }
T=$(mktemp -d)
git archive export-vibe | tar -x -C "$T"
rm -f "$T/test/count.test.js"
if ! (cd "$T" && node --test >/dev/null 2>&1); then
  rm -rf "$T"
  echo "Des tests du projet échouent sur la branche export-vibe (hors test/count.test.js). Lancez npm test, corrigez ou appliquez le résultat de secours, puis commitez."
  exit 1
fi
rm -rf "$T"
```
