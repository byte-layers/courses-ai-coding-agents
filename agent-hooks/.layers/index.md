---
title: "Hooks : demander ou garantir"
description: "Lab : écrire les hooks qui protègent le fichier .env et vérifient la syntaxe après chaque écriture, en plugin OpenCode puis au format Claude Code, et éviter le piège du code de sortie 1."
tags:
  - lab
  - ai
difficulty: intermediate
duration: 45
introTitle: "Le lab"
requires: "0.4.0"
options:
  allowManualProgress: false
  autoCompleteOnNext: unverifiable
---

Une règle de permission `ask` **demande** : elle compte sur vous pour refuser au bon moment. Un hook **garantit** : c'est votre propre code, que le harness lance à chaque appel d'outil, et qui peut refuser l'appel quoi qu'en pense le modèle. Dans ce lab, vous écrivez deux hooks pour le projet `todo-api` : l'un bloque toute lecture du fichier `.env`, l'autre vérifie la syntaxe de chaque fichier JavaScript que l'agent écrit avec ses outils d'écriture. Vous les écrivez d'abord en plugin OpenCode, puis le premier au format de Claude Code, que plusieurs autres outils lisent aussi.

Le lab n'a pas besoin de modèle : chaque hook se teste en lui envoyant un faux appel d'outil, exactement celui que le harness lui enverrait. Si votre modèle local tourne, des essais réels sont proposés en plus.

**Niveau** : à l'aise avec un terminal, Git, et la lecture d'un peu de JavaScript.

**Ce qu'il faut** : le lab [Une configuration d'agent portable d'abord](https://byte-layers.com/fr/labs/portable-agent-config) terminé. Le projet `~/byte-layers/ai/todo-api` est sur votre poste, avec la branche `agent-config` et son `opencode.json`, et OpenCode est installé.

**Vos fichiers** vivent dans `~/byte-layers/ai/todo-api`, sur une nouvelle branche `hooks`. Le dossier est gardé à la fin.

**Sous Windows**, tapez les commandes dans le terminal **Git Bash**.
