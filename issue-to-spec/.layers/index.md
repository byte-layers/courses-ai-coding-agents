---
title: "De l'issue floue à la spécification"
description: "Lab : transformer une issue vague en spécification écrite (exigences EARS, conception, tâches), y repérer l'exigence réglementaire que personne n'a écrite, et la proposer dans une merge request sur gitlab.com."
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

Une issue arrive rarement prête à coder. Celle de ce lab demande un export des tâches, en deux phrases. Elle ne dit ni le format, ni ce qu'il faut exporter, et elle tait une obligation légale que le développeur doit repérer seul. Vous allez la transformer en spécification, les trois fichiers `requirements.md`, `design.md` et `tasks.md`, puis la proposer à l'équipe dans une merge request. La spécification sera relue avant qu'une seule ligne de code soit écrite : c'est le principe du développement piloté par les spécifications.

**Niveau** : à l'aise avec un terminal et Git, avec un compte sur gitlab.com.

**Ce qu'il faut** :
- le bit [Le développement piloté par les spécifications et EARS](https://byte-layers.com/fr/bits/spec-driven-development) lu : les trois fichiers d'une spécification et les cinq motifs EARS ;
- le lab [Une configuration d'agent portable d'abord](https://byte-layers.com/fr/labs/portable-agent-config) terminé : le projet `~/byte-layers/ai/todo-api` a sa branche `agent-config`, avec la skill `ears-spec` ;
- un compte sur **gitlab.com**, et une façon de pousser du code depuis votre poste : une clé SSH enregistrée sur votre compte, ou un jeton d'accès personnel.

Un agent de code peut vous aider à écrire un premier jet des exigences, si vous avez fait le lab [Un modèle, deux harness](https://byte-layers.com/fr/labs/one-model-two-harnesses) : OpenCode y est relié à un modèle local. Il n'est pas indispensable : la relecture, qui est le cœur du lab, se fait de toute façon à la main.

**Une simplification** : le projet `todo-api` n'a pas de comptes utilisateurs. On considère que chaque client a sa propre instance du service, si bien que toutes les tâches d'une instance sont les siennes. Dans un vrai service partagé, « ses tâches » voudrait dire « celles du client authentifié », et l'exporter tout entier livrerait les données des autres clients.

**Vos fichiers** vivent dans `~/byte-layers/ai/todo-api`, sur une nouvelle branche `spec/export`, poussée sur votre projet gitlab.com.

**Sous Windows**, tapez les commandes dans le terminal **Git Bash**.
