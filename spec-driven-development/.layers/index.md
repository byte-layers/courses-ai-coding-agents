---
title: "Le développement piloté par les spécifications et EARS"
description: "Donner à un agent de code une spécification écrite avant de le laisser coder : les trois documents (exigences, conception, tâches) de Kiro et de Spec Kit, la syntaxe EARS pour des exigences testables, les trois niveaux d'engagement et les limites de la méthode."
tags:
  - bit
  - ai
difficulty: beginner
duration: 13
requires: "0.4.0"
---

Un agent de code fait ce qu'on lui demande, au mieux de ce qu'il a compris. Ce bit montre comment lui donner, avant qu'il code, une demande écrite qu'il peut suivre et que vous pouvez vérifier : une **spécification**, et une façon d'en écrire les exigences, EARS.

**Pour qui** : un développeur qui a déjà demandé une modification à un agent de code et qui connaît les mots modèle, harness et contexte, expliqués dans [La boucle d'un agent de code](https://byte-layers.com/fr/bits/agent-loop). Kiro et GitHub Copilot sont présentés dans [Le paysage des harness de code](https://byte-layers.com/fr/bits/coding-harness-landscape).

# Le développement piloté par les spécifications et EARS

Tous les exemples portent sur la même petite application : une **API de tâches** (une *todo API*), qui permet de créer, lister, terminer et supprimer des tâches par des requêtes web.

## 1 - Le problème : coder « aux vibes » ou partir d'une spécification

Le terme **vibe coding** (coder « aux vibes », au ressenti) a été lancé par Andrej Karpathy, chercheur en IA et cofondateur d'OpenAI, dans un message publié sur X le 2 février 2025. Il désigne une façon de programmer où l'on décrit ce qu'on veut à un agent en langage courant, où l'on accepte le code produit sans le relire de près, et où l'on corrige en relançant des demandes. Pour un prototype jetable, cela va vite. Pour du code qu'on devra maintenir, la demande n'existe que dans une conversation : personne ne peut dire ensuite ce qui était voulu.

Le **développement piloté par les spécifications** (*spec-driven development*, SDD) inverse l'ordre : vous écrivez d'abord, dans des fichiers du dépôt, ce que le logiciel doit faire, puis l'agent code à partir de ce texte. Une **spécification** (*spec*) est ce document : une description écrite et précise du comportement attendu.

Une spécification aide un agent pour trois raisons, résumées dans le tableau :

| Raison | Ce que la spécification apporte |
|---|---|
| **Le contexte** | l'agent relit un fichier stable à chaque tour, au lieu de reconstituer votre intention à partir d'une conversation qui s'allonge |
| **Des critères d'acceptation vérifiables** | chaque exigence dit ce qu'on doit observer pour la considérer remplie ; l'agent peut en tirer des tests, et les tests disent quand il a fini |
| **La relecture** | vous relisez une demande de quelques lignes avant que le code existe, et vous comparez ensuite le code à ce texte plutôt qu'à vos souvenirs |

## 2 - Les trois documents, dans l'ordre

Une spécification se découpe en trois documents, écrits l'un après l'autre parce que chacun s'appuie sur le précédent :

1. les **exigences** (*requirements*) disent **quoi** : ce que le système doit faire, du point de vue de l'utilisateur, sous une forme testable ;
2. la **conception** (*design*) dit **comment** : l'architecture, les données, les composants et leurs échanges ;
3. les **tâches** (*tasks*) découpent la conception en **petites étapes ordonnées**, que l'agent réalise et que vous cochez une à une.

On relit et on valide chaque document avant de passer au suivant : une erreur dans les exigences coûte une phrase à corriger, la même erreur découverte dans le code coûte une tâche à refaire.

Deux outils répandus suivent ce découpage. **Kiro**, un éditeur lié à AWS, range une spécification dans trois fichiers, `requirements.md`, `design.md` et `tasks.md`, et sa documentation y écrit les exigences avec une syntaxe nommée EARS. **Spec Kit**, une boîte à outils libre publiée par GitHub, ajoute des commandes à votre agent de code (GitHub Copilot et d'autres) ; chaque commande produit un fichier dans le dépôt.

Le tableau met les deux côte à côte, phase par phase :

| Phase | Kiro | Spec Kit |
|---|---|---|
| Principes du projet, écrits une fois | (hors des specs) | commande `constitution`, fichier `memory/constitution.md` |
| Exigences : quoi | `requirements.md` | commande `specify`, fichier `spec.md` |
| Conception : comment | `design.md` | commande `plan`, fichier `plan.md` (et des documents annexes : modèle de données, contrats d'API) |
| Tâches | `tasks.md` | commande `tasks`, fichier `tasks.md` |
| Réalisation | l'agent exécute les tâches de `tasks.md` | commande `implement` |

Dans Spec Kit, les commandes portent le préfixe `speckit` et s'appellent selon la syntaxe de votre agent : la documentation montre par exemple `/speckit-specify` pour GitHub Copilot, et `/speckit.specify` dans d'autres pages. Vérifiez la forme exacte dans la documentation de l'outil installé, qui évolue vite.

## 3 - EARS : des exigences écrites avec des mots-clés fixes

**EARS** (*Easy Approach to Requirements Syntax*, « une approche simple de la syntaxe des exigences ») a été mise au point par Alistair Mavin et ses collègues chez Rolls-Royce, en analysant les règles de navigabilité du système de contrôle d'un moteur d'avion, et publiée pour la première fois en 2009. Elle propose cinq modèles de phrase, chacun reconnaissable à son mot-clé, et une façon de les combiner.

Le modèle de base est la phrase **toujours vraie** : « Le <système> doit <réponse du système> ». Les autres modèles ajoutent devant elle une condition, introduite par un mot-clé.

Le tableau donne chaque modèle avec le gabarit anglais d'origine, une adaptation française et un exemple sur l'API de tâches :

| Modèle | Gabarit anglais | Adaptation française | Exemple sur l'API de tâches |
|---|---|---|---|
| **Universel** (*ubiquitous*), toujours actif, sans mot-clé | The <system> shall <response> | Le <système> doit <réponse> | L'API de tâches doit renvoyer ses réponses au format JSON. |
| **Événementiel** (*event-driven*), mot-clé *When* | When <trigger>, the <system> shall <response> | Quand <déclencheur>, le <système> doit <réponse> | Quand un client envoie une tâche avec un titre, l'API de tâches doit la créer et renvoyer le code 201. |
| **Lié à un état** (*state-driven*), mot-clé *While* | While <precondition>, the <system> shall <response> | Pendant que <précondition>, le <système> doit <réponse> | Pendant que la base de données est en maintenance, l'API de tâches doit refuser les écritures avec le code 503. |
| **Comportement indésirable** (*unwanted behaviour*), mots-clés *If* et *then* | If <trigger>, then the <system> shall <response> | Si <déclencheur>, alors le <système> doit <réponse> | Si une tâche est envoyée sans titre, alors l'API de tâches doit renvoyer le code 400 et le message « titre obligatoire ». |
| **Fonctionnalité optionnelle** (*optional feature*), mot-clé *Where* | Where <feature is included>, the <system> shall <response> | Là où <la fonctionnalité est présente>, le <système> doit <réponse> | Là où le module d'étiquettes est activé, l'API de tâches doit permettre de filtrer les tâches par étiquette. |

Une exigence peut combiner plusieurs mots-clés, dans l'ordre état, puis événement : c'est le modèle **complexe**. Par exemple : « Pendant que l'utilisateur est connecté, quand il supprime une tâche, l'API de tâches doit la retirer de sa liste. » Un comportement indésirable peut aussi se combiner avec un état.

Des mots-clés fixes rendent une exigence **testable**, parce que chaque morceau de la phrase devient un morceau du test :

| Morceau de la phrase EARS | Morceau du test |
|---|---|
| *Pendant que* <précondition> | la préparation : mettre le système dans cet état |
| *Quand* ou *Si* <déclencheur> | l'action : envoyer la requête, provoquer l'erreur |
| le <système> *doit* <réponse> | la vérification : le code, le message, la donnée attendus |

Un agent qui lit « Si une tâche est envoyée sans titre, alors l'API doit renvoyer le code 400 » sait écrire le test sans rien deviner : envoyer une tâche sans titre, vérifier le code 400. À l'inverse, « l'API doit gérer correctement les erreurs » ne dit ni quelle erreur ni quel résultat : ni l'agent ni vous ne pouvez dire quand c'est fait.

## 4 - Trois niveaux d'engagement

Écrire une spécification ne dit pas encore ce qu'on en fait une fois le code écrit. Birgitta Böckeler (Thoughtworks) distingue trois niveaux, dans un article de martinfowler.com d'octobre 2025 qui compare Kiro, Spec Kit et Tessl. Le tableau les résume avec leurs compromis :

| Niveau | Ce que ça veut dire | Ce qu'on y gagne | Ce que ça coûte |
|---|---|---|---|
| **Spec-first** (la spec d'abord) | on écrit la spécification avant de coder la tâche ; une fois la tâche finie, on ne la maintient plus | une demande claire pour l'agent, sans charge sur la durée | la spécification devient vite fausse : elle raconte le passé, pas le code actuel |
| **Spec-anchored** (le code ancré à la spec) | on garde la spécification après la tâche, et on la met à jour pour faire évoluer la fonctionnalité | une description du comportement qui reste vraie et sert aux évolutions suivantes | chaque changement demande de modifier deux choses, le texte et le code |
| **Spec-as-source** (la spec comme source) | la spécification est le document principal : on ne modifie qu'elle, et le code est régénéré à partir d'elle sans y toucher à la main | une seule source à relire et à modifier | le même texte peut produire un code différent à chaque génération ; on dépend entièrement de l'outil |

## 5 - Les limites de la méthode

Une spécification a un coût, et trois limites reviennent dans les retours d'expérience, dont l'article cité plus haut :

- **La sur-spécification.** Un processus en trois documents est trop lourd pour un petit correctif : pour changer un message d'erreur, écrire exigences, conception et tâches prend plus de temps que la correction. On réserve la méthode aux fonctionnalités qui en valent la peine.
- **La dérive entre la spec et le code.** Dès qu'on corrige le code sans toucher au texte, ou l'inverse, les deux ne disent plus la même chose, et l'agent qui relit la spécification travaille sur une description fausse.
- **Le coût de la relecture.** Un agent produit facilement de longs documents répétitifs. Relire trois fichiers Markdown verbeux peut coûter plus cher que relire le code, et une spécification qu'on ne relit pas donne une fausse impression de contrôle : l'agent peut aussi ignorer une consigne écrite.

## Récap

- Le **vibe coding** accepte le code de l'agent sans le relire ; le **développement piloté par les spécifications** écrit d'abord, dans le dépôt, ce que le logiciel doit faire.
- Une spécification se découpe en **exigences** (quoi), **conception** (comment) et **tâches** (petites étapes ordonnées), validées dans cet ordre. Kiro les range dans `requirements.md`, `design.md` et `tasks.md` ; Spec Kit les produit avec ses commandes `specify`, `plan` et `tasks`.
- **EARS** écrit chaque exigence avec un mot-clé fixe : aucun (universel), *Quand* (événement), *Pendant que* (état), *Si… alors* (comportement indésirable), *Là où* (fonctionnalité optionnelle), ou une combinaison. Chaque morceau de la phrase devient un morceau du test.
- Trois niveaux d'engagement : **spec-first**, **spec-anchored**, **spec-as-source**.
- Limites : la **sur-spécification**, la **dérive** entre texte et code, le **coût de la relecture**.

> 📖 [EARS, par Alistair Mavin](https://alistairmavin.com/ears/) · [Les specs dans Kiro](https://kiro.dev/docs/specs/) · [Les specs de fonctionnalité dans Kiro](https://kiro.dev/docs/specs/feature-specs/) · [Spec Kit (GitHub)](https://github.com/github/spec-kit) · [La méthode de Spec Kit](https://github.com/github/spec-kit/blob/main/spec-driven.md) · [Understanding Spec-Driven-Development: Kiro, spec-kit, and Tessl (Birgitta Böckeler)](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) · [Vibe coding (Wikipédia)](https://en.wikipedia.org/wiki/Vibe_coding)

```quiz
title: Le développement piloté par les spécifications et EARS
tracksProgress: true
submitMode: last-only
questions:
  - prompt: Qu'est-ce qui distingue le vibe coding du développement piloté par les spécifications ?
    answers:
      - "Le vibe coding n'utilise pas de modèle de langage"
      - "Le vibe coding accepte le code produit sans le relire de près, alors que le SDD écrit d'abord ce que le logiciel doit faire"
      - "Le SDD interdit à l'agent d'écrire du code"
      - "Le SDD ne fonctionne qu'avec Kiro"
    correct: 2
    explanation: |
      En vibe coding, la demande n'existe que dans la conversation et le code est accepté au
      ressenti. Le SDD écrit d'abord une spécification dans le dépôt, puis l'agent code à partir d'elle.
  - prompt: Dans quel ordre écrit-on les trois documents d'une spécification ?
    answers:
      - "Tâches, conception, exigences"
      - "Conception, exigences, tâches"
      - "Exigences, tâches, conception"
      - "Exigences, conception, tâches"
    correct: 4
    explanation: |
      Les exigences disent quoi, la conception dit comment, et les tâches découpent la
      conception en petites étapes : chacun s'appuie sur le précédent.
  - prompt: "Quel modèle EARS reconnaissez-vous dans : « Pendant que la base de données est en maintenance, l'API de tâches doit refuser les écritures » ?"
    answers:
      - "Lié à un état"
      - "Événementiel"
      - "Comportement indésirable"
      - "Fonctionnalité optionnelle"
    correct: 1
    explanation: |
      Le mot-clé « Pendant que » (While) introduit une précondition qui dure : c'est le modèle
      lié à un état.
  - prompt: "« Si le titre dépasse 200 caractères, alors l'API de tâches doit renvoyer le code 400. » Que devient « Si le titre dépasse 200 caractères » dans un test ?"
    answers:
      - "La vérification du résultat"
      - "La préparation de l'état du système"
      - "L'action qui déclenche le comportement : envoyer un titre de plus de 200 caractères"
      - "Rien : cette partie ne se teste pas"
    correct: 3
    explanation: |
      Le déclencheur introduit par « Si » ou « Quand » devient l'action du test ; ce que le
      système « doit » faire devient la vérification.
  - prompt: Une équipe écrit une spécification avant chaque tâche, puis ne la met plus jamais à jour. Quel niveau d'engagement est-ce ?
    answers:
      - "Spec-as-source"
      - "Spec-first"
      - "Spec-anchored"
      - "Aucun : ce n'est pas du SDD"
    correct: 2
    explanation: |
      En spec-first, la spécification guide la tâche en cours puis n'est plus maintenue.
      En spec-anchored on la garde à jour ; en spec-as-source, on ne modifie qu'elle.
```
