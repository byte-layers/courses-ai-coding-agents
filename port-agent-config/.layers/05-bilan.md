---
title: "05 - Écrire le bilan du portage"
description: "Consigner, brique par brique, ce qui passe, ce qui se refait et ce qui ne passe pas, puis commiter"
---

Le portage n'est utile à l'équipe que s'il est écrit : le prochain développeur qui ouvre le projet avec dsh doit savoir ce qui le protège et ce qui ne le protège plus.

## 05.1 - Le fichier de bilan

Le bilan tient dans un fichier de clés et de valeurs, une ligne par brique. On écrit d'abord le fichier avec ses clés, sans réponse :

```exec
cd ~/byte-layers/ai/todo-api && mkdir -p docs && cat > docs/port-dsh.txt <<'FIN'
# Portage de la configuration d'agent vers DeepSeek Harness 0.2.2-rc.3
# Valeurs possibles : passe, a-refaire (se refait), ne-passe-pas
agents_md =
test_agents_md =
skill =
permissions =
hooks =
# En quelques mots : quelle protection n'a plus aucune garantie dans dsh ?
garantie_perdue =
FIN
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `mkdir -p docs` | crée le dossier `docs` s'il n'existe pas encore |
> | `cat > docs/port-dsh.txt <<'FIN'` … `FIN` | écrit les lignes entre les deux `FIN` dans le fichier |

**À vous** : ouvrez `docs/port-dsh.txt` dans votre éditeur. Complétez les cinq premières lignes avec une des trois valeurs : `passe`, `a-refaire` pour une brique qui se refait, ou `ne-passe-pas`. Sur la dernière ligne, nommez en quelques mots la protection qui n'a plus aucune garantie dans dsh, et le fichier qu'elle protégeait.

```hint
Les instructions et la skill suivent des formats ouverts, que dsh lit sans changement. Les permissions ont un équivalent partiel. Les hooks n'ont pas d'équivalent dans cette version de dsh. Pour la dernière ligne, relisez le tableau de la section 03.2 : une seule règle n'y a « aucun » équivalent.
```

````solution
```text
agents_md = passe
test_agents_md = passe
skill = passe
permissions = a-refaire
hooks = ne-passe-pas
garantie_perdue = l'interdiction de lire le fichier .env
```
````

```verify
F=~/byte-layers/ai/todo-api/docs/port-dsh.txt
[ -f "$F" ] || { echo "Le fichier docs/port-dsh.txt n'existe pas. Lancez la commande de cette section."; exit 1; }
v() { sed -n "s/^[[:space:]]*$1[[:space:]]*=//p" "$F" | tail -1 | tr 'A-Z' 'a-z' | tr -d '"\r' | sed 's/^[[:space:]]*//;s/[[:space:]]*$//;s/[[:space:]_][[:space:]_]*/-/g;s/à/a/g'; }
check() {
  got=$(v "$1")
  [ -n "$got" ] || { echo "La ligne $1 n'a pas de valeur."; exit 1; }
  case "$got" in passe|a-refaire|ne-passe-pas) ;; *) echo "La valeur de $1 doit être passe, a-refaire ou ne-passe-pas (lu : $got)."; exit 1;; esac
  [ "$got" = "$2" ] || { echo "$1 : $3"; exit 1; }
}
check agents_md passe "relisez la section 02.1 : dsh envoie AGENTS.md au modèle sans changement."
check test_agents_md passe "relisez la section 02.1 : dsh ajoute test/AGENTS.md quand l'agent touche un fichier de test/."
check skill passe "relisez la section 02.2 : la skill est dans .agents/skills/, un dossier que dsh lit."
check permissions a-refaire "relisez la section 03.2 : dsh a un équivalent partiel, à régler dans son propre modèle."
check hooks ne-passe-pas "relisez la section 04.1 : dsh 0.2.2-rc.3 n'a pas d'équivalent sûr aux hooks du projet."
G=$(sed -n "s/^[[:space:]]*garantie_perdue[[:space:]]*=//p" "$F" | tail -1)
[ -n "$(echo "$G" | tr -d '[:space:]')" ] || { echo "La ligne garantie_perdue n'a pas de valeur : nommez la protection perdue."; exit 1; }
echo "$G" | grep -qi 'env' || { echo "garantie_perdue : la protection perdue concerne un fichier précis. Relisez la section 04.2."; exit 1; }
```

## 05.2 - Commiter le bilan

Le bilan rejoint la configuration qu'il décrit, avec le script de lancement de l'étape précédente. L'option `--chmod=+x` enregistre dans Git que le script est exécutable, y compris sous Windows, où `chmod` ne suffit pas :

```exec
cd ~/byte-layers/ai/todo-api && git add docs/port-dsh.txt && git add --chmod=+x scripts/dsh-web.sh && git commit -m "Ajoute le portage vers dsh : script de lancement et bilan"
```

```verify
cd ~/byte-layers/ai/todo-api 2>/dev/null || { echo "Le dossier ~/byte-layers/ai/todo-api n'existe pas."; exit 1; }
for f in docs/port-dsh.txt scripts/dsh-web.sh; do
  git cat-file -e "port-dsh:$f" 2>/dev/null || { echo "$f n'est pas commité sur la branche port-dsh. Lancez la commande de cette section."; exit 1; }
  [ -z "$(git status --porcelain -- "$f")" ] || { echo "$f a des modifications qui ne sont pas commitées. Relancez git add puis git commit."; exit 1; }
done
git ls-tree port-dsh scripts/dsh-web.sh | grep -q '^100755' || { echo "Git n'a pas enregistré scripts/dsh-web.sh comme exécutable. Lancez git add --chmod=+x scripts/dsh-web.sh puis commitez."; exit 1; }
```

## Récap

- Porter une configuration, c'est la reprendre **brique par brique**, et classer chacune : passe, se refait, ne passe pas.
- Ce qui suit un **format ouvert** passe dans dsh, parce que dsh lit ces formats : `AGENTS.md`, à la racine comme dans un sous-dossier, et une skill rangée dans `.agents/skills/`.
- Les **permissions** se refont : dsh n'a pas de règles par fichier ni par commande, mais un bac à sable et une politique d'accord, réglés pour la session. Un **script de lancement** commité partage le réglage de l'équipe, et fixe le dossier du bac à sable.
- Les **hooks** ne passent pas dans dsh 0.2.2-rc.3 : le plugin qui lirait le format Claude Code est déclaré incompatible avec cette version.
- Quand une garantie se perd, retirez ce qu'elle protégeait : pas de vrai secret à la portée d'un agent, ni dans ses fichiers, ni dans ses variables d'environnement.
- Écrivez le bilan dans le dépôt, pour le prochain développeur qui changera d'outil.

> 📖 [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) · [Le format AGENTS.md](https://agents.md/) · [La spécification Agent Skills](https://agentskills.io/specification)
