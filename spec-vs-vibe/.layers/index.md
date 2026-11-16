---
title: "Spécification contre vibe coding"
description: "Lab : faire coder deux fois la même fonctionnalité par un agent, une fois à partir de l'issue seule, une fois à partir d'une spécification, puis départager les deux versions avec les mêmes tests d'acceptation."
tags:
  - lab
  - ai
difficulty: intermediate
duration: 60
introTitle: "Le lab"
requires: "0.4.0"
options:
  allowManualProgress: false
  autoCompleteOnNext: unverifiable
---

Le *vibe coding*, c'est demander à un agent ce qu'on veut en une phrase, et accepter ce qui marche à peu près. La spécification, c'est écrire d'abord ce qu'on veut, précisément, puis faire coder. Dans ce lab, vous essayez les deux sur la même fonctionnalité, l'export des tâches de `todo-api`, avec le même agent et le même modèle, chacun sur sa branche. Un même juge les départage : des tests d'acceptation gardés hors des deux branches. Ce juge n'est pas neutre, et c'est voulu : il est écrit d'après la spécification, c'est-à-dire d'après ce que l'équipe attend. L'essai au feeling est donc noté sur des critères qu'il n'a jamais reçus, exactement comme un code livré sans spécification est jugé par un client qui avait ses attentes.

**Niveau** : à l'aise avec un terminal et Git.

**Ce qu'il faut** :
- le lab [De l'issue floue à la spécification](https://byte-layers.com/fr/labs/issue-to-spec) terminé : le projet `~/byte-layers/ai/todo-api` a une branche `spec/export` avec `requirements.md`, `design.md` et `tasks.md` ;
- OpenCode configuré pour votre modèle local, comme au lab [Un modèle, deux harness](https://byte-layers.com/fr/labs/one-model-two-harnesses).

Avec un petit modèle sur un poste sans carte graphique, chaque essai peut prendre de longues minutes, voire ne pas aboutir. Le lab prévoit ce cas : chaque étape donne un résultat de secours, pour que vous puissiez continuer. Vous noterez à chaque fois le score de l'agent **avant** tout secours ou correction, pour que la comparaison finale porte sur ce que l'agent a réellement produit.

**Vos fichiers** vivent dans `~/byte-layers/ai/todo-api`, sur deux nouvelles branches, `export-vibe` et `export-spec`, et dans `~/byte-layers/ai/acceptance`. Les dossiers sont gardés à la fin.

**Sous Windows**, tapez les commandes dans le terminal **Git Bash**.
