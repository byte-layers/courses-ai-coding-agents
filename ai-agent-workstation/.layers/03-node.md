---
title: "03 - Installer Node.js 24"
description: "La version de Node.js que demande DeepSeek Harness"
---

DeepSeek Harness est un programme écrit en JavaScript : il s'installe avec **npm**, le gestionnaire de paquets livré avec **Node.js**, et il exige **Node.js 24.9 ou plus récent**. Cette étape est un **support**.

## 03.1 - Regarder la version installée

Affichez la version de Node.js présente sur le poste :

```exec
node -v
```

Si la commande affiche `v24.9.0` ou un numéro plus grand (`v24.22.3`, `v26.1.0`…), passez à la vérification. Si elle affiche une version plus ancienne (`v22.18.0`) ou `command not found`, installez Node.js 24 :

| Système | Installation |
|---|---|
| macOS, Windows | l'installateur de la version **24 LTS** (*long-term support*, la version maintenue le plus longtemps) sur [nodejs.org](https://nodejs.org/fr/download) |
| Linux | le gestionnaire de versions **nvm**, ci-dessous |
| Déjà nvm sur le poste | `nvm install 24`, puis `nvm use 24` |

Sous Linux, installez nvm avec son script officiel, puis ouvrez un nouveau terminal pour qu'il soit chargé :

```exec
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.41.0/install.sh | bash
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `curl -o- …/v0.41.0/install.sh` | télécharge le script d'installation de nvm, version 0.41.0 ; `-o-` l'écrit sur la sortie au lieu d'un fichier |
> | `\|` | envoie ce script en entrée de la commande suivante |
> | `bash` | exécute le script reçu |

Le script télécharge nvm dans `~/.nvm` et ajoute son chargement à votre fichier `~/.bashrc`. Dans le nouveau terminal, installez Node.js 24 :

```exec
nvm install 24
```

nvm installe Node.js dans votre dossier personnel : vous n'aurez pas besoin de `sudo` pour installer des paquets npm à l'étape suivante, alors que le paquet Node.js d'une distribution l'exige souvent.

Ouvrez ensuite un **nouveau terminal** et relancez `node -v`.

> 💡 **Remarque : nvm.** Si vous utilisez nvm, `nvm use 24` ne vaut que pour le terminal courant. `nvm alias default 24` fait de la version 24 celle de chaque nouveau terminal.

> 📖 [Télécharger Node.js](https://nodejs.org/fr/download) · [nvm](https://github.com/nvm-sh/nvm)

```verify
if ! command -v node >/dev/null 2>&1; then
  echo "Node.js est introuvable. Installez la version 24, puis ouvrez un nouveau terminal."
  exit 1
fi
V=$(node -v | sed 's/^v//')
MAJ=${V%%.*}; REST=${V#*.}; MIN=${REST%%.*}
if [ "$MAJ" -lt 24 ] || { [ "$MAJ" -eq 24 ] && [ "$MIN" -lt 9 ]; }; then
  echo "Node.js $V est trop ancien : DeepSeek Harness demande 24.9 ou plus. Installez la version 24 LTS."
  exit 1
fi
```
