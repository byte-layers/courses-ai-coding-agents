---
title: "Un modèle, deux harness"
description: "Lab : confier la même tâche au même modèle local, d'abord dans DeepSeek Harness puis dans OpenCode, et comparer ce que chaque harness change."
tags:
  - lab
  - ai
difficulty: beginner
duration: 50
introTitle: "Le lab"
requires: "0.4.0"
options:
  allowManualProgress: false
  autoCompleteOnNext: unverifiable
---

Dans ce lab, vous donnez la même tâche, faire passer un test, au même modèle local `qwen3.5:4b`, dans deux harness : DeepSeek Harness (`dsh`), puis OpenCode. Chaque harness travaille sur sa propre branche Git, ce qui permet ensuite de comparer les deux résultats ligne à ligne. Le modèle ne change pas : tout ce qui diffère vient du harness.

**Niveau** : à l'aise avec un terminal, et avec Git pour créer une branche et commiter.

**Ce qu'il faut** :
- le lab [Préparer son poste](https://byte-layers.com/fr/labs/ai-agent-workstation) terminé : Ollama sert `qwen3.5:4b` avec une fenêtre de 16k, et dsh est configuré pour l'utiliser ;
- le lab [Une configuration d'agent portable d'abord](https://byte-layers.com/fr/labs/portable-agent-config) terminé : le projet `~/byte-layers/ai/todo-api` est configuré sur la branche `agent-config`, et OpenCode est installé ;
- un poste de **16 Go** de mémoire vive. Sans carte graphique ni puce Apple, un petit modèle est très lent : une tâche peut prendre de longues minutes par harness, voire ne pas aboutir. Le lab prévoit ce cas.

**Vos fichiers** vivent dans `~/byte-layers/ai/todo-api`. Le dossier est gardé à la fin, avec les deux branches du lab.

**Sous Windows**, tapez les commandes dans le terminal **Git Bash**.
