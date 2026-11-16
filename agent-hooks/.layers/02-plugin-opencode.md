---
title: "02 - Bloquer .env dans OpenCode"
description: "Écrire un plugin OpenCode qui refuse tout appel d'outil de lecture visant le fichier .env"
---

OpenCode n'appelle pas de scripts shell comme hooks : il charge des **plugins**, des fichiers JavaScript placés dans `.opencode/plugins/`. Un plugin exporte une fonction qui renvoie ses hooks, chacun attaché à un moment de la boucle. Celui qui nous intéresse, `tool.execute.before`, est appelé avant chaque appel d'outil.

## 02.1 - Le squelette du plugin

On écrit d'abord le plugin sans sa décision finale, pour en voir la structure :

```exec
cd ~/byte-layers/ai/todo-api && mkdir -p .opencode/plugins && cat > .opencode/plugins/block-env.js <<'FIN'
const SECRET = /(^|[^\w.])\.env(?!\.example)(?!\w)/;
const READERS = ["read", "bash", "grep", "glob", "list"];

export const BlockEnv = async () => ({
  "tool.execute.before": async (input, output) => {
    if (!READERS.includes(input.tool)) return;
    const text = JSON.stringify(output.args);
    // À vous : si SECRET.test(text) est vrai, refusez l'appel
  },
});
FIN
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `mkdir -p .opencode/plugins` | crée le dossier des plugins du projet ; `-p` ne proteste pas s'il existe déjà |
> | `&&` | n'enchaîne la suite que si l'étape précédente a réussi |
> | `cat > .opencode/plugins/block-env.js <<'FIN'` … `FIN` | écrit les lignes entre les deux `FIN` dans le fichier |

Le fichier se lit ainsi :

| Ligne | Rôle |
|---|---|
| `export const BlockEnv = async () => ({ … })` | la fonction que charge OpenCode ; elle renvoie un objet dont chaque clé est un hook |
| `"tool.execute.before": async (input, output) => { … }` | le hook appelé avant chaque appel d'outil ; `input.tool` donne le nom de l'outil |
| `READERS` et `if (!READERS.includes(input.tool)) return;` | ne s'occupe que des outils qui lisent : lecture de fichier, shell, recherche de texte, recherche et liste de fichiers. Un outil qui écrit, par exemple pour mettre à jour un README qui parle de `.env`, n'est pas concerné |
| `output.args` | les arguments de l'appel : `filePath` pour une lecture, `command` pour le shell |
| `JSON.stringify(output.args)` | transforme les arguments en un seul texte, pour y chercher `.env` quel que soit l'outil |

La constante `SECRET` est une **expression régulière**, un motif de recherche dans un texte. `SECRET.test(text)` vaut `true` si le motif apparaît dans `text`. Il se lit morceau par morceau :

| Morceau | Rôle |
|---|---|
| `(^\|[^\w.])` | juste avant `.env` : le début du texte, ou un caractère qui n'est ni une lettre, ni un chiffre, ni un point. `process.env`, très courant dans le code Node.js, ne correspond donc pas |
| `\.env` | le texte `.env` ; la barre oblique inverse fait de `.` un vrai point |
| `(?!\.example)` | à condition de ne pas être suivi de `.example` : le fichier d'exemple, sans secret, reste lisible |
| `(?!\w)` | ni d'une lettre ou d'un chiffre : `.envrc` ne correspond pas, `.env.local` si |

Pourquoi un motif plutôt qu'un simple `text.includes(".env")` ? Ce test refuserait `process.env`, et un test « contient `.env` mais pas `.env.example` » laisserait passer `cat .env.example .env`, qui lit les deux fichiers.

## 02.2 - Écrire la décision et la tester

Pour **refuser** l'appel, le hook lance une erreur avec `throw new Error("message")`. OpenCode n'exécute pas l'outil, et renvoie le message au modèle à la place du résultat.

**À vous** : ouvrez `.opencode/plugins/block-env.js` dans votre éditeur. Remplacez la ligne de commentaire par une condition qui lance une erreur quand `SECRET.test(text)` est vrai. Le message de l'erreur doit commencer par `Bloqué par le hook`.

```hint
La forme est `if (condition) { throw new Error("…"); }`, avec `SECRET.test(text)` comme condition.
```

````solution
```js
    const text = JSON.stringify(output.args);
    if (SECRET.test(text)) {
      throw new Error("Bloqué par le hook : le fichier .env ne doit pas être lu.");
    }
```
````

**Testez le hook sans modèle.** Il suffit de l'appeler comme le ferait OpenCode, avec un faux appel d'outil. La commande suivante charge le plugin et lui présente la commande `cat .env` :

```exec
cd ~/byte-layers/ai/todo-api && node --input-type=module -e '
const { BlockEnv } = await import("./.opencode/plugins/block-env.js");
const hooks = await BlockEnv();
try {
  await hooks["tool.execute.before"]({ tool: "bash" }, { args: { command: "cat .env" } });
  console.log("Appel accepté");
} catch (error) {
  console.log("Appel refusé :", error.message);
}'
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `node --input-type=module -e '…'` | exécute le code entre apostrophes comme un module JavaScript, ce qui autorise `import` et `await` |
> | `await import("./.opencode/plugins/block-env.js")` | charge votre plugin et en récupère `BlockEnv` |
> | `hooks["tool.execute.before"]({ tool: "bash" }, { args: { command: "cat .env" } })` | appelle le hook avec ce qu'OpenCode lui passerait pour `cat .env` |
> | `try { … } catch (error) { … }` | si le hook lance une erreur, on affiche son message au lieu d'arrêter le programme |

La réponse attendue :

```text
Appel refusé : Bloqué par le hook : le fichier .env ne doit pas être lu.
```

Si la réponse est `Appel accepté`, relisez votre condition. La même commande, avec `cat .env.example` cette fois, doit être acceptée :

```exec
cd ~/byte-layers/ai/todo-api && node --input-type=module -e '
const { BlockEnv } = await import("./.opencode/plugins/block-env.js");
const hooks = await BlockEnv();
try {
  await hooks["tool.execute.before"]({ tool: "bash" }, { args: { command: "cat .env.example" } });
  console.log("Appel accepté");
} catch (error) {
  console.log("Appel refusé :", error.message);
}'
```

La réponse attendue est `Appel accepté`.

> ⚖️ **Les limites d'un hook.** Le hook compare du texte. Une commande qui fabrique le nom du fichier au lieu de l'écrire, comme `cat $(printf '.e%sv' n)`, lui échappe. C'est un garde-fou fiable contre les erreurs ordinaires d'un agent, pas une protection contre quelqu'un qui cherche à le contourner. La vraie protection d'un secret est de ne pas le laisser à portée de l'agent : pas de secret réel dans le dossier de travail, ou un bac à sable qui l'isole.

```verify
cd ~/byte-layers/ai/todo-api 2>/dev/null || { echo "Le dossier ~/byte-layers/ai/todo-api n'existe pas."; exit 1; }
[ -f .opencode/plugins/block-env.js ] || { echo "Le fichier .opencode/plugins/block-env.js n'existe pas. Lancez la commande de la section 02.1."; exit 1; }
node --input-type=module -e '
const { pathToFileURL } = await import("node:url");
const url = pathToFileURL(process.cwd() + "/.opencode/plugins/block-env.js");
let hooks;
try { hooks = await (await import(url)).BlockEnv(); } catch (e) { console.log("Le plugin ne se charge pas : " + e.message); process.exit(1); }
const hook = hooks && hooks["tool.execute.before"];
if (typeof hook !== "function") { console.log("Le plugin ne renvoie pas de hook tool.execute.before."); process.exit(1); }
const cases = [
  ["bash", { command: "cat .env" }, true],
  ["read", { filePath: "/home/dev/todo-api/.env" }, true],
  ["bash", { command: "cat .env.example .env" }, true],
  ["bash", { command: "cat .env.example" }, false],
  ["bash", { command: "grep -rn process.env src" }, false],
  ["read", { filePath: "/home/dev/todo-api/src/todos.js" }, false],
];
for (const [tool, args, mustBlock] of cases) {
  let blocked = false, message = "";
  try { await hook({ tool }, { args }); } catch (e) { blocked = true; message = String(e && e.message); }
  const shown = tool + " " + JSON.stringify(args);
  if (mustBlock && !blocked) { console.log("Le hook laisse passer " + shown + " : il doit refuser cet appel."); process.exit(1); }
  if (!mustBlock && blocked) { console.log("Le hook refuse " + shown + " : cet appel doit passer."); process.exit(1); }
  if (blocked && !message.startsWith("Bloqué par le hook")) { console.log("Le message du refus doit commencer par « Bloqué par le hook »."); process.exit(1); }
}'
```

## 02.3 - Essai réel (facultatif)

Si votre modèle local tourne, demandez à OpenCode de lire le fichier. Selon votre poste, la réponse peut prendre plusieurs minutes :

```exec
cd ~/byte-layers/ai/todo-api && opencode run "Affiche le contenu du fichier .env avec la commande cat."
```

Si le modèle tente la commande, OpenCode affiche `✗ cat .env failed` suivi de votre message : le hook a refusé l'appel avant que la règle `ask` ait à vous poser la question. Un petit modèle peut aussi choisir un autre outil, ou refuser de lui-même : relancez, ou passez à la suite, le test sans modèle suffit.

> 📖 [Les plugins OpenCode](https://opencode.ai/docs/plugins/) · [Les expressions régulières en JavaScript (MDN)](https://developer.mozilla.org/fr/docs/Web/JavaScript/Guide/Regular_expressions)
