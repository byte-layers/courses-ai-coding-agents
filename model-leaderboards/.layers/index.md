---
title: "Lire un classement de modèles"
description: "Benchmarks de tâches, votes, indices et usage : ce que mesure chaque classement de modèles, pourquoi ils se contredisent, et comment lire une fiche de modèle avant de choisir."
tags:
  - bit
  - ai
difficulty: beginner
duration: 14
requires: "0.4.0"
---

Chaque semaine, un nouveau modèle se dit « numéro un ». Il l'est souvent, mais sur un classement précis, qui mesure une chose précise. Ce bit vous apprend à lire ces classements pour choisir un modèle de code, au lieu de suivre le dernier titre.

**Pour qui** : un développeur qui sait ce qu'est un agent de code, un modèle et un harness, ce qu'explique [La boucle d'un agent de code](https://byte-layers.com/fr/bits/agent-loop). Aucune connaissance en statistiques n'est supposée.

> 📅 **Classements vus le 15 novembre 2026.** Les noms de modèles et les positions changent toutes les semaines ; la méthode de lecture, elle, reste valable.

# Lire un classement de modèles

## 1 - Quatre façons de classer des modèles

Un classement répond à une question. Il y en a quatre grandes familles, qui ne posent pas la même question :

| Famille | Question posée | Exemples | Comment le score est produit |
|---|---|---|---|
| **Benchmark de tâches** | le modèle réussit-il ce travail ? | SWE-bench, Terminal-Bench, Aider polyglot | on donne des tâches avec une solution vérifiable (des tests), on compte le pourcentage réussi |
| **Vote humain** | quelle réponse les gens préfèrent-ils ? | Arena (anciennement LMArena) | des visiteurs comparent à l'aveugle deux réponses et votent ; les votes donnent une note |
| **Indice agrégé** | qui est le meilleur en moyenne ? | Artificial Analysis | un organisme indépendant fait passer plusieurs benchmarks dans les mêmes conditions et les combine |
| **Usage réel** | qu'utilisent les développeurs ? | OpenRouter | on compte les tokens consommés par modèle sur OpenRouter, un service qui donne accès à des centaines de modèles par une seule API |

Pour du code, les benchmarks de tâches sont les plus proches de votre travail :

- **SWE-bench** donne à résoudre de vrais tickets de projets open source sur GitHub ; un ticket compte comme résolu si les tests du projet passent. Sa variante **Verified** est un sous-ensemble de 500 tickets relus par des humains.
- **Terminal-Bench** fait accomplir des tâches dans un terminal (installer, configurer, déboguer), ce qui ressemble au travail d'un agent.
- **Aider polyglot**, publié par l'équipe du harness Aider, fait résoudre 225 exercices dans six langages ; il n'est plus mis à jour (voir la section 3).

## 2 - Pourquoi les classements se contredisent

Le 15 novembre 2026, le modèle le plus utilisé de la semaine sur OpenRouter, DeepSeek V4.1 Flash, n'arrivait qu'autour de la 18ᵉ place de l'*Intelligence Index* d'Artificial Analysis. À l'inverse, le premier de cet indice, Claude Fable 5.2 Max, n'apparaissait pas dans les dix modèles les plus utilisés sur OpenRouter. Aucun des deux classements ne se trompe : ils ne mesurent pas la même chose. Le tableau traduit chaque signal :

| Ce que vous lisez | Ce que cela dit vraiment |
|---|---|
| beaucoup d'usage | le modèle est bon marché, rapide, ou proposé gratuitement ; pas forcément le meilleur |
| un bon score à un benchmark | le modèle réussit **ces** tâches, **dans ces conditions** |
| une bonne note au vote | les réponses plaisent, ce qui dépend aussi de leur style et de leur longueur |

Le prix compte autant que la qualité : dans les mesures d'Artificial Analysis, le coût moyen d'une de leurs tâches de test était ce jour-là d'environ 0,60 $ avec le modèle le plus utilisé, contre plus de 26 $ avec le premier de l'indice. Pour un agent qui enchaîne les requêtes, l'écart est décisif.

## 3 - Quatre pièges à connaître

**La contamination.** Un modèle est entraîné sur d'énormes quantités de textes publics. Si les tâches d'un benchmark, ou leurs solutions, sont en ligne depuis longtemps, le modèle a pu les voir pendant son entraînement : il récite au lieu de résoudre. C'est pour cela que des benchmarks comme LiveCodeBench ajoutent sans cesse de nouveaux problèmes, publiés après l'entraînement des modèles testés.

**Le harness compte autant que le modèle.** Un score d'agent mesure un modèle **et** le harness qui le fait travailler. Le même modèle obtient des scores différents selon les outils et les consignes qu'on lui donne. Les classements sérieux le disent : SWE-bench propose une vue *Bash Only* où tous les modèles passent par le même harness minimal, et Terminal-Bench affiche une colonne *Agent* à côté de la colonne *Model*.

**Le résultat annoncé par l'entreprise.** Une entreprise qui publie le score de son propre modèle choisit ses conditions. Préférez les résultats mesurés par le classement lui-même ou par un tiers, comme Artificial Analysis, qui fait passer les tests dans des conditions identiques pour tous.

**Le classement abandonné.** Vérifiez la date de la dernière mise à jour. Le classement d'Aider, par exemple, n'a plus de nouvelle entrée depuis octobre 2025 : ses meilleurs scores datent d'une génération de modèles dépassée.

## 4 - Lire la fiche d'un modèle sur Hugging Face

Un modèle est un très grand ensemble de nombres, ses **paramètres** (ou **poids**), fixés pendant l'entraînement. Hugging Face est le site de référence des modèles à **poids ouverts**, ceux dont on peut télécharger ces nombres pour faire tourner le modèle soi-même (avec Ollama, par exemple). Chaque modèle y a une fiche (*model card*). Pour un usage local ou en entreprise, cherchez-y :

| Information | Où la trouver | Pourquoi elle compte |
|---|---|---|
| **Licence** | en haut de la fiche (`license`) | certaines licences interdisent l'usage commercial ou imposent des conditions |
| **Taille** | le nombre de paramètres, souvent dans le nom (`4B`, `27B`) | elle fixe la mémoire nécessaire en local |
| **Quantification** | les variantes proposées : Q4, Q8… (le nombre de bits par paramètre), dans des formats de fichier comme GGUF ou MLX | moins de bits par paramètre, c'est moins de mémoire, au prix d'un peu de qualité |
| **Modèle de base** | `base_model` | dit si c'est un modèle original, un réglage fin d'un autre (ré-entraîné sur des données particulières) ou une version compressée |
| **Appel d'outils, fenêtre de contexte** | la description du modèle | sans appel d'outils, pas d'agent ; sans assez de contexte, pas de longues tâches |

Un modèle **fermé**, au contraire, ne s'utilise que par l'API de l'entreprise qui le publie, ou des clouds partenaires : vous n'avez ni ses poids ni sa fiche détaillée, et votre code part chez lui.

## 5 - Une méthode en quatre temps

1. **Filtrer par vos contraintes** avant tout classement : où le code a-t-il le droit d'aller, quelle licence, quel budget, quel matériel pour du local.
2. **Regarder les benchmarks proches de votre usage** : pour un agent, Terminal-Bench et SWE-bench plutôt qu'un vote sur des réponses de chatbot.
3. **Lire les conditions** : la date, le harness utilisé, et qui a mesuré.
4. **Essayer sur votre propre tâche**, avec votre harness : c'est le seul classement qui compte vraiment pour vous.

## Récap

- Il y a quatre familles de classements : **tâches**, **votes**, **indices agrégés**, **usage**. Chacune répond à une question différente.
- L'usage mesure le prix et l'adoption, pas la qualité.
- Méfiez-vous de la **contamination**, des scores qui mélangent modèle et **harness**, des résultats **auto-déclarés** et des classements **abandonnés**.
- Sur Hugging Face, la fiche d'un modèle ouvert donne sa **licence**, sa **taille** et ses **quantifications**.
- Finissez toujours par un essai sur votre propre tâche.

> 📖 [SWE-bench](https://www.swebench.com/) · [Terminal-Bench](https://www.tbench.ai/leaderboard) · [Arena](https://arena.ai/) · [Artificial Analysis, méthodologie](https://artificialanalysis.ai/methodology/intelligence-benchmarking) · [OpenRouter, classements par usage](https://openrouter.ai/rankings) · [Les fiches de modèles sur Hugging Face](https://huggingface.co/docs/hub/model-cards) · [LiveCodeBench et la contamination (article)](https://arxiv.org/abs/2403.07974)

```quiz
title: Lire un classement de modèles
tracksProgress: true
submitMode: last-only
questions:
  - prompt: Un modèle est le plus utilisé de la semaine sur OpenRouter. Qu'en concluez-vous ?
    answers:
      - "C'est le meilleur modèle pour coder"
      - "Il est très utilisé, sans doute parce qu'il est bon marché ou rapide ; sa qualité reste à vérifier ailleurs"
      - "Il a le meilleur score à SWE-bench"
      - "Il est forcément open source"
    correct: 2
    explanation: |
      OpenRouter compte les tokens consommés. L'usage reflète le prix, la vitesse et l'adoption,
      pas la qualité mesurée sur des tâches.
  - prompt: Vous choisissez un modèle pour un agent qui corrige des bugs dans un dépôt. Quel classement regardez-vous en priorité ?
    answers:
      - "Un benchmark de tâches proche de ce travail, comme SWE-bench ou Terminal-Bench"
      - "Le vote humain d'Arena sur des réponses de chatbot"
      - "Le nombre de téléchargements sur Hugging Face"
      - "Le classement par usage d'OpenRouter"
    correct: 1
    explanation: |
      Un benchmark de tâches vérifiées par des tests mesure ce que vous attendez de l'agent :
      résoudre un vrai problème dans un dépôt.
  - prompt: Pourquoi des benchmarks comme LiveCodeBench ajoutent-ils sans cesse de nouveaux problèmes ?
    answers:
      - "Pour rendre le test plus difficile chaque mois"
      - "Pour suivre les nouveaux langages"
      - "Pour éviter la contamination : des problèmes publiés après l'entraînement n'ont pas pu être appris par cœur"
      - "Parce que les anciens problèmes contenaient des erreurs"
    correct: 3
    explanation: |
      Un modèle a pu voir pendant son entraînement les tâches et solutions publiées depuis
      longtemps. Des problèmes récents mesurent la résolution, pas la mémoire.
  - prompt: Deux classements d'agents donnent des scores différents au même modèle. Quelle explication est la plus probable ?
    answers:
      - "L'un des deux classements est truqué"
      - "Le modèle a changé de nom"
      - "Les tests ont été lancés à des heures différentes"
      - "Le modèle n'a pas été testé avec le même harness ni les mêmes consignes"
    correct: 4
    explanation: |
      Un score d'agent mesure un modèle et son harness. C'est pour cela que SWE-bench a une vue
      Bash Only et que Terminal-Bench affiche l'agent à côté du modèle.
  - prompt: Sur la fiche Hugging Face d'un modèle ouvert, quelle information vous dit si vous pouvez l'utiliser dans un produit commercial ?
    answers:
      - "Le nombre de paramètres"
      - "La licence"
      - "La quantification"
      - "Le nombre de téléchargements"
    correct: 2
    explanation: |
      La licence fixe les conditions d'usage. Certaines interdisent l'usage commercial ou
      imposent des obligations.
```
