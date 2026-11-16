---
title: "02 - Lire l'issue en analyste"
description: "Créer l'issue sur gitlab.com, écrire les questions qu'elle laisse ouvertes, et repérer l'exigence légale qu'elle tait"
---

Une issue est écrite par quelqu'un qui connaît son besoin, pas la façon de le construire. Le travail du développeur commence par la lire en analyste : qu'est-ce qui est dit, qu'est-ce qui manque, et qu'est-ce qui s'impose sans avoir été demandé ?

## 02.1 - Créer l'issue

Dans votre projet `todo-api` sur gitlab.com, ouvrez le menu **Plan › Issues** (ou **Plan › Work items**, selon la version de l'interface), puis cliquez sur **New issue** (ou **New item**, de type *Issue*). Donnez-lui le titre `Export des tâches`, et collez cette description :

```markdown
Un client nous a écrit : il quitte notre service pour un autre outil, et voudrait récupérer ses tâches. Ce serait bien d'avoir un export, il en a besoin assez vite.

Proposition de la tech lead : une route `GET /todos/export`.
```

Créez l'issue. gitlab.com lui donne un numéro, affiché après le titre : `#1` si c'est la première issue du projet. Retenez-le : la spécification et la merge request le citeront.

## 02.2 - Les questions que l'issue laisse ouvertes

L'issue tient en deux phrases. Avant d'écrire une seule exigence, listez ce qu'elle ne dit pas. Ces questions, vous les poseriez en commentaire de l'issue ; vous les écrivez d'abord dans un fichier, sur la branche de la spécification.

La spécification se prépare sur sa propre branche, `spec/export`, partie de `agent-config`. La commande crée la branche et le dossier de la spécification :

```exec
cd ~/byte-layers/ai/todo-api && git switch -c spec/export agent-config && mkdir -p docs/specs/export
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `cd ~/byte-layers/ai/todo-api &&` | se place dans le projet, puis lance la suite si le `cd` a réussi |
> | `git switch -c spec/export agent-config` | crée la branche `spec/export` à partir de `agent-config`, et s'y place |
> | `mkdir -p docs/specs/export` | crée le dossier de la spécification, selon la règle de `docs/specs/README.md` : un dossier par fonctionnalité |

Si la branche existe déjà, parce que vous reprenez le lab, lancez seulement `git switch spec/export`.

**À vous** : créez `docs/specs/export/questions.md` dans votre éditeur, et écrivez-y au moins quatre questions, une par ligne, chacune terminée par un point d'interrogation. Pensez à ce qu'un développeur devrait décider seul si personne ne répondait : la forme du document, son contenu, la façon dont le client le reçoit, les cas limites.

Une phrase de l'issue mérite un regard à part : « il en a besoin assez vite ». Ce n'est pas une exigence, qu'un test pourrait vérifier, mais une **priorité**, qui se traite dans le planning de l'équipe.

```hint
Partez des mots de l'issue qui ne sont pas définis. « Un export » : sous quel format ? « Ses tâches » : lesquelles, avec quels champs ? « Récupérer » : affiché ou téléchargé ? Et que se passe-t-il s'il n'en a aucune ?
```

````solution
```markdown
# Questions sur l'issue Export des tâches

- Dans quel format, JSON, CSV ou autre ?
- Faut-il exporter toutes les tâches, ou seulement celles qui ne sont pas terminées ?
- Quels champs de chaque tâche faut-il exporter ?
- Le document contient-il autre chose que les tâches, une date par exemple ?
- Le document s'affiche-t-il dans le navigateur, ou se télécharge-t-il ?
- Que renvoie l'export quand il n'y a aucune tâche ?
```
````

```verify
D=~/byte-layers/ai/todo-api
cd "$D" 2>/dev/null || { echo "Le dossier $D n'existe pas."; exit 1; }
B=$(git branch --show-current)
[ "$B" = "spec/export" ] || { echo "Vous êtes sur la branche $B : lancez la commande de cette section, ou git switch spec/export."; exit 1; }
F=docs/specs/export/questions.md
[ -f "$F" ] || { echo "Le fichier docs/specs/export/questions.md n'existe pas. Créez-le dans le dossier docs/specs/export."; exit 1; }
Q=$(tr -d '\r' < "$F" | grep -cE '[?][[:space:]*_]*$')
[ "$Q" -ge 4 ] || { echo "Le fichier contient $Q ligne(s) terminée(s) par un point d'interrogation : écrivez au moins quatre questions, une par ligne."; exit 1; }
```

## 02.3 - Les réponses de l'équipe

Dans une vraie équipe, l'auteur de l'issue répondrait en commentaire. Ici, l'équipe a déjà répondu. Comparez avec vos questions :

| Question | Réponse de l'équipe |
|---|---|
| Quel format ? | JSON |
| Quelles tâches ? | toutes, terminées ou non |
| Quels champs pour chaque tâche ? | les trois du projet : `id`, `title`, `done` |
| Que contient le document en plus des tâches ? | la date et l'heure de l'export, dans un champ `exportedAt`, au format ISO 8601 (`2026-11-16T09:30:00.000Z`) |
| Affiché ou téléchargé ? | téléchargé, sous le nom `todos.json` |
| Et s'il n'y a aucune tâche ? | un export valide, avec une liste vide, pas une erreur |

Chacune de ces réponses deviendra une exigence. Sans elles, un agent de code, ou un collègue pressé, aurait choisi à votre place, et chaque choix aurait pu être différent. Si vous aviez une question que le tableau ne couvre pas, gardez-la dans `questions.md` : elle accompagnera la spécification dans la merge request.

## 02.4 - L'exigence que personne n'a écrite

Avant de lire la suite, relisez la première phrase de l'issue, et demandez-vous : **pourquoi** ce client écrit-il, et **a-t-il un droit** à ce qu'il demande ?

Il **quitte le service** et veut **récupérer ses données** pour un autre outil. Le **RGPD**, le règlement européen sur la protection des données personnelles, prévoit exactement ce cas : le **droit à la portabilité** (article 20). Il s'applique sous conditions :

| Condition | Dans notre cas |
|---|---|
| la personne est une **personne physique**, pas une entreprise | le client est un particulier qui a ouvert un compte |
| les données **la concernent** et elle les a **fournies** | il a saisi ses tâches lui-même, sous son compte |
| le traitement repose sur son **consentement** ou sur un **contrat**, et il est **automatisé** | le service exécute son contrat d'abonnement, par logiciel |

Quand ces conditions sont réunies, le service doit lui remettre ses données dans un format **structuré, couramment utilisé et lisible par machine**, gratuitement et dans un délai d'un mois (article 12). Un PDF ou une capture d'écran ne conviendraient pas ; un JSON documenté, si. Si le client était une entreprise, le RGPD ne s'appliquerait pas, et l'export resterait un simple service rendu.

Le RGPD n'impose pas une route d'API : un export préparé à la main, sur demande, respecterait aussi la loi. La route est une façon de tenir cette obligation sans effort à chaque départ ; si on la retirait un jour, il faudrait la tenir autrement. L'exigence légale change donc peu le code, mais elle fixe le format, et elle explique pourquoi la fonctionnalité existe.

Ce cas est fréquent : une demande métier anodine cache une obligation réglementaire, que l'auteur de l'issue ne pense pas à écrire. La spécification est le bon endroit pour la rendre visible, et pour qu'un relecteur la vérifie.

> 📖 [Le droit à la portabilité (CNIL)](https://www.cnil.fr/fr/le-droit-la-portabilite-obtenir-et-reutiliser-une-copie-de-vos-donnees) · [Article 20 du RGPD (CNIL)](https://www.cnil.fr/fr/reglement-europeen-protection-donnees/chapitre3#Article20)

```acknowledge
J'ai lu les réponses de l'équipe et les conditions du droit à la portabilité.
```
