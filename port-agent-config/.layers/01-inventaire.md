---
title: "01 - Faire l'inventaire"
description: "Lister les briques de configuration du projet et créer la branche du lab"
---

Avant de porter une configuration, il faut savoir ce qu'elle contient. Le projet `todo-api` en a accumulé au fil des labs précédents, dans des fichiers dont certains ne servent qu'à un outil.

## 01.1 - Créer la branche du lab

Le portage part de la configuration la plus complète, celle de la branche `hooks` :

```exec
cd ~/byte-layers/ai/todo-api && git switch -c port-dsh hooks
```

Git répond `Switched to a new branch 'port-dsh'`. Si la branche existe déjà, parce que vous reprenez le lab, placez-vous dessus avec `git switch port-dsh`.

## 01.2 - Lister les briques

Les fichiers de configuration d'agent sont tous des fichiers que Git suit. Listez-les :

```exec
cd ~/byte-layers/ai/todo-api && git ls-files | grep -E 'AGENTS\.md|^\.agents/|^\.opencode/|^\.claude/|opencode\.json'
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `cd ~/byte-layers/ai/todo-api &&` | se place dans le projet, puis lance la suite si le `cd` a réussi |
> | `git ls-files` | liste les fichiers que Git suit dans la branche courante |
> | `\|` | envoie cette liste à la commande suivante au lieu de l'écran |
> | `grep -E '…'` | ne garde que les lignes qui correspondent au motif ; `-E` autorise `\|` pour dire « ou » |
> | `AGENTS\.md\|^\.agents/\|…` | les noms de fichiers ou de dossiers de la configuration ; `^` veut dire « au début du chemin », `\.` un vrai point |

La liste contient au moins ces huit fichiers, que l'on range par brique :

```text
.agents/skills/ears-spec/SKILL.md
.claude/hooks/block-env.js
.claude/settings.json
.opencode/plugins/block-env.js
.opencode/plugins/check-syntax.js
AGENTS.md
opencode.json
test/AGENTS.md
```

| Brique | Fichiers | Écrits pour |
|---|---|---|
| Instructions du projet | `AGENTS.md` | le format ouvert AGENTS.md |
| Consigne de sous-dossier | `test/AGENTS.md` | le format ouvert AGENTS.md |
| Skill | `.agents/skills/ears-spec/SKILL.md` | la spécification Agent Skills |
| Permissions | `opencode.json` | OpenCode seulement |
| Hooks | `.opencode/plugins/*.js`, `.claude/hooks/block-env.js` et `.claude/settings.json` | OpenCode, et le format de Claude Code |

```verify
D=~/byte-layers/ai/todo-api
cd "$D" 2>/dev/null || { echo "Le dossier $D n'existe pas : faites d'abord le lab Hooks : demander ou garantir."; exit 1; }
if [ "$(git branch --show-current)" != "port-dsh" ]; then
  echo "Vous n'êtes pas sur la branche port-dsh. Lancez : git switch -c port-dsh hooks (ou git switch port-dsh si elle existe déjà)."
  exit 1
fi
V=$(dsh --version 2>/dev/null)
case "$V" in *0.2.2-rc.3*) ;; *) echo "Ce lab décrit dsh 0.2.2-rc.3 (lu : ${V:-dsh introuvable}). Installez cette version : npm install -g @deepseek-ai/dsh@0.2.2-rc.3"; exit 1;; esac
for f in AGENTS.md .agents/skills/ears-spec/SKILL.md opencode.json .opencode/plugins/block-env.js; do
  git ls-files --error-unmatch "$f" >/dev/null 2>&1 || { echo "$f manque sur la branche : la branche hooks du lab Hooks : demander ou garantir doit le contenir."; exit 1; }
done
```
