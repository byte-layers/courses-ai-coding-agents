---
title: "04 - Ce qui ne passe pas"
description: "Constater que les hooks ne passent pas dans dsh, et compenser la protection perdue"
---

Il reste les hooks : le plugin OpenCode qui bloque `.env`, celui qui vérifie la syntaxe après chaque écriture, et le script au format Claude Code.

## 04.1 - Les hooks dans dsh

dsh 0.2.2-rc.3 ne lit ni les plugins de `.opencode/plugins/`, propres à OpenCode, ni les hooks de `.claude/settings.json`, et sa documentation ne décrit pas de hooks à déclarer dans le dépôt. Un plugin expérimental de dsh, `@deepseek-ai/dsh-hooks-claude-code`, sait lire les hooks au format Claude Code. Sa version publiée, essayée le 16 novembre 2026, est déclarée incompatible avec dsh 0.2.2-rc.3 : dsh refuse de l'installer, sauf à accepter explicitement le risque par une commande dédiée. Le refus, que vous n'avez pas besoin de reproduire, se lit ainsi :

```text
dsh: installation rejected: Plugin @deepseek-ai/dsh-hooks-claude-code@0.0.1-rc.5 is incompatible with dsh 0.2.2-rc.3 … Running it may cause crashes or data loss.
```

On ne force pas un plugin qui annonce lui-même un risque de perte de données. Les hooks **ne passent pas**, et chacun de leurs rôles redevient une simple demande :

| Hook | Ce qui reste dans dsh |
|---|---|
| bloquer `.env` | la consigne de la section Sécurité de `AGENTS.md` : « Ne lisez jamais le fichier `.env` » |
| vérifier la syntaxe après chaque écriture | la consigne de la section Commandes : lancer `npm test` après chaque modification |

## 04.2 - Compenser la protection perdue

Faites le bilan pour `.env`. Dans OpenCode, deux garanties le protégeaient : la règle `read` en `deny` et le hook. Dans dsh, il ne reste que la consigne, qu'un modèle peut ignorer, et l'outil de lecture de dsh lit le fichier sans rien demander.

Quand une garantie ne passe pas, il reste à retirer ce qu'elle protégeait. Vérifiez que le `.env` de votre projet ne contient aucun vrai secret :

```exec
cd ~/byte-layers/ai/todo-api && cat .env
```

Il doit contenir les valeurs d'exemple copiées de `.env.example` au lab Une configuration d'agent portable d'abord, sans rien de réel :

```text
PORT=3000
API_TOKEN=change-me
```

Si `cat` répond `No such file or directory`, le projet n'a pas de `.env` : il n'y a rien à protéger.

C'est la règle à retenir pour tout outil : un agent qui n'a pas de garantie sur un fichier ne doit pas travailler là où un vrai secret est à sa portée. Cela vaut pour les fichiers du dossier de travail, mais aussi pour les variables d'environnement du shell où il lance ses commandes : `env` les affiche toutes. Dans un vrai projet, les secrets restent dans un gestionnaire de secrets, et seul le programme qui en a besoin les reçoit, au moment de son exécution.

```verify
F=~/byte-layers/ai/todo-api/.env
[ -f "$F" ] || exit 0
K=$(node -e '
const lines = require("fs").readFileSync(process.argv[1], "utf8").split(/\r?\n/);
for (const line of lines) {
  const m = line.replace(/^\s*export\s+/, "").match(/^\s*([A-Za-z_][A-Za-z0-9_]*)\s*=\s*(.*?)\s*$/);
  if (!m) continue;
  const value = m[2].replace(/^["\x27](.*)["\x27]$/, "$1");
  if (/TOKEN|SECRET|PASSWORD|PASSWD|KEY/i.test(m[1]) && value !== "" && value !== "change-me") { console.log(m[1]); break; }
}' "$F")
if [ -n "$K" ]; then
  echo "La variable $K de .env a une valeur réelle. Dans ce projet, .env ne doit garder que les valeurs d'exemple : lancez cp .env.example .env."
  exit 1
fi
```
