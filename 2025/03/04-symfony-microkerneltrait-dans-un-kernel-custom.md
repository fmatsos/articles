---
title: "Symfony MicroKernelTrait dans un Kernel custom"
description: "Exploration du Symfony MicroKernelTrait lors de révisions pour la certification Symfony, avec une architecture proche du DDD et intégrant ADR."
pubDate: 2025-03-04
lang: fr
published: true
draft: false
tags: [blog,symfony]
---

Dans le cadre de mes révisions pour la certification Symfony, j'ai créé un projet afin d'explorer les différentes facettes
du framework que je ne connaissais pas ou peu. Ce projet, suivant une architecture proche du DDD et intégrant ADR, m'a
conduit à modifier fortement la configuration de base de Symfony, y compris le Kernel.

<!-- excerpt -->

Pour rappel, voici le Kernel fourni par défaut :

```php
<?php

namespace App;

```

