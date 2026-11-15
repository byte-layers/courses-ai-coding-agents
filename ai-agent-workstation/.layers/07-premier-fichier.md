---
title: "07 - Faire créer un premier fichier par l'agent local"
description: "Lancer l'interface web de dsh, demander un fichier, constater le travail de l'agent, puis nettoyer"
---

Tout est en place : Ollama sert le modèle, et dsh sait où le trouver. Il reste à confier une première tâche à l'agent et à constater qu'il agit vraiment sur vos fichiers, sans rien envoyer hors du poste.

## 07.1 - Lancer l'interface web

dsh prend le dossier où on le lance comme **espace de travail** (workspace), c'est-à-dire le dossier où l'agent a le droit de lire et d'écrire. On se place donc dans le dossier de travail, puis on lance l'interface web :

```exec
cd ~/byte-layers/ai/setup && OLLAMA_API_KEY=ollama dsh web
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `cd ~/byte-layers/ai/setup` | se place dans le dossier de travail |
> | `&&` | lance la suite seulement si le `cd` a réussi |
> | `OLLAMA_API_KEY=ollama` | une variable placée devant une commande n'existe que pour ce lancement : c'est la clé quelconque que dsh réclame (étape précédente) et qu'Ollama ignore |
> | `dsh web` | démarre l'interface web de dsh |

dsh affiche l'adresse de l'interface, avec un jeton d'accès à usage local, et l'ouvre dans votre navigateur :

```text
dsh web: http://127.0.0.1:3081/?token=…
```

Si le navigateur ne s'ouvre pas tout seul, copiez cette adresse complète, jeton compris, dans votre navigateur. Le terminal reste occupé tant que l'interface tourne : laissez-le ouvert.

Dans le navigateur :

1. une fenêtre **Preview Notice** rappelle que dsh est en préversion : cliquez sur **Continue** ;
2. sous la zone de saisie, le sélecteur de modèle doit afficher **`qwen3.5:4b`** ;
3. au-dessus de la zone de saisie, l'espace de travail doit être **`setup`**. S'il affiche **Choose workspace**, cliquez dessus, ouvrez `byte-layers`, puis `ai`, sélectionnez `setup` et cliquez sur **Open**.

> ⚠️ **Piège : une fenêtre « Get started » demande un compte DeepSeek.**
> - *Symptôme :* à l'ouverture, une fenêtre propose **Sign in** ou **Add API Key**, et le sélecteur de modèle est vide.
> - *Cause :* dsh n'a pas lu votre réglage, et il revient à son fournisseur par défaut, DeepSeek.
> - *Correctif :* fermez l'interface (`Ctrl+C` dans le terminal), relancez la vérification de l'étape précédente, corrigez le fichier, puis relancez `OLLAMA_API_KEY=ollama dsh web`. Ne créez pas de compte DeepSeek : ce lab n'en a pas besoin.

```acknowledge
L'interface de dsh affiche qwen3.5:4b et l'espace de travail setup
```

## 07.2 - Demander un fichier à l'agent

Dans la zone de saisie, tapez la demande suivante, puis validez avec Entrée :

```text
Crée le fichier hello.txt dans ce dossier, avec le seul mot : bonjour
```

Sans carte graphique, comptez une à deux minutes : le modèle doit d'abord lire les quelque 9 800 tokens de consignes de l'agent. L'interface affiche **Deep diving** pendant ce temps. Ensuite, la conversation montre deux choses :

- un bloc **Think**, où le modèle explique ce qu'il va faire ;
- un bloc **Write** avec le nom `hello.txt` : c'est l'appel d'outil que vous aviez vu à l'étape 05, cette fois **exécuté** par le harness.

Si dsh vous demande l'autorisation d'écrire le fichier, acceptez-la. Si l'interface affiche **Retried model request**, dsh relance une requête qui a échoué : attendez. Si l'erreur revient avec `500` ou `unexpected EOF`, le modèle a manqué de mémoire : voyez le piège de l'étape 05.

Vérifiez dans le terminal (un **nouveau** terminal, puisque le premier fait tourner dsh) que le fichier existe bien :

```exec
cat ~/byte-layers/ai/setup/hello.txt
```

Le terminal affiche le contenu écrit par l'agent :

```text
bonjour
```

> 📖 [DeepSeek Harness sur GitHub](https://github.com/deepseek-ai/deepseek-harness)

```verify
F=~/byte-layers/ai/setup/hello.txt
if [ ! -f "$F" ]; then
  echo "Le fichier $F n'existe pas. Vérifiez que l'espace de travail de dsh est bien setup, puis redemandez le fichier à l'agent."
  exit 1
fi
if ! grep -qi 'bonjour' "$F"; then
  echo "Le fichier $F existe mais ne contient pas « bonjour ». Contenu actuel : $(head -c 100 "$F")"
  exit 1
fi
```

## 07.3 - Arrêter dsh et nettoyer

Revenez au terminal qui fait tourner dsh et arrêtez-le avec `Ctrl+C` (si un prochain lancement signale le port `3081` déjà occupé, fermez simplement cette fenêtre de terminal). Supprimez ensuite le fichier d'essai ; le dossier `~/byte-layers/ai/setup` reste, pour les labs suivants :

```exec
rm ~/byte-layers/ai/setup/hello.txt
```

Ollama, lui, continue de tourner en arrière-plan : c'est voulu, les labs suivants s'en servent. Il décharge le modèle de la mémoire après quelques minutes sans demande.

```verify
if [ -f ~/byte-layers/ai/setup/hello.txt ]; then
  echo "Le fichier d'essai hello.txt est toujours là. Supprimez-le avec la commande rm de cette section."
  exit 1
fi
```

## Récap

- Un **agent de code** est un harness qui fait tourner une boucle : il envoie votre demande à un modèle, puis exécute les **appels d'outils** que le modèle renvoie.
- Le **harness** et le **modèle** sont séparés : il a suffi à dsh d'une adresse (`baseURL`) et d'un nom de modèle pour passer du cloud de DeepSeek à Ollama sur votre poste.
- **Kilo** (extension, modèles gratuits dans le cloud) est rapide, mais votre code part chez des fournisseurs. **dsh + Ollama** garde tout sur le poste, au prix de la vitesse et de la mémoire.
- Pour un agent, la **fenêtre de contexte** compte autant que le modèle : ses seules consignes occupent près de 10 000 tokens.
