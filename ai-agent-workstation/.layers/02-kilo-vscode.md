---
title: "02 - Installer Kilo dans VS Code"
description: "VS Code, l'extension Kilo, un compte gratuit et le modèle kilo-auto/free"
---

Kilo est une extension qui ajoute un agent à VS Code. Vous lui parlez dans un panneau de l'éditeur, et il lit et modifie les fichiers du dossier ouvert. Cette étape est un **support** : elle prépare l'outil, chaque geste est donné.

## 02.1 - Installer VS Code et sa commande `code`

Si VS Code n'est pas installé, téléchargez-le sur [code.visualstudio.com](https://code.visualstudio.com/) et installez-le. Les vérifications de ce lab utilisent la commande `code` dans le terminal. Testez-la :

```exec
code --version
```

La commande affiche un numéro de version, par exemple `1.134.0`. Si le terminal répond `command not found` :

- **sous macOS**, ouvrez VS Code, puis la palette de commandes (`Cmd+Maj+P`), tapez **Shell Command: Install 'code' command in PATH** et validez ; ouvrez ensuite un nouveau terminal ;
- **sous Windows**, l'installateur ajoute `code` au **PATH** (la liste des dossiers où le terminal cherche les commandes) si l'option est cochée ; relancez Git Bash après l'installation ;
- **sous Linux**, les paquets officiels `.deb`, `.rpm` et snap installent la commande `code`.

> 📖 [Lancer VS Code depuis la ligne de commande](https://code.visualstudio.com/docs/configure/command-line)

```verify
if ! command -v code >/dev/null 2>&1; then
  echo "La commande code est introuvable. Suivez les indications de cette section pour votre système, puis ouvrez un nouveau terminal."
  exit 1
fi
```

## 02.2 - Installer l'extension Kilo

L'extension s'installe depuis la vue **Extensions** de VS Code : cherchez **Kilo Code** et cliquez sur **Install**. Vous pouvez aussi l'installer depuis le terminal, avec son identifiant sur la place de marché, `kilocode.kilo-code` :

```exec
code --install-extension kilocode.kilo-code
```

Une icône Kilo apparaît dans la barre latérale de VS Code.

> 📖 [Kilo Code sur la place de marché de VS Code](https://marketplace.visualstudio.com/items?itemName=kilocode.kilo-code)

```verify
if ! code --list-extensions 2>/dev/null | grep -qi '^kilocode\.kilo-code$'; then
  echo "L'extension Kilo Code n'est pas installée dans VS Code. Installez-la depuis la vue Extensions ou avec la commande de cette section."
  exit 1
fi
```

## 02.3 - Créer un compte et choisir un modèle gratuit

Ouvrez le panneau Kilo depuis son icône. À la première ouverture, il vous demande de vous connecter : cliquez sur **Sign In**, puis créez un compte gratuit dans le navigateur qui s'ouvre (e-mail, Google, GitHub, GitLab… au choix). Aucune carte bancaire n'est demandée pour les modèles gratuits.

De retour dans VS Code, réglez deux choses :

1. Dans le sélecteur de modèle, sous la zone de saisie du panneau, choisissez **`kilo-auto/free`**. Ce mode envoie chaque demande vers l'un des modèles gratuits du moment, que Kilo choisit pour vous.
2. Kilo utilise aussi, en arrière-plan, un petit modèle pour des tâches comme titrer les conversations. Par défaut, c'est **Auto Small**, qui consomme des crédits payants. Ouvrez les réglages de Kilo (roue dentée), onglet **Models**, et choisissez un modèle gratuit pour ce petit modèle : tapez `free` dans la liste pour les filtrer.

Pour tester, ouvrez le dossier `~/byte-layers/ai/setup` dans VS Code (**File**, **Open Folder**). Si VS Code demande si vous faites confiance aux auteurs des fichiers, répondez **Yes, I trust the authors** : c'est votre propre dossier. Demandez ensuite à Kilo : « Dis bonjour en une phrase. » Il répond en quelques secondes.

> ⚠️ **Des données qui sortent.** La documentation de Kilo prévient que `kilo-auto/free` peut passer par des fournisseurs qui enregistrent les demandes et les réponses pour améliorer leurs services. N'y envoyez aucune donnée personnelle ni confidentielle.

> 📖 [Utiliser Kilo gratuitement](https://kilo.ai/docs/getting-started/using-kilo-for-free) · [Se connecter à Kilo](https://kilo.ai/docs/getting-started/setting-up)

```acknowledge
Kilo répond dans VS Code avec kilo-auto/free
```
