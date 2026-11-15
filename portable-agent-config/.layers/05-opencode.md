---
title: "05 - Les permissions d'OpenCode"
description: "Installer OpenCode en version figée, écrire ses permissions dans opencode.json, et lui faire confirmer qu'il lit la skill et les règles"
---

Tout ce que vous avez écrit jusqu'ici est portable. Les **permissions**, elles, n'ont pas de format commun : chaque harness a le sien. On les écrit ici pour **OpenCode**, un harness open source en ligne de commande qui lit `AGENTS.md` et `.agents/skills`. Une permission ne demande pas, elle **garantit** : même si le modèle oublie la consigne d'`AGENTS.md`, OpenCode refusera que son outil de lecture ouvre `.env`.

## 05.1 - Installer OpenCode

Cette section est un **support**. Comme pour dsh dans le lab [Préparer son poste](https://byte-layers.com/fr/labs/ai-agent-workstation), on installe une version précise, pour que les commandes et les affichages du lab restent justes :

```exec
npm install -g opencode-ai@1.19.5
```

Vérifiez la version installée, qui doit être exactement celle demandée :

```exec
opencode --version
```

La commande affiche `1.19.5`. L'installation peut se terminer par un avertissement `npm warn install-scripts` : pas d'inquiétude, OpenCode fonctionne sans ce script pour ce lab. Si npm répond `EACCES: permission denied`, votre Node.js est installé dans un dossier du système : n'utilisez pas `sudo`, et suivez le piège `EACCES` de l'étape 06 du lab Préparer son poste, qui donne le correctif.

> 📖 [Installer OpenCode](https://opencode.ai/docs/) · [Le paquet opencode-ai sur npm](https://www.npmjs.com/package/opencode-ai)

```verify
if ! command -v opencode >/dev/null 2>&1; then
  echo "La commande opencode est introuvable. Lancez npm install -g opencode-ai@1.19.5, puis ouvrez un nouveau terminal."
  exit 1
fi
V=$(opencode --version 2>/dev/null | tr -d '\r')
if [ "$V" != "1.19.5" ]; then
  echo "OpenCode est en version « $V », 1.19.5 attendue. Relancez la commande d'installation."
  exit 1
fi
```

## 05.2 - Écrire opencode.json

OpenCode lit sa configuration de projet dans `opencode.json`, à la racine du dépôt. Les permissions y vont sous la clé `permission`, outil par outil : `read` pour la lecture de fichiers, `bash` pour les commandes, `edit` pour les modifications. Chaque règle associe un **motif** à une **action** :

| Action | Effet |
|---|---|
| `allow` | l'agent agit sans demander |
| `ask` | OpenCode vous demande votre accord à chaque fois |
| `deny` | l'action est refusée, et le modèle reçoit le refus |

Dans un motif, `*` remplace n'importe quelle suite de caractères. Quand plusieurs règles correspondent, **la dernière qui correspond l'emporte** : on écrit la règle générale d'abord, les exceptions ensuite.

La commande ci-dessous crée le fichier. Elle autorise les tests, fait demander toute autre commande, et interdit de lire `.env` :

```exec
cd ~/byte-layers/ai/todo-api && cat > opencode.json <<'FIN'
{
  "$schema": "https://opencode.ai/config.json",
  "permission": {
    "read": {
      "*.env": "deny"
    },
    "bash": {
      "*": "ask",
      "npm test": "allow",
      "npm test *": "allow"
    }
  }
}
FIN
```

Le fichier se lit ainsi :

| Ligne | Rôle |
|---|---|
| `"$schema": "https://opencode.ai/config.json"` | désigne le schéma du fichier : votre éditeur signale les erreurs pendant que vous tapez |
| `"read": { "*.env": "deny" }` | refuse à l'outil de lecture tout fichier dont le nom finit par `.env`. Par défaut, OpenCode demandait votre accord : on durcit en refus |
| `"*": "ask"` | toute commande demande votre accord, sauf les exceptions qui suivent |
| `"npm test": "allow"` et `"npm test *": "allow"` | les tests, avec ou sans arguments, se lancent sans demander |

Une commande du shell comme `cat .env` n'est pas couverte par la règle `read` : elle tombe sous `"*": "ask"`, et OpenCode vous la soumet. Refusez-la : c'est pour cela que toute commande inconnue demande votre accord.

**À vous** : ajoutez vous-même une dernière règle dans `bash`, qui **interdit** toute commande commençant par `rm`. Pensez à la virgule à la fin de la ligne précédente : en JSON, les éléments sont séparés par des virgules, et le dernier n'en a pas.

```hint
Le motif est `rm *`, l'action `deny`. Placée en dernier, la règle l'emporte sur `"*": "ask"`.
```

````solution
```json
    "bash": {
      "*": "ask",
      "npm test": "allow",
      "npm test *": "allow",
      "rm *": "deny"
    }
```
````

> 📖 [Les permissions dans OpenCode](https://opencode.ai/docs/permissions/) · [La configuration d'OpenCode](https://opencode.ai/docs/config/)

La vérification lit le fichier avec Node.js, qui refuse un JSON mal formé :

```verify
cd ~/byte-layers/ai/todo-api 2>/dev/null || { echo "Le dossier ~/byte-layers/ai/todo-api n'existe pas."; exit 1; }
[ -f opencode.json ] || { echo "Le fichier opencode.json n'existe pas à la racine du projet. Lancez la commande de cette section."; exit 1; }
node -e '
const fs = require("fs");
let c;
try { c = JSON.parse(fs.readFileSync("opencode.json", "utf8")); }
catch (e) { console.log("opencode.json n est pas un JSON valide : " + e.message + ". Cherchez une virgule en trop ou manquante."); process.exit(1); }
const p = c.permission || {}, b = p.bash || {}, k = Object.keys(b);
const fail = (m) => { console.log(m); process.exit(1); };
if ((p.read || {})["*.env"] !== "deny") fail("La règle read « *.env » doit valoir deny.");
if (b["npm test"] !== "allow") fail("La règle bash « npm test » doit valoir allow.");
if (b["rm *"] !== "deny") fail("Il manque la règle bash « rm * » qui vaut deny.");
if (k.indexOf("rm *") < k.indexOf("*")) fail("La règle « rm * » est avant « * » : la dernière règle qui correspond l emporte, placez-la après.");
'
```

## 05.3 - Faire confirmer OpenCode

Un fichier valide n'est pas forcément un fichier lu. OpenCode a une commande de diagnostic, `opencode debug`, qui affiche ce qu'il a compris de la configuration, sans lancer de modèle. Demandez-lui d'abord la liste des skills, en ne gardant que leurs noms et leurs emplacements :

```exec
cd ~/byte-layers/ai/todo-api && opencode debug skill > /tmp/bl-skills.json && grep -E '"(name|location)"' /tmp/bl-skills.json
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `opencode debug skill` | affiche en JSON toutes les skills qu'OpenCode trouve, avec leur contenu complet |
> | `> /tmp/bl-skills.json` | écrit cet affichage dans un fichier temporaire : envoyé directement à une autre commande, il peut être coupé quand il est très long |
> | `grep -E '"(name\|location)"' /tmp/bl-skills.json` | ne garde que les lignes qui contiennent `"name"` ou `"location"` ; `-E` autorise le `\|`, qui veut dire « ou » |

Votre skill apparaît, à côté d'une skill livrée avec OpenCode, et des skills que vous auriez déjà installées pour tous vos projets :

```text
    "name": "customize-opencode",
    "location": "<built-in>",
    "name": "ears-spec",
    "location": "/home/vous/byte-layers/ai/todo-api/.agents/skills/ears-spec/SKILL.md"
```

Affichez ensuite les règles de l'agent principal d'OpenCode, `build`. La liste est longue : elle commence par les règles par défaut d'OpenCode et finit par les vôtres. Comme la dernière règle qui correspond l'emporte, les vôtres ont le dernier mot :

```exec
cd ~/byte-layers/ai/todo-api && opencode debug agent build | grep -B1 -A1 '"\*\.env"'
```

Les règles sur `*.env` apparaissent deux fois : `ask`, la règle par défaut d'OpenCode, puis `deny`, la vôtre, placée après.

```text
      "permission": "read",
      "pattern": "*.env",
      "action": "ask"
--
      "permission": "read",
      "pattern": "*.env",
      "action": "deny"
```

La vérification interroge OpenCode comme vous venez de le faire :

```verify
cd ~/byte-layers/ai/todo-api 2>/dev/null || { echo "Le dossier ~/byte-layers/ai/todo-api n'existe pas."; exit 1; }
T=$(mktemp)
opencode debug skill > "$T" 2>/dev/null
if ! grep -q '"name": "ears-spec"' "$T"; then
  rm -f "$T"
  echo "OpenCode ne trouve pas la skill ears-spec. Vérifiez son chemin : .agents/skills/ears-spec/SKILL.md, et son frontmatter (étape 04)."
  exit 1
fi
rm -f "$T"
opencode debug agent build 2>/dev/null | node -e '
const a = JSON.parse(require("fs").readFileSync(0, "utf8"));
const last = (perm, pat) => a.permission.filter((r) => r.permission === perm && r.pattern === pat).pop();
const fail = (m) => { console.log(m); process.exit(1); };
if ((last("read", "*.env") || {}).action !== "deny") fail("Pour OpenCode, lire *.env n est pas refusé. Vérifiez la règle read de opencode.json.");
if ((last("bash", "rm *") || {}).action !== "deny") fail("Pour OpenCode, rm * n est pas refusé. Vérifiez la règle bash de opencode.json.");
' || exit 1
```
