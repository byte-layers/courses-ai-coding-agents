---
title: "Passer sa configuration à un autre harness"
description: "Lab : reprendre dans DeepSeek Harness la configuration d'agent écrite pour OpenCode, brique par brique : ce qui passe tel quel, ce qui se refait, ce qui ne passe pas."
tags:
  - lab
  - ai
difficulty: intermediate
duration: 40
introTitle: "Le lab"
requires: "0.4.0"
options:
  allowManualProgress: false
  autoCompleteOnNext: unverifiable
---

Le projet `todo-api` a maintenant une configuration d'agent complète : des instructions dans `AGENTS.md`, une skill, des permissions et des hooks pour OpenCode. Un collègue préfère DeepSeek Harness (`dsh`). Dans ce lab, vous reprenez cette configuration dans dsh, brique par brique, et vous notez pour chacune si elle passe telle quelle, si elle se refait, ou si elle ne passe pas. C'est le travail réel d'une équipe où chacun garde son outil.

Le lab n'a pas besoin de modèle. Ce que dsh envoie au modèle a été constaté pour vous, sur dsh 0.2.2-rc.3, en capturant ses requêtes : le lab vous donne ces constats, et vous les reliez à vos propres fichiers. Si votre modèle local tourne, un essai réel vous permet de le voir vous-même.

**Niveau** : à l'aise avec un terminal et Git.

**Ce qu'il faut** :
- le lab [Préparer son poste](https://byte-layers.com/fr/labs/ai-agent-workstation) terminé : dsh 0.2.2-rc.3 est installé, et son profil `web` utilise votre modèle Ollama ;
- le lab [Hooks : demander ou garantir](https://byte-layers.com/fr/labs/agent-hooks) terminé : le projet `~/byte-layers/ai/todo-api` a une branche `hooks`, avec la configuration d'OpenCode et les hooks.

**Vos fichiers** vivent dans `~/byte-layers/ai/todo-api`, sur une nouvelle branche `port-dsh`. Le dossier est gardé à la fin.

**Sous Windows**, tapez les commandes dans le terminal **Git Bash**.
