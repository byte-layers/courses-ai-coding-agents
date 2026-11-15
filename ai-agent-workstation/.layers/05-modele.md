---
title: "05 - Télécharger un modèle local et régler son contexte"
description: "Le modèle qwen3.5:4b, la mémoire qu'il demande, et une fenêtre de contexte assez grande pour un agent"
---

Ollama est installé, mais il ne contient encore aucun modèle. Vous allez en télécharger un petit, puis régler la quantité de texte qu'il peut lire d'un coup. Pour un agent, ce second réglage compte autant que le premier.

## 05.1 - Choisir un modèle selon sa mémoire

Un modèle local se charge entièrement en mémoire : sa taille sur le disque donne un ordre de grandeur de la mémoire qu'il occupe. Ce lab utilise **`qwen3.5:4b`** : un modèle ouvert de la famille Qwen, à 4 milliards de paramètres, compressé (quantifié) pour peser 3,3 Go. Il sait appeler des **outils**, ce qui est indispensable pour un agent : sans cela, le modèle peut répondre à une question mais pas écrire un fichier.

| Mémoire vive du poste | Ce que vous pouvez attendre |
|---|---|
| 8 Go | le modèle tourne, mais avec le système, le navigateur et l'éditeur, l'agent est très lent et peut s'arrêter faute de mémoire. Gardez Kilo pour le travail réel |
| 16 Go | `qwen3.5:4b` avec un contexte de 16k (16 384 tokens) : l'agent fonctionne, en une à deux minutes par demande sans carte graphique |
| 32 Go et plus, ou une carte graphique | des modèles plus gros deviennent possibles : la documentation de Kilo cite `qwen3-coder:30b` à partir de 32 Go |

Pour estimer si un **autre** modèle tiendra sur votre poste, deux outils en ligne :

- **Hugging Face**, le site de référence des modèles ouverts : déclarez votre matériel dans les [réglages Hardware](https://huggingface.co/settings/hardware) de votre compte (processeur, carte graphique ou puce Apple, et sa mémoire). Les pages de modèles qui proposent des fichiers GGUF ou MLX affichent alors un panneau **Hardware compatibility** pour chaque niveau de compression. Ce matériel est public par défaut : décochez **Publicly Visible** si vous préférez le garder privé.
- [**Can I Use LLM?**](https://www.caniusellm.com/), sans compte, détecte votre matériel depuis le navigateur. À utiliser avec prudence : sur un Mac de 24 Go, il a vu 16 Go de mémoire et environ 4 Go de mémoire graphique, et il affichait le même minimum pour un petit modèle et pour un modèle de plusieurs centaines de milliards de paramètres. Prenez-le comme une première indication, pas comme une réponse.

Dans les deux cas, gardez de la marge : l'estimation porte sur le modèle seul, sans la fenêtre de contexte de l'agent (section suivante) ni les autres applications ouvertes.

> 📖 [Déclarer son matériel sur Hugging Face](https://huggingface.co/docs/hub/en/hardware)

Téléchargez le modèle. Ollama affiche une barre de progression, et termine par `success` :

```exec
ollama pull qwen3.5:4b
```

Listez ensuite les modèles présents sur le poste, pour vérifier que le téléchargement est complet :

```exec
ollama list
```

La liste contient votre modèle, avec sa taille sur le disque :

```text
NAME          ID              SIZE      MODIFIED
qwen3.5:4b    d8b0f5e9760c    3.3 GB    Less than a second ago
```

> 📖 [Le modèle qwen3.5 sur ollama.com](https://ollama.com/library/qwen3.5) · [Ollama et Kilo Code](https://kilo.ai/docs/ai-providers/ollama)

```verify
if ! ollama list 2>/dev/null | grep -q '^qwen3\.5:4b '; then
  echo "Le modèle qwen3.5:4b n'est pas dans ollama list. Lancez ollama pull qwen3.5:4b et attendez le message success."
  exit 1
fi
```

## 05.2 - Agrandir la fenêtre de contexte

La **fenêtre de contexte** est la quantité de texte, comptée en **tokens** (des morceaux de mots), que le modèle lit à chaque demande. Un agent y met beaucoup plus que votre phrase : ses consignes, la description de chaque outil, les fichiers qu'il a lus. Au démarrage, DeepSeek Harness envoie déjà environ **9 800 tokens** avant même votre demande.

Or Ollama choisit une fenêtre de **4k** (4 096 tokens) par défaut sur un poste qui a moins de 24 Go de mémoire graphique, ce qui est le cas de la plupart des portables. Avec ce réglage, le début des consignes de l'agent serait coupé et il se comporterait au hasard. Réglez la fenêtre sur **16k** (16 384 tokens) :

| Système | Réglage |
|---|---|
| macOS, Windows | ouvrez les réglages de l'application Ollama (**Settings**, depuis sa fenêtre ou son icône dans la barre de menus ou la zone de notification) et placez le curseur **Context length** sur **16k** |
| Linux (service) | les trois commandes ci-dessous |

Sous Linux, Ollama tourne comme un service **systemd**, et lit ses réglages dans des variables d'environnement. On ajoute au service un petit fichier de réglage qui définit `OLLAMA_CONTEXT_LENGTH`, puis on le redémarre. La première commande crée le dossier de ces fichiers :

```exec
sudo mkdir -p /etc/systemd/system/ollama.service.d
```

La deuxième écrit le fichier `context.conf` dans ce dossier :

```exec
printf '[Service]\nEnvironment="OLLAMA_CONTEXT_LENGTH=16384"\n' | sudo tee /etc/systemd/system/ollama.service.d/context.conf
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `printf '[Service]\nEnvironment=…\n'` | écrit les deux lignes du réglage ; `\n` est un retour à la ligne |
> | `\|` | envoie ce texte à la commande suivante |
> | `sudo tee …/context.conf` | écrit ce qu'elle reçoit dans le fichier, avec les droits d'administrateur nécessaires dans `/etc` (et l'affiche) |

La troisième fait relire ses fichiers à systemd, puis redémarre Ollama avec le nouveau réglage ; `&&` ne lance la seconde que si la première a réussi :

```exec
sudo systemctl daemon-reload && sudo systemctl restart ollama
```

Une fenêtre plus grande occupe plus de mémoire : chargé avec 16k, `qwen3.5:4b` occupe 4,2 Go au lieu de 3,9 Go avec 8k. C'est pour cela qu'on ne met pas la valeur maximale.

> ⚠️ **Piège : le modèle s'arrête en pleine réponse.**
> - *Symptôme :* l'agent affiche une erreur `500` ou `unexpected EOF`, et le journal d'Ollama contient `signal: killed`. Ce journal est dans `~/.ollama/logs/server.log` sous macOS, `%LOCALAPPDATA%\Ollama\server.log` sous Windows, et s'affiche avec `journalctl -u ollama` sous Linux.
> - *Cause :* le système a tué le modèle faute de mémoire. Cela arrive quand la fenêtre de contexte est trop grande pour la mémoire libre.
> - *Correctif :* fermez les applications gourmandes, ou redescendez la fenêtre à 12k (12 288 tokens) ; dans ce cas, mettez aussi `contextWindow: 12288` dans le réglage de dsh, à l'étape suivante. En dessous de 12k, les consignes de l'agent ne tiennent plus.

> 📖 [La longueur de contexte dans Ollama](https://docs.ollama.com/context-length) · [Régler une variable d'environnement d'Ollama sous Linux](https://docs.ollama.com/faq)

La vérification décharge le modèle, le recharge, puis lit la taille de fenêtre qu'Ollama lui a donnée. Le chargement peut prendre jusqu'à une minute :

```verify
API=http://localhost:11434/api
curl -s --max-time 30 $API/generate -d '{"model":"qwen3.5:4b","keep_alive":0}' >/dev/null
for i in 1 2; do
  curl -s --max-time 300 $API/generate -d '{"model":"qwen3.5:4b"}' >/dev/null
  C=$(curl -s --max-time 10 $API/ps | grep -oE '"name":"[^"]*"|"context_length":[0-9]+' | grep -A1 '^"name":"qwen3\.5:4b"$' | sed -n 's/^"context_length"://p')
  [ -n "$C" ] && break
done
if [ -z "$C" ]; then
  echo "Le modèle ne se charge pas. Vérifiez qu'Ollama tourne (curl http://localhost:11434) et que qwen3.5:4b est téléchargé. Si les deux sont bons, il manque de la mémoire : fermez des applications et relancez la vérification (voir le piège de cette section)."
  exit 1
fi
if [ "$C" -lt 12288 ]; then
  echo "La fenêtre de contexte est de $C tokens : trop petite pour un agent. Réglez-la sur 16k (Settings de l'application, ou OLLAMA_CONTEXT_LENGTH sous Linux), puis relancez Ollama."
  exit 1
fi
```

## 05.3 - Vérifier que le modèle sait appeler un outil

Un agent ne fonctionne que si le modèle répond à une demande par un **appel d'outil** : au lieu d'écrire du texte, il renvoie le nom d'une fonction et ses arguments, que le harness exécute. Vous pouvez le constater sans agent, en décrivant vous-même un outil `write_file` à l'API d'Ollama.

La commande ci-dessous envoie une demande en JSON (un format texte pour des données structurées, entre accolades) : le modèle visé, le message, et la description d'un outil. L'option `-d` de `curl` envoie ce texte comme corps de la requête :

```exec
curl -s http://localhost:11434/api/chat -d '{"model":"qwen3.5:4b","stream":false,"think":false,"messages":[{"role":"user","content":"Create the file hello.txt containing bonjour"}],"tools":[{"type":"function","function":{"name":"write_file","description":"Write a file","parameters":{"type":"object","properties":{"path":{"type":"string"},"content":{"type":"string"}},"required":["path","content"]}}}]}'
```

Après quelques secondes, la réponse ne contient pas de texte mais un champ `tool_calls` : le modèle demande d'appeler `write_file` avec les bons arguments. Rien n'est écrit sur le disque, car personne n'exécute l'appel : c'est justement le travail du harness.

```text
{"model":"qwen3.5:4b",...,"message":{"role":"assistant","content":"","tool_calls":[{"id":"call_yfel2ed3","function":{"index":0,"name":"write_file","arguments":{"path":"hello.txt","content":"bonjour"}}}]},"done":true,...}
```

> 📖 [L'appel d'outils dans Ollama](https://docs.ollama.com/capabilities/tool-calling)

Un petit modèle ne répond pas toujours de la même façon : la vérification fait donc jusqu'à deux essais.

```verify
for i in 1 2; do
  R=$(curl -s --max-time 180 http://localhost:11434/api/chat -d '{"model":"qwen3.5:4b","stream":false,"think":false,"messages":[{"role":"user","content":"Create the file hello.txt containing bonjour"}],"tools":[{"type":"function","function":{"name":"write_file","description":"Write a file","parameters":{"type":"object","properties":{"path":{"type":"string"},"content":{"type":"string"}},"required":["path","content"]}}}]}')
  echo "$R" | grep -q '"tool_calls"' && exit 0
done
echo "Le modèle n'a pas renvoyé d'appel d'outil en deux essais. Relancez la vérification ; si l'échec revient, vérifiez le nom du modèle. Réponse reçue : $(echo "$R" | head -c 300)"
exit 1
```
