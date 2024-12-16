---
layout: post-fr
title: "Comprendre l'injection de dépendances"
date: 2024-12-16
category: fr
lang: fr
---

## Comprendre l'injection de dépendances
(Publié le 12 déc 2024 - [English version](/en/dependancy-injection))

Dans le monde du développement, l'utilisation de frameworks modernes est devenue courante.
Ces outils offrent des solutions pratiques et rapides pour créer des applications, en standardisant les approches de développement et en réduisant la complexité du code.
L'un de leurs atouts majeurs réside dans leur capacité à gérer efficacement les dépendances entre les différents composants, notamment grâce aux conteneurs d'injection de dépendances.
L'injection de dépendances, si courante aujourd'hui, repose sur des principes fondamentaux bien plus anciens que les frameworks eux-mêmes.

Pourquoi l'injection de dépendances est-elle essentielle ?
Quels problèmes résout-elle, et comment peut-elle améliorer notre code ?
Voyons cela ensemble dans un exemple concret.

### La problématique
Regardons le code suivant :

```php
<?php
class ContentManagement {
    public function __construct(private string $name) {}
    
    public function getContent(): string {
        $filename = '/var/contents/' . $this->name . '.txt';
        return file_exists($filename) ? file_get_contents($filename) : 'unknown';
    }
}

class Renderer {
    public function render(string $name): string {
        return sprintf('<p>%s</p>', htmlspecialchars(new ContentManagement($name)->getContent()));
    }
}

echo new Renderer()->render('foo');
```

Pour fonctionner, la classe `Renderer` a besoin de la classe `ContentManagement`, elle **dépend** donc de cette classe.
En soi, ce n'est pas un problème, surtout avec une exemple aussi simple, mais on constate qu'il y a une limitation : La classe `Renderer` ne va pas être facile à tester.
Il va falloir écrire des jeux de tests dans des fichiers, et comme le chemin d'accès à ces fichiers est fixé par la classe `ContentManager`, cela va poser des problèmes entre les fichiers de tests et les fichiers de contenu *réels*.

### Patchons la solution avec un passe-plat.

Pour résoudre le problème du chemin d'accès au système de fichiers, nous pourrions écrire ceci :
```php
<?php
class ContentManagement {
    public function __construct(private string $name, private ?string $path = null) {}
    
    public function getContent(): string {
        $filename = ($this->path ?? '/var/contents/') . $this->name . '.txt';
        return file_exists($filename) ? file_get_contents($filename) : 'unknown';
    }
}

class Renderer {
    public function render(string $name, ?string $path = null): string {
        return sprintf('<p>%s</p>', htmlspecialchars(new ContentManagement($name, $path)->getContent()));
    }
}

echo new Renderer()->render('foo', '/var/contents-test/');
```

Cette approche est souvent utilisée par les développeurs et les développeuses qui veulent aller vite : On vient ajouter une rustine (un "patch") pour régler un problème et on augmente l'entropie.
C'est la voie vers le chaos.
Car même si on semble avoir régler le problème de la testabilité, on a en fait renforcé la dépendance (le couplage) entre les classes car la classe `Renderer` dépend toujours de `ContentManagement`, et maintenant, elles sont liées entre elles par le support d'un système de fichiers.
Quand on analyse `Renderer`, on s'aperçoit d'ailleurs qu'il n'est qu'un passe-plat entre l'appelant et la classe `ContentManager` : On fait circuler la variable `$path` sans en faire quoi que ce soi.

Dites vous qu'à chaque fois que vous avez une variable **passe-plat**, c'est qu'il y a quelque chose que vous faites peut-être mal.

### Injection de la dépendance

On va donc dissocier la création de la classe `ContentManagement` et la classe `Renderer`, puis on va injecter le `ContentManagement` dans le `Renderer`.
On appelle cela, de l'[inversion de contrôle](https://fr.wikipedia.org/wiki/Inversion_de_contr%C3%B4le) (<abbr title="Inversion of Control">IoC</abbr>) : On déplace le contrôle de l'instantiation dans un autre composant du programme.
On parle aussi d'[injection de dépendance](https://fr.wikipedia.org/wiki/Injection_de_d%C3%A9pendances) (<abbr title="Dependancy Injection">DI</abbr>) qui est une technique particulière d'inversion de contrôle.

```php
<?php
class ContentManagement {
    public function __construct(private string $name, private ?string $path = null) {}
    
    public function getContent(): string {
        $filename = ($this->path ?? '/var/contents/') . $this->name . '.txt';
        return file_exists($filename) ? file_get_contents($filename) : 'unknown';
    }
}

class Renderer {
    public function __construct(private ContentManagement $contentManagement) {}
    public function render(): string {
        return sprintf('<p>%s</p>', htmlspecialchars($this->contentManagement->getContent()));
    }
}

$contentManagement = new ContentManagement('foo', '/var/contents-test/');

echo new Renderer($contentManagement)->render();
```

Dans l'état actuel, `Renderer` dépend toujours de `ContentManagement`, mais il n'y a plus de passe-plat.
On vient configurer `ContentManagement`, et `Renderer` devient agnostique de la façon de gérer le contenu.
D'ailleurs, `Renderer` n'ayant plus la responsabilité de créer une instance de `ContentManagement`, elle peut se concentrer sur le rendu, c'est le [principe de la responsabilité unique](https://fr.wikipedia.org/wiki/Principe_de_responsabilit%C3%A9_unique) (Single Responsability) 

À partir de maintenant, on peut changer le comportement du code, en faisant de l'héritage comme ceci : 
```php
<?php
class SimpleContentManagement extends ContentManagement {
    public function __construct(private string $content) {}
    public function getContent(): string {
        return $this->content;
    }
}

$contentManagement = new SimpleContentManagement('The <<content>>');

echo new Renderer($contentManagement)->render();
```

Faire de l'injection de dépendances implique que l'on ne crée plus d'instance directement, et donc il ne devrait plus y avoir de `new` dans nos classes. Nous verrons plus tard, que ce n'est pas si automatique que ça.

Dans l'exemple ci-dessus, l'instantiation est faite dans le code applicatif, juste avant d'appeler le rendu.
Seulement, il arrive souvent que ce travail de création soit très complexe, alors on va utiliser une autre classe qui se chargera de ça : les **factories**.

Voici un exemple :
```php
<?php
final class ContentManagementFactory {
    public function makeFor(string $name): ContentManagement {
        return new ApiContentManagement(
            endpoint: 'https://api.acme.com/content/' . $name,
            apiToken: 'KxxiLKU47555Lkks_124888324',
            timeout: 10
        );
    }
}

$contentManagement = new ContentManagementFactory()->makeFor('foo');

echo new Renderer($contentManagement)->render();
```

Et là, vous vous dites : "Attendez, on vient de réintroduire une classe avec un couplage fort !".
En effet, seulement, dans cette architecture, l'utilisation de cette factory n'est pas obligatoire; il est toujours possible de créer une instance de classe pour des tests par exemple.

Les factories sont des facilitatrices : elles rassemblent le code fastidieux de création d'instances qui peuvent aussi nécessiter que l'on injecte des sous-dépendances, et cela sur de nombreux niveaux.

Par contre, les factories sont souvent spécialisées pour la création de classes d'un domaine précis, mais se limitent **toujours** à l'instanciation et ne doivent jamais participer à l'exécution d'un processus.

À un niveau plus global, on retrouvera le conteneur d'injection de dépendances.
Celui-ci se charge de la création de la plupart des objets, y compris des fatories, en s'appuyant sur une configuration.
Dans l'exemple avec `ApiContentManagement`, les éléments de configuration comme l'URL ou le token d'API seront dans des fichiers de configuration, et le conteneur d'injection utilisera cette configuration pour créer les objets.

### Injection de comportements

En fait, quand on fait de l'injection de dépendances, cela ne veut pas dire que l'on arrête de dépendre d'autres classes, mais que l'on réduit le couplage entre les classes en permettant de substituer le comportement d'une dépendance par un autre.

D'ailleurs, cette substitution par de l'héritage n'est pas non plus une très bonne approche, et il est préférable d'utiliser des interfaces pour cela. Ce qui donnerait :

```php
<?php
interface ContentResolver {
    public function getContent(): string;
}

final class SimpleContentManagement implements ContentResolver {
    public function __construct(private string $content) {}
    public function getContent(): string {
        return $this->content;
    }
}

final class Renderer {
    public function __construct(private ContentResolver $contentResolver) {}
    public function render(): string {
        return sprintf('<p>%s</p>', htmlspecialchars($this->contentResolver->getContent()));
    }
}

$contentManagement = new SimpleContentManagement('The <<content>>');

echo new Renderer($contentManagement)->render();
```

Vous noterez au passage, que la classe concrète est "final" afin de mettre l'accent sur le fait que l'on ne souhaite pas promouvoir l'héritage dans cette architecture objet.

Maintenant, nous n'avons plus une dépendance à une classe, mais à un comportement de classe, et c'est ce comportement que l'on injecte.
Au final, dans vos classes, il ne devrait plus y avoir de `new`, et dans les paramètres de vos constructeurs, on ne devrait avoir que des interfaces pour agréger des comportements.

### Tout ne s'injecte pas

En fait, quand on affirme "il ne devrait plus y avoir de `new`", ce n'est pas tout à fait vrai, car il existe de nombreux cas où l'injection n'est pas pertinente.

Par exemple, il y a l'utilisation de *[value object](https://acairns.co.uk/posts/primitive-obsession)* (VO), de *[data transfert object](https://fr.wikipedia.org/wiki/Objet_de_transfert_de_donn%C3%A9es)* (DTO), d'événements (Event), des exceptions et finalement tous les objets transitoires ou à cycle de vie court.

L'un des grands intérêts de l'injection de dépendances, c'est de pouvoir substituer des comportements, ce qui les rend beaucoup plus testable.

Il existe toutefois des compositions de classes qui ont été conçues ainsi pour permettre de découper le code en plus petits morceaux afin de rendre l'ensemble moins lourds à lire et à maintenir.
Quand vous avez une très longue méthode, vous allez la refactorer en un ensemble d'appels de sous méthodes privées pour la rendre plus digeste, et bien c'est la même chose avec les classes : 
vous subdivisez votre longue classe en plusieurs plus petites classes dans lesquelles vous déplacez votre code. Ces classes n'ont pas d'intérêts en dehors de l'usage de la classe qui les agrège et l'injection de dépendances n'y apportera aucune valeur.
D'ailleurs, vous allez peut-être même ajouter le tag `@internal` ou un analyseur statique comme [Deptrac](https://qossmic.github.io/deptrac/) pour limiter les interactions avec vos classes internes.
Un jour, peut-être, PHP offrira la possibilité d'adapter la [visibilité des classes](https://wiki.php.net/rfc/namespace-visibility) ou introduira les [classes amies](https://wiki.php.net/rfc/friend-classes).

Vous devez trouvez un équilibre.
Si l'injection apporte de la flexibilité pour faire évoluer votre code, un usage excessif ou inadapté peut le rendre complexe et difficile à comprendre.

Restez pragmatique !