---
title: "L'IA de code en entreprise, en France"
description: "Où part votre code quand vous utilisez un agent, ce que changent les offres gratuites et entreprise, ce que disent le RGPD, l'AI Act et l'ANSSI, et les règles d'une politique d'usage."
tags:
  - bit
  - ai
difficulty: beginner
duration: 15
requires: "0.4.0"
---

Utiliser un agent de code au travail, c'est envoyer du code de l'entreprise à un tiers, à chaque demande. Ce bit explique ce que cela implique et les règles qui s'appliquent en France, pour que vous sachiez quoi vérifier avant d'installer un outil sur un projet réel.

**Pour qui** : un développeur qui sait comment un agent envoie le contexte à un modèle, ce qu'explique [La boucle d'un agent de code](https://byte-layers.com/fr/bits/agent-loop). Aucune connaissance juridique n'est supposée. Ce bit n'est pas un avis juridique : dans une entreprise, le délégué à la protection des données (DPO) et le responsable de la sécurité des systèmes d'information (RSSI) ont le dernier mot.

> 📅 **Politiques et textes vérifiés le 15 novembre 2026.** Les conditions des offres changent souvent : relisez la page de l'éditeur avant de décider.

# L'IA de code en entreprise, en France

## 1 - Où part votre code

À chaque tour de la boucle, le harness envoie au modèle le contexte complet : vos consignes, les fichiers lus, les sorties de commandes. Si le modèle tourne dans le cloud, tout ce texte quitte le poste. Chez le fournisseur, trois choses peuvent lui arriver :

| Ce qui peut arriver | Ce que cela veut dire |
|---|---|
| **Conservation** | les échanges sont gardés un certain temps (de quelques jours à plusieurs années), pour détecter les abus ou par défaut |
| **Entraînement** | les échanges servent à entraîner les prochains modèles : un extrait de votre code pourrait ressortir ailleurs |
| **Relecture humaine** | des employés du fournisseur peuvent lire des échanges, pour évaluer ou modérer |

Le code n'est pas seul en jeu. Un fichier de configuration lu par l'agent peut contenir une clé d'API ; une sortie de commande, un nom de client ; une base de test, des données personnelles. Tout cela entre dans le contexte au même titre que le code.

## 2 - Gratuit ou entreprise : ce n'est pas la même offre

Pour un même outil, les conditions changent souvent entre l'offre pour les particuliers et l'offre pour les entreprises. Le tableau résume quelques politiques officielles :

| Offre | Utilisation des échanges pour l'entraînement |
|---|---|
| GitHub Copilot Free, Pro et Pro+ | oui depuis le 24 avril 2026, sauf si vous le désactivez dans les réglages |
| GitHub Copilot Business et Enterprise | non |
| Claude pour les particuliers (Pro, Max) | selon un réglage que chaque utilisateur doit choisir depuis 2025 ; activé, les échanges sont conservés jusqu'à cinq ans |
| Claude Team, Enterprise et l'API | non |
| Gemini API, niveau gratuit | oui, avec relecture humaine possible ; Google déconseille d'y envoyer des données sensibles |
| Gemini API, niveau payant | non |
| Mistral pour les particuliers | oui par défaut, désactivable |
| Mistral Team et Enterprise | non par défaut |
| Kilo, modèles gratuits (`kilo-auto/free`) | peut passer par des fournisseurs qui enregistrent les échanges et s'en servent |
| Cursor | non si le *Privacy Mode* est activé ; possible sinon |

La règle qui s'en dégage : **une offre gratuite se paie souvent avec vos données**, et une offre entreprise s'accompagne d'un contrat qui les protège. Dans une entreprise, on utilise donc des comptes entreprise fournis par l'employeur, jamais un compte personnel : l'ANSSI le recommande explicitement, pour éviter le *shadow IT* (des outils utilisés hors de tout contrôle).

## 3 - Ce que disent les textes

**Le RGPD et la CNIL.** Le RGPD (Règlement général sur la protection des données) encadre les données personnelles dans l'Union européenne, et la CNIL (Commission nationale de l'informatique et des libertés) veille à son application en France. Dès qu'une donnée permet d'identifier une personne (un nom, un e-mail, un identifiant client), le RGPD s'applique. La CNIL recommande de ne mettre ni données personnelles ni informations confidentielles dans un service d'IA grand public, de vérifier si le fournisseur réutilise les données et de désactiver cette réutilisation quand c'est possible, et d'encadrer l'usage par une charte interne.

**L'AI Act européen.** Ce règlement classe les systèmes d'IA selon leur risque. Un assistant de code ne fait pas partie des usages à haut risque listés par le texte. Pour un développeur qui s'en sert, l'obligation qui compte est celle de l'article 4 : l'employeur doit prendre des mesures pour **accompagner la formation** de son personnel à l'IA. Le texte s'applique par étapes, et son calendrier a été révisé en 2026 : consultez la page de la Commission européenne pour les dates à jour.

**L'ANSSI.** L'agence nationale de la sécurité des systèmes d'information a publié en 2024, avec son homologue allemand (le BSI), un document sur les assistants de code. Ses recommandations, pour un développeur :

- les assistants ne remplacent pas des développeurs expérimentés : le code généré doit être **relu et compris**, et passer les tests, les revues de sécurité et les scanners de vulnérabilités ;
- **aucun secret** dans les demandes : ni clé d'API, ni clé de chiffrement, ni détail d'infrastructure ;
- se méfier des **paquets hallucinés** : dans une étude citée par le document, 19,7 % des paquets importés par du code généré n'existaient pas. Un attaquant peut publier un paquet sous ce nom inventé ; vérifiez toute bibliothèque inconnue avant de l'installer ;
- garder la capacité de travailler **sans** l'assistant.

## 4 - Souveraineté : pourquoi la question revient

Un fournisseur américain est soumis aux lois américaines, dont le *CLOUD Act*, qui permet aux autorités des États-Unis de lui demander des données, même stockées en Europe. C'est pour cela que l'ANSSI a créé la qualification **SecNumCloud** : un service qualifié doit être protégé contre ces lois extra-européennes, avec des critères sur le siège, l'actionnariat et l'autonomie du fournisseur. L'État l'impose pour ses données les plus sensibles.

Pour l'IA de code, il y a trois façons de réduire cette exposition :

| Option | Exemple | Ce qu'il faut vérifier |
|---|---|---|
| un fournisseur européen | Mistral AI, entreprise française, héberge les données dans l'Union européenne par défaut | ses sous-traitants, et l'offre souscrite (particulier ou entreprise) |
| un hébergement qualifié | des offres cloud qualifiées SecNumCloud | que le **service d'IA lui-même** est qualifié, pas seulement les serveurs : le catalogue de l'ANSSI fait foi |
| l'auto-hébergement d'un modèle ouvert | Devstral Small 2 de Mistral (licence Apache 2.0), servi par Ollama ou un serveur interne | le matériel nécessaire, et la licence du modèle |

L'auto-hébergement est la seule option où le code ne quitte pas l'entreprise : c'est ce que fait, en petit, le couple DeepSeek Harness et Ollama du lab [Préparer son poste](https://byte-layers.com/fr/labs/ai-agent-workstation).

## 5 - Les règles d'une politique d'usage

Il n'existe pas de modèle officiel de politique d'usage, mais les recommandations de la CNIL et de l'ANSSI se recoupent. Une politique d'usage fixe au minimum :

1. **les outils autorisés**, avec des comptes entreprise, et l'interdiction des comptes personnels ;
2. **un contrat vérifié** : pas d'entraînement sur les données de l'entreprise, lieu d'hébergement connu, contrat de sous-traitance des données (*Data Processing Agreement*, DPA) signé ;
3. **ce qui n'entre jamais dans une demande** : secrets, données personnelles, informations confidentielles ;
4. **la relecture obligatoire** du code généré, avec tests et scanners, et la vérification de tout nouveau paquet ;
5. **la formation** avant le premier usage, et des contrôles réguliers, avec le DPO dans la boucle.

Avant d'installer un agent sur un projet de votre employeur, cherchez cette politique. S'il n'y en a pas, posez la question : c'est à l'entreprise de décider où part son code.

## Récap

- Avec un modèle dans le cloud, tout le contexte part chez le fournisseur, qui peut le **conserver**, s'en servir pour **l'entraînement** ou le faire **relire**.
- Les offres **gratuites** utilisent souvent vos échanges ; les offres **entreprise** les protègent par contrat. Au travail : comptes entreprise uniquement.
- **RGPD** : pas de données personnelles dans les demandes. **AI Act** : l'employeur accompagne votre formation. **ANSSI** : relire le code, aucun secret, vérifier les paquets.
- La **souveraineté** passe par un fournisseur européen, un service qualifié **SecNumCloud**, ou un modèle **auto-hébergé**.

> 📖 [CNIL : questions-réponses sur l'IA générative](https://cnil.fr/fr/les-questions-reponses-de-la-cnil-sur-lutilisation-dun-systeme-dia-generative) · [ANSSI et BSI : AI Coding Assistants (en anglais)](https://cyber.gouv.fr/nous-connaitre/publications/publications-internationales/ai-coding-assistants/) · [L'AI Act et son calendrier (Commission européenne)](https://digital-strategy.ec.europa.eu/en/policies/regulatory-framework-ai) · [L'article 4 de l'AI Act](https://ai-act-service-desk.ec.europa.eu/en/ai-act/article-4) · [Le cloud et SecNumCloud (ANSSI)](https://cyber.gouv.fr/enjeux-technologiques/cloud/) · [GitHub Copilot et les données d'interaction](https://github.blog/news-insights/company-news/updates-to-github-copilot-interaction-data-usage-policy/) · [Claude : conditions pour les particuliers](https://www.anthropic.com/news/updates-to-our-consumer-terms) · [Gemini API : conditions](https://aistudio.google.com/docs/terms) · [Cursor : usage des données](https://cursor.com/data-use) · [Kilo : modèles et fournisseurs](https://kilo.ai/docs/gateway/models-and-providers) · [Mistral : où sont stockées vos données](https://help.mistral.ai/en/articles/347629-where-do-you-store-my-data-or-my-organization-s-data)

```quiz
title: L'IA de code en entreprise, en France
tracksProgress: true
submitMode: last-only
questions:
  - prompt: Un collègue utilise son compte gratuit personnel d'un assistant de code sur le dépôt de l'entreprise. Quel est le problème principal ?
    answers:
      - "L'assistant sera plus lent qu'avec un compte payant"
      - "Le code de l'entreprise part sous des conditions pensées pour les particuliers, souvent avec réutilisation pour l'entraînement, et hors de tout contrat"
      - "Le compte gratuit ne donne pas accès aux appels d'outils"
      - "Aucun : un dépôt privé ne peut pas être lu par l'assistant"
    correct: 2
    explanation: |
      Les offres gratuites réutilisent souvent les échanges. Au travail, on utilise les comptes
      entreprise fournis par l'employeur, couverts par un contrat ; l'ANSSI le recommande.
  - prompt: L'agent a lu un fichier .env qui contient une clé d'API, avant d'envoyer le contexte à un modèle dans le cloud. Que s'est-il passé ?
    answers:
      - "La clé est partie chez le fournisseur avec le reste du contexte"
      - "Rien : les harness retirent automatiquement les secrets"
      - "La clé a été chiffrée par le modèle"
      - "Le modèle refuse de lire les fichiers .env"
    correct: 1
    explanation: |
      Tout ce que l'agent lit entre dans le contexte envoyé au modèle. D'où la règle de
      l'ANSSI, aucun secret dans les demandes, et les permissions qui interdisent la lecture
      des fichiers de secrets.
  - prompt: Pour un développeur qui utilise un assistant de code, quelle obligation de l'AI Act compte vraiment ?
    answers:
      - "Faire certifier chaque fonction générée"
      - "Déclarer l'assistant comme système à haut risque"
      - "L'employeur accompagne la formation de son personnel à l'IA (article 4)"
      - "Publier le code généré sous licence libre"
    correct: 3
    explanation: |
      Un assistant de code ne fait pas partie des usages à haut risque. L'article 4 demande à
      l'employeur d'accompagner la formation de son personnel à l'IA.
  - prompt: Le code généré importe un paquet npm que vous ne connaissez pas. Que recommande l'ANSSI ?
    answers:
      - "L'installer : l'assistant ne cite que des paquets populaires"
      - "Demander à l'assistant si le paquet est sûr"
      - "Le remplacer par la dernière version"
      - "Vérifier qu'il existe et qu'il est fiable avant de l'installer, car il peut s'agir d'un nom inventé qu'un attaquant a occupé"
    correct: 4
    explanation: |
      Les modèles inventent parfois des noms de paquets. Un attaquant peut publier un paquet
      malveillant sous ce nom : c'est l'attaque par paquet halluciné.
  - prompt: Votre entreprise ne veut pas que son code quitte son infrastructure. Quelle option répond à cette exigence ?
    answers:
      - "Un fournisseur européen dans le cloud"
      - "Un modèle ouvert auto-hébergé, servi par un serveur interne"
      - "Une offre entreprise d'un fournisseur américain"
      - "Le niveau payant d'une API, sans conservation"
    correct: 2
    explanation: |
      Seul l'auto-hébergement garde le code dans l'entreprise. Les autres options l'envoient
      chez un tiers, avec plus ou moins de garanties.
```
