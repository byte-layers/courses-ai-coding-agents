---
title: "04 - La conception et les tâches"
description: "Écrire design.md, le comment de la fonctionnalité, puis tasks.md, les étapes ordonnées qui la réalisent"
---

Les exigences disent **quoi**. Les deux fichiers suivants disent **comment**, puis **dans quel ordre**. Ce sont eux qu'un agent de code suivra quand viendra le moment de coder.

## 04.1 - Le téléchargement en HTTP

Une exigence demande que l'export soit **téléchargé** sous le nom `todos.json`. En HTTP, c'est l'affaire d'un en-tête de réponse, `Content-Disposition` :

| En-tête de réponse | Effet |
|---|---|
| `Content-Type: application/json` | annonce un document JSON |
| `Content-Disposition: attachment; filename="todos.json"` | demande au navigateur d'enregistrer le document dans un fichier nommé `todos.json`, au lieu de l'afficher |

Le corps de la réponse, lui, suit les réponses de l'équipe : un objet avec la date de l'export et la liste des tâches. Rappel de la simplification du lab : le service n'a qu'un client par instance, donc « toutes les tâches » sont les siennes.

```json
{
  "exportedAt": "2026-11-16T09:30:00.000Z",
  "todos": [
    { "id": 1, "title": "Lire la spécification", "done": true },
    { "id": 2, "title": "Écrire les tests", "done": false }
  ]
}
```

## 04.2 - Écrire design.md

La conception tient en quatre rubriques. On écrit le fichier avec ses rubriques et ce qui ne dépend pas de vous ; vous compléterez les trous marqués `À compléter`. La commande n'écrit rien si le fichier existe déjà :

```exec
cd ~/byte-layers/ai/todo-api && if [ -e docs/specs/export/design.md ]; then echo "design.md existe déjà : il n'est pas réécrit."; else cat > docs/specs/export/design.md <<'FIN'
# Conception : export des tâches

## Route

À compléter : la méthode HTTP et le chemin.

## Réponse

- Code : 200, y compris quand il n'y a aucune tâche.
- En-têtes : À compléter, les deux en-têtes de la réponse.
- Corps : un objet JSON avec deux champs, `exportedAt` (date ISO 8601, `new Date().toISOString()`) et `todos` (la liste complète, `store.list()`).

## Code

- `src/server.js` : traiter la route avant la règle générale des routes `/todos`, qui répondrait 404 à `/todos/export`.
- Aucune nouvelle dépendance : `node:http` suffit.

## Tests

- Un fichier `test/export.test.js`, qui démarre le serveur comme `test/server.test.js` et vérifie chaque exigence de `requirements.md`.
FIN
fi
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `cd ~/byte-layers/ai/todo-api &&` | se place dans le projet, puis lance la suite si le `cd` a réussi |
> | `if [ -e docs/specs/export/design.md ]; then echo …` | si le fichier existe déjà, parce que vous reprenez le lab, le laisse tel quel : vos réponses ne sont pas effacées |
> | `else cat > docs/specs/export/design.md <<'FIN'` … `FIN` | sinon, écrit les lignes entre les deux `FIN` dans le fichier |
> | `fi` | termine le `if` |

La rubrique **Code** contient une remarque qu'aucune exigence ne pouvait donner : le serveur reconnaît aujourd'hui `/todos` et `/todos/<nombre>`, et toute autre adresse reçoit une erreur 404. C'est le genre de connaissance du code qui distingue une conception d'une simple reformulation des exigences.

**À vous** : ouvrez `docs/specs/export/design.md` dans votre éditeur, et remplacez les deux lignes `À compléter` : la route, puis les deux en-têtes de la réponse avec leurs valeurs exactes.

```hint
La route est celle que propose la tech lead dans l'issue. Les deux en-têtes sont dans le tableau de la section 04.1.
```

````solution
```markdown
## Route

`GET /todos/export`

## Réponse

- Code : 200, y compris quand il n'y a aucune tâche.
- En-têtes : `Content-Type: application/json` et `Content-Disposition: attachment; filename="todos.json"`.
```
````

```verify
F=~/byte-layers/ai/todo-api/docs/specs/export/design.md
[ -f "$F" ] || { echo "Le fichier docs/specs/export/design.md n'existe pas. Lancez la commande de cette section."; exit 1; }
if grep -q 'À compléter' "$F"; then echo "Le fichier contient encore « À compléter » : remplacez ces lignes."; exit 1; fi
T=$(tr -d '`*' < "$F")
printf '%s\n' "$T" | grep -qE 'GET[^A-Za-z]*/todos/export' || { echo "La route GET /todos/export n'apparaît pas dans design.md."; exit 1; }
printf '%s\n' "$T" | grep -qiE 'content-type[^a-z]*application/json' || { echo "L'en-tête Content-Type: application/json manque."; exit 1; }
printf '%s\n' "$T" | grep -qiE 'content-disposition[^a-z]*attachment' || { echo "L'en-tête Content-Disposition: attachment manque."; exit 1; }
printf '%s\n' "$T" | grep -q 'todos\.json' || { echo "Le nom du fichier téléchargé, todos.json, manque dans l'en-tête Content-Disposition."; exit 1; }
```

## 04.3 - Écrire tasks.md

Les tâches découpent la réalisation en petites étapes, dans l'ordre où on les fera. Chacune est assez petite pour être faite et vérifiée en une fois, et dit quelles exigences elle couvre : un relecteur voit d'un coup d'œil qu'aucune n'est oubliée. On les écrit comme une liste à cocher, la syntaxe que gitlab.com affiche avec de vraies cases :

```markdown
- [ ] Écrire test/export.test.js : un test par exigence, qui échoue (exigences 1 à 7)
```

La parenthèse finale relie la tâche aux exigences qu'elle couvre ; `(1, 2)` suffit aussi.

**À vous** : créez `docs/specs/export/tasks.md` dans votre éditeur. Écrivez un titre, puis au moins quatre tâches à cocher, dans l'ordre de réalisation. Chaque tâche cite entre parenthèses les numéros des exigences qu'elle couvre, et chaque exigence de votre `requirements.md` est couverte par au moins une tâche. La vérification compte les tâches et leurs parenthèses ; la couverture de chaque exigence, c'est à vous de la relire.

```hint
Un ordre qui marche : écrire les tests qui échouent ; ajouter la route qui renvoie la liste ; ajouter `exportedAt` ; ajouter les en-têtes du téléchargement ; vérifier le cas sans tâche et lancer `npm test`.
```

````solution
```markdown
# Tâches : export des tâches

- [ ] Écrire test/export.test.js : un test par exigence, qui échoue pour l'instant (exigences 1 à 7)
- [ ] Ajouter la route GET /todos/export dans src/server.js, avant la règle générale, qui renvoie toutes les tâches en JSON (exigences 1, 2, 3, 4)
- [ ] Ajouter le champ exportedAt au format ISO 8601 (exigence 5)
- [ ] Ajouter les en-têtes Content-Type et Content-Disposition du téléchargement (exigences 2, 6)
- [ ] Vérifier le cas sans tâche, puis lancer npm test : tous les tests passent (exigence 7)
```
````

```verify
F=~/byte-layers/ai/todo-api/docs/specs/export/tasks.md
[ -f "$F" ] || { echo "Le fichier docs/specs/export/tasks.md n'existe pas. Créez-le dans le dossier docs/specs/export."; exit 1; }
T=$(grep -cE '^[[:space:]]*[-*][[:space:]]+\[[[:space:]]\]' "$F")
[ "$T" -ge 4 ] || { echo "Le fichier contient $T tâche(s) à cocher, de la forme - [ ] … : il en faut au moins quatre."; exit 1; }
R=$(grep -E '^[[:space:]]*[-*][[:space:]]+\[[[:space:]]\]' "$F" | grep -ciE '\([^)]*[0-9]')
[ "$R" -ge "$T" ] || { echo "$((T - R)) tâche(s) ne citent pas de numéro d'exigence entre parenthèses, par exemple (exigences 1, 2)."; exit 1; }
```
