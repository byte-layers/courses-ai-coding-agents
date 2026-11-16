---
title: "02 - Ce qui passe tel quel"
description: "Relier vos fichiers d'instructions et votre skill à ce que dsh envoie au modèle"
---

Les instructions et la skill suivent des formats ouverts, pensés pour être lus par plusieurs outils. Reste à savoir si dsh les lit vraiment, et à quel moment. Les constats de cette étape ont été faits avec dsh 0.2.2-rc.3, en capturant les requêtes qu'il envoie au modèle.

## 02.1 - Les instructions AGENTS.md

dsh lit les fichiers d'instructions de deux façons :

| Fichier | Comment dsh le trouve | Quand il l'envoie au modèle |
|---|---|---|
| `AGENTS.md` à la racine | en remontant les dossiers depuis celui où il est lancé, jusqu'à la racine du projet | dès le début de la session, dans les instructions de départ |
| `test/AGENTS.md` | quand l'agent lit ou modifie un fichier de `test/` | à ce moment-là, dans un rappel qui commence par `Additional instructions from: test/AGENTS.md` |

Ses réglages par défaut cherchent aussi `CLAUDE.md` au même endroit : un projet qui n'a qu'un `CLAUDE.md` est donc lu, lui aussi. Le second comportement suit la règle du format AGENTS.md : le fichier le plus proche du travail en cours s'applique. Vos deux fichiers passent donc **tels quels**.

## 02.2 - La skill

dsh cherche les skills d'un projet dans deux dossiers : `.agents/skills/`, le dossier partagé entre outils, et `.dsh/skills/`, le sien. Au début d'une session, il annonce au modèle le nom et la description de chaque skill trouvée. Pour `ears-spec`, la capture montre :

```text
The following skills are available in this session:
- `ears-spec`: Écrit ou relit des exigences en syntaxe EARS. À utiliser pour rédiger une spécification, un fichier requirements.md ou des critères d'acceptation.
```

Ces deux informations viennent de l'en-tête de votre fichier. Affichez-le :

```exec
cd ~/byte-layers/ai/todo-api && grep -m2 -E '^(name|description):' .agents/skills/ears-spec/SKILL.md
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `cd ~/byte-layers/ai/todo-api &&` | se place dans le projet, puis lance la suite si le `cd` a réussi |
> | `grep -E '^(name\|description):'` | garde les lignes qui commencent par `name:` ou par `description:` |
> | `-m2` | s'arrête après les deux premières lignes trouvées, celles de l'en-tête |

Vous retrouvez mot pour mot ce que dsh annonce. Le corps de la skill, lui, n'est lu que si le modèle décide de l'utiliser : c'est le chargement progressif de la spécification Agent Skills. La skill passe donc, elle aussi, **telle quelle**, parce qu'elle a été rangée dans le dossier partagé.

## 02.3 - Essai réel (facultatif)

Si Ollama tourne et sert votre modèle, comme au lab Préparer son poste, posez la question à dsh. La commande utilise le mode `headless` de dsh, qui traite une seule demande sans interface. Ce mode a son propre profil : `--patch` lui ajoute le réglage de votre profil `web`, qui désigne votre modèle Ollama.

```exec
cd ~/byte-layers/ai/todo-api && OLLAMA_API_KEY=ollama dsh headless --patch ~/.dsh/profiles/web/cordis.patch.yml "Sans lancer d'outil : quelles skills sont disponibles, et quelle commande AGENTS.md demande-t-il pour les tests ?"
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `cd ~/byte-layers/ai/todo-api &&` | se place dans le projet : dsh y cherchera `AGENTS.md` et les skills |
> | `OLLAMA_API_KEY=ollama` | la clé quelconque que dsh réclame pour Ollama, valable pour ce seul lancement |
> | `dsh headless "…"` | lance dsh sans interface : il traite la demande, affiche la réponse et s'arrête |
> | `--patch ~/.dsh/profiles/web/cordis.patch.yml` | applique le réglage de votre profil `web` au profil `headless` |

Selon votre poste, la réponse peut prendre une à plusieurs minutes. Elle doit citer `ears-spec` et `npm test`, la commande de la section Commandes de votre `AGENTS.md`. Un petit modèle peut se tromper dans sa formulation : relancez si besoin.
