---
title: "06 - Commiter la configuration"
description: "Versionner la configuration de l'agent avec le code, sans le fichier .env"
---

La configuration de l'agent fait partie du projet : commitée, elle s'applique à toute l'équipe et à chaque clone, et elle évolue avec le code. Le fichier `.env`, lui, reste sur votre poste.

## 06.1 - Commiter sur la branche agent-config

Regardez d'abord ce que Git voit. Le fichier `.env` n'apparaît pas, puisque `.gitignore` l'exclut :

```exec
cd ~/byte-layers/ai/todo-api && git status --short --untracked-files=all
```

L'option `--untracked-files=all` détaille les fichiers à l'intérieur des nouveaux dossiers. La liste contient vos cinq fichiers :

```text
?? .agents/skills/ears-spec/SKILL.md
?? AGENTS.md
?? docs/specs/README.md
?? opencode.json
?? test/AGENTS.md
```

Ajoutez ces cinq fichiers à l'index, puis commitez-les en une fois :

```exec
cd ~/byte-layers/ai/todo-api && git add AGENTS.md test/AGENTS.md docs/specs .agents opencode.json && git commit -m "Ajoute la configuration de l'agent"
```

Si Git répond `Please tell me who you are`, votre identité n'est pas réglée : lancez les deux commandes `git config --global user.name "Votre Nom"` et `git config --global user.email "vous@exemple.fr"` qu'il propose, avec vos valeurs, puis relancez le commit.

Ne poussez pas ce commit : le dépôt d'origine appartient à byte_layers. Le projet restera sur votre poste, sur la branche `agent-config`, pour la suite du stack.

```verify
cd ~/byte-layers/ai/todo-api 2>/dev/null || { echo "Le dossier ~/byte-layers/ai/todo-api n'existe pas."; exit 1; }
for f in AGENTS.md test/AGENTS.md docs/specs/README.md .agents/skills/ears-spec/SKILL.md opencode.json; do
  if ! git ls-files --error-unmatch "$f" >/dev/null 2>&1; then
    echo "$f n'est pas commité. Lancez git add puis git commit (commande de cette section)."
    exit 1
  fi
done
if git ls-files --error-unmatch .env >/dev/null 2>&1; then
  echo "Le fichier .env est commité : il ne doit pas l'être. Retirez-le avec git rm --cached .env, puis commitez."
  exit 1
fi
if [ -n "$(git status --porcelain -- AGENTS.md test docs .agents opencode.json)" ]; then
  echo "Des modifications de la configuration ne sont pas commitées. Ajoutez-les et commitez."
  exit 1
fi
```

## Récap

- **Portable d'abord** : `AGENTS.md` à la racine pour les instructions du projet, des `AGENTS.md` de sous-dossier pour les règles ciblées (le plus proche l'emporte), et des skills au format `SKILL.md` dans `.agents/skills/`.
- Une skill est un dossier ; son `name` est identique au nom du dossier, et sa `description` dit quand la charger.
- Les **permissions** sont propres à chaque outil. Dans OpenCode, `opencode.json` les range sous `permission`, et la dernière règle qui correspond l'emporte.
- Une instruction **demande**, une permission **garantit** : les interdits importants vont dans les deux.
- `opencode debug skill` et `opencode debug agent build` montrent ce qu'OpenCode a réellement compris.
- La configuration se commite avec le code ; `.env` jamais.

> 📖 [AGENTS.md](https://agents.md/) · [La spécification Agent Skills](https://agentskills.io/specification) · [Les permissions dans OpenCode](https://opencode.ai/docs/permissions/)
