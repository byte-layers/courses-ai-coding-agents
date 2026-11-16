---
title: "03 - Écrire les exigences"
description: "Écrire requirements.md en syntaxe EARS, avec l'aide d'un agent si vous le voulez, et le relire"
---

Les réponses de l'équipe et l'exigence légale sont réunies : il reste à les écrire en exigences, une par phrase, chacune testable. C'est le premier des trois fichiers de la spécification.

## 03.1 - Un premier jet par l'agent (facultatif)

Votre projet a une skill, `ears-spec`, écrite pour ce travail. Si vous avez relié OpenCode à un modèle local dans le lab [Un modèle, deux harness](https://byte-layers.com/fr/labs/one-model-two-harnesses), demandez-lui un premier jet. Sinon, passez directement à la section suivante.

La demande donne à l'agent tout ce que vous savez : l'issue, les réponses de l'équipe, l'exigence légale. Elle cite le numéro de votre issue : tapez-la vous-même, en remplaçant les deux `#1` si votre issue a un autre numéro.

```bash
cd ~/byte-layers/ai/todo-api && git switch spec/export && opencode run "Utilise la skill ears-spec. Écris docs/specs/export/requirements.md pour l'issue #1, Export des tâches : un client quitte le service et veut récupérer ses tâches. Route GET /todos/export. Réponses de l'équipe : JSON, toutes les tâches terminées ou non, champs id, title et done, un champ exportedAt au format ISO 8601, téléchargé sous le nom todos.json, une liste vide s'il n'y a aucune tâche. Le format sert le droit à la portabilité du RGPD (article 20) : structuré, couramment utilisé et lisible par machine. Commence le fichier par un titre et la ligne Issue : #1."
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `cd ~/byte-layers/ai/todo-api &&` | se place dans le projet, puis lance la suite si le `cd` a réussi |
> | `git switch spec/export &&` | s'assure que l'agent écrit sur la branche de la spécification |
> | `opencode run "…"` | envoie une seule demande à OpenCode, qui travaille dans le projet puis rend la main |

Selon votre poste, la réponse peut prendre plusieurs minutes. L'agent n'a besoin que d'écrire un fichier, ce que vos permissions autorisent. S'il essaie de lancer une commande du shell, la règle `"*": "ask"` de votre `opencode.json` la soumet à un accord que `opencode run`, qui ne pose pas de question, refuse d'office : c'est sans conséquence ici. S'il abandonne sans écrire le fichier, écrivez-le vous-même à la section suivante.

Ce que l'agent écrit n'est qu'un **premier jet** : la section suivante vous le fait relire, et c'est votre version qui sera proposée à l'équipe.

## 03.2 - Écrire et relire requirements.md

**À vous** : ouvrez ou créez `docs/specs/export/requirements.md` dans votre éditeur. Le fichier doit contenir :

- un titre, puis une ligne `Issue : #N`, avec le numéro de votre issue ;
- des exigences numérotées, une par ligne de liste, et non dans un tableau, chacune écrite avec un des motifs EARS de la skill : « Le système doit… », « Quand…, le système doit… », etc. ;
- une exigence pour chaque réponse de l'équipe, soit au moins six ;
- au moins une exigence **Quand** : l'appel de la route ;
- l'exigence légale : le **RGPD** et le droit à la **portabilité**, sur une ligne `Justification :` sous l'exigence de format qu'ils motivent.

Une justification n'est pas une exigence : elle dit **pourquoi**, et un test ne peut pas la vérifier. La séparer garde l'exigence testable, et la raison visible pour le relecteur.

Le cas sans tâche se lit comme une condition, mais il ne s'écrit pas avec **Si…, alors** : ce motif EARS est réservé aux comportements indésirables, comme une erreur ou une panne. Une liste vide est un état normal du service ; écrivez-la avec **Quand**, par exemple « Quand un client demande l'export alors qu'aucune tâche n'existe, … ».

Relisez chaque exigence avec trois questions, celles d'un relecteur :

| Question | Ce qu'elle attrape |
|---|---|
| Puis-je en écrire un test ? | les mots vagues : « rapide », « simple », « correct » |
| Dit-elle *quoi*, pas *comment* ? | une décision de code glissée dans une exigence : nom de fonction, de fichier source |
| Est-elle seule dans sa phrase ? | deux exigences collées par un « et », qui ne pourront pas être validées séparément |

Si l'agent a écrit le premier jet, ces questions trouvent presque toujours quelque chose à corriger. La troisième, en particulier : « répondre avec le code 200 et un document JSON », ce sont deux exigences.

```hint
Partez des lignes du tableau de la section 02.3 : chacune donne au moins une exigence. Exemple pour l'appel de la route : « Quand un client envoie GET /todos/export, le système doit répondre avec le code 200. » Le format JSON fait l'objet d'une exigence à part, suivie de sa ligne Justification.
```

````solution
```markdown
# Export des tâches

Issue : #1

1. Quand un client envoie GET /todos/export, le système doit répondre avec le code 200.
2. Le système doit produire l'export en JSON valide, annoncé avec le type application/json.
   Justification : le droit à la portabilité du RGPD (article 20) demande un format structuré, couramment utilisé et lisible par machine.
3. Le système doit inclure dans l'export toutes les tâches, terminées ou non.
4. Le système doit donner pour chaque tâche exportée son identifiant (id), son titre (title) et son état (done).
5. Le système doit indiquer dans l'export la date et l'heure de sa production, dans un champ exportedAt au format ISO 8601.
6. Quand un client demande l'export, le système doit proposer le document au téléchargement sous le nom todos.json.
7. Quand un client demande l'export alors qu'aucune tâche n'existe, le système doit renvoyer un export valide avec une liste de tâches vide.
```
````

```verify
D=~/byte-layers/ai/todo-api
F=$D/docs/specs/export/requirements.md
[ -f "$F" ] || { echo "Le fichier docs/specs/export/requirements.md n'existe pas. Créez-le dans le dossier docs/specs/export."; exit 1; }
B=$(cd "$D" && git branch --show-current)
[ "$B" = "spec/export" ] || { echo "Vous êtes sur la branche $B : placez-vous sur spec/export (git switch spec/export)."; exit 1; }
T=$(LC_ALL=C sed -e $'s/\xc2\xa0/ /g' -e 's/[*_`]//g' -e 's/\r$//' "$F")
printf '%s\n' "$T" | grep -qiE 'issue[^#]{0,6}#[0-9]+' || { echo "Le fichier ne cite pas l'issue : ajoutez une ligne Issue : #N, avec le numéro de votre issue sur gitlab.com."; exit 1; }
REQ=$(printf '%s\n' "$T" | grep -iE '^[[:space:]]*([-+]|[a-z]*-?[0-9]+[.):-]?|exigence[[:space:]]+[0-9]+[[:space:]]*[.:-]?)[[:space:]]' | grep -iE '(^|[^[:alpha:]])doi(t|vent)([^[:alpha:]]|$)')
N=$(printf '%s' "$REQ" | grep -c .)
if [ "$N" -lt 6 ]; then
  if printf '%s\n' "$T" | grep -q '^[[:space:]]*|'; then echo "Le fichier contient $N exigence(s) en liste : écrivez-les en liste numérotée (1. …), une par ligne, et non dans un tableau."; exit 1; fi
  echo "Le fichier contient $N exigence(s) numérotées avec « doit » : il en faut au moins six, une par réponse de l'équipe."; exit 1
fi
printf '%s\n' "$REQ" | grep -qiE '(^|[^[:alpha:]])(quand|lorsqu)' || { echo "Aucune exigence ne suit le motif Quand …, le système doit … : écrivez l'appel de la route avec ce motif."; exit 1; }
printf '%s\n' "$T" | grep -qi 'rgpd' || { echo "L'exigence légale manque : aucune ligne ne cite le RGPD."; exit 1; }
printf '%s\n' "$T" | grep -qi 'portabilit' || { echo "L'exigence légale doit nommer le droit en jeu : la portabilité."; exit 1; }
printf '%s\n' "$T" | grep -qi 'justification' || { echo "Le RGPD doit figurer sur une ligne Justification :, sous l'exigence de format qu'il motive."; exit 1; }
```
