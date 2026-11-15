---
title: "Les sept briques de configuration d'un agent"
description: "Instructions de projet, règles ciblées, skills, commandes, sous-agents, serveurs MCP, permissions et hooks : à quoi sert chaque brique, quand elle entre dans le contexte du modèle, et ce qu'elle coûte en tokens."
tags:
  - bit
  - ai
difficulty: beginner
duration: 14
requires: "0.4.0"
---

Tous les agents de code se configurent avec les mêmes sortes de pièces, même si chaque outil les range dans ses propres fichiers. Ce bit présente ces sept briques sans parler d'un outil en particulier : à quoi sert chacune, et surtout **quand** elle entre dans le contexte du modèle. C'est ce moment qui décide de ce qu'elle coûte.

**Pour qui** : un développeur qui connaît les mots modèle, harness, contexte, tokens et permissions, ce qu'explique [La boucle d'un agent de code](https://byte-layers.com/fr/bits/agent-loop), et qui a vu passer les noms `AGENTS.md`, Agent Skills et MCP dans [Le paysage des harness de code](https://byte-layers.com/fr/bits/coding-harness-landscape). Aucun outil n'est à installer.

# Les sept briques de configuration d'un agent

## 1 - Le contexte, une ressource qui se paie

Toute la configuration se juge avec une seule idée : ce qui entre dans le contexte se paie à chaque tour de la boucle. Le modèle relit tout son contexte à chaque requête ; un texte de 2 000 tokens chargé au démarrage est donc renvoyé à chaque appel d'outil, dix fois pour une tâche de dix tours. Il prend aussi de la place dans la fenêtre de contexte, et un modèle suit moins bien une consigne noyée dans un long texte.

Les sept briques se distinguent donc par leur **moment de chargement**. Le tableau donne les quatre moments possibles, du plus coûteux au moins coûteux :

| Moment de chargement | Ce que ça veut dire | Coût en tokens |
|---|---|---|
| **toujours** | le texte est ajouté au contexte dès le début de chaque conversation | payé à chaque tour, que la tâche en ait besoin ou non |
| **selon le chemin** | le texte est ajouté quand l'agent travaille sur un fichier qui correspond à un motif | payé seulement pour les tâches qui touchent ces fichiers |
| **à la demande** | seuls un nom et une description sont chargés au départ ; le modèle charge le reste s'il en a besoin | quelques dizaines de tokens par brique, puis le texte complet si elle sert |
| **à l'invocation** | rien n'est chargé tant que vous ne l'appelez pas par son nom | nul tant que vous ne l'utilisez pas |

Une brique a enfin un cas à part : celle qui n'entre **jamais** dans le contexte, parce que c'est le harness qui l'applique, pas le modèle. C'est la septième.

## 2 - Les instructions de projet : toujours chargées

Les **instructions de projet** sont un fichier Markdown à la racine du dépôt, lu par le harness et ajouté au contexte au début de chaque conversation. Le format commun est `AGENTS.md`. On y met ce que tout contributeur doit savoir : comment lancer les tests, les conventions du code, ce qu'il ne faut pas toucher.

Un fichier d'instructions court ressemble à ceci :

```markdown
# AGENTS.md

- Installer : `npm ci`. Tester : `npm test` avant chaque commit.
- Le code est en TypeScript strict ; pas de `any`.
- Ne jamais modifier les fichiers de `migrations/` déjà publiés.
```

Comme ce fichier est chargé à chaque tour, chaque ligne a un coût permanent. Gardez-y ce qui vaut pour toutes les tâches, et déplacez le reste dans les briques suivantes.

## 3 - Les règles ciblées : chargées selon le chemin

Une **règle ciblée** est un fichier d'instructions qui ne vaut que pour une partie du dépôt. Elle porte un **motif de chemin** (un *glob*, une expression comme `src/api/**/*.ts` qui désigne un ensemble de fichiers), et le harness ne l'ajoute au contexte que lorsque l'agent lit ou modifie un fichier qui correspond.

L'exemple suivant montre la forme habituelle : un en-tête qui donne le motif, puis la consigne. Le nom exact du champ (`paths`, `globs`, `applyTo`…) change d'un outil à l'autre :

```markdown
---
paths: "src/api/**/*.ts"
---

Chaque route valide son entrée avec le schéma Zod du dossier `schemas/`.
```

Une règle ciblée coûte zéro token dans une tâche qui ne touche pas l'API. C'est la bonne place pour les conventions d'un dossier, d'un langage ou d'un type de fichier.

## 4 - Les skills : chargées à la demande

Une **skill** (compétence) est un dossier qui contient un fichier `SKILL.md` : un en-tête avec un nom et une description, puis les instructions détaillées d'une tâche, et parfois des scripts. C'est le format ouvert Agent Skills.

Une skill tient en quelques lignes au minimum ; la description dit au modèle quand s'en servir :

```markdown
---
name: release-notes
description: Rédige les notes de version à partir des commits depuis le dernier tag. À utiliser quand on prépare une release.
---

1. Lister avec `git log --oneline` les commits faits depuis le dernier tag.
2. Les regrouper en Ajouts, Corrections, Ruptures.
3. Écrire le résultat dans `CHANGELOG.md`, sous un titre avec la date du jour.
```

Le chargement se fait en trois temps, ce que la spécification appelle la **divulgation progressive** (*progressive disclosure*) :

| Étape | Ce qui entre dans le contexte | Ordre de grandeur donné par la spécification |
|---|---|---|
| au démarrage | le nom et la description de chaque skill | environ 100 tokens par skill |
| quand le modèle juge la skill utile | tout le corps de `SKILL.md` | moins de 5 000 tokens recommandés |
| si les instructions le demandent | les fichiers annexes (scripts, références) | selon le besoin |

Vingt skills coûtent donc environ 2 000 tokens au repos, au lieu de dizaines de milliers si leurs instructions étaient dans `AGENTS.md`. La contrepartie : c'est le modèle qui décide de charger une skill, d'après sa description. Une description vague, et la skill ne sert jamais.

## 5 - Les commandes personnalisées : chargées à l'invocation

Une **commande personnalisée** (*custom command*, ou *prompt file* selon les outils) est une consigne réutilisable, rangée dans un fichier, que vous lancez en tapant son nom précédé d'une barre oblique : `/review`. Le texte du fichier remplace alors votre message.

Une commande de relecture peut ressembler à ceci ; `$ARGUMENTS` est remplacé par ce que vous tapez après le nom, dans les outils qui le prennent en charge :

```markdown
Relis les changements non commités ($ARGUMENTS).
Cherche d'abord les bugs, puis les tests manquants. Ne modifie aucun fichier.
```

Une commande ne coûte rien tant que vous ne l'appelez pas : c'est vous qui décidez, pas le modèle. Plusieurs outils rapprochent aujourd'hui les deux briques, et acceptent qu'une skill soit aussi appelée par `/nom` ; la différence qui reste est l'initiative, celle du modèle pour une skill, la vôtre pour une commande.

## 6 - Les sous-agents : un contexte à part

Un **sous-agent** (*subagent*) est un second agent, avec ses propres consignes, ses propres outils autorisés et **sa propre fenêtre de contexte**. L'agent principal lui délègue une tâche par un message ; le sous-agent travaille seul, puis renvoie un résumé.

Un sous-agent se décrit souvent par un fichier comme celui-ci : un nom, une description qui dit quand lui déléguer, la liste des outils permis, puis ses consignes :

```markdown
---
name: code-reviewer
description: Relit un changement et signale les bugs. À utiliser après une modification de code.
tools: read, grep
---

Tu relis du code. Tu ne modifies rien. Tu rends une liste de problèmes, du plus grave au moins grave.
```

L'intérêt est le coût pour l'agent principal : les vingt fichiers que le sous-agent a lus restent dans son contexte à lui, et seul le résumé revient. Le prix est ailleurs : le sous-agent consomme ses propres tokens, et il ne voit pas la conversation, donc le message de délégation doit tout lui dire. Sa description, elle, est chargée dans le contexte principal, comme celle d'une skill.

## 7 - Les serveurs MCP : des outils en plus

Un **serveur MCP** est un programme qui ajoute des outils ou des sources de données à l'agent, en parlant le protocole MCP (*Model Context Protocol*) : par exemple un accès à une base de données, à un gestionnaire de tickets ou à un navigateur. Le harness le lance ou s'y connecte au démarrage.

La configuration donne au harness la commande à lancer, ou l'adresse à joindre. La forme suivante, en JSON, est fréquente ; le nom du fichier et de la clé de premier niveau changent selon l'outil :

```json
{
  "mcpServers": {
    "tickets": { "command": "npx", "args": ["-y", "example-tickets-mcp"] }
  }
}
```

Chaque outil d'un serveur MCP arrive avec sa description et ses paramètres, et ces descriptions sont en général envoyées au modèle à chaque tour, comme celles des outils intégrés. Un serveur qui expose quarante outils peut ainsi coûter plusieurs milliers de tokens permanents. Certains harness chargent ces descriptions à la demande, par une recherche d'outils ; vérifiez ce que fait le vôtre, et n'activez que les serveurs dont le projet se sert.

## 8 - Les permissions et les hooks : appliqués par le harness

Les **permissions** sont des règles que le harness vérifie avant d'exécuter un outil : **autoriser** sans demander (*allow*), **demander** votre accord (*ask*), ou **refuser** (*deny*). Une règle vise un outil et un motif, par exemple « autoriser `npm test` », « refuser la lecture de `.env` ».

Un **hook** (crochet) est un script que le harness lance à un moment précis de la boucle : avant un appel d'outil, après une modification de fichier, à la fin d'une tâche. Un hook placé avant un appel d'outil peut le bloquer ; un hook placé après peut lancer le formateur sur le fichier modifié.

Ces deux briques n'entrent pas dans le contexte : c'est le harness qui les applique, pas le modèle. Elles ne coûtent donc aucun token, et surtout elles ne dépendent pas de la bonne volonté du modèle. Le tableau compare les deux façons de dire « ne lis pas `.env` » :

| Où vous l'écrivez | Qui l'applique | Ce que vous obtenez |
|---|---|---|
| une phrase dans `AGENTS.md` | le modèle, s'il la suit | une **demande** : le modèle peut l'oublier ou l'ignorer |
| une règle `deny` ou un hook qui bloque la lecture | le harness, avant d'exécuter l'outil | une **garantie** : l'outil n'est pas exécuté |

Retenez la formule : une instruction **demande**, un hook ou une permission **garantit**. Tout ce qui protège un secret ou évite une commande destructrice relève de cette septième brique.

## 9 - Les sept briques côte à côte

Le tableau résume les sept briques, dans l'ordre de ce bit :

| Brique | À quoi elle sert | Quand elle est chargée | Coût en contexte |
|---|---|---|---|
| **Instructions de projet** (`AGENTS.md`) | ce qui vaut pour toute tâche du dépôt | toujours | permanent, à chaque tour |
| **Règles ciblées** | les conventions d'un dossier ou d'un type de fichier | selon le chemin des fichiers touchés | seulement quand le motif correspond |
| **Skills** (`SKILL.md`) | une procédure détaillée pour une tâche précise | à la demande : description au départ, corps si le modèle s'en sert | faible au repos, complet à l'usage |
| **Commandes personnalisées** | une consigne réutilisable que vous lancez | à l'invocation (`/nom`) | nul tant que vous ne l'appelez pas |
| **Sous-agents** | déléguer une tâche dans un contexte séparé | description au départ ; le travail se fait ailleurs | seul le résumé revient dans le contexte principal |
| **Serveurs MCP** | des outils ou des données en plus | au démarrage, en général | une description par outil, souvent à chaque tour |
| **Permissions et hooks** | autoriser, demander, refuser ; agir avant ou après une action | jamais : appliqués par le harness | aucun |

Pour placer une consigne, posez-vous deux questions dans l'ordre. Faut-il une garantie ? Alors c'est une permission ou un hook. Sinon, quand le modèle en a-t-il besoin : toujours, pour certains fichiers, pour une tâche précise, ou seulement quand vous le décidez ?

## Récap

- Ce qui entre dans le contexte se paie à **chaque tour** : la place d'une brique se choisit par son **moment de chargement**.
- **Toujours** : les instructions de projet. **Selon le chemin** : les règles ciblées. **À la demande** : les skills. **À l'invocation** : les commandes.
- Un **sous-agent** travaille dans son propre contexte et ne renvoie qu'un résumé ; un **serveur MCP** ajoute des outils, dont les descriptions pèsent souvent à chaque tour.
- Les **permissions** et les **hooks** sont appliqués par le harness : une instruction demande, un hook garantit.

> 📖 [Le format AGENTS.md](https://agents.md/) · [La spécification Agent Skills](https://agentskills.io/specification) · [Le Model Context Protocol](https://modelcontextprotocol.io/docs/getting-started/intro) · [Exemple d'un outil : les sous-agents de Claude Code](https://code.claude.com/docs/en/sub-agents) · [Exemple d'un outil : les hooks de Claude Code](https://code.claude.com/docs/en/hooks)

```quiz
title: Les sept briques de configuration d'un agent
tracksProgress: true
submitMode: last-only
questions:
  - prompt: Votre fichier AGENTS.md fait 3 000 tokens, et une tâche demande douze tours de boucle. Combien de fois ces 3 000 tokens sont-ils envoyés au modèle ?
    answers:
      - "Une fois, au début de la conversation"
      - "Douze fois, une par tour"
      - "Jamais : le harness les garde pour lui"
      - "Seulement quand le modèle les demande"
    correct: 2
    explanation: |
      Les instructions de projet sont toujours chargées, et le modèle relit tout son contexte
      à chaque requête : elles sont renvoyées à chaque tour.
  - prompt: Vous avez une convention qui ne concerne que les fichiers de migration de la base de données. Quelle brique évite de la payer dans les autres tâches ?
    answers:
      - "Une ligne de plus dans AGENTS.md"
      - "Un serveur MCP"
      - "Un hook"
      - "Une règle ciblée avec un motif de chemin sur ces fichiers"
    correct: 4
    explanation: |
      Une règle ciblée n'est ajoutée au contexte que lorsque l'agent touche un fichier qui
      correspond à son motif.
  - prompt: Quelle différence entre une skill et une commande personnalisée ?
    answers:
      - "C'est le modèle qui décide de charger une skill, d'après sa description ; c'est vous qui lancez une commande par son nom"
      - "Une skill est toujours chargée, une commande jamais"
      - "Une commande peut contenir des scripts, une skill non"
      - "Il n'y en a aucune"
    correct: 1
    explanation: |
      Une skill se charge à la demande, à l'initiative du modèle ; une commande se charge à
      l'invocation, à la vôtre.
  - prompt: Pourquoi déléguer une recherche dans tout le dépôt à un sous-agent ?
    answers:
      - "Parce qu'il ne consomme aucun token"
      - "Parce qu'il voit toute la conversation en cours"
      - "Parce que les fichiers qu'il lit restent dans son contexte et que seul son résumé revient à l'agent principal"
      - "Parce qu'il contourne les permissions"
    correct: 3
    explanation: |
      Le sous-agent a sa propre fenêtre de contexte. Il consomme ses propres tokens, mais il
      épargne le contexte de l'agent principal.
  - prompt: Vous voulez être certain que l'agent ne lancera jamais `git push --force`. Que configurez-vous ?
    answers:
      - "Une phrase dans AGENTS.md"
      - "Une skill qui explique les dangers du push forcé"
      - "Une commande /push"
      - "Une règle de permission deny, ou un hook qui bloque la commande"
    correct: 4
    explanation: |
      Une instruction demande, le modèle peut l'ignorer. Une permission ou un hook est appliqué
      par le harness avant l'exécution : c'est une garantie.
```
