---
title: "05 - Commiter les hooks"
description: "Versionner les hooks avec le projet, pour qu'ils protègent toute l'équipe"
---

Les hooks protègent tous ceux qui ouvrent le projet, à condition d'être commités : un hook resté sur votre poste ne garantit rien aux autres.

## 05.1 - Commiter sur la branche hooks

Regardez ce que Git voit :

```exec
cd ~/byte-layers/ai/todo-api && git status --short --untracked-files=all
```

Les quatre fichiers des hooks apparaissent, précédés de `??`, la marque des fichiers nouveaux :

```text
?? .claude/hooks/block-env.js
?? .claude/settings.json
?? .opencode/plugins/block-env.js
?? .opencode/plugins/check-syntax.js
```

D'autres fichiers peuvent apparaître selon ce que vous avez lancé. `broken.js` ou `notes.md` sont des restes des tests de l'étape précédente : supprimez-les avec `rm`. `.claude/settings.local.json` est créé par Claude Code pour vos réglages personnels : il ne se commite pas. Ajoutez donc les quatre fichiers par leur nom, puis commitez :

```exec
cd ~/byte-layers/ai/todo-api && git add .claude/hooks/block-env.js .claude/settings.json .opencode/plugins/block-env.js .opencode/plugins/check-syntax.js && git commit -m "Ajoute les hooks : .env bloqué, syntaxe vérifiée"
```

Un hook commité est du code qui s'exécute sur le poste de chaque développeur qui lance l'agent dans le projet. Relisez donc les hooks d'un dépôt que vous clonez comme vous reliriez un script d'installation : c'est la contrepartie de leur pouvoir.

```verify
cd ~/byte-layers/ai/todo-api 2>/dev/null || { echo "Le dossier ~/byte-layers/ai/todo-api n'existe pas."; exit 1; }
for f in .claude/hooks/block-env.js .claude/settings.json .opencode/plugins/block-env.js .opencode/plugins/check-syntax.js; do
  if ! git cat-file -e "hooks:$f" 2>/dev/null; then
    echo "$f n'est pas commité sur la branche hooks. Lancez git add puis git commit (commande de cette section)."
    exit 1
  fi
  if [ -n "$(git status --porcelain -- "$f")" ]; then
    echo "$f a des modifications qui ne sont pas commitées. Relancez git add puis git commit."
    exit 1
  fi
done
for f in broken.js notes.md .claude/settings.local.json; do
  if git ls-tree -r --name-only hooks | grep -qxF "$f"; then
    echo "$f est commité sur la branche hooks : retirez-le avec git rm --cached $f, puis commitez."
    exit 1
  fi
done
```

## Récap

- Une règle `deny` garantit pour un outil et un motif ; une règle `ask` **demande** et compte sur votre vigilance ; un hook **garantit** sur tout le texte de chaque appel, parce que c'est votre code qui décide.
- Un hook **avant** l'outil peut refuser l'appel ; un hook **après** ne peut plus l'empêcher, mais il vérifie et prévient le modèle.
- OpenCode charge des plugins JavaScript dans `.opencode/plugins/` : `throw` refuse, `output.output` complète le résultat.
- Au format Claude Code, repris par d'autres outils, le hook est un script qui lit l'appel en JSON. Le code 2 bloque avant l'outil et transmet le message au modèle après ; **le code 1 laisse tout passer**, y compris quand le script plante.
- Un hook compare du texte : c'est un garde-fou contre les erreurs d'un agent, pas un coffre-fort. Le secret le mieux protégé est celui qui n'est pas dans le dossier de travail.
- Un hook se teste sans modèle, en lui envoyant le faux appel que le harness lui enverrait.
- Un hook commité s'exécute chez chacun : relisez ceux des dépôts que vous clonez.

> 📖 [Les plugins OpenCode](https://opencode.ai/docs/plugins/) · [Les hooks de Claude Code](https://code.claude.com/docs/en/hooks) · [node --check](https://nodejs.org/api/cli.html#-c---check)
