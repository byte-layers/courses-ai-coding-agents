---
title: "Une configuration d'agent portable d'abord"
description: "Lab : sur un petit projet Node.js, écrire les instructions du projet dans AGENTS.md, une consigne de sous-dossier, une skill au format standard, puis les permissions d'un harness, OpenCode, et vérifier qu'il les lit."
tags:
  - lab
  - ai
difficulty: beginner
duration: 45
introTitle: "Le lab"
requires: "0.4.0"
options:
  allowManualProgress: false
  autoCompleteOnNext: unverifiable
---

Dans ce lab, vous préparez un projet pour qu'un agent de code y travaille bien, quel que soit l'outil. Vous écrivez d'abord ce qui est **portable**, lu par la plupart des harness : les instructions du projet dans `AGENTS.md`, une consigne propre au dossier des tests, un dossier pour les spécifications et une skill au format `SKILL.md`. Vous écrivez ensuite ce qui est **propre à un outil** : les permissions d'OpenCode, un harness open source en ligne de commande. Pour finir, OpenCode lui-même confirme qu'il lit votre skill et vos règles.

**Niveau** : à l'aise avec un terminal et un éditeur de texte. Vous savez ce que sont les sept briques de configuration d'un agent, ce qu'explique [Les sept briques de configuration d'un agent](https://byte-layers.com/fr/bits/agent-config-building-blocks), et où chaque outil les range, ce que montre [Où chaque harness range sa configuration](https://byte-layers.com/fr/bits/agent-config-crosswalk).

**Ce qu'il faut** :
- Git, et Node.js 22.9 ou plus récent, installé comme dans le lab [Préparer son poste](https://byte-layers.com/fr/labs/ai-agent-workstation) ;
- une connexion internet, pour récupérer le projet et installer OpenCode.

Aucun modèle n'est nécessaire : ce lab écrit et vérifie des fichiers de configuration, il ne lance pas l'agent.

**Vos fichiers** vivent dans `~/byte-layers/ai/todo-api`. Le dossier est gardé à la fin : les labs suivants du stack repartent de ce projet configuré.

**Sous Windows**, tapez les commandes dans le terminal **Git Bash**, livré avec [Git pour Windows](https://git-scm.com/download/win).
