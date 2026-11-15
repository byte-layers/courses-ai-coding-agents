---
title: "Où chaque harness range sa configuration"
description: "Une table de correspondance pour huit agents de code (Claude Code, OpenCode, Kilo, GitHub Copilot, Cursor, Kiro, Mistral Vibe, Codex CLI) : le fichier de chaque brique de configuration, ce qui est standard et portable, et ce qu'il faut refaire en changeant d'outil."
tags:
  - bit
  - ai
difficulty: beginner
duration: 15
requires: "0.4.0"
---

Chaque agent de code range sa configuration dans ses propres fichiers. Ce bit est une table de correspondance : pour chaque brique, le fichier à créer dans huit outils, ce qui passe d'un outil à l'autre sans changement, et ce qu'il faut réécrire. Il se consulte plus qu'il ne se lit d'une traite.

**Pour qui** : un développeur qui connaît les sept briques de configuration d'un agent (instructions de projet, règles ciblées, skills, commandes, sous-agents, serveurs MCP, permissions et hooks), ce qu'explique [Les sept briques de configuration d'un agent](https://byte-layers.com/fr/bits/agent-config-building-blocks), et les outils présentés dans [Le paysage des harness de code](https://byte-layers.com/fr/bits/coding-harness-landscape).

> 📅 **Correspondances vérifiées le 16 novembre 2026** sur la documentation officielle de chaque outil. Ces emplacements changent souvent : en cas de doute, la page officielle fait foi. Une case « non documenté » veut dire que la documentation officielle ne le dit pas à cette date, pas que la fonction n'existe pas.

# Où chaque harness range sa configuration

## 1 - Ce qui est standard, ce qui est propre à chaque outil

Avant les détails, une seule distinction compte : deux briques ont un format commun, les cinq autres non. Le tableau dit lesquelles :

| Brique | Format commun ? | Conséquence |
|---|---|---|
| Instructions de projet | **oui** : `AGENTS.md`, lu par les huit outils | écrite une fois, elle sert partout |
| Skills | **oui** : le dossier `SKILL.md` du format Agent Skills ; le dossier `.agents/skills/` est lu par six des huit outils | le contenu se copie tel quel, seul le dossier peut changer |
| Règles ciblées | non : le principe est le même, le champ et le dossier changent | à recopier en adaptant l'en-tête |
| Commandes | non | à recopier, en vérifiant la syntaxe des arguments |
| Sous-agents | non : souvent Markdown, parfois TOML, avec des champs différents | à réécrire |
| Serveurs MCP | le protocole est commun, **pas le fichier de configuration** | à retranscrire dans le format de l'outil |
| Permissions et hooks | non | à réécrire entièrement |

Trois formats de fichiers reviennent dans les tableaux qui suivent. **JSON** et **YAML** sont des formats de données structurées, l'un avec accolades et guillemets, l'autre par indentation ; **TOML** est un format de configuration par sections entre crochets (`[tools.bash]`). L'**en-tête** (*frontmatter*) désigne le bloc YAML entre deux lignes `---` en haut d'un fichier Markdown.

Les chemins sont donnés depuis la racine du dépôt ; la plupart des outils ont aussi un équivalent dans votre dossier personnel, pour les réglages valables dans tous vos projets. DeepSeek Harness, présent dans le paysage, n'est pas dans ces tableaux : son dépôt ne documente pas ces emplacements à la date de vérification.

## 2 - Les instructions de projet

Les huit outils lisent `AGENTS.md`. Certains lisent en plus leur propre fichier, et un outil peut ignorer `AGENTS.md` quand son fichier à lui existe :

| Outil | Fichier lu |
|---|---|
| **Claude Code** | `CLAUDE.md` ; `AGENTS.md` seulement s'il n'y a aucun `CLAUDE.md`, sinon via un import `@AGENTS.md` dans `CLAUDE.md` |
| **OpenCode** | `AGENTS.md`, puis `CLAUDE.md` à défaut |
| **Kilo** | `AGENTS.md` (`AGENT.md` à défaut) |
| **GitHub Copilot** | `.github/copilot-instructions.md`, `AGENTS.md`, ou un `CLAUDE.md` à la racine |
| **Cursor** | `AGENTS.md` |
| **Kiro** | `.kiro/steering/*.md` (inclusion `always` par défaut) et `AGENTS.md` |
| **Mistral Vibe** | `AGENTS.md`, dans un dossier marqué de confiance |
| **Codex CLI** | `AGENTS.md`, ou `AGENTS.override.md` qui le remplace dans le même dossier |

Le piège est dans la première ligne : un dépôt qui contient à la fois `CLAUDE.md` et `AGENTS.md` ne donne à Claude Code que `CLAUDE.md`. Pour garder une seule source, écrivez `AGENTS.md` et faites de `CLAUDE.md` une ligne qui l'importe. Un **dossier de confiance** (*trusted folder*), dans Mistral Vibe et Codex CLI, est un dossier que vous avez approuvé à la première ouverture ; la configuration d'un dépôt cloné n'est pas lue avant.

## 3 - Les règles ciblées

Une règle ciblée porte un motif de chemin, un *glob*. Le principe est partout le même, mais le dossier et le nom du champ changent, et trois outils n'ont pas de règle par motif :

| Outil | Fichier | Champ du motif |
|---|---|---|
| **Claude Code** | `.claude/rules/*.md` | `paths` dans l'en-tête |
| **OpenCode** | non documenté (la clé `instructions` ajoute des fichiers, chargés à chaque conversation) | non documenté |
| **Kilo** | pas de règle par motif ; un `AGENTS.md` de sous-dossier est chargé quand l'agent lit un fichier de ce dossier | sans objet |
| **GitHub Copilot** | `.github/instructions/*.instructions.md` | `applyTo` dans l'en-tête |
| **Cursor** | `.cursor/rules/*.mdc` (extension `.mdc` obligatoire) | `globs` dans l'en-tête |
| **Kiro** | `.kiro/steering/*.md` | `inclusion: fileMatch` et `fileMatchPattern` |
| **Mistral Vibe** | pas de règle par motif ; un `AGENTS.md` de sous-dossier est découvert quand l'agent y lit un fichier | sans objet |
| **Codex CLI** | pas de règle par motif ; un `AGENTS.md` par dossier, lu de la racine jusqu'au dossier courant | sans objet |

Le `AGENTS.md` de sous-dossier est donc la solution la plus portable pour une convention propre à un dossier : Kilo, Mistral Vibe, Codex CLI et Cursor le lisent.

## 4 - Les skills

Le contenu d'une skill est le même partout : un dossier avec un `SKILL.md`. Ce qui change, c'est le dossier où l'outil va la chercher :

| Outil | Dossiers lus dans le projet |
|---|---|
| **Claude Code** | `.claude/skills/` |
| **OpenCode** | `.opencode/skills/`, `.claude/skills/`, `.agents/skills/` |
| **Kilo** | `.kilo/skills/`, `.agents/skills/` ; `.claude/skills/` si la compatibilité Claude Code est activée |
| **GitHub Copilot** | `.github/skills/` ; selon le mode, aussi `.claude/skills/` et `.agents/skills/` |
| **Cursor** | `.cursor/skills/`, `.agents/skills/` ; `.claude/skills/` et `.codex/skills/` par compatibilité |
| **Kiro** | `.kiro/skills/` |
| **Mistral Vibe** | `.vibe/skills/`, `.agents/skills/` |
| **Codex CLI** | `.agents/skills/` |

`.agents/skills/` est lu par six des huit outils, et `.claude/skills/` par cinq. Claude Code et Kiro ne documentent que leur propre dossier : pour eux, un lien symbolique (un raccourci de fichier, créé avec `ln -s`) vers `.agents/skills/` évite de copier.

## 5 - Les commandes personnalisées

Les commandes sont la brique la moins stable : plusieurs outils les remplacent par des skills que l'on appelle avec `/nom`. Le tableau dit où ranger une consigne réutilisable que vous lancez vous-même :

| Outil | Fichier | Appel |
|---|---|---|
| **Claude Code** | `.claude/commands/*.md` (toujours lu) ; les skills sont recommandées pour un nouveau fichier | `/nom` |
| **OpenCode** | `.opencode/commands/*.md` | `/nom` |
| **Kilo** | `.kilo/commands/*.md` (appelées *workflows*) | `/nom` |
| **GitHub Copilot** | `.github/prompts/*.prompt.md` (*prompt files*) | `/nom` |
| **Cursor** | une skill avec `disable-model-invocation: true` ; la commande `/migrate-to-skills` convertit les anciennes commandes | `/nom` |
| **Kiro** | une skill, appelée comme une commande | `/nom` |
| **Mistral Vibe** | une skill : c'est la façon recommandée d'ajouter une commande | `/nom` |
| **Codex CLI** | une skill ; les *custom prompts* de `~/.codex/prompts/` sont dépréciés | `$nom` ou `/skills` |

La tendance est nette : en écrivant vos commandes comme des skills, vous les rendez portables. Le champ `disable-model-invocation: true`, reconnu notamment par Claude Code, Cursor et Mistral Vibe, empêche le modèle de la lancer seul.

## 6 - Les sous-agents

Un sous-agent se décrit dans un fichier par agent, mais le format et les champs diffèrent d'un outil à l'autre :

| Outil | Fichier | Format |
|---|---|---|
| **Claude Code** | `.claude/agents/*.md` | Markdown avec en-tête (`name`, `description`, `tools`, `model`) |
| **OpenCode** | `.opencode/agents/*.md` | Markdown avec en-tête, `mode: subagent` |
| **Kilo** | `.kilo/agents/*.md`, ou la clé `agent` de `kilo.jsonc` | Markdown avec en-tête |
| **GitHub Copilot** | `.github/agents/*.agent.md` | Markdown avec en-tête |
| **Cursor** | `.cursor/agents/` ; aussi `.claude/agents/` et `.codex/agents/` | Markdown avec en-tête (`name`, `description`, `model`, `readonly`) |
| **Kiro** | `.kiro/agents/` | non documenté sur la page des sous-agents |
| **Mistral Vibe** | `.vibe/agents/*.toml` | TOML, `agent_type = "subagent"` |
| **Codex CLI** | `.codex/agents/*.toml` | TOML (`name`, `description`, `developer_instructions`) |

Seul Cursor lit directement les sous-agents écrits pour Claude Code. Partout ailleurs, la consigne du sous-agent se recopie, mais l'en-tête se réécrit : la liste d'outils, les noms de modèles et les droits n'ont pas les mêmes noms.

## 7 - Les serveurs MCP

Le protocole MCP est le même partout : un serveur écrit une fois fonctionne avec les huit outils. Ce qui change, c'est le fichier qui dit au harness comment le lancer :

| Outil | Fichier | Clé |
|---|---|---|
| **Claude Code** | `.mcp.json` | `mcpServers` (JSON) |
| **OpenCode** | `opencode.json` | `mcp` (JSON) |
| **Kilo** | `kilo.jsonc` ou `.kilo/kilo.jsonc` | `mcp` (JSON avec commentaires) |
| **GitHub Copilot** | `.mcp.json` ; `.vscode/mcp.json` dans l'ancien mode local de VS Code | `mcpServers` ; `servers` dans `.vscode/mcp.json` |
| **Cursor** | `.cursor/mcp.json` | `mcpServers` (JSON) |
| **Kiro** | `.kiro/settings/mcp.json` | `mcpServers` (JSON) |
| **Mistral Vibe** | `.vibe/config.toml` | `[[mcp_servers]]` (TOML) |
| **Codex CLI** | `.codex/config.toml` | `[mcp_servers.<nom>]` (TOML) |

Le bloc `mcpServers` se copie presque tel quel entre Claude Code, Cursor, Kiro et Copilot. Pour OpenCode et Kilo, la clé `mcp` a un autre schéma (la commande et ses arguments dans un seul tableau, par exemple) ; pour Mistral Vibe et Codex CLI, il faut passer en TOML. Les clés d'API ne voyagent jamais avec le fichier : chaque outil a sa façon de lire une variable d'environnement.

## 8 - Les permissions et les hooks

C'est la brique où les outils diffèrent le plus, dans le format comme dans ce qu'ils savent faire. Le tableau donne, pour chaque outil, le fichier des permissions, les décisions possibles, et le fichier des hooks :

| Outil | Permissions | Décisions | Hooks |
|---|---|---|---|
| **Claude Code** | `.claude/settings.json`, clé `permissions` | `allow`, `ask`, `deny` | clé `hooks` du même fichier ; un code de sortie 2 bloque |
| **OpenCode** | `opencode.json`, clé `permission` | `allow`, `ask`, `deny` | pas de hooks en script shell : des plugins JavaScript ou TypeScript dans `.opencode/plugins/` |
| **Kilo** | `kilo.jsonc`, clé `permission` | `allow`, `ask`, `deny` | pas de hooks en script shell : des plugins TypeScript, d'après son guide de migration |
| **GitHub Copilot** | réglage de VS Code `chat.tools.terminal.autoApprove` | `true` (sans demander), `false` (demander) ; pas de refus | `.github/hooks/*.json` ; lit aussi les hooks de `.claude/settings.json` |
| **Cursor** | `.cursor/cli.json` (pour la CLI) | `allow`, `deny` | `.cursor/hooks.json` ; un code de sortie 2 bloque |
| **Kiro** | `~/.kiro/settings/permissions.yaml` ; les règles d'un projet sont gardées hors du dépôt | `allow`, `deny` | `.kiro/hooks/*.json` |
| **Mistral Vibe** | `.vibe/config.toml`, sections `[tools.<outil>]` | `always`, `ask`, `never` | `.vibe/hooks.toml` |
| **Codex CLI** | `.codex/rules/*.rules`, règles `prefix_rule` | `allow`, `prompt`, `forbidden` | `.codex/hooks.json`, ou clé `hooks` de `.codex/config.toml` |

Deux remarques. OpenCode documente aussi un nouveau format, une liste `permissions` de règles `action`, `resource`, `effect` : il ne vaut que pour la préversion `opencode2`, la version stable le refuse. Et chez Copilot, une règle `false` force une demande sans bloquer la commande : pour un vrai refus, il faut un hook.

## 9 - Migrer d'un outil à l'autre

Changer d'outil ne veut pas dire tout réécrire. Le tableau classe les briques selon l'effort, en partant de Claude Code, qui a le plus de configuration à migrer chez la plupart des équipes :

| Ce qui passe tel quel | Ce qui se recopie en adaptant | Ce qui se réécrit |
|---|---|---|
| `AGENTS.md` (si `CLAUDE.md` l'importe au lieu de le dupliquer) | les règles ciblées : même texte, autre en-tête | les permissions : chaque outil a sa syntaxe |
| les skills, si elles sont dans un dossier que l'outil cible lit | les commandes : même texte, syntaxe des arguments à vérifier | les hooks : autre format, parfois un plugin dans un autre langage |
| les serveurs MCP eux-mêmes | le fichier de configuration MCP | l'en-tête des sous-agents |

Deux outils documentent ce passage. Kilo publie un guide de migration depuis Claude Code, qui reprend brique par brique ce tableau : skills lues dans `.claude/skills/`, commandes à copier dans `.kilo/commands/`, permissions et `.mcp.json` à retranscrire, hooks à réécrire en plugins. OpenCode lit de lui-même une partie de la configuration de Claude Code (`CLAUDE.md` à défaut d'`AGENTS.md`, les skills de `.claude/skills/`) et se règle par des variables d'environnement comme `OPENCODE_DISABLE_CLAUDE_CODE=1`, qui coupe cette lecture.

Pour une équipe qui veut rester libre de changer d'outil, la règle est simple : mettez le plus possible dans `AGENTS.md` et dans des skills rangées dans `.agents/skills/`, et gardez le reste court, puisqu'il faudra le refaire.

## Récap

- Deux briques sont **portables** : `AGENTS.md`, lu par les huit outils, et les skills (`SKILL.md`), dont le dossier `.agents/skills/` est lu par six d'entre eux.
- Claude Code ignore `AGENTS.md` quand un `CLAUDE.md` existe : faites importer l'un par l'autre.
- Les règles ciblées, les commandes et les sous-agents se **recopient en adaptant** l'en-tête ; plusieurs outils remplacent les commandes par des skills.
- Le protocole MCP est commun, mais pas le **fichier** ; les **permissions** et les **hooks** se réécrivent outil par outil.

> 📖 [Claude Code, CLAUDE.md et règles](https://code.claude.com/docs/en/memory) · [Claude Code, skills](https://code.claude.com/docs/en/skills) · [Claude Code, réglages](https://code.claude.com/docs/en/settings) · [OpenCode, instructions](https://opencode.ai/docs/rules/) · [OpenCode, skills](https://opencode.ai/docs/skills/) · [OpenCode, agents](https://opencode.ai/docs/agents/) · [Kilo, AGENTS.md](https://kilo.ai/docs/customize/agents-md) · [Kilo, skills](https://kilo.ai/docs/customize/skills) · [Kilo, migrer depuis Claude Code](https://kilo.ai/articles/claude-code-to-kilo-code-migration-guide) · [GitHub Copilot, instructions de dépôt](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions/add-repository-instructions) · [VS Code, skills](https://code.visualstudio.com/docs/agent-customization/agent-skills) · [VS Code, hooks](https://code.visualstudio.com/docs/agent-customization/hooks) · [Cursor, règles](https://cursor.com/docs/context/rules) · [Cursor, sous-agents](https://cursor.com/docs/context/subagents) · [Cursor, hooks](https://cursor.com/docs/agent/hooks) · [Kiro, steering](https://kiro.dev/docs/steering/) · [Kiro, hooks](https://kiro.dev/docs/hooks/) · [Mistral Vibe, permissions](https://docs.mistral.ai/vibe/code/safety-approvals-permissions) · [Mistral Vibe, skills](https://docs.mistral.ai/vibe/code/cli/skills) · [Codex, AGENTS.md](https://learn.chatgpt.com/docs/agent-configuration/agents-md) · [Codex, règles](https://learn.chatgpt.com/docs/agent-configuration/rules) · [Codex, hooks](https://learn.chatgpt.com/docs/hooks)

```quiz
title: Où chaque harness range sa configuration
tracksProgress: true
submitMode: last-only
questions:
  - prompt: Votre dépôt contient un CLAUDE.md et un AGENTS.md aux contenus différents. Que lit Claude Code ?
    answers:
      - "Les deux, fusionnés"
      - "Seulement AGENTS.md, le standard"
      - "Seulement CLAUDE.md, sauf si CLAUDE.md importe AGENTS.md"
      - "Aucun des deux"
    correct: 3
    explanation: |
      Claude Code ne lit AGENTS.md que s'il n'y a aucun CLAUDE.md. Pour une seule source,
      CLAUDE.md importe AGENTS.md.
  - prompt: Vous voulez écrire une skill une fois et qu'elle serve dans OpenCode, Kilo, Cursor, Mistral Vibe et Codex CLI. Où la rangez-vous ?
    answers:
      - "Dans .agents/skills/<nom>/SKILL.md"
      - "Dans AGENTS.md"
      - "Dans .cursor/rules/"
      - "Dans .mcp.json"
    correct: 1
    explanation: |
      Ces cinq outils lisent .agents/skills/. Claude Code et Kiro demandent leur propre dossier,
      ou un lien symbolique.
  - prompt: Vous passez de Claude Code à Cursor. Quelle brique devez-vous réécrire, et non simplement copier ?
    answers:
      - "AGENTS.md"
      - "Les skills de .claude/skills/"
      - "Les sous-agents de .claude/agents/"
      - "Les règles de permission de .claude/settings.json"
    correct: 4
    explanation: |
      Cursor lit les skills et les sous-agents de .claude/, et AGENTS.md est commun. Les
      permissions ont une syntaxe propre à chaque outil.
  - prompt: "Dans GitHub Copilot, vous mettez la règle rm à false dans le réglage d'approbation des commandes. Qu'obtenez-vous ?"
    answers:
      - "La commande rm est bloquée"
      - "La commande rm demande votre accord, mais n'est pas bloquée"
      - "La commande rm s'exécute sans demander"
      - "Rien : le réglage n'existe pas"
    correct: 2
    explanation: |
      Une règle false force une demande d'approbation. Pour refuser vraiment une commande dans
      Copilot, il faut un hook.
  - prompt: Quelle affirmation sur les serveurs MCP est juste ?
    answers:
      - "Chaque outil demande un serveur MCP écrit pour lui"
      - "Le fichier .mcp.json est lu par les huit outils"
      - "Le serveur fonctionne partout, mais le fichier qui le déclare change d'un outil à l'autre"
      - "Les clés d'API sont copiées avec la configuration"
    correct: 3
    explanation: |
      Le protocole est commun. Le fichier et son format changent : mcpServers en JSON chez
      plusieurs outils, mcp chez OpenCode et Kilo, du TOML chez Mistral Vibe et Codex CLI.
```
