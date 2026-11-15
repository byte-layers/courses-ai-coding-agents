---
title: "Choisir son modèle et son outil, sans guide"
description: "Trois situations à trancher : un modèle d'après un classement, un harness d'après des contraintes, et l'usage de l'IA de code dans trois entreprises. Vos décisions sont vérifiées."
tags:
  - challenge
  - ai
difficulty: beginner
duration: 20
introTitle: "Le défi"
requires: "0.4.0"
options:
  allowManualProgress: false
  autoCompleteOnNext: unverifiable
---

Ce challenge prouve que vous savez choisir un modèle, un outil et un usage de l'IA de code d'après des contraintes, au lieu de suivre le dernier titre. Il n'y a **aucune commande à recopier** : chaque tâche décrit une situation et des critères, et vous inscrivez votre décision dans un fichier. Un bouton vérifie ce fichier.

Si vous êtes bloqué, les **indices** se dévoilent un par un, et la **solution** reste en dernier recours. Les utiliser ne coûte rien : seule la vérification compte.

## Avant de commencer

- **Ce qu'il faut savoir** : ce qu'enseignent [Lire un classement de modèles](https://byte-layers.com/fr/bits/model-leaderboards), [Le paysage des harness de code](https://byte-layers.com/fr/bits/coding-harness-landscape) et [L'IA de code en entreprise, en France](https://byte-layers.com/fr/bits/ai-coding-at-work).
- **Ce qu'il faut avoir** : un terminal (Git Bash sous Windows) et un éditeur de texte.
- **Le fichier de décisions** : `~/byte-layers/ai/choose/decisions.txt` est créé pour vous à l'ouverture. Il contient une ligne `clé=` par décision, à compléter après le signe `=`. Les majuscules, les espaces et les accents sont ignorés. Les lignes qui commencent par `#` sont des commentaires. Le fichier reste sur votre poste à la fin : supprimez le dossier `~/byte-layers/ai/choose` quand vous n'en avez plus besoin.

# Tâche 1 : un modèle d'après un classement

## Choisir le modèle d'un agent local

```setup
D=~/byte-layers/ai/choose
if [ ! -f "$D/decisions.txt" ]; then
  mkdir -p "$D"
  cat > "$D/decisions.txt" <<'EOF'
# Tâche 1 : le modèle choisi, puis la famille de classement qui compte ici
modele=
famille=
# Tâche 2 : le harness choisi, puis le fichier d'instructions à écrire
harness=
instructions=
# Tâche 3 : une décision par entreprise (autoriser, refuser ou auto-heberger)
entreprise1=
entreprise2=
entreprise3=
EOF
fi
```

Une équipe veut un agent qui corrige des bugs dans ses dépôts. Ses contraintes :

- le code ne doit **pas quitter le poste** des développeurs, qui ont **16 Go** de mémoire vive ;
- le modèle sera intégré à un **produit commercial** de l'entreprise ;
- l'équipe ne croit que les résultats **mesurés par un tiers**.

Voici un extrait de classement. Les modèles sont **fictifs**, inventés pour l'exercice :

| Modèle | Rang par usage, cette semaine | Score à SWE-bench Verified | Score mesuré par | Licence | Taille |
|---|---|---|---|---|---|
| `orion-xl` | 1ᵉʳ | 71 % | le classement | propriétaire, par API seulement | non publiée |
| `lynx-8b` | 3ᵉ | 64 % | l'entreprise qui le publie | Apache 2.0 | 8B |
| `vega-7b` | 12ᵉ | 52 % | le classement | Apache 2.0 | 7B |
| `atlas-70b` | 25ᵉ | 66 % | le classement | usage non commercial | 70B |

Complétez dans `decisions.txt` :

- `modele=` avec le nom du seul modèle qui respecte les trois contraintes ;
- `famille=` avec la famille de classement qui dit si un modèle sait corriger des bugs vérifiés par des tests, parmi `taches`, `votes`, `indice` et `usage`.

**Critères :**

- le modèle choisi respecte les trois contraintes ;
- la famille désigne les classements qui mesurent la réussite de tâches vérifiées par des tests.

```hint
Éliminez les modèles un par un : lequel ne se télécharge pas ? lequel interdit l'usage commercial ? lequel n'a qu'un score annoncé par sa propre entreprise ?
```

```hint
Le rang par usage ne dit rien de la qualité : il reflète le prix et l'adoption. La question de l'équipe est « le modèle réussit-il ce travail ? ».
```

```verify
F=~/byte-layers/ai/choose/decisions.txt
v() { sed -n "s/^$1[[:space:]]*=//p" "$F" | tail -1 | tr 'A-Z' 'a-z' | tr -d '[:space:]"' | sed 's/é/e/g;s/è/e/g;s/ê/e/g;s/â/a/g;s/^\.\///'; }
[ -f "$F" ] || { echo "Le fichier $F n'existe pas. Rechargez la page du challenge pour le recréer."; exit 1; }
M=$(v modele)
case "$M" in
  vega-7b) ;;
  "") echo "La ligne modele= est vide."; exit 1 ;;
  orion-xl) echo "orion-xl ne s'utilise que par API : le code quitterait le poste."; exit 1 ;;
  lynx-8b) echo "Le score de lynx-8b est annoncé par l'entreprise qui le publie, pas mesuré par un tiers."; exit 1 ;;
  atlas-70b) echo "atlas-70b interdit l'usage commercial."; exit 1 ;;
  *) echo "modele=$M ne correspond à aucun modèle du tableau. Recopiez son nom exact."; exit 1 ;;
esac
case "$(v famille)" in
  taches) ;;
  "") echo "La ligne famille= est vide."; exit 1 ;;
  indice) echo "Un indice agrégé mesure par un tiers, mais il fait la moyenne de plusieurs benchmarks : il ne dit pas si le modèle sait corriger des bugs. Quelle famille donne des tâches vérifiées par des tests ?"; exit 1 ;;
  *) echo "Ce n'est pas la bonne famille : l'équipe veut savoir si le modèle réussit des tâches vérifiées par des tests."; exit 1 ;;
esac
echo "Le modèle et la famille de classement sont les bons."
```

````solution
```text
modele=vega-7b
famille=taches
```

`orion-xl` est fermé, donc le code partirait chez son entreprise. `atlas-70b` interdit l'usage commercial. `lynx-8b` n'a qu'un score auto-déclaré. `vega-7b` a le score le plus bas des quatre, mais c'est le seul qui respecte les contraintes : un essai sur votre propre tâche dira s'il suffit.
````

# Tâche 2 : un harness d'après des contraintes

## Choisir l'outil de l'équipe

La même équipe choisit maintenant son harness. Ses contraintes :

- l'outil s'utilise dans le **terminal** ;
- il accepte le **modèle local** choisi à la tâche 1, servi par Ollama ;
- il est **open source** et lit le fichier d'instructions standard, pour que l'équipe puisse **changer d'outil** plus tard sans réécrire sa configuration.

Complétez dans `decisions.txt` :

- `harness=` avec un outil qui respecte les trois contraintes, sous son nom du bit sur le paysage des harness (`opencode`, `claude-code`, `mistral-vibe`…) ;
- `instructions=` avec le nom du fichier d'instructions standard à écrire à la racine du dépôt.

**Critères :**

- l'outil choisi s'utilise dans le terminal, accepte un modèle local, est open source et lit le fichier standard ;
- le fichier d'instructions est celui que lisent la plupart des outils.

```hint
Dans le tableau des outils dans le terminal du bit sur le paysage des harness, regardez les colonnes « Entreprise, licence », « Modèles » et « Instructions lues ».
```

```verify
F=~/byte-layers/ai/choose/decisions.txt
v() { sed -n "s/^$1[[:space:]]*=//p" "$F" | tail -1 | tr 'A-Z' 'a-z' | tr -d '[:space:]"' | sed 's/é/e/g;s/è/e/g;s/ê/e/g;s/â/a/g;s/^\.\///'; }
[ -f "$F" ] || { echo "Le fichier $F n'existe pas. Rechargez la page du challenge pour le recréer."; exit 1; }
H=$(v harness)
case "$H" in
  opencode|pi|kilo|kilo-code|kilocode|mistral-vibe|mistralvibe|vibe|codex|codex-cli|codexcli|dsh|deepseek-harness|deepseekharness) ;;
  "") echo "La ligne harness= est vide."; exit 1 ;;
  claude-code|claudecode|claude|cursor|kiro|copilot|github-copilot|githubcopilot) echo "$H est propriétaire : l'équipe veut un outil open source."; exit 1 ;;
  gemini-cli|geminicli|gemini) echo "Gemini CLI ne sert que les modèles Gemini : pas de modèle local."; exit 1 ;;
  *) echo "harness=$H n'est pas un outil du bit sur le paysage des harness. Écrivez son nom en minuscules, avec des tirets à la place des espaces."; exit 1 ;;
esac
I=$(v instructions)
case "$I" in
  agents.md) ;;
  "") echo "La ligne instructions= est vide."; exit 1 ;;
  *) echo "$I est un fichier propre à un outil. Le fichier standard, lu par la plupart des outils, est un autre."; exit 1 ;;
esac
echo "L'outil et le fichier d'instructions conviennent."
```

````solution
```text
harness=opencode
instructions=agents.md
```

OpenCode, Pi, Kilo (qui a une CLI), Mistral Vibe, Codex CLI (avec `--oss`) et DeepSeek Harness sont tous acceptés : ils sont open source, acceptent un modèle local et lisent `AGENTS.md`. Claude Code, Cursor, Kiro et Copilot sont propriétaires ; Gemini CLI ne sert que Gemini.
````

# Tâche 3 : l'IA de code dans trois entreprises

## Trancher trois demandes

On vous soumet trois demandes. Pour chacune, choisissez une décision parmi trois :

| Décision | Ce qu'elle veut dire |
|---|---|
| `autoriser` | l'usage demandé respecte les règles, tel quel |
| `refuser` | l'usage demandé enfreint une règle de base |
| `auto-heberger` | l'usage est légitime, mais seul un modèle servi dans l'entreprise respecte la contrainte |

Les trois demandes :

1. **Entreprise 1.** Une développeuse veut utiliser son compte **personnel et gratuit** d'un assistant de code sur le code d'un client de son employeur, parce que l'entreprise n'a encore choisi aucun outil.
2. **Entreprise 2.** L'entreprise a souscrit une **offre entreprise** d'un assistant de code : contrat de sous-traitance des données signé, hébergement dans l'Union européenne, pas d'entraînement sur ses données, comptes fournis par l'employeur. Un développeur veut s'en servir sur le code de l'entreprise, sans secret ni donnée personnelle dans ses demandes.
3. **Entreprise 3.** Un éditeur de logiciels de santé veut un agent de code. Sa direction impose que **le code ne quitte jamais l'entreprise**, et aucune offre cloud ne remplit cette condition pour lui.

Complétez `entreprise1=`, `entreprise2=` et `entreprise3=` dans `decisions.txt`, avec l'une des trois décisions du tableau.

**Critères :**

- chaque décision applique les règles d'une politique d'usage : comptes entreprise uniquement, contrat vérifié ;
- la contrainte « le code ne quitte jamais l'entreprise » mène à la seule option qui la respecte.

```hint
Dans une entreprise, quel type de compte l'ANSSI recommande-t-elle d'utiliser, et pourquoi ?
```

```hint
Parmi les trois façons de réduire l'exposition du code, une seule garde le code dans l'entreprise.
```

```verify
F=~/byte-layers/ai/choose/decisions.txt
v() { sed -n "s/^$1[[:space:]]*=//p" "$F" | tail -1 | tr 'A-Z' 'a-z' | tr -d '[:space:]"' | sed 's/é/e/g;s/è/e/g;s/ê/e/g;s/â/a/g;s/^\.\///'; }
[ -f "$F" ] || { echo "Le fichier $F n'existe pas. Rechargez la page du challenge pour le recréer."; exit 1; }
for k in entreprise1 entreprise2 entreprise3; do
  case "$(v $k)" in
    autoriser|refuser|auto-heberger) ;;
    "") echo "La ligne $k= est vide."; exit 1 ;;
    *) echo "$k doit valoir autoriser, refuser ou auto-heberger (sans accent)."; exit 1 ;;
  esac
done
[ "$(v entreprise1)" = refuser ] || { echo "Entreprise 1 : un compte personnel et gratuit, sur le code d'un client, échappe à tout contrat. Relisez la règle sur les comptes."; exit 1; }
[ "$(v entreprise2)" = autoriser ] || { echo "Entreprise 2 : comptes entreprise, contrat signé, pas d'entraînement, pas de secret dans les demandes. Que manque-t-il ?"; exit 1; }
[ "$(v entreprise3)" = auto-heberger ] || { echo "Entreprise 3 : l'usage est légitime, mais le code ne doit pas sortir. Quelle option le garde dans l'entreprise ?"; exit 1; }
echo "Les trois décisions sont justes. Challenge terminé."
```

````solution
```text
entreprise1=refuser
entreprise2=autoriser
entreprise3=auto-heberger
```

L'entreprise 1 doit d'abord choisir un outil et fournir des comptes entreprise : un compte personnel gratuit, c'est du *shadow IT*, et les échanges peuvent servir à l'entraînement. L'entreprise 2 coche les règles d'une politique d'usage. L'entreprise 3 n'a qu'une option : un modèle ouvert servi en interne, comme Ollama le fait en petit sur votre poste.
````
