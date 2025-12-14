---
title: "Symfony MicroKernelTrait dans un Kernel custom"
description: "Exploration du Symfony MicroKernelTrait lors de révisions pour la certification Symfony, avec une architecture proche du DDD et intégrant ADR."
pubDate: 2025-03-04
lang: fr
published: true
draft: false
tags: [blog,symfony]
---

Dans le cadre de mes révisions pour la certification Symfony, j'ai créé un projet afin d'explorer les différentes facettes du framework que je ne connaissais pas ou peu. Ce projet, suivant une architecture proche du DDD et intégrant ADR, m'a conduit à modifier fortement la configuration de base de Symfony, y compris le Kernel.

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

    private const CONFIG_EXTS = '.{php,xml,yaml,yml}';

    public function getCacheDir(): string
    {
        return $this->getProjectDir() . '/var/cache/' . $this->environment;
    }

    public function getLogDir(): string
    {
        return $this->getProjectDir() . '/var/log';
    }

    public function registerBundles(): iterable
    {
        $contents = require $this->getProjectDir() . '/config/bundles.php';
        foreach ($contents as $class => $envs) {
            if ($envs[$this->environment] ?? $envs['all'] ?? false) {
                yield new $class();
            }
        }
    }

    protected function configureContainer(ContainerBuilder $container, LoaderInterface $loader): void
    {
        $container->addResource(new FileResource($this->getProjectDir() . '/config/bundles.php'));
        $container->setParameter('container.dumper.inline_class_loader',
            \PHP_VERSION_ID < 70400 || $this->debug);
        $confDir = $this->getProjectDir() . '/config';

        $loader->load($confDir . '/services' . self::CONFIG_EXTS, 'glob');
        $loader->load($confDir . '/services_' . $this->environment . self::CONFIG_EXTS, 'glob');
    }

    protected function configureRoutes(RouteCollectionBuilder $routes): void
    {
        $confDir = $this->getProjectDir() . '/config';

        $routes->import($confDir . '/routes' . self::CONFIG_EXTS, '/', 'glob');
        $routes->import($confDir . '/routes_' . $this->environment . self::CONFIG_EXTS, '/', 'glob');
    }
}
```

Ce Kernel utilise le trait `MicroKernelTrait` qui fournit une implémentation minimaliste de l'interface `HttpKernelInterface`. C'est un excellent point de départ, mais dans le contexte d'une architecture DDD, j'ai eu besoin de l'adapter.

## Besoins spécifiques du projet

Mon objectif était de créer une architecture où :
- Les bundles seraient organisés par domaine métier
- Les services seraient chargés différemment selon le contexte
- Les routes seraient namespaced par domaine
- Le conteneur de services serait plus granulaire

## Modifications du Kernel

Voici comment j'ai personnalisé le Kernel :

```php
<?php

namespace App;

use Symfony\Bundle\FrameworkBundle\Kernel\MicroKernelTrait;
use Symfony\Component\Config\Loader\LoaderInterface;
use Symfony\Component\Config\Resource\FileResource;
use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\HttpKernel\Kernel as BaseKernel;
use Symfony\Component\Routing\RouteCollectionBuilder;

class Kernel extends BaseKernel
{
    use MicroKernelTrait;

    private const CONFIG_EXTS = '.{php,xml,yaml,yml}';

    public function getCacheDir(): string
    {
        return $this->getProjectDir() . '/var/cache/' . $this->environment;
    }

    public function getLogDir(): string
    {
        return $this->getProjectDir() . '/var/log';
    }

    public function registerBundles(): iterable
    {
        $contents = require $this->getProjectDir() . '/config/bundles.php';
        foreach ($contents as $class => $envs) {
            if ($envs[$this->environment] ?? $envs['all'] ?? false) {
                yield new $class();
            }
        }
    }

    protected function configureContainer(ContainerBuilder $container, LoaderInterface $loader): void
    {
        $container->addResource(new FileResource($this->getProjectDir() . '/config/bundles.php'));
        $container->setParameter('container.dumper.inline_class_loader',
            \PHP_VERSION_ID < 70400 || $this->debug);
        
        $confDir = $this->getProjectDir() . '/config';

        // Load global services
        $loader->load($confDir . '/services' . self::CONFIG_EXTS, 'glob');
        
        // Load domain-specific services
        $domainsDir = $this->getProjectDir() . '/src/Domain';
        if (is_dir($domainsDir)) {
            foreach (scandir($domainsDir) as $domain) {
                if ($domain !== '.' && $domain !== '..' && is_dir("$domainsDir/$domain")) {
                    $servicesFile = "$domainsDir/$domain/config/services.yaml";
                    if (file_exists($servicesFile)) {
                        $loader->load($servicesFile);
                    }
                }
            }
        }
        
        // Load environment-specific services
        $loader->load($confDir . '/services_' . $this->environment . self::CONFIG_EXTS, 'glob');
    }

    protected function configureRoutes(RouteCollectionBuilder $routes): void
    {
        $confDir = $this->getProjectDir() . '/config';

        // Load global routes
        $routes->import($confDir . '/routes' . self::CONFIG_EXTS, '/', 'glob');
        
        // Load domain-specific routes
        $domainsDir = $this->getProjectDir() . '/src/Domain';
        if (is_dir($domainsDir)) {
            foreach (scandir($domainsDir) as $domain) {
                if ($domain !== '.' && $domain !== '..' && is_dir("$domainsDir/$domain")) {
                    $routesFile = "$domainsDir/$domain/config/routes.yaml";
                    if (file_exists($routesFile)) {
                        $routes->import($routesFile, "/$domain", 'yaml');
                    }
                }
            }
        }
        
        // Load environment-specific routes
        $routes->import($confDir . '/routes_' . $this->environment . self::CONFIG_EXTS, '/', 'glob');
    }
}
```

## Points clés

1. **Chargement dynamique des services par domaine** : Le Kernel scanne le répertoire `src/Domain` et charge automatiquement les fichiers `services.yaml` de chaque domaine.

2. **Préfixage des routes par domaine** : Les routes de chaque domaine sont préfixées automatiquement, créant une séparation claire.

3. **Maintien de la compatibilité** : Le chargement global des services et routes reste intact, assurant que le code existant fonctionne toujours.

Cette approche offre une grande flexibilité tout en maintenant la simplicité du `MicroKernelTrait`. Elle permet à chaque domaine d'être autonome en termes de configuration, facilitant ainsi la maintenabilité du projet.
