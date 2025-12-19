---
title: "La face cachée des MCP"
pubDate: 2025-12-19
lang: fr
published: true
description: "Les MCP sont puissants, mais ils ont un coût caché : la saturation de la fenêtre de contexte. Découvrez pourquoi je privilégie des custom tools Opencode ciblés pour gagner en fiabilité et en stabilité."
tags: [blog, ia, llm, mcp, opencode, tooling, architecture, r&d]
---

Après plusieurs mois de R&D et d'utilisation de l'IA, le constat est clair : les MCP sont un levier puissant, mais ils exercent une pression invisible sur la fenêtre de contexte. 

Voici comment je suis passé d'un catalogue exhaustif à une architecture plus sobre.

<!-- excerpt -->

Les MCP (*Model Context Protocol*) se sont imposés comme une évidence : un standard ouvert pour connecter une application "agent" à des outils externes sans réinventer la roue à chaque intégration. Sur le papier, la promesse est irrésistible.

Pourtant, après plusieurs mois d'itérations et de tests, mon enthousiasme s'est nuancé. Les MCP sont puissants, certes, mais ils ne sont pas "gratuits". Et leur coût le plus pénible n'est ni l'infrastructure, ni la latence, ni l'authentification. C'est un coût plus insidieux, qui se paie directement dans la **fenêtre de contexte**.

## La facture invisible : quand l’abondance devient du bruit

Quand on travaille avec des LLM, la fenêtre de contexte est un budget fini. Idéalement, ce budget doit être dépensé pour porter le problème réel : le code utilisateur, l'historique, les contraintes métier. Mais chaque outil ajouté via un serveur MCP consomme une part de ce budget pour sa propre "documentation machine" (schémas, descriptions, exemples).

C'est là que le piège se referme. Parce que les MCP facilitent l'accès à un écosystème d'outils, on a tendance à en ajouter "au cas où". Un connecteur supplémentaire, puis deux, puis dix.

Le résultat n'est pas une explosion immédiate, mais une dégradation progressive de la fiabilité. Plus on augmente la surface d'outils exposée, plus on augmente l'ambiguïté pour le modèle. L'agent commence à hésiter, à appeler des outils pour "vérifier", ou pire, à choisir un outil plausible mais incorrect. C'est une forme de bruit cognitif : à force de voir trop de portes, l'agent ne sait plus laquelle ouvrir.

## L’antidote : le "Custom Tool" comme filtre d’intention

Faut-il pour autant abandonner les MCP ? Absolument pas. Ils restent excellents pour l'interopérabilité. Mais le secret réside dans ce qu'on choisit de *montrer* au modèle.

C'est ici que l'approche des **tools custom** (tels qu'on les définit simplement dans `.opencode/tool/`) prend tout son sens. Plutôt que d'exposer un catalogue brut de primitives via un serveur MCP, je préfère désormais construire des outils qui encodent une **intention**.

Prenons un cas concret : **GitHub**.
Le MCP officiel est très complet, mais il expose une surface immense. Bien qu'il soit possible de filtrer les outils d'un MCP dans Opencode, cela crée une dépendance fragile : si le mainteneur du MCP change le nom d'un outil, votre configuration casse.

J'ai donc fini par créer mes propres outils qui communiquent directement avec l'API (REST ou GraphQL) pour mes besoins réels.
*   Au lieu d'exposer tout le graphe GitHub, je fournis des outils ciblés comme `github_get_pr_diff` ou `jira_get_issue`.
*   Ces outils ne font pas de "magie" (ils remontent la donnée brute), mais leur signature est stable et leur description est optimisée pour *mon* cas d'usage.

Le gain est double :
1.  **Déterminisme** : En maîtrisant la description de l'outil, je guide le LLM bien plus efficacement qu'avec les descriptions génériques d'un MCP tiers.
2.  **Stabilité** : Je ne suis plus à la merci d'un changement de nommage dans un package externe.

En forçant ce niveau d'abstraction, on soulage le modèle. Il n'a plus à improviser une chorégraphie complexe d'appels d'API ; il déclenche une action unique, cohérente et testable. On réduit la surface d'erreur tout en augmentant la densité de valeur par token consommé.

## Vers une architecture raisonnée : la précision plutôt que l'exhaustivité

Au fil de mes usages, une règle de design s'est imposée naturellement.

Concrètement, pour une intégration donnée (comme GitHub ou Jira), je ne branche pas le serveur MCP complet. Je développe directement les 3 à 5 outils "custom" qui couvrent 90% du workflow réel.

Ce pattern a un effet immédiat : la fenêtre de contexte respire. L'agent n'est pas distrait par 40 outils inutiles. La décision d'appel devient évidente.

**La parcimonie n'est pas ici une contrainte, c'est une optimisation de performance.**

La prochaine étape logique — et le vrai défi technique — sera de charger ces outils "à la demande" pour ne payer leur coût contextuel qu'au moment exact où ils deviennent nécessaires.

---

### Pour aller plus loin

- **MCP (documentation)** : [modelcontextprotocol.io](https://modelcontextprotocol.io/)
- **Annonce Anthropic** : [Introduction du MCP](https://www.anthropic.com/news/model-context-protocol)
- **OpenAI Apps SDK** : [Concepts MCP Server](https://developers.openai.com/apps-sdk/concepts/mcp-server/)
- **Opencode** : [Custom tools](https://opencode.ai/docs/custom-tools/)
