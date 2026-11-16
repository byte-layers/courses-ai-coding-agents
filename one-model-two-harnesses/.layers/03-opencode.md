---
title: "03 - Second essai : OpenCode"
description: "Déclarer Ollama à OpenCode, puis confier la même tâche sur une branche try-opencode"
---

Le second essai repart du même point, la branche `agent-config`, avec le même modèle. Seul le harness change : OpenCode, installé dans le lab précédent. Il faut d'abord lui dire où trouver le modèle.

## 03.1 - Déclarer Ollama à OpenCode

Le choix d'un modèle dépend de votre poste, pas du projet : on le range dans la configuration **globale** d'OpenCode, `~/.config/opencode/opencode.json`, valable pour tous vos projets. OpenCode la fusionne avec le `opencode.json` du projet, qui garde ses permissions.

Comme pour dsh, OpenCode parle à Ollama par son API compatible OpenAI. La commande crée le dossier de configuration, puis le fichier :

```exec
mkdir -p ~/.config/opencode && cat > ~/.config/opencode/opencode.json <<'FIN'
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "ollama": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Ollama",
      "options": { "baseURL": "http://localhost:11434/v1" },
      "models": { "qwen3.5:4b": { "name": "qwen3.5:4b" } }
    }
  },
  "model": "ollama/qwen3.5:4b"
}
FIN
```

Le fichier se lit ainsi :

| Champ | Rôle |
|---|---|
| `provider.ollama` | déclare un fournisseur de modèles nommé `ollama` |
| `"npm": "@ai-sdk/openai-compatible"` | la bibliothèque qu'OpenCode utilise pour parler à une API compatible OpenAI ; il la télécharge au premier lancement |
| `options.baseURL` | l'adresse de l'API d'Ollama, la même que dans la configuration de dsh |
| `models` | les modèles proposés, sous leur nom dans Ollama |
| `"model": "ollama/qwen3.5:4b"` | le modèle par défaut, sous la forme `fournisseur/modèle` |

Si ce fichier existe déjà parce que vous utilisez OpenCode avec un autre fournisseur, ajoutez-y le bloc `ollama` au lieu de le remplacer.

> 📖 [Les fournisseurs dans OpenCode, dont Ollama](https://opencode.ai/docs/providers/) · [L'API compatible OpenAI d'Ollama](https://docs.ollama.com/api/openai-compatibility)

```verify
F=~/.config/opencode/opencode.json
[ -f "$F" ] || { echo "Le fichier $F n'existe pas. Lancez la commande de cette section."; exit 1; }
node -e '
const c = JSON.parse(require("fs").readFileSync(process.argv[1], "utf8"));
const o = ((c.provider || {}).ollama || {});
const fail = (m) => { console.log(m); process.exit(1); };
if (!/:11434\/v1\/?$/.test((o.options || {}).baseURL || "")) fail("Le fournisseur ollama doit avoir baseURL http://localhost:11434/v1.");
if (!(o.models || {})["qwen3.5:4b"]) fail("Le modèle qwen3.5:4b n est pas déclaré dans le fournisseur ollama.");
if (c.model !== "ollama/qwen3.5:4b") fail("Le modèle par défaut doit être ollama/qwen3.5:4b.");
' "$F" || exit 1
```

## 03.2 - Confier la même tâche

Revenez sur la branche de départ et créez la branche `try-opencode`. Le code écrit par dsh reste sur `try-dsh` :

```exec
cd ~/byte-layers/ai/todo-api && git switch -c try-opencode agent-config
```

`opencode run` envoie une seule demande, affiche le travail de l'agent dans le terminal, puis rend la main. Envoyez la même demande qu'à dsh :

```exec
cd ~/byte-layers/ai/todo-api && opencode run "Le test test/count.test.js échoue. Ajoute la méthode countRemaining() au store de src/todos.js pour qu'il passe, puis lance npm test."
```

OpenCode affiche le modèle utilisé, `build · qwen3.5:4b`, puis chaque action de l'agent. Notez les mêmes choses que pour dsh : le temps, le nombre d'actions, les autorisations. Les permissions du lab précédent s'appliquent : `npm test` se lance sans demander, toute autre commande est soumise à votre accord. Le piège de l'étape précédente vaut ici aussi, avec la même solution en recours.

Lancez les tests vous-même, puis commitez le résultat sur `try-opencode` :

```exec
cd ~/byte-layers/ai/todo-api && npm test && git add src/todos.js && git commit -m "countRemaining, écrit avec OpenCode"
```

La vérification rejoue les tests de la branche `try-opencode` :

```verify
D=~/byte-layers/ai/todo-api
cd "$D" 2>/dev/null || { echo "Le dossier $D n'existe pas."; exit 1; }
git rev-parse --verify -q try-opencode >/dev/null || { echo "La branche try-opencode n'existe pas. Lancez la commande git switch de cette section."; exit 1; }
git show try-opencode:src/todos.js | grep -q countRemaining || { echo "La méthode countRemaining n'est pas commitée sur try-opencode. Commitez src/todos.js."; exit 1; }
T=$(mktemp -d)
git archive try-opencode | tar -x -C "$T"
if ! (cd "$T" && npm test >/dev/null 2>&1); then
  rm -rf "$T"
  echo "Les tests échouent sur la branche try-opencode. Lancez npm test, corrigez src/todos.js, puis commitez."
  exit 1
fi
rm -rf "$T"
```
