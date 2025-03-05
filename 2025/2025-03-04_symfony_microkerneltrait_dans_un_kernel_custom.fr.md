---
title: "Symfony MicroKernelTrait dans un Kernel custom"
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

use Symfony\Bundle\FrameworkBundle\Kernel\MicroKernelTrait;
use Symfony\Component\HttpKernel\Kernel as BaseKernel;

class Kernel extends BaseKernel
{
    use MicroKernelTrait;
}
```

On ne peut faire plus minimaliste : nous héritons du Kernel par défaut (que nous renommons `BaseKernel`), qui inclut toute
la logique permettant de faire la _glue_ du framework et d'assurer son bon fonctionnement. Pour les méthodes abstraites de
celui-ci, nous utilisons le `MicroKernelTrait`, qui fournit le corps des méthodes et facilite la configuration de Symfony.

Maintenant, voici ma problématique : dans le projet, je définis certaines configurations, comme mes Workflows, dans le
namespace `App\Infrastructure\Symfony\Config`. Je dois donc indiquer au Kernel où et comment charger ces fichiers.

Par défaut, je pars sur cette implémentation :

```php
<?php

namespace App;

use Symfony\Bundle\FrameworkBundle\Kernel\MicroKernelTrait;
use Symfony\Component\Config\Loader\LoaderInterface;
use Symfony\Component\Finder\Finder;
use Symfony\Component\HttpKernel\Kernel as BaseKernel;

class Kernel extends BaseKernel
{
    use MicroKernelTrait;

    public function registerContainerConfiguration(LoaderInterface $loader): void
    {
        $files = new Finder()
            ->files()
            ->in($this->getProjectDir().'/src/Infrastructure/Symfony/Config')
            ->name('*.php');

        foreach ($files as $file) {
            $loader->load($file->getRealPath());
        }
    }
}
```

Cependant, j'ai rencontré une erreur lorsque j'exécutais `bin/console` :

```bash
Attempted to load class "BaseCommand" from namespace "Composer\Command".
Did you forget a "use" statement for another namespace?
```

Je vous épargne les détails du débogage pour trouver la source du problème. Ce qu'il faut savoir, c'est que la méthode
`registerContainerConfiguration` est déjà définie dans `MicroKernelTrait`. C'est elle qui se charge de configurer le
container. Étant donné que mon implémentation vient surcharger celle définie dans le trait, je court-circuite toutes les
étapes permettant de charger les configurations du container, rendant ainsi ma console inopérante.

Il faut donc corriger le code précédent pour que, dans mon implémentation de registerContainerConfiguration, je puisse
appeler l'implémentation disponible dans le trait. Mais comme nous sommes dans un trait, nous ne pouvons pas utiliser
`parent::registerContainerConfiguration`.

La solution consiste à utiliser un alias pour la méthode du trait :

```php
<?php

namespace App;

use Symfony\Bundle\FrameworkBundle\Kernel\MicroKernelTrait;
use Symfony\Component\Config\Loader\LoaderInterface;
use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\Finder\Finder;
use Symfony\Component\HttpKernel\Kernel as BaseKernel;

class Kernel extends BaseKernel
{
    use MicroKernelTrait {
        registerContainerConfiguration as microKernelRegisterContainerConfiguration;
    }

    public function registerContainerConfiguration(LoaderInterface $loader): void
    {
        $this->microKernelRegisterContainerConfiguration($loader);

        $files = new Finder()
            ->files()
            ->in($this->getProjectDir().'/src/Infrastructure/Symfony/Config')
            ->name('*.php');

        foreach ($files as $file) {
            $loader->load($file->getRealPath());
        }
    }
}
```

Et voilà !

L'utilisation des alias sur les traits est assez peu fréquente, car elle répond souvent à des besoins particuliers. Je
peux maintenant avoir ma propre implémentation tout en conservant celle fournie par Symfony, qui se charge pour moi du
chargement de toutes les configurations liées au framework.

*[DDD]: Domain Driven Design
*[ADR]: Action-Domain-Responder
