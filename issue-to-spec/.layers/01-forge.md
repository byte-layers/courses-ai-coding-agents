---
title: "01 - Mettre le projet sur gitlab.com"
description: "Créer un projet vide sur gitlab.com et y pousser la branche agent-config"
---

Une spécification se discute en équipe, dans la forge où vit le code. Cette étape est un **support** : elle installe le projet `todo-api` sur votre compte gitlab.com, pour y ouvrir l'issue puis la merge request.

## 01.1 - Créer le projet

Sur gitlab.com, connecté à votre compte :

1. Cliquez sur **New project**, puis **Create blank project**.
2. Nommez le projet `todo-api`. Dans **Project URL**, choisissez votre nom d'utilisateur comme espace de noms, s'il n'est pas déjà sélectionné. Laissez la visibilité sur **Private**.
3. **Décochez** toutes les cases de *Project Configuration* : *Initialize repository with a README*, et *Enable Static Application Security Testing (SAST)* si elle apparaît. Chacune créerait un premier commit sur une branche `main`. Le projet doit rester vide, puisque votre dépôt local a déjà son historique.
4. Cliquez sur **Create project**.

La page du projet vide affiche son adresse de clonage sous le bouton **Code**, et aussi dans les instructions *Push an existing Git repository*. Elle a la forme `git@gitlab.com:<votre-identifiant>/todo-api.git` en SSH, ou `https://gitlab.com/<votre-identifiant>/todo-api.git` en HTTPS.

## 01.2 - Pousser la branche agent-config

Votre dépôt local connaît déjà un dépôt distant, `origin` : le dépôt d'origine du projet, sur GitHub, qui appartient à byte_layers. On ne pousse **jamais** sur `origin`. Ajoutez votre projet gitlab.com comme second dépôt distant, nommé `gitlab`. Cette commande contient votre identifiant : tapez-la vous-même, en remplaçant `<votre-identifiant>`, et en prenant la forme SSH ou HTTPS selon votre façon de vous connecter :

```bash
cd ~/byte-layers/ai/todo-api && git remote add gitlab git@gitlab.com:<votre-identifiant>/todo-api.git
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `cd ~/byte-layers/ai/todo-api &&` | se place dans le projet, puis lance la suite si le `cd` a réussi |
> | `git remote add gitlab <adresse>` | enregistre un dépôt distant nommé `gitlab`, à cette adresse ; rien n'est encore envoyé |

Si vous vous êtes trompé d'adresse, corrigez-la avec `git remote set-url gitlab <adresse>`.

Poussez ensuite la branche `agent-config`. Comme c'est la première branche du projet, gitlab.com en fait la branche par défaut, celle vers laquelle les merge requests seront proposées. Il la marque aussi **protégée** : seuls les responsables du projet peuvent y pousser, ce que vous êtes.

```exec
cd ~/byte-layers/ai/todo-api && git push -u gitlab agent-config
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `cd ~/byte-layers/ai/todo-api &&` | se place dans le projet, puis lance la suite si le `cd` a réussi |
> | `git push gitlab agent-config` | envoie la branche `agent-config` au dépôt distant `gitlab` |
> | `-u` | retient ce lien : un simple `git push` depuis cette branche ira désormais vers `gitlab` |

Si le `push` échoue, votre poste n'est pas encore autorisé à pousser sur gitlab.com :

| Message | Ce qu'il faut faire |
|---|---|
| `Permission denied (publickey)`, en SSH | enregistrez votre clé publique dans les préférences de votre compte, rubrique **SSH Keys** |
| `Are you sure you want to continue connecting`, en SSH | c'est la première connexion à gitlab.com : répondez `yes` |
| une demande de nom et de mot de passe, en HTTPS | le nom est votre nom d'utilisateur gitlab.com ; le mot de passe est un **jeton d'accès personnel** avec le droit `write_repository`, jamais le mot de passe du compte, refusé dès que la double authentification est active |

Sous Windows, Git Credential Manager peut ouvrir une fenêtre de connexion à gitlab.com dans le navigateur : acceptez-la, elle remplace le jeton.

> 📖 [Créer un projet sur GitLab](https://docs.gitlab.com/user/project/) · [Les clés SSH sur GitLab](https://docs.gitlab.com/user/ssh/) · [Les jetons d'accès personnels](https://docs.gitlab.com/user/profile/personal_access_tokens/)

La vérification interroge gitlab.com sans pouvoir vous demander de mot de passe. Si votre `push` a marché mais qu'elle dit le projet injoignable, votre accès dépend d'une saisie : en SSH, chargez votre clé avec `ssh-add` ; en HTTPS, laissez Git retenir le jeton avec `git config --global credential.helper store`, qui l'enregistre en clair dans votre dossier personnel, ou le gestionnaire de votre système.

```verify
D=~/byte-layers/ai/todo-api
cd "$D" 2>/dev/null || { echo "Le dossier $D n'existe pas : faites d'abord le lab Une configuration d'agent portable d'abord."; exit 1; }
URL=$(git config --get remote.gitlab.url) || { echo "Le dépôt distant gitlab n'existe pas. Lancez git remote add gitlab … (section 01.2), avec votre identifiant."; exit 1; }
case "$URL" in *gitlab.com*) ;; *) echo "Le dépôt distant gitlab ne pointe pas vers gitlab.com (lu : $URL). Corrigez-le avec git remote set-url gitlab <adresse>."; exit 1;; esac
case "$URL" in *"<"*|*votre-identifiant*) echo "L'adresse du dépôt distant contient encore <votre-identifiant> : corrigez-la avec git remote set-url gitlab <adresse>."; exit 1;; esac
OUT=$(GIT_TERMINAL_PROMPT=0 git ls-remote --symref gitlab HEAD refs/heads/agent-config 2>/dev/null) || { echo "Le projet gitlab.com n'est pas joignable sans saisie. Si le push a marché, voyez le paragraphe juste avant la vérification (ssh-add, ou credential.helper)."; exit 1; }
REMOTE=$(printf '%s\n' "$OUT" | awk -F '\t' '$2 == "refs/heads/agent-config" { print $1 }')
[ -n "$REMOTE" ] || { echo "La branche agent-config n'est pas sur gitlab.com. Lancez git push -u gitlab agent-config."; exit 1; }
[ "$REMOTE" = "$(git rev-parse agent-config)" ] || { echo "La branche agent-config de gitlab.com n'est pas à jour. Relancez git push gitlab agent-config."; exit 1; }
HEADB=$(printf '%s\n' "$OUT" | sed -n 's|^ref: refs/heads/\([^[:space:]]*\)[[:space:]]*HEAD$|\1|p')
if [ -n "$HEADB" ] && [ "$HEADB" != "agent-config" ]; then
  echo "La branche par défaut du projet gitlab.com est $HEADB, pas agent-config : le projet n'était pas vide. Dans Settings › Repository › Branch defaults, choisissez agent-config."
  exit 1
fi
```
