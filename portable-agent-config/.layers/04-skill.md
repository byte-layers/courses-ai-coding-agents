---
title: "04 - Écrire une skill au format standard"
description: "Une skill ears-spec dans .agents/skills, avec le frontmatter exigé par la spécification Agent Skills"
---

Les instructions du projet sont chargées à chaque demande. Une **skill**, au contraire, n'est chargée qu'au besoin : l'agent ne voit d'abord que son nom et sa description, et ne lit le reste que si la demande correspond. C'est la bonne brique pour un savoir-faire précis et un peu long, comme écrire des exigences en syntaxe EARS (*Easy Approach to Requirements Syntax*), que [Le développement piloté par les spécifications et EARS](https://byte-layers.com/fr/bits/spec-driven-development) présente.

## 04.1 - Le format d'une skill

Une skill est un **dossier** qui contient un fichier `SKILL.md`. Le fichier commence par un **frontmatter** : un bloc de métadonnées en YAML, encadré par deux lignes `---`. La spécification Agent Skills y exige deux champs :

| Champ | Règle |
|---|---|
| `name` | de 1 à 64 caractères : minuscules, chiffres et tirets, sans tiret au début, à la fin ni deux à la suite. Il doit être **identique au nom du dossier** |
| `description` | de 1 à 1 024 caractères : ce que fait la skill **et quand l'utiliser**. C'est tout ce que l'agent lit pour décider de la charger. Si le texte contient un deux-points suivi d'une espace (`Usage : …`), mettez-le entre guillemets : sinon le YAML est invalide et l'outil ignore la skill |

Où ranger le dossier ? Chaque outil a son emplacement (`.claude/skills`, `.opencode/skills`…), mais `.agents/skills/` est l'emplacement **partagé** que lisent plusieurs outils, dont OpenCode et Kilo. On choisit celui-là, pour rester portable.

> 📖 [La spécification Agent Skills](https://agentskills.io/specification) · [Les skills dans OpenCode](https://opencode.ai/docs/skills/)

## 04.2 - Créer la skill ears-spec

Créez d'abord le dossier de la skill. Son nom, `ears-spec`, sera aussi le `name` du frontmatter :

```exec
mkdir -p ~/byte-layers/ai/todo-api/.agents/skills/ears-spec
```

Créez maintenant, dans votre éditeur, le fichier `.agents/skills/ears-spec/SKILL.md`. Le corps de la skill est fourni ci-dessous ; **écrivez vous-même le frontmatter** au-dessus, d'après le tableau de la section précédente. Les dossiers qui commencent par un point sont masqués par défaut : dans VS Code, ouvrez le dossier `todo-api` entier, ils apparaissent dans l'explorateur.

```markdown
# Écrire des exigences EARS

Chaque exigence suit l'un des cinq motifs EARS, une phrase par exigence :

- Toujours : « Le système doit <réponse>. »
- Événement : « Quand <déclencheur>, le système doit <réponse>. »
- État : « Pendant que <état>, le système doit <réponse>. »
- Comportement indésirable : « Si <condition>, alors le système doit <réponse>. »
- Option : « Là où <fonctionnalité est présente>, le système doit <réponse>. »

Écrivez les exigences dans docs/specs/<fonctionnalité>/requirements.md, numérotées, une par ligne.
Chaque exigence doit pouvoir devenir un test.
```

**Critères :**

- le fichier commence par une ligne `---`, et une seconde ligne `---` ferme le frontmatter ;
- `name` vaut le nom du dossier ;
- `description` dit ce que fait la skill et quand l'utiliser, en moins de 1 024 caractères ;
- le corps fourni suit le frontmatter.

```hint
Le frontmatter tient en quatre lignes : `---`, `name: …`, `description: …`, `---`. Pour la description, pensez à la phrase qui ferait dire à l'agent « c'est le moment de charger cette skill » : écrire ou relire des exigences, une spécification, `requirements.md`.
```

````solution
```markdown
---
name: ears-spec
description: Écrit ou relit des exigences en syntaxe EARS. À utiliser pour rédiger une spécification, un fichier requirements.md ou des critères d'acceptation.
---

# Écrire des exigences EARS
…
```
````

```verify
F=~/byte-layers/ai/todo-api/.agents/skills/ears-spec/SKILL.md
BOM=$(printf '\357\273\277')
if [ ! -f "$F" ]; then
  echo "Le fichier $F n'existe pas. Vérifiez le nom du dossier et du fichier (SKILL.md en majuscules)."
  exit 1
fi
if [ "$(head -1 "$F" | tr -d '\r' | sed "1s/^$BOM//")" != "---" ]; then
  echo "La première ligne de SKILL.md doit être exactement ---."
  exit 1
fi
FM=$(tr -d '\r' < "$F" | sed "1s/^$BOM//" | awk 'NR>1 && /^---$/ {exit} NR>1 {print}')
if ! tr -d '\r' < "$F" | awk 'NR>1 && /^---$/ {f=1} END {exit !f}'; then
  echo "Le frontmatter n'est pas fermé : ajoutez une ligne --- après la description."
  exit 1
fi
N=$(echo "$FM" | sed -n 's/^name:[[:space:]]*//p' | tr -d "\"' ")
if [ "$N" != "ears-spec" ]; then
  echo "name vaut « $N » : il doit être identique au nom du dossier, ears-spec."
  exit 1
fi
D=$(echo "$FM" | sed -n 's/^description:[[:space:]]*//p')
if [ -z "$D" ]; then
  echo "Le champ description est vide ou absent."
  exit 1
fi
case "$D" in
  \"*|\'*) ;;
  *": "*) echo "La description contient « : » sans guillemets : le YAML est invalide. Mettez toute la description entre guillemets."; exit 1 ;;
esac
if [ "${#D}" -gt 1024 ]; then
  echo "La description fait ${#D} caractères : 1 024 au plus."
  exit 1
fi
if ! grep -q 'Quand <déclencheur>' "$F"; then
  echo "Le corps de la skill manque après le frontmatter : recopiez le bloc fourni."
  exit 1
fi
```
