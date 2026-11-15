---
title: "04 - Installer Ollama, le serveur de modèles locaux"
description: "Installer Ollama et vérifier que son API répond"
---

**Ollama** fait tourner des modèles de langage sur votre poste. Il les télécharge, les charge en mémoire et les sert par une **API** HTTP sur le port `11434` (un numéro qui désigne un service sur la machine). Un harness comme DeepSeek Harness envoie ses demandes à cette API, exactement comme il les enverrait à un fournisseur dans le cloud. Cette étape est un **support**.

## 04.1 - Installer Ollama

Ollama s'installe différemment selon le système. Choisissez la ligne qui vous concerne.

| Système | Ce qu'il faut | Installation |
|---|---|---|
| macOS | macOS 14 Sonoma ou plus récent | téléchargez `Ollama.dmg` sur [ollama.com/download](https://ollama.com/download), glissez **Ollama** dans **Applications**, lancez-le, et acceptez qu'il installe la commande `ollama` |
| Windows | Windows 10 22H2 ou plus récent | téléchargez et lancez `OllamaSetup.exe` depuis [ollama.com/download](https://ollama.com/download) ; pas besoin d'être administrateur |
| Linux | `curl` | le script officiel, ci-dessous |

Sous Linux, le script officiel installe Ollama et le déclare comme service système, qui démarre tout seul. Il demande votre mot de passe, car il écrit dans des dossiers du système :

```exec
curl -fsSL https://ollama.com/install.sh | sh
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `curl -fsSL https://ollama.com/install.sh` | télécharge le script d'installation : `-f` échoue sur une erreur HTTP, `-s` coupe la barre de progression, `-S` garde les messages d'erreur, `-L` suit les redirections |
> | `\|` | le pipe : envoie ce que `curl` télécharge en entrée de la commande suivante |
> | `sh` | exécute le script reçu |
>
> Exécuter un script téléchargé revient à lui faire confiance : faites-le seulement depuis le site officiel de l'outil.

> 💡 **Remarque : pourquoi une installation directe, et pas un conteneur ?** Ollama existe aussi en image Docker (`ollama/ollama`). Mais sous macOS et Windows, Docker fait tourner ses conteneurs dans une machine virtuelle Linux. Le modèle y perd l'accès à la carte graphique et reste limité à la mémoire donnée à cette machine virtuelle. Installé directement, Ollama utilise la puce graphique des Mac Apple Silicon (M1 et suivants) et les cartes NVIDIA. Les Mac Intel et les postes sans carte graphique calculent sur le processeur dans les deux cas : attendez-vous aux temps les plus longs de ce lab.

> 📖 [Ollama sous macOS](https://docs.ollama.com/macos) · [sous Windows](https://docs.ollama.com/windows) · [sous Linux](https://docs.ollama.com/linux) · [Les cartes graphiques prises en charge](https://docs.ollama.com/gpu)

## 04.2 - Vérifier que le serveur répond

Affichez d'abord la version de la commande `ollama` :

```exec
ollama -v
```

Elle affiche le numéro de version, par exemple :

```text
ollama version is 0.40.16
```

Interrogez ensuite l'API, sur le port `11434` de votre propre poste (`localhost`) :

```exec
curl http://localhost:11434
```

Le serveur répond par une seule phrase :

```text
Ollama is running
```

Si `curl` répond `Failed to connect` juste après l'installation, le serveur est peut-être encore en train de démarrer : attendez quelques secondes et relancez la commande. Si l'erreur persiste, le serveur n'est pas lancé : sous macOS et Windows, lancez l'application Ollama ; sous Linux, lancez `sudo systemctl start ollama`.

```verify
if ! command -v ollama >/dev/null 2>&1; then
  echo "La commande ollama est introuvable. Installez Ollama, puis ouvrez un nouveau terminal."
  exit 1
fi
if ! curl -s http://localhost:11434 | grep -q "Ollama is running"; then
  echo "L'API d'Ollama ne répond pas sur http://localhost:11434. Lancez l'application Ollama (macOS, Windows) ou le service (sudo systemctl start ollama sous Linux)."
  exit 1
fi
```
