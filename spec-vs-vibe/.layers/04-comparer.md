---
title: "04 - Départager les deux essais"
description: "Passer les deux branches au même juge, comparer leur code, et en tirer la règle"
---

Les deux branches ont chacune un export qui ne casse pas les tests du projet. Reste à savoir laquelle fait ce que l'équipe voulait, et quelle part de ce résultat vient vraiment de l'agent.

## 04.1 - Le même juge pour les deux

Lancez les tests d'acceptation sur la branche `export-vibe`, puis sur la branche `export-spec` :

```exec
cd ~/byte-layers/ai/todo-api && git switch export-vibe && node --test ~/byte-layers/ai/acceptance/export.test.mjs; git switch export-spec && node --test ~/byte-layers/ai/acceptance/export.test.mjs
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `git switch export-vibe && node --test …` | se place sur `export-vibe`, puis lance le juge sur son code |
> | `;` | passe à la suite même si des tests ont échoué : on veut voir les deux résultats |
> | `git switch export-spec && node --test …` | même chose sur la branche `export-spec` |

Ces scores sont ceux des branches commitées, après vos éventuelles corrections. Mettez-les à côté des scores bruts que vous avez notés, ce que l'agent a produit seul :

```exec
cd ~ && cat ~/byte-layers/ai/acceptance/scores.txt
```

Comment lire les quatre chiffres :

| Ce que vous voyez | Ce qu'il faut en conclure |
|---|---|
| `spec` brut plus haut que `vibe` brut | l'effet attendu : avec les mêmes moyens, l'agent a fait davantage ce que l'équipe voulait |
| deux scores bruts bas, tous deux à 0 par exemple | le modèle est trop petit pour la tâche ; la comparaison porte alors sur ce que vous avez corrigé, pas sur l'agent |
| `vibe_auteur` et `spec_auteur` à `secours` | vous comparez deux réalisations écrites à la main : la conclusion vient du raisonnement, pas de votre essai |
| `vibe` brut à 5 | l'agent a deviné toutes les décisions de l'équipe : c'est possible, mais rien ne le garantissait |

Sur `export-vibe`, un score bas ne veut pas dire que l'agent a mal codé : il a pu prendre des décisions raisonnables, qui n'étaient pas celles de l'équipe. Le format, le nom du fichier, la date, le cas vide : rien de tout cela n'était dans l'issue.

## 04.2 - Comparer le code

Voyez ce que chaque essai a changé, fichier par fichier :

```exec
cd ~/byte-layers/ai/todo-api && git diff --stat agent-config export-vibe; git diff --stat spec/export export-spec
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `git diff --stat agent-config export-vibe` | résume ce que `export-vibe` a changé depuis son point de départ : fichiers touchés, lignes ajoutées et retirées |
> | `git diff --stat spec/export export-spec` | même chose pour `export-spec`, depuis la spécification |

Regardez ensuite le code lui-même et ses tests, avec `git diff agent-config export-vibe -- src test` puis `git diff spec/export export-spec -- src test`. Les questions à vous poser :

| Question | Pourquoi elle compte |
|---|---|
| Quelle branche a des tests pour l'export ? | sans test, rien ne protège la fonctionnalité de la prochaine modification |
| Quelle route, quel format, quels en-têtes ? | ce sont les décisions que la spécification fixait, et que l'agent a dû deviner sur `export-vibe` |
| L'exigence du RGPD est-elle respectée ? | un JSON est structuré, couramment utilisé et lisible par machine : les deux branches la respectent sans doute. La spécification ajoute ce que l'équipe a choisi en plus, les noms des champs, `exportedAt`, le nom du fichier |
| Quelle part du code vient de l'agent ? | vos lignes `_auteur` : un score obtenu par le secours ou par vos corrections ne dit rien du modèle |
| Qu'a-t-il fallu écrire avant de coder ? | trois fichiers de spécification sur `export-spec`, une phrase sur `export-vibe` : c'est le prix de la précision |

## 04.3 - Ce qu'il faut retenir

La spécification ne rend pas l'agent plus intelligent. Elle lui retire les choix qui n'étaient pas les siens, et elle donne à l'équipe un juge pour vérifier le résultat. Le vibe coding garde son intérêt pour un prototype jetable, ou pour explorer une idée : on accepte alors que l'agent décide. Dès que le résultat doit correspondre à une attente, la vôtre, celle d'un client ou celle de la loi, l'attente doit être écrite avant le code.

```acknowledge
J'ai passé les deux branches aux tests d'acceptation et comparé leur code.
```

## Récap

- Pour comparer deux façons de faire coder un agent, donnez-leur **la même fonctionnalité**, le même agent et le même modèle, chacune sur sa branche.
- Le juge est un jeu de **tests d'acceptation**, écrits d'après la spécification et gardés **hors des branches** : ni vous ni l'agent ne les adaptez au code.
- Sans spécification, l'agent **décide à votre place** : le résultat peut marcher, et ne pas correspondre à ce que l'équipe voulait.
- Notez le **score brut** avant toute correction : sinon, vous mesurez votre travail, pas celui de l'agent.
- Avec une spécification, l'agent suit des exigences écrites, et chaque exigence a son test.
- Le vibe coding convient à un prototype jetable ; dès qu'il y a une attente à respecter, écrivez-la d'abord.

> 📖 [Le lanceur de tests de Node.js](https://nodejs.org/api/test.html) · [Understanding Spec-Driven-Development: Kiro, spec-kit, and Tessl (Birgitta Böckeler)](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) · [Vibe coding (Wikipedia, en anglais)](https://en.wikipedia.org/wiki/Vibe_coding)
