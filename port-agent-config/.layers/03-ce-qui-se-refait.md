---
title: "03 - Ce qui se refait"
description: "Traduire les permissions d'OpenCode dans le modèle de dsh, et fixer le réglage de l'équipe dans un script de lancement"
---

Les permissions n'ont pas de format commun. OpenCode les écrit dans `opencode.json`, sous forme de règles par outil et par motif. dsh ne lit pas ce fichier, et son modèle de permissions est construit autrement.

## 03.1 - Le modèle de dsh

dsh règle deux choses, pour toute la session : un **bac à sable** (*sandbox*), qui limite les fichiers que les commandes peuvent toucher, et une **politique d'accord**, `ask` par défaut, qui le fait vous demander votre accord pour les opérations qu'il juge sensibles. Affichez le réglage du bac à sable dans la configuration de votre profil :

```exec
dsh --profile web --dump-config | grep -A4 'id: sandbox-policy'
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `dsh --profile web --dump-config` | affiche la configuration complète du profil `web` |
> | `\|` | envoie cet affichage à la commande suivante |
> | `grep -A4 'id: sandbox-policy'` | garde la ligne du réglage du bac à sable et les 4 lignes qui la suivent |

Le bloc affiché donne la règle qui choisit le mode, pas le mode lui-même :

```text
- id: sandbox-policy
  name: '@deepseek-ai/dsh-sandbox-policy'
  config:
    mode: !!js process.env.DSH_PERMISSION_MODE ?? 'workspace-write'
    workspaceRoot: !!js process.cwd()
```

Les deux lignes de `config` se lisent ainsi :

| Ligne | Ce qu'elle dit |
|---|---|
| `mode: … DSH_PERMISSION_MODE ?? 'workspace-write'` | le mode est la valeur de la variable d'environnement `DSH_PERMISSION_MODE` si elle existe, sinon `workspace-write` |
| `workspaceRoot: … process.cwd()` | le « projet » du bac à sable est le dossier **depuis lequel vous lancez dsh**. Lancé depuis votre dossier personnel, `workspace-write` autorise l'écriture dans tout ce dossier |

Les trois modes possibles :

| Mode | Ce que les commandes peuvent faire |
|---|---|
| `read-only` | lire, sans rien écrire |
| `workspace-write` | écrire dans le dossier de lancement, pas ailleurs |
| `danger-full-access` | tout, sans limite |

Le bac à sable s'appuie sur le système : `sandbox-exec` sur macOS, `bubblewrap` ou Landlock sur Linux, un compte restreint sous Windows. Sans lui, dsh refuse de lancer une commande en `workspace-write` plutôt que de la lancer sans protection.

## 03.2 - Traduire les règles d'OpenCode

Affichez vos règles d'OpenCode, pour les traduire une à une :

```exec
cd ~/byte-layers/ai/todo-api && cat opencode.json
```

Le tableau cherche l'équivalent de chacune dans dsh 0.2.2-rc.3 :

| Règle d'OpenCode | Équivalent dans dsh |
|---|---|
| `"read": { "*.env": "deny" }` | **aucun** : le bac à sable limite l'écriture, pas la lecture d'un fichier du projet. L'outil de lecture de dsh lit `.env` sans rien demander |
| `"bash": { "*": "ask" }` | la politique d'accord `ask` : dsh décide lui-même des opérations qui demandent votre accord, sans liste réglable par commande |
| `"npm test": "allow"` et `"npm test *": "allow"` | pas de règle par commande : ce sont le bac à sable et la politique d'accord qui décident, pour toutes les commandes à la fois |
| `"rm *": "deny"`, votre règle | pas de règle qui interdise une commande précise ; seul le mode `read-only` empêche toute écriture, `rm` compris, mais aussi toute modification du code |

Deux différences changent la façon de travailler. Les permissions de dsh se règlent pour la session, pas commande par commande. Et elles ne vivent pas dans un fichier du projet comme `opencode.json`, mais dans votre profil, sous `~/.dsh`, ou dans une variable d'environnement : nous n'avons pas trouvé, dans sa documentation, de réglage de permissions que dsh lirait dans le dépôt.

## 03.3 - Fixer le réglage de l'équipe

Ce que dsh ne lit pas dans le dépôt, l'équipe peut quand même le partager : un **script de lancement**, commité, qui lance dsh toujours de la même façon. Il règle les deux points du tableau de la section 03.1 : le dossier de lancement et le mode. On l'écrit avec un manque, que vous compléterez :

```exec
cd ~/byte-layers/ai/todo-api && mkdir -p scripts && cat > scripts/dsh-web.sh <<'FIN'
#!/bin/sh
# Lance dsh web pour ce projet, toujours depuis sa racine.
cd "$(dirname "$0")/.." || exit 1
# À vous : exportez DSH_PERMISSION_MODE avec le mode choisi par l'équipe
export OLLAMA_API_KEY="${OLLAMA_API_KEY:-ollama}"
exec dsh web "$@"
FIN
chmod +x scripts/dsh-web.sh
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `mkdir -p scripts` | crée le dossier `scripts` s'il n'existe pas encore |
> | `cat > scripts/dsh-web.sh <<'FIN'` … `FIN` | écrit les lignes entre les deux `FIN` dans le script |
> | `chmod +x scripts/dsh-web.sh` | rend le script exécutable : on pourra le lancer par son chemin |

Le script se lit ainsi :

| Ligne | Rôle |
|---|---|
| `#!/bin/sh` | indique le programme qui exécute le script : le shell standard |
| `cd "$(dirname "$0")/.." \|\| exit 1` | `$0` est le chemin du script, `dirname` en garde le dossier, `scripts`, et `/..` remonte à la racine du projet. Le bac à sable sera donc toujours le projet, d'où que vous lanciez le script. `\|\| exit 1` arrête tout si le `cd` échoue |
| `export OLLAMA_API_KEY="${OLLAMA_API_KEY:-ollama}"` | garde votre clé si la variable existe déjà, sinon met la valeur quelconque qu'attend dsh |
| `exec dsh web "$@"` | remplace le script par dsh web, en lui passant les arguments reçus (`"$@"`) |

**À vous** : ouvrez `scripts/dsh-web.sh` dans votre éditeur, et remplacez le commentaire par une ligne qui exporte `DSH_PERMISSION_MODE` avec le mode qui laisse l'agent modifier le code du projet, et rien en dehors.

```hint
La ligne suivante du script montre la forme : `export NOM=valeur`. Le mode est l'un des trois du tableau de la section 03.1.
```

````solution
```sh
cd "$(dirname "$0")/.." || exit 1
export DSH_PERMISSION_MODE=workspace-write
export OLLAMA_API_KEY="${OLLAMA_API_KEY:-ollama}"
```
````

Lancez le script depuis votre dossier personnel, avec l'option `--help`, qui affiche l'aide de dsh web sans démarrer l'interface : cela prouve que le script trouve le projet et dsh d'où que vous soyez.

```exec
cd ~ && ~/byte-layers/ai/todo-api/scripts/dsh-web.sh --help
```

L'aide de dsh web s'affiche, et commence par ces lignes :

```text
Usage: dsh --profile web [options]

Serve the DeepSeek Harness browser UI.
```

Pour travailler avec dsh sur ce projet, lancez désormais `scripts/dsh-web.sh` plutôt que `dsh web`. La brique se **refait** donc, avec une perte : la règle qui protégeait `.env` n'a pas d'équivalent.

```verify
S=~/byte-layers/ai/todo-api/scripts/dsh-web.sh
[ -f "$S" ] || { echo "Le script scripts/dsh-web.sh n'existe pas. Lancez la commande de la section 03.3."; exit 1; }
M=$(sed -n 's/^[[:space:]]*export[[:space:]]\{1,\}DSH_PERMISSION_MODE=//p' "$S" | tail -1 | tr -d '"'"'"' \r')
[ -n "$M" ] || { echo "Le script n'exporte pas DSH_PERMISSION_MODE. Remplacez le commentaire par une ligne export DSH_PERMISSION_MODE=…"; exit 1; }
case "$M" in
  workspace-write) ;;
  read-only) echo "En read-only, l'agent ne pourrait plus modifier le code du projet. Choisissez le mode qui permet d'écrire dans le projet seulement."; exit 1;;
  danger-full-access) echo "danger-full-access ne limite rien : l'agent pourrait écrire hors du projet. Choisissez un mode plus restreint."; exit 1;;
  *) echo "Le mode $M n'existe pas. Les trois modes sont read-only, workspace-write et danger-full-access."; exit 1;;
esac
grep -q 'cd "$(dirname "$0")/.."' "$S" || { echo "Le script ne se place plus à la racine du projet : gardez la ligne cd \"\$(dirname \"\$0\")/..\"."; exit 1; }
OUT=$(cd ~ && "$S" --help 2>&1 | head -3)
echo "$OUT" | grep -q 'dsh --profile web' || { echo "Le script ne lance pas dsh web (sortie : $OUT). Vérifiez qu'il est exécutable (chmod +x scripts/dsh-web.sh) et que sa dernière ligne est exec dsh web \"\$@\"."; exit 1; }
```
