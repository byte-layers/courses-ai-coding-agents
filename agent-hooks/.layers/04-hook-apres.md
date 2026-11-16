---
title: "04 - Vérifier chaque fichier écrit"
description: "Écrire un hook qui vérifie la syntaxe de chaque fichier JavaScript juste après son écriture, et prévient le modèle"
---

Un hook n'agit pas seulement avant un outil. Placé **après**, il ne peut plus empêcher l'action, puisque le fichier est déjà écrit, mais il peut la vérifier et prévenir le modèle. Le fichier `AGENTS.md` demande à l'agent de lancer les tests : c'est une instruction, qu'un petit modèle oublie souvent. Le hook de cette étape garantit une vérification plus modeste mais systématique : la **syntaxe** de chaque fichier JavaScript écrit par les outils d'écriture d'OpenCode. Un fichier écrit par une commande du shell, comme `cat > x.js`, ne passe pas par ces outils et lui échappe.

## 04.1 - Le plugin de vérification

Le contrôle choisi est le plus simple : `node --check fichier.js` vérifie la syntaxe d'un fichier JavaScript sans l'exécuter. Il ne demande rien à installer. Dans un vrai projet, on y mettrait le linter de l'équipe. On écrit le plugin avec un manque, que vous compléterez :

```exec
cd ~/byte-layers/ai/todo-api && cat > .opencode/plugins/check-syntax.js <<'FIN'
import { execFileSync } from "node:child_process";

export const CheckSyntax = async ({ directory }) => ({
  "tool.execute.after": async (input, output) => {
    if (input.tool !== "write" && input.tool !== "edit") return;
    const file = input.args.filePath;
    // À vous : arrêtez ici si file ne se termine pas par ".js"
    try {
      execFileSync("node", ["--check", file], { cwd: directory, stdio: "pipe" });
    } catch (error) {
      output.output += "\n\nnode --check a trouvé une erreur de syntaxe :\n" + error.stderr;
    }
  },
});
FIN
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `cat > .opencode/plugins/check-syntax.js <<'FIN'` … `FIN` | écrit les lignes entre les deux `FIN` dans un second plugin, à côté du premier |

Le plugin se lit ainsi :

| Ligne | Rôle |
|---|---|
| `async ({ directory }) => …` | OpenCode passe au plugin le dossier du projet, `directory` |
| `"tool.execute.after"` | le hook appelé **après** chaque appel d'outil, une fois l'outil exécuté |
| `if (input.tool !== "write" && input.tool !== "edit") return;` | ne s'occupe que des outils qui écrivent un fichier : `write` (créer ou remplacer) et `edit` (modifier) |
| `input.args.filePath` | le fichier que l'outil vient d'écrire. Attention : après l'outil, les arguments sont dans `input.args`, alors qu'avant l'outil ils étaient dans `output.args` |
| `execFileSync("node", ["--check", file], …)` | lance `node --check` sur ce fichier et attend la fin ; en cas d'erreur de syntaxe, la fonction lance une erreur |
| `{ cwd: directory, stdio: "pipe" }` | lance la commande depuis le dossier du projet, et capture ses sorties au lieu de les afficher : c'est ce qui remplit `error.stderr` |
| `output.output += "…" + error.stderr` | ajoute le message de `node --check` au résultat de l'outil : c'est ce que le modèle lira, et il pourra corriger au tour suivant |

**À vous** : ouvrez `.opencode/plugins/check-syntax.js` dans votre éditeur. Remplacez le commentaire par une ligne qui arrête le hook quand `file` ne se termine pas par `.js`. Sans elle, le hook lancerait `node --check` sur un fichier Markdown ou JSON, et signalerait au modèle de fausses erreurs.

```hint
Une chaîne de caractères a une méthode `endsWith(".js")`. La ligne au-dessus montre comment arrêter le hook : `return;`.
```

````solution
```js
    const file = input.args.filePath;
    if (!file.endsWith(".js")) return;
```
````

**Testez le hook sans modèle.** Comme à l'étape 02, on appelle le hook avec un faux appel d'outil. On écrit d'abord un fichier volontairement cassé, `broken.js`, puis on présente au hook l'écriture de ce fichier :

```exec
cd ~/byte-layers/ai/todo-api && printf 'function f( {\n' > broken.js && node --input-type=module -e '
const { CheckSyntax } = await import("./.opencode/plugins/check-syntax.js");
const hooks = await CheckSyntax({ directory: process.cwd() });
const output = { output: "Wrote file successfully." };
await hooks["tool.execute.after"]({ tool: "write", args: { filePath: process.cwd() + "/broken.js" } }, output);
console.log(output.output);'; rm -f broken.js
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `printf 'function f( {\n' > broken.js` | écrit une ligne de JavaScript à laquelle il manque une parenthèse |
> | `CheckSyntax({ directory: process.cwd() })` | charge le plugin en lui passant le dossier courant comme dossier du projet |
> | `const output = { output: "Wrote file successfully." }` | le résultat que l'outil `write` d'OpenCode renvoie normalement |
> | `hooks["tool.execute.after"]({ tool: "write", args: { … } }, output)` | appelle le hook comme OpenCode juste après l'écriture de `broken.js` |
> | `; rm -f broken.js` | supprime le fichier de test, que la commande précédente ait réussi ou non |

Le résultat de l'outil contient maintenant le message de `node --check`, qui commence ainsi :

```text
Wrote file successfully.

node --check a trouvé une erreur de syntaxe :
…/broken.js:2

SyntaxError: Unexpected end of input
```

C'est exactement ce que le modèle recevrait à la place du simple `Wrote file successfully.` Vérifiez enfin votre ligne : la même commande, pour un fichier `notes.md` qui contient le même texte, ne doit rien ajouter au résultat.

```exec
cd ~/byte-layers/ai/todo-api && printf 'function f( {\n' > notes.md && node --input-type=module -e '
const { CheckSyntax } = await import("./.opencode/plugins/check-syntax.js");
const hooks = await CheckSyntax({ directory: process.cwd() });
const output = { output: "Wrote file successfully." };
await hooks["tool.execute.after"]({ tool: "write", args: { filePath: process.cwd() + "/notes.md" } }, output);
console.log(output.output);'; rm -f notes.md
```

La réponse attendue est seulement `Wrote file successfully.`

> 📖 [Les plugins OpenCode](https://opencode.ai/docs/plugins/) · [node --check](https://nodejs.org/api/cli.html#-c---check)

```verify
cd ~/byte-layers/ai/todo-api 2>/dev/null || { echo "Le dossier ~/byte-layers/ai/todo-api n'existe pas."; exit 1; }
[ -f .opencode/plugins/check-syntax.js ] || { echo "Le fichier .opencode/plugins/check-syntax.js n'existe pas. Lancez la commande de la section 04.1."; exit 1; }
node --input-type=module -e '
import { mkdtempSync, writeFileSync, rmSync } from "node:fs";
import { tmpdir } from "node:os";
import { join } from "node:path";
import { pathToFileURL } from "node:url";
let hooks;
try { hooks = await (await import(pathToFileURL(process.cwd() + "/.opencode/plugins/check-syntax.js"))).CheckSyntax({ directory: process.cwd() }); }
catch (e) { console.log("Le plugin ne se charge pas : " + e.message); process.exit(1); }
const hook = hooks && hooks["tool.execute.after"];
if (typeof hook !== "function") { console.log("Le plugin ne renvoie pas de hook tool.execute.after."); process.exit(1); }
const dir = mkdtempSync(join(tmpdir(), "hooks-"));
const files = { "bad.js": "function f( {\n", "good.js": "export const a = 1;\n", "notes.md": "# Notes ( {\n" };
for (const [name, text] of Object.entries(files)) writeFileSync(join(dir, name), text);
const run = async (tool, name) => {
  const output = { output: "Wrote file successfully." };
  try { await hook({ tool, args: { filePath: join(dir, name) } }, output); }
  catch (e) { return "ERREUR " + e.message; }
  return output.output;
};
let problem = "";
const bad = await run("write", "bad.js");
if (bad.startsWith("ERREUR")) problem = "Le hook plante sur bad.js : " + bad.slice(7);
else if (!bad.includes("erreur de syntaxe")) problem = "Le hook ne signale pas l’erreur de syntaxe de bad.js au modèle.";
else if ((await run("write", "good.js")) !== "Wrote file successfully.") problem = "Le hook modifie le résultat pour un fichier JavaScript correct.";
else if ((await run("write", "notes.md")) !== "Wrote file successfully.") problem = "Le hook vérifie aussi les fichiers qui ne sont pas en .js : ajoutez la ligne qui l’arrête.";
rmSync(dir, { recursive: true, force: true });
if (problem) { console.log(problem); process.exit(1); }'
```

## 04.2 - Le même hook chez Claude Code

Chez Claude Code, ce hook serait un script déclaré sous `PostToolUse`, avec `"matcher": "Write|Edit"`. Le chemin du fichier arrive dans `tool_input.file_path`. Pour que le modèle lise le message, le script sort avec le code 2 : l'écriture n'est pas annulée, puisqu'elle a déjà eu lieu, mais stderr est transmis au modèle. Le piège du code 1 vaut aussi ici : avec lui, le message ne s'affiche que pour vous, et le modèle ne saura jamais que le fichier est cassé.

| | Avant l'outil | Après l'outil |
|---|---|---|
| OpenCode | `tool.execute.before` : `throw` refuse l'appel | `tool.execute.after` : compléter `output.output` |
| Claude Code | `PreToolUse` : le code 2 refuse l'appel | `PostToolUse` : le code 2 transmet stderr au modèle |
| Peut empêcher l'action | oui | non, elle a déjà eu lieu |
