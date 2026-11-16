---
title: "05 - Proposer la spécification"
description: "Commiter les trois fichiers, pousser la branche et ouvrir une merge request liée à l'issue"
---

Une spécification n'engage l'équipe qu'une fois relue et acceptée. Sur gitlab.com, cela passe par une **merge request** : la proposition de fusionner votre branche, que vos collègues commentent ligne par ligne avant de l'accepter.

## 05.1 - Commiter et pousser

Commitez les fichiers de la spécification, vos questions comprises, puis poussez la branche sur gitlab.com. La branche part sur `gitlab`, jamais sur `origin` :

```exec
cd ~/byte-layers/ai/todo-api && git add docs/specs/export && git commit -m "Spécification de l'export des tâches" && git push -u gitlab spec/export
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `cd ~/byte-layers/ai/todo-api &&` | se place dans le projet, puis lance la suite si le `cd` a réussi |
> | `git add docs/specs/export` | ajoute à l'index les fichiers du dossier : `questions.md`, `requirements.md`, `design.md` et `tasks.md` |
> | `git commit -m "…"` | enregistre le commit sur la branche `spec/export` |
> | `git push -u gitlab spec/export` | envoie la branche sur votre projet gitlab.com, et retient ce lien |
> | `&&` | n'enchaîne chaque commande que si la précédente a réussi |

En réponse au `push`, gitlab.com affiche un lien de la forme :

```text
remote: To create a merge request for spec/export, visit:
remote:   https://gitlab.com/<votre-identifiant>/todo-api/-/merge_requests/new?merge_request%5Bsource_branch%5D=spec%2Fexport
```

Si vous relancez la commande alors que le commit est déjà fait, `git commit` répond `nothing to commit` et le `push` ne part pas : lancez alors seulement `git push -u gitlab spec/export`.

```verify
cd ~/byte-layers/ai/todo-api 2>/dev/null || { echo "Le dossier ~/byte-layers/ai/todo-api n'existe pas."; exit 1; }
for f in questions.md requirements.md design.md tasks.md; do
  git cat-file -e "spec/export:docs/specs/export/$f" 2>/dev/null || { echo "docs/specs/export/$f n'est pas commité sur la branche spec/export. Lancez la commande de cette section."; exit 1; }
  [ -z "$(git status --porcelain -- "docs/specs/export/$f")" ] || { echo "docs/specs/export/$f a des modifications qui ne sont pas commitées. Relancez git add, git commit puis git push."; exit 1; }
done
OUT=$(GIT_TERMINAL_PROMPT=0 git ls-remote gitlab refs/heads/spec/export 2>/dev/null) || { echo "Le projet gitlab.com n'est pas joignable sans saisie : voyez le paragraphe avant la vérification de l'étape 01 (ssh-add, ou credential.helper)."; exit 1; }
REMOTE=$(printf '%s\n' "$OUT" | cut -f1)
[ -n "$REMOTE" ] || { echo "La branche spec/export n'est pas sur gitlab.com. Lancez git push -u gitlab spec/export."; exit 1; }
[ "$REMOTE" = "$(git rev-parse spec/export)" ] || { echo "La branche spec/export de gitlab.com n'a pas votre dernier commit. Relancez git push."; exit 1; }
```

## 05.2 - Ouvrir la merge request

Ouvrez le lien affiché par le `push`. Sinon, dans votre projet sur gitlab.com, ouvrez **Code › Merge requests › New merge request**, choisissez `spec/export` comme branche source et `agent-config` comme branche cible, puis cliquez sur **Compare branches and continue**.

La branche cible est `agent-config` parce que c'est la branche par défaut de votre projet : c'est une simplification du lab. Dans un vrai projet, la cible serait la branche principale, souvent `main`.

Remplissez le formulaire. Le titre arrive pré-rempli avec le message du commit : remplacez-le.

| Champ | Valeur |
|---|---|
| Title | `Spécification : export des tâches` |
| Description | une phrase qui résume la spécification, puis `Relates to #1`, avec le numéro de votre issue. Signalez à part l'exigence de portabilité du RGPD, celle que le relecteur doit trouver sans la chercher, et les questions de `questions.md` restées sans réponse |

Cliquez sur **Create merge request**. C'est la mention `#1` qui relie la merge request à l'issue : l'issue affiche désormais un lien vers votre proposition. *Relates to* n'est pas un mot-clé de gitlab.com, il dit la relation au lecteur. On n'écrit pas `Closes #1`, qui fermerait l'issue à la fusion : la spécification ne livre pas encore l'export, elle le décrit.

Dans une équipe, la suite se passe dans l'onglet **Changes** de la merge request : un relecteur commente les exigences ligne par ligne, vous corrigez sur la même branche, et un nouveau `push` met la merge request à jour. Le code ne commence qu'après l'accord.

> 📖 [Créer une merge request](https://docs.gitlab.com/user/project/merge_requests/creating_merge_requests/) · [Relier une issue et une merge request](https://docs.gitlab.com/user/project/issues/crosslinking_issues/)

```acknowledge
J'ai ouvert sur gitlab.com la merge request de la branche spec/export vers agent-config, liée à mon issue.
```

## Récap

- Une issue dit un besoin, pas une solution : écrivez les questions qu'elle **ne dit pas**, et faites trancher chacune avant d'écrire. « Assez vite » est une priorité, pas une exigence.
- Cherchez l'**exigence implicite** : une demande anodine, comme un export pour un client qui part, peut cacher une obligation légale, ici le droit à la **portabilité** du RGPD, sous ses conditions : une personne physique, des données qu'elle a fournies. Elle va sur une ligne de justification, l'exigence restant testable.
- `requirements.md` dit **quoi**, en exigences EARS testables, une par phrase ; `design.md` dit **comment**, avec ce que vous savez du code ; `tasks.md` dit **dans quel ordre**, chaque tâche reliée à ses exigences.
- Un agent peut écrire le premier jet ; la relecture reste la vôtre : testable, quoi et pas comment, une exigence par phrase. **Si…, alors** est réservé aux comportements indésirables.
- La spécification se propose dans une **merge request** liée à l'issue, et le code attend qu'elle soit acceptée.
