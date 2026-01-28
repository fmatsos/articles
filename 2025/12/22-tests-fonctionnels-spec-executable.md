---
title: "Tests fonctionnels : la spec exécutable (et la dette qui ment)"
description: "Un test fonctionnel non maintenu est une dette qui ment. Découvrez comment transformer vos tests en specs exécutables pour sécuriser vos règles métier critiques."
pubDate: 2025-12-22
lang: fr
draft: true
tags:
  - "tests"
  - "behat"
  - "qualite"
  - "metier"
  - "fiabilite"
---

Le métier bouge, le code doit suivre. Mais si vos tests fonctionnels ne sont pas maintenus au même rythme, ils deviennent une dette qui ment.

<!-- excerpt -->

Les règles métier sont vivantes : elles évoluent, se nuancent, et s'enrichissent d'exceptions. À l'inverse, le code vise la stabilité et le déterminisme. C'est dans cet écart que se joue la fiabilité d'un produit.

Un test fonctionnel bien écrit — qu'il soit en Gherkin, Behat ou autre — ne devrait pas être vu comme une simple validation technique. C'est une **spécification exécutable**. Lorsqu'elle n'est pas maintenue, elle ne devient pas juste obsolète : elle devient une **dette qui ment**. Elle rassure à tort, tout en laissant la réalité du métier s'éloigner du code.

## Une vérité terrain, pas juste un filet de sécurité

Sur la plupart des projets, le produit évolue en continu (nouveaux workflows, règles de gestion, contraintes légales), et le code suit tant bien que mal au fil des tickets. Pourtant, les régressions métier finissent par passer. Pourquoi ? Parce qu'il manque souvent une **trace exécutable** de la règle.

Sans tests fonctionnels, la connaissance métier s'éparpille : une phrase dans un ticket Jira fermé, une discussion Slack oubliée, ou un "savoir tribal" dans la tête d'un développeur. Tout cela tient... jusqu'au jour où une refonte touche le mauvais module.

C'est ici que la distinction est cruciale :
*   **Les tests unitaires** sont indispensables pour le design du code, la validation des invariants et la boucle de feedback rapide.
*   **Les tests fonctionnels** (ou d'acceptation) valident la **promesse produit**. Ils matérialisent ce que l'application *doit* faire, indépendamment de la façon dont elle est codée.

Une spec exécutable capture cette promesse. Elle dit "ce qui doit être vrai" dans un langage compréhensible par l'équipe, et elle l'exécute réellement. Si elle casse, ce n'est pas juste un test qui échoue, c'est le contrat métier qui est rompu.

## Quand le "technique" cache du métier

Prenons un cas concret. Votre produit synchronise un catalogue vers une marketplace externe. Une décision tombe : "On change le format des images exportées pour standardiser". Sur le papier, c'est une tâche technique. Dans la réalité, c'est une règle métier déguisée : *ce partenaire accepte tel format, sous telles conditions*.

Sans spec exécutable, le changement est déployé. Quelques jours plus tard, le support s'affole : les produits ne s'affichent plus correctement chez le partenaire. Le diagnostic est flou, on fouille les logs.

Avec une spec exécutable, le contrat aurait été explicite dès le départ :

```gherkin
Feature: Export catalogue partenaire

  Scenario: Export d'un produit avec image compatible
    Given a product "SKU-123" with an image in "jpeg" format
    When I export the product to the partner
    Then the export payload should contain an image in an accepted format
    And the export should be accepted by the partner
```

Si la règle change (par exemple, le partenaire n'accepte le nouveau format que pour le canal "Web"), le scénario doit évoluer **en même temps** que le code. C'est ce synchronisme qui transforme le test en garde-fou. Il force à se poser la question : "Est-ce que je casse le contrat existant ?" avant même de merger.

## Cibler pour ne pas subir

L'erreur classique est de vouloir tout couvrir. Une suite de tests fonctionnels trop large devient lente, verbeuse et fragile (le fameux "flaky test"). Résultat : l'équipe finit par les ignorer.

La bonne approche est celle du **ciblage critique**. Demandez-vous : si cette fonctionnalité casse, est-ce que cela nous coûte de l'argent, de la confiance ou des données ? Si oui, elle mérite un scénario. Le reste peut souvent être couvert par des tests unitaires ou du monitoring.

Au final, la règle est simple : **peu de scénarios, mais vivants**.

Un test fonctionnel doit être traité comme du code de production. S'il est maintenu avec rigueur, il devient un fil conducteur pour les développeurs. S'il est laissé à l'abandon, supprimez-le avant qu'il ne commence à vous mentir.
