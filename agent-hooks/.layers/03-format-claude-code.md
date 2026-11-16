---
title: "03 - Le même hook au format Claude Code"
description: "Réécrire le hook en script au format de Claude Code, et éviter le piège du code de sortie 1"
---

Le plugin de l'étape précédente ne sert qu'à OpenCode. Claude Code, lui, lance des **scripts** : l'appel d'outil arrive en JSON sur l'entrée standard du script, et son **code de sortie** décide de la suite. Ce format de script est repris par plusieurs autres outils, comme le montre le tableau des hooks du bit [Où chaque harness range sa configuration](https://byte-layers.com/fr/bits/agent-config-crosswalk). Vous n'avez pas besoin de Claude Code pour cette étape : le script se teste seul.

## 03.1 - Écrire le script

Le script lit tout le JSON reçu, y cherche `.env` avec le même motif que votre plugin, et refuse si besoin. On l'écrit dans `.claude/hooks/`, le dossier habituel de ces scripts :

```exec
cd ~/byte-layers/ai/todo-api && mkdir -p .claude/hooks && cat > .claude/hooks/block-env.js <<'FIN'
// Hook PreToolUse au format Claude Code : l'appel d'outil arrive en JSON sur l'entrée standard
const SECRET = /(^|[^\w.])\.env(?!\.example)(?!\w)/;
let data = "";
for await (const chunk of process.stdin) data += chunk;
let event;
try {
  event = JSON.parse(data);
} catch {
  console.error("Hook block-env : appel illisible, refusé par prudence.");
  process.exit(1);
}
if (SECRET.test(JSON.stringify(event.tool_input ?? {}))) {
  console.error("Bloqué par le hook : le fichier .env ne doit pas être lu.");
  process.exit(1);
}
FIN
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `mkdir -p .claude/hooks` | crée le dossier des scripts de hooks ; `-p` crée aussi `.claude` et ne proteste pas s'il existe déjà |
> | `cat > .claude/hooks/block-env.js <<'FIN'` … `FIN` | écrit les lignes entre les deux `FIN` dans le fichier |

L'**entrée standard** est ce qu'un programme reçoit quand on lui envoie du texte, par un tube `|` ou depuis un autre programme. Claude Code y écrit l'appel d'outil en JSON. Le script se lit ainsi :

| Ligne | Rôle |
|---|---|
| `for await (const chunk of process.stdin) data += chunk;` | lit toute l'entrée standard, morceau par morceau. Cette écriture n'est permise qu'en module JavaScript : elle marche ici parce que le `package.json` du projet déclare `"type": "module"` |
| `try { event = JSON.parse(data); } catch { … }` | transforme le texte reçu en objet ; si le texte n'est pas du JSON valide, le script refuse par prudence au lieu de planter |
| `event.tool_input ?? {}` | les arguments de l'outil ; `??` donne `{}` à la place si `tool_input` est absent |
| `console.error("…")` | écrit le message sur la **sortie d'erreur** (stderr), celle que lit Claude Code |
| `process.exit(1)` | termine le script avec le code de sortie 1 |

Le nom de l'outil arrive dans `event.tool_name` (`Read`, `Bash`…). Le script n'en a pas besoin : on choisira les outils concernés dans la déclaration du hook, en fin d'étape. La seule différence avec le plugin est la manière de refuser : un code de sortie au lieu d'une erreur JavaScript.

## 03.2 - Tester le script

Claude Code enverrait au script un JSON comme celui-ci pour la commande `cat .env`. On le lui passe à la main, puis on affiche le code de sortie :

```exec
cd ~/byte-layers/ai/todo-api && echo '{"hook_event_name":"PreToolUse","tool_name":"Bash","tool_input":{"command":"cat .env"}}' | node .claude/hooks/block-env.js; echo "code de sortie : $?"
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `echo '{…}'` | écrit le JSON de l'appel d'outil, tel que Claude Code l'enverrait |
> | `\| node .claude/hooks/block-env.js` | le tube `\|` branche ce texte sur l'entrée standard du script |
> | `;` | lance la commande suivante, que la précédente ait réussi ou non |
> | `echo "code de sortie : $?"` | affiche `$?`, le code de sortie de la dernière commande |

Le script affiche son message, puis le code 1 :

```text
Bloqué par le hook : le fichier .env ne doit pas être lu.
code de sortie : 1
```

Le hook semble fonctionner. Il ne protège pourtant rien.

## 03.3 - Le piège du code de sortie 1

Claude Code ne lit pas le code de sortie comme un shell, où tout code autre que 0 signale un échec. Il distingue trois cas :

| Code de sortie | Ce que fait Claude Code |
|---|---|
| 0 | l'appel continue normalement, en passant par les règles de permission |
| **2** | l'appel est **bloqué** : l'outil n'est pas exécuté, et le texte de stderr est renvoyé au modèle comme raison |
| tout autre code, dont 1 | une **erreur non bloquante** : Claude Code affiche la première ligne de stderr pour vous, et **l'outil s'exécute quand même** |

Avec `exit 1`, le script affiche donc son avertissement, puis Claude Code lit le fichier `.env`. Le piège vient de l'habitude du shell, où `exit 1` veut dire « échec ». Un hook de garde qui plante produit lui aussi un code 1, et laisse tout passer : c'est pour cela que le script attrape l'erreur de `JSON.parse` au lieu de planter.

**À vous** : ouvrez `.claude/hooks/block-env.js` dans votre éditeur, et corrigez-le pour qu'il bloque vraiment l'appel, dans les deux cas où il refuse.

```hint
Le script appelle `process.exit` à deux endroits : le JSON illisible et le fichier `.env`. Les deux doivent bloquer.
```

````solution
```js
} catch {
  console.error("Hook block-env : appel illisible, refusé par prudence.");
  process.exit(2);
}
if (SECRET.test(JSON.stringify(event.tool_input ?? {}))) {
  console.error("Bloqué par le hook : le fichier .env ne doit pas être lu.");
  process.exit(2);
}
```
````

Relancez le test de la section précédente : il doit afficher `code de sortie : 2`. Le même test avec `cat .env.example` doit, lui, donner le code 0 :

```exec
cd ~/byte-layers/ai/todo-api && echo '{"hook_event_name":"PreToolUse","tool_name":"Bash","tool_input":{"command":"cat .env.example"}}' | node .claude/hooks/block-env.js; echo "code de sortie : $?"
```

Le script n'affiche rien d'autre que `code de sortie : 0`. Claude Code accepte aussi une réponse en JSON sur la sortie standard pour refuser un appel ; ce lab s'en tient au code 2, que les autres outils comprennent aussi.

```verify
cd ~/byte-layers/ai/todo-api 2>/dev/null || { echo "Le dossier ~/byte-layers/ai/todo-api n'existe pas."; exit 1; }
H=.claude/hooks/block-env.js
[ -f "$H" ] || { echo "Le script $H n'existe pas. Lancez la commande de la section 03.1."; exit 1; }
run() { printf '%s' "$1" | node "$H" >/dev/null 2>&1; echo $?; }
c=$(run '{"hook_event_name":"PreToolUse","tool_name":"Bash","tool_input":{"command":"cat .env"}}')
if [ "$c" = 1 ]; then echo "Le script sort avec le code 1 pour cat .env : Claude Code exécuterait quand même la commande. Il faut le code 2."; exit 1; fi
if [ "$c" != 2 ]; then echo "Le script sort avec le code $c pour cat .env : il doit sortir avec le code 2."; exit 1; fi
c=$(run '{"hook_event_name":"PreToolUse","tool_name":"Read","tool_input":{"file_path":"/home/dev/todo-api/.env"}}')
[ "$c" = 2 ] || { echo "Le script ne bloque pas la lecture de .env par l'outil Read (code $c au lieu de 2)."; exit 1; }
c=$(run '{"hook_event_name":"PreToolUse","tool_name":"Bash","tool_input":{"command":"cat .env.example .env"}}')
[ "$c" = 2 ] || { echo "Le script laisse passer cat .env.example .env (code $c) : cette commande lit aussi .env."; exit 1; }
c=$(run 'pas du JSON')
[ "$c" = 2 ] || { echo "Le script sort avec le code $c sur un appel illisible : il doit refuser par prudence, avec le code 2."; exit 1; }
c=$(run '{"hook_event_name":"PreToolUse","tool_name":"Bash","tool_input":{"command":"cat .env.example"}}')
[ "$c" = 0 ] || { echo "Le script bloque cat .env.example (code $c) : le fichier d'exemple doit rester lisible."; exit 1; }
c=$(run '{"hook_event_name":"PreToolUse","tool_name":"Bash","tool_input":{"command":"grep -rn process.env src"}}')
[ "$c" = 0 ] || { echo "Le script bloque grep -rn process.env src (code $c) : process.env n'est pas le fichier .env."; exit 1; }
```

## 03.4 - Déclarer le hook

Un script de hook ne fait rien tant que l'outil ne sait pas quand le lancer. Claude Code le déclare dans `.claude/settings.json`, le fichier de réglages du projet, que l'on commite. Ce fichier est fourni complet : lancez la commande sans la modifier.

```exec
cd ~/byte-layers/ai/todo-api && cat > .claude/settings.json <<'FIN'
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Read|Grep|Bash",
        "hooks": [
          {
            "type": "command",
            "command": "node \"$CLAUDE_PROJECT_DIR\"/.claude/hooks/block-env.js"
          }
        ]
      }
    ]
  }
}
FIN
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `cat > .claude/settings.json <<'FIN'` … `FIN` | écrit les lignes entre les deux `FIN` dans le fichier, en remplaçant son contenu s'il existait |

Le fichier se lit ainsi :

| Clé | Rôle |
|---|---|
| `"hooks"` (la première) | la section des hooks du fichier de réglages |
| `"PreToolUse"` | le moment : avant chaque appel d'outil |
| `"matcher": "Read\|Grep\|Bash"` | les outils concernés : lecture, recherche dans les fichiers, et shell |
| `"hooks"` (la seconde) | la liste des commandes à lancer quand un de ces outils est appelé |
| `"type": "command"` | le hook est une commande à lancer |
| `"command": "node \"$CLAUDE_PROJECT_DIR\"/…"` | la commande ; Claude Code remplit `$CLAUDE_PROJECT_DIR` avec le dossier du projet, ce qui marche quel que soit le dossier courant |

Si vous utilisez Claude Code, la commande `/hooks` dans une session ouverte sur le projet affiche ce hook. Les autres outils qui lancent des scripts de hooks ont chacun leur fichier de déclaration, et parfois des noms de champs différents dans le JSON reçu : vérifiez-les dans leur documentation avant d'y brancher ce script.

> 📖 [Les hooks de Claude Code](https://code.claude.com/docs/en/hooks) · [Le guide des hooks de Claude Code](https://code.claude.com/docs/en/hooks-guide)

```verify
cd ~/byte-layers/ai/todo-api 2>/dev/null || { echo "Le dossier ~/byte-layers/ai/todo-api n'existe pas."; exit 1; }
node -e '
let s;
try { s = JSON.parse(require("fs").readFileSync(".claude/settings.json", "utf8")); }
catch (e) { console.log(".claude/settings.json est absent ou n’est pas un JSON valide. Relancez la commande de la section 03.4."); process.exit(1); }
const groups = (s.hooks && s.hooks.PreToolUse) || [];
const ok = groups.some(g => /Bash/.test(g.matcher || "") && (g.hooks || []).some(h => h.type === "command" && /block-env\.js/.test(h.command || "")));
if (!ok) { console.log(".claude/settings.json ne déclare pas block-env.js en PreToolUse pour Bash."); process.exit(1); }
'
```
