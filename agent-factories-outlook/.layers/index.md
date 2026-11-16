---
title: "Conteneurs de développement, machines distantes et usines d'agents"
description: "Où va le poste du développeur quand l'agent travaille seul : le conteneur de développement comme environnement et comme cage, les machines de développement distantes, les agents dans le cloud, et l'usine d'agents qui assemble tout ce que le stack a construit."
tags:
  - bit
  - ai
difficulty: beginner
duration: 15
requires: "0.4.0"
---

Tout au long de ce stack, l'agent a travaillé sur votre poste, sous vos yeux : vous lanciez la demande, vous lisiez ce qu'il faisait, vous commitiez. Ce bit regarde la suite. L'agent travaille de plus en plus **sans vous** : dans un conteneur, sur une machine distante, puis dans le cloud, plusieurs à la fois. Chaque étape demande un environnement plus isolé et plus reproductible, et des garde-fous qui ne dépendent plus de votre attention.

**Pour qui** : un développeur qui a configuré un agent et l'a fait coder, ce que font les labs [Une configuration d'agent portable d'abord](https://byte-layers.com/fr/labs/portable-agent-config) et [Spécification contre vibe coding](https://byte-layers.com/fr/labs/spec-vs-vibe).

> 📅 **Sources vérifiées le 16 novembre 2026.** Les offres des agents dans le cloud changent vite : leurs noms et leurs conditions sont à revérifier.

# Conteneurs de développement, machines distantes et usines d'agents

## 1 - Le conteneur de développement

**Ce que c'est.** Un **conteneur de développement** (*dev container*) est un conteneur qui contient tout ce qu'il faut pour travailler sur un projet : le langage dans la bonne version, les outils, les extensions de l'éditeur. Il est décrit par un fichier, `.devcontainer/devcontainer.json`, commité avec le code. Le format est une spécification ouverte, publiée sur [containers.dev](https://containers.dev/), et lue par VS Code, les IDE JetBrains, GitHub Codespaces et une interface en ligne de commande de référence.

Pour le projet `todo-api` des labs, le fichier tient en quelques lignes :

```json
{
  "name": "todo-api",
  "image": "mcr.microsoft.com/devcontainers/javascript-node:22",
  "postCreateCommand": "npm test",
  "customizations": {
    "vscode": {
      "extensions": ["dbaeumer.vscode-eslint"]
    }
  }
}
```

| Champ | Rôle |
|---|---|
| `image` | l'image de départ : ici Node.js 22, préparée par Microsoft pour les conteneurs de développement |
| `postCreateCommand` | une commande lancée une fois le conteneur créé : ici les tests, qui prouvent que l'environnement marche |
| `customizations.vscode.extensions` | les extensions que VS Code installe dans le conteneur |

Avec ce fichier, chaque développeur, et chaque agent, travaille dans le même environnement : fini « chez moi ça marche ». La commande `devcontainer up --workspace-folder .`, de l'interface `@devcontainers/cli`, construit et démarre ce conteneur sans éditeur ; avec ce fichier, elle lance les sept tests du projet, qui passent.

**Pourquoi ça compte pour un agent.** Le conteneur est aussi une **cage**. L'agent qui y travaille ne voit que le projet et ce que le conteneur contient, pas vos clés SSH ni vos autres dépôts. C'est ce qui permet de lui laisser plus de liberté. La documentation de Claude Code fournit un conteneur de développement de référence, avec un pare-feu qui n'autorise que quelques destinations réseau, pour le faire tourner sans demander d'accord à chaque commande. Elle prévient aussi : même dans ce conteneur, un projet malveillant peut faire sortir tout ce qui s'y trouve, identifiants de l'agent compris. La cage limite les dégâts, elle ne remplace ni les permissions ni la relecture.

## 2 - Les machines de développement distantes

**Ce que c'est.** Le même fichier `devcontainer.json` peut démarrer le conteneur ailleurs que sur votre poste : sur une machine du cloud ou un serveur de l'entreprise. Vous y travaillez depuis votre éditeur ou un navigateur. On parle d'**environnement de développement dans le cloud** (*cloud development environment*, CDE).

| Outil | Où tourne l'environnement | Ce qu'il faut savoir |
|---|---|---|
| GitHub Codespaces | sur les machines de GitHub | lit `devcontainer.json` ; facturé à l'usage au-delà d'un quota gratuit |
| Coder | sur votre infrastructure : machines virtuelles, Kubernetes, Docker | open source (licence AGPL), auto-hébergé ; environnements décrits en Terraform |
| DevPod | là où vous le pointez : votre poste, un serveur, un cloud | open source, un simple client : rien n'est stocké chez un éditeur |
| Ona, ex-Gitpod | sur les machines d'Ona ou dans votre cloud | Gitpod a changé de nom en septembre 2025 pour se tourner vers les agents |

L'intérêt pour une équipe : un nouvel arrivant code en quelques minutes, un poste modeste suffit, et le code reste sur des machines que l'entreprise choisit. Le dernier point rejoint les règles de [L'IA de code en entreprise, en France](https://byte-layers.com/fr/bits/ai-coding-at-work) : savoir où part le code.

## 3 - Les agents dans le cloud

**Ce que c'est.** L'étape suivante supprime l'éditeur : vous confiez une tâche, et l'agent la réalise seul, sur une machine qu'il démarre pour l'occasion. Il revient avec une branche ou une demande de fusion. Les principaux éditeurs proposent ce mode :

| Agent | Point de départ | Où il travaille |
|---|---|---|
| GitHub Copilot, agent cloud (ex-*coding agent*) | une issue assignée à Copilot, ou une demande | un runner GitHub Actions ; il ouvre une pull request, qu'il ne peut ni approuver ni fusionner |
| OpenAI Codex, tâches cloud | une demande dans l'interface ou l'outil en ligne de commande | une machine virtuelle d'OpenAI ; il rend un correctif ou une pull request |
| Google Jules | une demande sur un dépôt GitHub | une machine virtuelle de Google |
| Claude Code, sessions cloud (ex-Claude Code *on the web*) | une demande sur claude.ai, l'application mobile ou le terminal | un environnement géré par Anthropic ; la session continue navigateur fermé |

Ce mode change la relecture. Vous ne voyez plus l'agent travailler : vous jugez le **résultat**, une demande de fusion. Tout ce que vous avez mis en place dans ce stack devient alors indispensable : des instructions dans `AGENTS.md`, des permissions et des hooks qui s'appliquent sans vous, une spécification qui dit ce qui est attendu, des tests qui jugent.

## 4 - L'usine d'agents

**Ce que c'est.** Une **usine d'agents** (*agent factory*) organise plusieurs agents comme une chaîne de production. Le travail entre sous forme d'issues ; chaque agent en prend une, dans son propre environnement isolé ; il produit une demande de fusion ; des contrôles automatiques et des humains la valident avant qu'elle n'entre dans le code. Le terme désigne une organisation, pas un produit : chaque équipe assemble la sienne, avec un orchestrateur, des agents dans le cloud ou des conteneurs, et sa forge.

Une usine reprend, à plus grande échelle, chaque brique de ce stack :

| Brique de l'usine | Ce qui la rend possible | Où vous l'avez pratiquée |
|---|---|---|
| des agents interchangeables | une configuration portable, lue par plusieurs harness | [Une configuration d'agent portable d'abord](https://byte-layers.com/fr/labs/portable-agent-config), [Passer sa configuration à un autre harness](https://byte-layers.com/fr/labs/port-agent-config) |
| des garde-fous sans surveillance | des permissions et des hooks qui garantissent, au lieu de demander | [Hooks : demander ou garantir](https://byte-layers.com/fr/labs/agent-hooks) |
| des tâches bien définies | une issue transformée en spécification relue | [De l'issue floue à la spécification](https://byte-layers.com/fr/labs/issue-to-spec) |
| un contrôle du résultat | des tests d'acceptation écrits avant le code | [Spécification contre vibe coding](https://byte-layers.com/fr/labs/spec-vs-vibe) |
| des environnements isolés | conteneurs de développement, machines distantes | sections 1 et 2 de ce bit |

Deux limites restent entières. La première est le **coût** : plusieurs agents en parallèle consomment beaucoup de jetons, et une usine sérieuse fixe un budget par tâche. La seconde est la **relecture humaine** : une usine produit plus de demandes de fusion qu'une équipe ne peut en relire attentivement. Les contrôles automatiques, l'analyse de code, les tests, les règles de qualité, deviennent le premier filtre. C'est l'objet du stack sur la qualité du code.

## Récap

- Un **conteneur de développement**, décrit par `.devcontainer/devcontainer.json`, donne à chacun, humain ou agent, le même environnement ; c'est aussi une **cage** qui limite ce que l'agent peut atteindre, sans remplacer les permissions.
- Le même fichier démarre l'environnement sur une **machine distante** : Codespaces, Coder, DevPod, Ona.
- Les **agents dans le cloud** travaillent seuls et rendent une demande de fusion : on juge le résultat, pas le travail.
- Une **usine d'agents** assemble tout le stack : configuration portable, garde-fous, spécifications, tests, environnements isolés. Ses limites sont le coût et la capacité de relecture.

> 📖 [La spécification Development Containers (en anglais)](https://containers.dev/implementors/spec/) · [Claude Code : conteneurs de développement (en anglais)](https://code.claude.com/docs/en/devcontainer) · [GitHub Copilot : l'agent cloud (en anglais)](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/create-a-pr) · [Claude Code : sessions cloud (en anglais)](https://code.claude.com/docs/en/claude-code-on-the-web) · [Coder (en anglais)](https://coder.com/docs) · [Gitpod devient Ona (en anglais)](https://ona.com/stories/gitpod-is-now-ona)

```quiz
title: Conteneurs de développement, machines distantes et usines d'agents
tracksProgress: true
submitMode: last-only
questions:
  - prompt: Quel fichier décrit un conteneur de développement, et où se range-t-il ?
    answers:
      - "Un Dockerfile à la racine, jamais commité"
      - "Un fichier devcontainer.json dans le dossier .devcontainer, commité avec le code"
      - "Un fichier AGENTS.md à la racine"
      - "Un fichier opencode.json dans le dossier personnel"
    correct: 2
    explanation: |
      Le format est une spécification ouverte (containers.dev). Le fichier est commité :
      chaque développeur et chaque agent obtiennent le même environnement.
  - prompt: Un agent tourne dans un conteneur de développement, sans demande d'accord à chaque commande. Qu'est-ce que le conteneur garantit ?
    answers:
      - "Rien de plus que sans conteneur"
      - "Que l'agent ne peut faire sortir aucune donnée"
      - "Qu'il limite ce que l'agent atteint : le projet et le contenu du conteneur, pas le reste de votre poste ; mais ce qui est dans le conteneur peut encore fuir"
      - "Que le code produit est correct"
    correct: 3
    explanation: |
      La documentation de Claude Code le dit : la cage limite les dégâts, elle n'empêche pas
      un projet malveillant de faire sortir ce qui est dans le conteneur, identifiants compris.
  - prompt: Quelle différence entre Coder et GitHub Codespaces ?
    answers:
      - "Coder est auto-hébergé sur votre infrastructure ; Codespaces tourne sur les machines de GitHub"
      - "Coder ne lit pas devcontainer.json"
      - "Codespaces est open source, Coder ne l'est pas"
      - "Aucune : ce sont deux noms du même produit"
    correct: 1
    explanation: |
      Les deux démarrent des environnements distants. Coder, open source, tourne sur des
      machines que l'entreprise choisit ; Codespaces sur celles de GitHub.
  - prompt: Un agent dans le cloud vous rend une pull request. Qu'est-ce qui change pour la relecture ?
    answers:
      - "Rien, vous voyez chaque commande au moment où il la lance"
      - "Vous jugez le résultat sans l'avoir vu travailler : instructions, garde-fous, spécification et tests doivent être en place avant"
      - "La pull request est fusionnée automatiquement"
      - "Les tests ne sont plus nécessaires"
    correct: 2
    explanation: |
      Sans vous à côté, ce sont la configuration, les garde-fous automatiques, la spécification
      et les tests qui cadrent le travail. L'agent cloud de Copilot ne peut d'ailleurs ni
      approuver ni fusionner sa pull request.
  - prompt: Quelles sont les deux limites d'une usine d'agents citées dans ce bit ?
    answers:
      - "La vitesse des modèles et la taille des dépôts"
      - "Le coût en jetons et la capacité des humains à relire toutes les demandes de fusion"
      - "Le nombre de langages et le choix de la forge"
      - "L'absence de standard et l'interdiction légale"
    correct: 2
    explanation: |
      Des agents en parallèle coûtent cher, et produisent plus qu'une équipe ne peut relire :
      les contrôles automatiques de qualité deviennent le premier filtre.
```
