---
title: "06 - Installer DeepSeek Harness et le brancher sur Ollama"
description: "Installer dsh en version figée et lui déclarer le modèle local servi par Ollama"
---

DeepSeek Harness (`dsh`) est un harness open source, livré par défaut pour les modèles de DeepSeek dans le cloud. Vous allez l'installer, puis lui dire d'utiliser à la place le modèle servi par Ollama. C'est l'occasion de voir concrètement qu'un harness et un modèle sont deux choses séparées : le harness a juste besoin d'une adresse et d'un nom de modèle.

## 06.1 - Installer une version figée

dsh est en **préversion** : son README annonce des changements incompatibles d'une version à l'autre. Pour que tout le monde ait le même outil, on installe une version précise plutôt que la dernière. L'option `-g` de `npm` installe la commande pour tout le poste, et `@0.2.2-rc.3` fixe la version :

```exec
npm install -g @deepseek-ai/dsh@0.2.2-rc.3
```

Selon la version de npm, l'installation peut se terminer par un avertissement `npm warn install-scripts` qui liste des scripts d'installation non exécutés. Pas d'inquiétude : dsh fonctionne sans eux pour ce lab.

> ⚠️ **Piège : `EACCES: permission denied`.**
> - *Symptôme :* npm refuse d'écrire dans `/usr/local/lib/node_modules` ou `/usr/lib/node_modules`.
> - *Cause :* votre Node.js est installé dans un dossier du système, où votre compte n'a pas le droit d'écrire.
> - *Correctif :* n'utilisez pas `sudo npm`. Installez Node.js avec nvm (étape 03), qui le place dans votre dossier personnel, puis relancez la commande.

Vérifiez la version installée :

```exec
dsh --version
```

La commande affiche la version installée, qui doit être exactement celle demandée :

```text
0.2.2-rc.3
```

> 📖 [DeepSeek Harness sur GitHub](https://github.com/deepseek-ai/deepseek-harness) · [Le paquet @deepseek-ai/dsh sur npm](https://www.npmjs.com/package/@deepseek-ai/dsh)

```verify
if ! command -v dsh >/dev/null 2>&1; then
  echo "La commande dsh est introuvable. Lancez npm install -g @deepseek-ai/dsh@0.2.2-rc.3, puis ouvrez un nouveau terminal."
  exit 1
fi
V=$(dsh --version 2>/dev/null | tr -d '\r')
if ! echo "$V" | grep -q '0\.2\.2-rc\.3'; then
  echo "dsh est en version « $V », 0.2.2-rc.3 attendue. Relancez la commande d'installation de cette section."
  exit 1
fi
```

## 06.2 - Créer le profil de l'interface web

dsh range sa configuration par **profil** : `web` pour l'interface dans le navigateur, `tui` pour le terminal… Chaque profil est un dossier sous `~/.dsh/profiles`, créé au premier usage. La commande suivante demande à dsh d'afficher la configuration du profil `web` ; au passage, elle crée son dossier. L'affichage, très long, ne nous intéresse pas : on le jette.

```exec
dsh --profile web --dump-config > /dev/null
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `dsh --profile web` | travaille sur le profil `web` |
> | `--dump-config` | affiche la configuration complète du profil, puis s'arrête |
> | `> /dev/null` | redirige l'affichage vers `/dev/null`, un fichier spécial qui jette tout ce qu'on y écrit |

Le dossier du profil contient maintenant un fichier `cordis.patch.yml`. C'est **votre** couche de réglages : dsh l'applique par-dessus sa configuration livrée.

```verify
if [ ! -f ~/.dsh/profiles/web/cordis.patch.yml ]; then
  echo "Le fichier ~/.dsh/profiles/web/cordis.patch.yml n'existe pas. Lancez la commande dsh --profile web --dump-config de cette section."
  exit 1
fi
```

## 06.3 - Déclarer le modèle local

Vous allez remplacer le contenu de `cordis.patch.yml` par deux réglages. Le premier déclare Ollama comme **fournisseur** de modèles, le second en fait le modèle par défaut de l'agent. Voici le fichier complet, puis le rôle de chaque champ.

```yaml
- id: llm-pi-ai
  config:
    providers:
      ollama:
        displayName: Ollama
        api: openai-completions
        baseURL: http://127.0.0.1:11434/v1
        apiKeyEnv: OLLAMA_API_KEY
        models:
          - id: qwen3.5:4b
            contextWindow: 16384
- id: agent-default-model
  config:
    provider: ollama
    model: qwen3.5:4b
```

| Champ | Rôle |
|---|---|
| `id: llm-pi-ai` | le module de dsh qui parle aux fournisseurs de modèles autres que DeepSeek |
| `api: openai-completions` | le format des requêtes : Ollama comprend le même format que l'API d'OpenAI, que presque tous les harness savent parler |
| `baseURL` | l'adresse de cette API compatible, chez Ollama : le port `11434`, chemin `/v1` |
| `apiKeyEnv` | le nom de la variable d'environnement qui contient la clé d'API. Ollama ne vérifie aucune clé, mais dsh en exige une : vous lui donnerez une valeur quelconque au lancement |
| `models` | les modèles proposés : `id` est le nom dans Ollama, `contextWindow` la fenêtre réglée à l'étape précédente |
| `id: agent-default-model` | le modèle que l'agent utilise par défaut : le fournisseur `ollama` et son modèle `qwen3.5:4b` |

La commande ci-dessous écrit ce contenu dans le fichier, en remplaçant l'ancien. Remplacez bien tout le fichier : sur un poste neuf, il contient `[]` (une liste vide), et des lignes ajoutées à la suite le rendraient illisible pour dsh.

```exec
cat > ~/.dsh/profiles/web/cordis.patch.yml <<'EOF'
- id: llm-pi-ai
  config:
    providers:
      ollama:
        displayName: Ollama
        api: openai-completions
        baseURL: http://127.0.0.1:11434/v1
        apiKeyEnv: OLLAMA_API_KEY
        models:
          - id: qwen3.5:4b
            contextWindow: 16384
- id: agent-default-model
  config:
    provider: ollama
    model: qwen3.5:4b
EOF
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `cat` | affiche ce qu'il reçoit en entrée |
> | `> ~/.dsh/profiles/web/cordis.patch.yml` | envoie cet affichage dans le fichier, en **remplaçant** son contenu |
> | `<<'EOF'` … `EOF` | un « here document » : les lignes entre les deux `EOF` sont l'entrée de `cat`. Les guillemets autour du premier `EOF` empêchent le shell d'interpréter le texte |

> 💡 **Remarque : pourquoi pas `ollama launch dsh` ?** La documentation d'Ollama propose `ollama launch dsh`, qui installe et configure dsh en une commande. Avec Ollama 0.40.16 et dsh 0.2.2, le réglage qu'il écrit n'est plus lu : l'interface s'ouvre sur une demande de compte DeepSeek. Écrire la configuration soi-même évite le problème, et montre ce qu'un lanceur fait à votre place.

Vérifiez que dsh lit bien votre réglage : la configuration composée du profil doit maintenant désigner Ollama.

```exec
dsh --profile web --dump-config | grep -A4 'id: agent-default-model'
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `dsh --profile web --dump-config` | affiche la configuration complète du profil `web`, votre patch compris |
> | `\|` | envoie cet affichage à la commande suivante au lieu de l'écran |
> | `grep -A4 'id: agent-default-model'` | garde la ligne qui contient `id: agent-default-model` et les 4 lignes qui la suivent (`-A4`, *after*) |

Le bloc affiché doit désigner le fournisseur `ollama` et le modèle `qwen3.5:4b` :

```text
- id: agent-default-model
  name: '@deepseek-ai/dsh-agent-default-model'
  config:
    provider: ollama
    model: qwen3.5:4b
```

> 📖 [Le format de patch des profils dsh](https://github.com/deepseek-ai/deepseek-harness#readme) · [L'API compatible OpenAI d'Ollama](https://docs.ollama.com/api/openai-compatibility)

```verify
if ! OUT=$(dsh --profile web --dump-config 2>&1) || echo "$OUT" | grep -qiE "failed to parse|YAMLException"; then
  echo "dsh ne peut pas lire ~/.dsh/profiles/web/cordis.patch.yml. Réécrivez tout le fichier avec la commande cat de cette section."
  exit 1
fi
if ! echo "$OUT" | grep -A4 'id: agent-default-model' | grep -q 'provider: ollama'; then
  echo "Le modèle par défaut de l'agent n'est pas sur Ollama. Vérifiez le bloc agent-default-model du fichier."
  exit 1
fi
if ! echo "$OUT" | grep -q 'baseURL: http://127.0.0.1:11434/v1'; then
  echo "Le fournisseur ollama n'a pas l'adresse http://127.0.0.1:11434/v1."
  exit 1
fi
```
