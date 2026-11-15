---
title: "Préparer son poste pour coder avec un agent d'IA"
description: "Lab d'installation : Kilo dans VS Code avec des modèles gratuits dans le cloud, puis DeepSeek Harness avec un modèle local servi par Ollama, et un premier fichier créé par l'agent."
tags:
  - lab
  - ai
  - setup
difficulty: beginner
duration: 75
introTitle: "Le lab"
requires: "0.4.0"
options:
  allowManualProgress: false
  autoCompleteOnNext: unverifiable
---

Dans ce lab, vous installez deux façons de coder avec un agent d'IA : **Kilo**, une extension de VS Code qui utilise des modèles gratuits hébergés dans le cloud, et **DeepSeek Harness** (`dsh`), un agent qui tourne sur votre poste avec un modèle local servi par **Ollama**. À la fin, l'agent local crée un premier fichier dans votre dossier de travail.

**Niveau** : à l'aise avec un terminal (se déplacer dans les dossiers, lancer une commande) et avec l'installation d'un logiciel. Aucune connaissance des agents d'IA n'est supposée.

**Ce qu'il faut** :
- un poste avec **16 Go de mémoire vive**. Le modèle local et l'agent occupent ensemble plus de 5 Go : avec 8 Go, l'agent est très lent, voire s'arrête faute de mémoire, et les étapes 05 à 07 risquent de ne pas aboutir ;
- environ **8 Go d'espace disque** libre, dont 3,3 Go pour le modèle ;
- le droit d'installer des logiciels sur le poste ;
- une connexion internet, et un compte Kilo gratuit, créé pendant le lab.

**Vos fichiers** vivent dans `~/byte-layers/ai/setup`. Le dossier est gardé à la fin : les labs suivants du stack s'en servent.

**Sous Windows**, tapez les commandes dans le terminal **Git Bash**, livré avec [Git pour Windows](https://git-scm.com/download/win) : installez Git pour Windows d'abord s'il n'est pas sur le poste. Les commandes de ce lab sont écrites pour un shell Unix ; PowerShell et l'invite de commandes ne les comprennent pas.

**Après chaque installation**, ouvrez un nouveau terminal : un terminal déjà ouvert ne voit pas les commandes ajoutées depuis. Si une vérification ne trouve pas une commande que votre nouveau terminal trouve, fermez et rouvrez aussi VS Code.

> 💡 **Remarque.** Comptez une vingtaine de minutes de téléchargements (VS Code, Node.js, Ollama, le modèle). Si vous suivez ce lab avant une formation, faites-le quelques jours avant, sur le poste que vous utiliserez.
