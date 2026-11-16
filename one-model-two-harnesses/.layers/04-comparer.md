---
title: "04 - Comparer les deux essais"
description: "Comparer le code des deux branches, et ce que chaque harness a fait différemment avec le même modèle"
---

Les deux branches contiennent la même fonctionnalité, écrite par le même modèle. Ce qui les sépare vient du harness : les consignes qu'il envoie, les outils qu'il donne au modèle, sa façon de demander des autorisations.

## 04.1 - Comparer le code

`git diff` compare deux branches. Limitez la comparaison au fichier modifié :

```exec
cd ~/byte-layers/ai/todo-api && git diff try-dsh try-opencode -- src/todos.js
```

Si l'affichage est vide, les deux harness ont écrit exactement le même code. Le plus souvent, il montre de petites différences : le nom d'une variable, une boucle à la place de `filter`, un commentaire en plus. Le même modèle ne répond pas deux fois de la même façon, même à la même demande, et chaque harness lui présente la demande dans un contexte différent.

## 04.2 - Comparer le travail des harness

Reprenez vos notes des deux essais. Le tableau donne ce qui ne dépend pas de vous, mesuré sur ces versions des deux outils :

| | DeepSeek Harness 0.2.2-rc.3 | OpenCode 1.19.5 |
|---|---|---|
| Consignes envoyées avant votre demande | environ 9 800 tokens | environ 7 700 tokens, dont plus de la moitié pour la description de ses dix outils |
| Où vous déclarez le modèle | `~/.dsh/profiles/web/cordis.patch.yml` | `~/.config/opencode/opencode.json` |
| Fichier d'instructions lu | `AGENTS.md` | `AGENTS.md` |
| Permissions | réglées par dsh | `opencode.json` du projet, écrit au lab précédent |
| Interface | web, dans le navigateur | terminal (`opencode run`), ou interface plein écran avec `opencode` |

Avec un petit modèle local, le poids des consignes compte : chaque tour de la boucle les renvoie, et le modèle doit les relire avant de répondre. C'est une des raisons pour lesquelles le même modèle paraît plus ou moins rapide, ou plus ou moins efficace, selon le harness.

## 04.3 - Ce qu'il faut retenir

```acknowledge
J'ai comparé les deux branches et noté ce que chaque harness a fait différemment.
```

## Récap

- Un harness et un modèle sont **séparés** : le même modèle Ollama se branche sur dsh comme sur OpenCode, par la même adresse, `localhost:11434`.
- Pour comparer deux outils, donnez-leur **la même demande sur le même point de départ**, chacun sur sa branche.
- Le résultat varie d'un harness à l'autre, et d'un essai à l'autre : vérifiez toujours avec les tests, pas avec le message de l'agent.
- Les **consignes** et les **outils** qu'un harness envoie à chaque tour pèsent sur la vitesse et la qualité, surtout avec un petit modèle local.

> 📖 [Les fournisseurs dans OpenCode](https://opencode.ai/docs/providers/) · [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) · [La longueur de contexte dans Ollama](https://docs.ollama.com/context-length)
