---
title: "02 - Écrire les instructions du projet dans AGENTS.md"
description: "Le fichier d'instructions lu par la plupart des harness : commandes, conventions, spécifications, sécurité"
---

Un agent qui découvre un projet ne sait pas comment lancer ses tests, quelles conventions il suit, ni ce qu'il ne faut pas toucher. **`AGENTS.md`**, à la racine du dépôt, le lui dit : c'est la brique « instructions du projet », chargée à chaque demande, et le format lu par la plupart des harness. On commence par lui, parce que c'est ce qui passera d'un outil à l'autre sans rien changer.

## 02.1 - Créer le fichier

`AGENTS.md` est du Markdown ordinaire : aucune rubrique n'est obligatoire. Le site du format conseille d'y mettre les commandes de build et de test, le style de code, les consignes de test et de sécurité. Comme ce fichier est chargé à chaque demande et occupe du contexte, on le garde court : des consignes concrètes, pas de documentation.

La commande ci-dessous crée le fichier avec quatre rubriques. Lancez-la depuis le dossier du projet :

```exec
cd ~/byte-layers/ai/todo-api && cat > AGENTS.md <<'FIN'
# todo-api

Une API de tâches en Node.js, sans dépendance. Le code est dans `src/`, les tests dans `test/`.

## Commandes

- Tester : `npm test` (lance `node --test`). Lancez-le après chaque modification.
- Démarrer : `npm start` (port 3000, ou la variable `PORT`).
- Rien à installer : le projet n'a aucune dépendance.

## Conventions

- JavaScript en modules ES (`import` et `export`), pour Node.js 22 ou plus récent.
- Identifiants et messages d'erreur en anglais, commentaires en français.
- Aucune nouvelle dépendance sans accord explicite.

## Spécifications

- Avant de coder une nouvelle fonctionnalité, lisez sa spécification dans `docs/specs/`.

## Sécurité

- Ne lisez jamais le fichier `.env` et ne recopiez aucun secret : les variables attendues sont dans `.env.example`.
- Ne supprimez aucun fichier sans le demander.
FIN
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `cd ~/byte-layers/ai/todo-api &&` | se place dans le projet, puis lance la suite seulement si le `cd` a réussi |
> | `cat > AGENTS.md` | écrit ce que `cat` reçoit dans le fichier `AGENTS.md` (et le remplace s'il existe) |
> | `<<'FIN'` … `FIN` | un *here-document* : les lignes entre les deux `FIN` sont envoyées à `cat`. Les guillemets autour du premier `FIN` empêchent le shell d'interpréter les `` ` `` et les `$` du texte |

Chaque rubrique répond à un besoin de l'agent :

| Rubrique | Ce que l'agent en fait |
|---|---|
| **Commandes** | il sait comment vérifier son travail, sans deviner ni lire tout le `package.json` |
| **Conventions** | le code qu'il écrit ressemble au reste du projet |
| **Spécifications** | il sait où chercher ce qu'on attend de lui avant de coder |
| **Sécurité** | il connaît les interdits. Une instruction **demande** : un modèle peut l'oublier. Plus loin, une permission d'OpenCode **garantit** le refus |

Ouvrez le fichier dans votre éditeur et relisez-le : c'est le vôtre, adaptez une convention si vous le souhaitez, en gardant les quatre rubriques.

> 📖 [Le format AGENTS.md](https://agents.md/)

```verify
F=~/byte-layers/ai/todo-api/AGENTS.md
if [ ! -f "$F" ]; then
  echo "Le fichier $F n'existe pas. Lancez la commande de cette section."
  exit 1
fi
for h in "## Commandes" "## Conventions" "## Spécifications" "## Sécurité"; do
  if ! grep -q "^$h" "$F"; then
    echo "La rubrique « $h » manque dans AGENTS.md."
    exit 1
  fi
done
if ! grep -q 'npm test' "$F"; then
  echo "AGENTS.md ne donne pas la commande de test, npm test."
  exit 1
fi
if ! grep -q '\.env' "$F"; then
  echo "La rubrique Sécurité d'AGENTS.md ne parle plus du fichier .env."
  exit 1
fi
```
