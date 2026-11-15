---
title: "Le paysage des harness de code"
description: "Une fiche par outil (Claude Code, OpenCode, Kilo, Pi, DeepSeek Harness, Mistral Vibe, Codex CLI, Gemini CLI, Cursor, Kiro, GitHub Copilot), les standards AGENTS.md et Agent Skills, et les questions qui décident d'un choix."
tags:
  - bit
  - ai
difficulty: beginner
duration: 14
requires: "0.4.0"
---

Il existe des dizaines d'agents de code, et ils changent chaque mois. Plutôt que d'en apprendre un par cœur, ce bit vous donne une grille de lecture et une fiche courte pour les outils les plus répandus, afin que vous sachiez en situer un nouveau en quelques minutes.

**Pour qui** : un développeur qui connaît les mots modèle, harness, contexte et permissions, et les trois formes d'un harness (CLI, éditeur dérivé, extension), ce qu'explique [La boucle d'un agent de code](https://byte-layers.com/fr/bits/agent-loop).

> 📅 **Fiches révisées le 15 novembre 2026.** Les offres, les prix et les fonctionnalités de ces outils changent vite : vérifiez la page officielle avant de décider, et regardez la date de révision de cette page.

# Le paysage des harness de code

## 1 - Cinq questions pour situer un outil

Pour chaque outil, cinq questions suffisent à le situer :

| Question | Pourquoi elle compte |
|---|---|
| **Quelle forme ?** CLI, éditeur dérivé de VS Code, extension | elle dit si vous gardez votre éditeur et si vous pouvez l'automatiser |
| **Quelle licence ?** open source ou propriétaire | un outil open source peut être audité, modifié, et ne disparaît pas avec l'entreprise qui le publie |
| **Quel compte ?** gratuit, abonnement, clé d'API | elle fixe le coût, et souvent la politique sur vos données |
| **Quels modèles ?** ceux de l'entreprise qui publie l'outil, n'importe quel fournisseur, un modèle local | c'est ce qui décide chez qui votre code peut partir |
| **Quels fichiers de configuration lit-il ?** `AGENTS.md` ou un fichier à lui | elle dit si votre configuration passera d'un outil à l'autre |

## 2 - Deux standards à connaître

Deux formats de fichiers sont devenus communs à la plupart des outils. Ils permettent d'écrire sa configuration une fois et de changer d'outil sans tout refaire.

**`AGENTS.md`** est un fichier Markdown à la racine du dépôt, qui donne à l'agent les instructions du projet : comment lancer les tests, les conventions de code, ce qu'il ne faut pas toucher. Le format est aujourd'hui confié à l'Agentic AI Foundation, sous l'égide de la Linux Foundation. Le site [agents.md](https://agents.md/) liste les outils qui le lisent.

**Agent Skills** (le fichier `SKILL.md`) est un format pour décrire une compétence réutilisable : un dossier avec un fichier `SKILL.md` qui explique quand et comment faire une tâche, plus d'éventuels scripts. Créé par Anthropic, il a été publié comme standard ouvert sur [agentskills.io](https://agentskills.io/).

Certains outils gardent en plus leur propre fichier d'instructions (`CLAUDE.md`, `GEMINI.md`, `.cursor/rules`, `.kiro/steering`…) : les connaître suffit pour retrouver la configuration d'un projet existant.

## 3 - Les outils dans le terminal

Le tableau donne la carte d'identité de chaque outil en ligne de commande. Une API compatible OpenAI désigne n'importe quel serveur qui comprend le format de requêtes d'OpenAI, celui d'Ollama comme ceux de la plupart des fournisseurs.

| Outil | Entreprise, licence | Compte | Modèles | Instructions lues |
|---|---|---|---|---|
| **Claude Code** | Anthropic, propriétaire | abonnement Claude ou compte API ; pas d'offre gratuite | ceux d'Anthropic, directement ou via les clouds d'Amazon (Bedrock) et de Google (Vertex) | `CLAUDE.md`, qui peut importer un `AGENTS.md` |
| **OpenCode** | open source (MIT) | aucun pour un usage local ; offres payantes en option | des dizaines de fournisseurs, toute API compatible OpenAI, et les serveurs locaux Ollama et LM Studio, détectés seuls | `AGENTS.md`, puis `CLAUDE.md` à défaut |
| **Pi** | open source (MIT) | aucun : vous vous connectez à un fournisseur | une trentaine de fournisseurs, et toute API compatible OpenAI | `AGENTS.md`, puis `CLAUDE.md` |
| **DeepSeek Harness** (`dsh`) | DeepSeek, open source (MIT), en préversion | compte ou clé DeepSeek, ou un autre fournisseur | DeepSeek par défaut ; toute API compatible OpenAI ou serveur local | `AGENTS.md` et `CLAUDE.md` |
| **Mistral Vibe** | Mistral AI, open source (Apache 2.0) | compte ou clé Mistral, plan gratuit compris | Mistral, d'autres fournisseurs, et les modèles locaux (Ollama, LM Studio, llama.cpp, vLLM) | `AGENTS.md` |
| **Codex CLI** | OpenAI, open source (Apache 2.0) | abonnement ChatGPT ou clé API | OpenAI ; modèles locaux avec l'option `--oss` (Ollama, LM Studio) | `AGENTS.md` |
| **Gemini CLI** | Google, open source (Apache 2.0) | depuis juin 2026, offres entreprise et clés payantes seulement | Gemini | `GEMINI.md` (`AGENTS.md` si on le configure) |

Deux remarques. Claude Code et Mistral Vibe existent aussi en extension de VS Code, et OpenCode en application de bureau ; la forme « CLI » n'est que leur forme d'origine. Et Gemini CLI montre la vitesse des changements : ses utilisateurs gratuits ont été basculés vers Antigravity CLI, un autre outil de Google, en juin 2026. L'API Gemini garde, elle, un niveau gratuit ; c'est l'outil en ligne de commande qui a changé d'offre.

## 4 - Les éditeurs et les extensions

Le même tableau, pour les outils qui s'utilisent dans un éditeur de code :

| Outil | Forme | Entreprise, licence | Compte | Modèles | Instructions lues |
|---|---|---|---|---|---|
| **Cursor** | éditeur dérivé de VS Code, plus une CLI | Anysphere, propriétaire | offre gratuite, puis abonnements | ceux proposés par Cursor ; vos clés OpenAI, Anthropic, Google, Azure ou Bedrock | `.cursor/rules` et `AGENTS.md` |
| **Kiro** | éditeur dérivé de VS Code (Code OSS), plus une CLI | lié à AWS (compte AWS Builder ID), propriétaire | 50 crédits gratuits par mois (une unité de consommation des requêtes), puis abonnements | ceux proposés par Kiro (Claude, GPT, modèles ouverts) | `.kiro/steering/` et `AGENTS.md` |
| **GitHub Copilot** | extension de VS Code, Visual Studio et JetBrains ; une CLI ; un agent dans le cloud | GitHub (Microsoft), propriétaire | offre gratuite (sans l'agent dans le cloud), puis abonnements | ceux de GitHub ; une API compatible OpenAI ou Ollama via un réglage | `.github/copilot-instructions.md`, `AGENTS.md`, `CLAUDE.md` |
| **Kilo** | extension de VS Code et JetBrains, plus une CLI | open source (MIT) | compte Kilo, modèles gratuits via sa passerelle | sa passerelle, vos clés, toute API compatible OpenAI, Ollama, LM Studio | `AGENTS.md` |

Presque tous ces outils prennent aussi en charge **MCP** (*Model Context Protocol*, un protocole pour brancher des outils supplémentaires sur l'agent), les **Agent Skills** et des **hooks** (des scripts lancés avant ou après une action de l'agent). Ces trois briques se configurent outil par outil, et c'est là que les différences entre outils sont les plus grandes.

## 5 - Comment choisir

Les fiches ne désignent pas un gagnant : le bon outil dépend de votre contexte. Trois situations reviennent souvent :

| Votre situation | Ce qui décide | Outils qui y répondent |
|---|---|---|
| le code ne doit pas quitter le poste ou l'entreprise | accepter un modèle local ou auto-hébergé | OpenCode, Pi, Mistral Vibe, Kilo, Codex CLI (`--oss`), DeepSeek Harness, GitHub Copilot (avec un réglage) |
| l'entreprise a déjà un contrat avec un fournisseur | l'outil de ce fournisseur, avec les comptes entreprise | Copilot pour GitHub, Claude Code pour Anthropic, Kiro pour AWS |
| vous voulez pouvoir changer d'outil | un outil open source qui lit `AGENTS.md` | OpenCode, Pi, Kilo, Mistral Vibe |

Dans tous les cas, regardez la politique de données de l'offre précise que vous allez utiliser : c'est elle, et non l'outil, qui dit où part votre code.

## Récap

- Cinq questions situent un outil : **forme**, **licence**, **compte**, **modèles**, **fichiers de configuration**.
- **`AGENTS.md`** et **Agent Skills** (`SKILL.md`) sont lus par la plupart des outils : écrivez votre configuration dans ces formats d'abord.
- Les outils open source (OpenCode, Pi, Kilo, Mistral Vibe, Codex CLI, DeepSeek Harness) acceptent en général n'importe quel modèle, local compris. Les outils propriétaires (Claude Code, Cursor, Kiro, Copilot) sont d'abord pensés pour les modèles de leur entreprise ; certains acceptent vos propres clés, voire un modèle local, avec des réglages.
- Ces fiches vieillissent vite : vérifiez toujours la page officielle.

> 📖 [AGENTS.md](https://agents.md/) · [Agent Skills](https://agentskills.io/) · [Claude Code](https://code.claude.com/docs/en/overview) · [OpenCode](https://opencode.ai/docs/) · [Pi](https://github.com/earendil-works/pi) · [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) · [Mistral Vibe](https://docs.mistral.ai/mistral-vibe/introduction) · [Codex CLI](https://github.com/openai/codex) · [Gemini CLI](https://github.com/google-gemini/gemini-cli) · [Cursor](https://cursor.com/docs/context/rules) · [Kiro](https://kiro.dev/docs/steering/) · [GitHub Copilot, instructions de dépôt](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions/add-repository-instructions) · [Kilo, AGENTS.md](https://kilo.ai/docs/customize/agents-md)

```quiz
title: Le paysage des harness de code
tracksProgress: true
submitMode: last-only
questions:
  - prompt: Vous voulez écrire les instructions de votre projet une seule fois, pour qu'elles servent dans plusieurs outils. Quel fichier écrivez-vous ?
    answers:
      - "AGENTS.md, à la racine du dépôt"
      - "CLAUDE.md"
      - "GEMINI.md"
      - ".cursor/rules"
    correct: 1
    explanation: |
      AGENTS.md est le format commun, lu par la plupart des outils. Les autres fichiers sont
      propres à un outil.
  - prompt: Votre entreprise interdit que le code quitte ses serveurs. Quel outil pouvez-vous brancher sur un modèle auto-hébergé ?
    answers:
      - "Gemini CLI"
      - "Kiro"
      - "OpenCode"
      - "Claude Code"
    correct: 3
    explanation: |
      OpenCode accepte toute API compatible OpenAI et détecte Ollama et LM Studio. Gemini CLI,
      Kiro et Claude Code sont liés aux modèles de leur éditeur.
  - prompt: Qu'est-ce qu'une Agent Skill ?
    answers:
      - "Un modèle spécialisé dans un langage"
      - "Un dossier avec un fichier SKILL.md qui décrit quand et comment faire une tâche, dans un format ouvert"
      - "Un abonnement qui donne accès à plus de requêtes"
      - "Une extension de VS Code"
    correct: 2
    explanation: |
      Le format Agent Skills, créé par Anthropic puis publié comme standard ouvert, décrit une
      compétence réutilisable que plusieurs outils savent lire.
  - prompt: Vous découvrez un nouvel agent de code. Quelle question vous dit chez qui votre code peut partir ?
    answers:
      - "Quelle est sa forme : CLI, éditeur dérivé ou extension ?"
      - "Lit-il le fichier AGENTS.md ?"
      - "Est-il gratuit ?"
      - "Quels modèles accepte-t-il : ceux de son entreprise seulement, n'importe quel fournisseur, un modèle local ?"
    correct: 4
    explanation: |
      Les modèles acceptés disent à qui le contexte peut être envoyé ; la politique de données
      de l'offre dit ensuite ce qui lui arrive.
  - prompt: Kiro et Cursor sont tous deux des éditeurs dérivés de VS Code. Qu'est-ce que cela implique pour vous ?
    answers:
      - "Vous changez d'éditeur pour utiliser leur agent"
      - "Ils ne fonctionnent que dans le terminal"
      - "Ils sont open source"
      - "Ils lisent uniquement vos fichiers .vscode"
    correct: 1
    explanation: |
      Un éditeur dérivé est une copie modifiée de VS Code : vous utilisez cet éditeur à la place
      du vôtre, avec l'agent intégré.
```
