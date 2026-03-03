---
layout: post-fr
title: "Pourquoi utiliser des closures statiques ?"
date: 2026-03-02
category: fr
lang: fr
---

## Pourquoi utiliser des closures statiques ?
(Publié le 2 mars 2026)

En PHP, Nous utilisons de plus en plus les [closures](https://www.php.net/closure), dans l'injection de dépendance, les middlewares, les callbacks de collections, et aussi dans le traitement de l'asynchronisme comme j'ai pu l'écrire dans mon article "[La programmation asynchrone en PHP](/fr/asynchrone)".

Cependant, elles ont un comportement qui peut surprendre : toute closure créée dans une méthode d'instance embarque automatiquement une référence à l'objet courant, même si elle n'utilise **pas** `$this`.
Ce comportement peut avoir des conséquences inattendues sur la durée de vie des objets et générer des fuites de mémoire si on n'y prend pas garde.

Pour comprendre pourquoi, il faut d'abord comprendre comment PHP gère la mémoire.
Contrairement à des langages comme Java qui s'appuie sur un garbage collector pour libérer la mémoire de manière différée, PHP utilise le **comptage de références** (Bon, en vrai, PHP dispose d'un [garbage collector](https://www.php.net/manual/fr/features.gc.php) pour les références cycliques, mais c'est un autre sujet).

Quand on affecte une variable, il faut stocker en mémoire son contenu, et quand on n'utilise plus la variable, on peut libérer la mémoire. Quand vous écrivez ceci :
```php
<?php
$a = 'Bonjour';
$b = $a;
```
PHP ne va pas créer un deuxième espace mémoire pour la variable `$b`, mais il va juste indiquer que celle-ci point vers le même espace mémoire que `$a`.
Si ensuite, vous affectez une nouvelle valeur à `$a`, disons `"Hello"`, un nouvel espace mémoire est alloué et `$a` pointe dessus pendant que `$b` continu de pointer vers l'ancien espace.
Maintenant, si vous affectez `NULL` à `$b`, l'espace mémoire qui contenait `"Bonjour"` n'est plus référencé par une variable, et il peut donc être libéré.
Pour faire cela, PHP maintient un compteur de nombre de références, et si ce compteur tombe à "0", l'espace est libéré.

### Le cycle de vie d'un objet

Avec un objet, quand le compteur de référence descend à 0, avant de libérer la mémoire, si la classe défini une méthode [`__destruct`](https://www.php.net/manual/fr/language.oop5.decon.php#language.oop5.decon.destructor), elle est appelée :

```php
<?php
class Foo {
    public function __construct() {
        echo "Construct\n";
    }

    public function __destruct() {
        echo "Destruct\n";
    }
}

new Foo();
echo "Fin\n";
```

```
Construct
Destruct
Fin
```

L'objet n'est affecté à aucune variable : son compteur tombe immédiatement à zéro après l'appel du constructeur et `__destruct` est invoqué juste après.

En revanche, si l'on affecte l'objet à une variable, la destruction est différée :

```php
<?php
$foo = new Foo();
echo "Fin\n";
```

```
Construct
Fin
Destruct
```

Tant que `$foo` pointe vers l'objet, le compteur reste à un. La destruction n'intervient qu'en fin de script, une fois que toutes les variables sont libérées. Pour forcer la destruction avant, il suffit de libérer la variable explicitement par l'affectation d'une nouvelle valeur ou l'appel de `unset()` :

```php
<?php
$foo = new Foo();
echo "Avant libération\n";
$foo = null;
echo "Après libération\n";
```

```
Construct
Avant libération
Destruct
Après libération
```

### Une closure retient l'objet en vie

Voyons ce qui se passe avec une classe `Bar` qui défini une méthode `getCallback()`, elle-même renvoyant une closure lisant la propriété `$this->id` :

```php
<?php
class Bar {
    public function __construct(private string $id) {
        echo "Construct\n";
    }

    public function __destruct() {
        echo "Destruct\n";
    }

    public function getCallback(): Closure {
        return function(): string {
            return $this->id;
        };
    }
}

$bar = new Bar('foo');
$getId = $bar->getCallback();

echo "Avant libération de l'objet\n";
$bar = null;
echo "Après libération de l'objet\n";

echo $getId() . "\n";

echo "Fin\n";
```

```
Construct
Avant libération de l'objet
Après libération de l'objet
foo
Fin
Destruct
```

L'objet n'est pas détruit quand on affecte `null` à la variable `$bar` car la closure accède à `$this->id`, elle constitue donc elle-même une référence à l'objet. Le compteur ne tombe pas à zéro tant que la closure existe, c'est à dire jusqu'à la fin du script. Si on avait ré-affecté `$getId`, l'appel à `__destruct` serait intervenu avant la fin, car en même temps qu'on libérait la variable, on libérait la référence à `$this`.

### Même sans $this, l'objet reste en vie

Que se passe-t-il si on n'utilise pas `$this` dans la closure ?

```php
<?php
class Bar {
    public function __construct() {
        echo "Construct\n";
    }

    public function __destruct() {
        echo "Destruct\n";
    }

    public function getCallback(): Closure {
        return function(): void {};
    }
}

$bar = new Bar();
$callback = $bar->getCallback();

echo "Avant libération de l'objet\n";
$bar = null;
echo "Après libération de l'objet\n";
$callback = null;
echo "Fin\n";
```

```
Construct
Avant libération de l'objet
Après libération de l'objet
Destruct
Fin
```

L'objet est toujours retenu en vie, car même si on n'utilise pas `$this`, la closure référence toujours l'objet, car PHP lie automatiquement `$this` à toute closure créée dans une méthode d'instance, qu'elle l'utilise ou non, qu'elle soit vide ou non.
La closure transporte donc toujours une référence à l'objet, invisible à la lecture du code.

Bien évidemment, si la closure est créée au sein d'une méthode statique, il n'y a pas de référence à `$this` et donc la destruction intervient au moment de la libération de la variable : 
```php
<?php
class Bar {
    public function __construct() {
        echo "Construct\n";
    }

    public function __destruct() {
        echo "Destruct\n";
    }

    public static function getCallback(): Closure {
        return function(): void {};
    }
}

$bar = new Bar();
$closure = $bar::getCallback();

echo "Avant libération de l'objet\n";
$bar = null;
echo "Fin\n";
```

```
Construct
Avant libération de l'objet
Destruct
Fin
```

### La closure statique

Le mot-clé `static` appliqué à une closure lui interdit explicitement d'être liée à `$this`.
PHP ne stocke alors plus de référence à l'objet, même implicitement.

```php
// ...
public function getCallback(): Closure {
    return static function(): void {};
}
// ...
```

```
Construct
Avant libération de l'objet
Destruct
Fin
```

Et si nous avons besoin de récupérer la valeur d'une propriété dans la closure, on peut utiliser `use` comme ceci :

```php
// ...
public function getCallback(): Closure {
    $id = $this->id;
    return static function() use ($id): string {
        return $id;
    };
}
...
```

Cette fois, PHP va détruire l'objet juste après la libération de la variable, car la closure n'en conserve aucune référence.

Si vous tentez d'utiliser `$this` dans une closure statique, PHP lèvera une erreur :

```php
return static function(): string {
    return $this->id; // Error: Using $this when not in object context
};
```

Le moteur PHP vous protège ainsi d'une capture accidentelle.

### Les short closures

Les short closures (`fn() =>`) offrent une syntaxe plus concise et capturent les variables de la portée englobante **automatiquement**, sans `use`.
Mais elles partagent le même comportement que les closures classiques vis-à-vis de `$this` :

```php
public function getCallback(): Closure {
    return fn(): string => $this->id;
}
```

Ici, `$this` est capturé implicitement, tout comme avec une closure ordinaire. L'objet reste en vie tant que la closure existe.

Le mot-clé `static` s'applique aussi aux short closures. Les variables de la portée englobante restent capturées automatiquement, mais `$this` ne l'est plus :

```php
public function getCallback(): Closure {
    return static fn(): string => $this->id; // Error: Using $this when not in object context
}
```

Pour passer la valeur sans capturer l'objet, il suffit de l'extraire avant :

```php
public function getCallback(): Closure {
    $id = $this->id;
    return static fn(): string => $id;
}
```

La variable `$id` est capturée par valeur, `$this` n'est plus impliqué, et l'objet peut être libéré dès que sa référence explicite disparaît.

### Ce que PHP 8.6 va changer

La [RFC Closure Optimizations](https://wiki.php.net/rfc/closure-optimizations), en vote pour PHP 8.6 au moment de l'écriture de cet article, s'attaque précisément à ce comportement.
Elle introduit une **inférence automatique** : si une closure ne fait aucun usage de `$this`, PHP la rendra statique de lui-même, sans que le développeur ait à l'écrire.

Notre exemple de closure avec `use ($id)` ou la short closure `fn(): string => $id` ne capturerait donc plus l'objet implicitement une fois cette RFC adoptée.

La RFC va encore plus loin avec une deuxième optimisation : les closures statiques qui ne capturent aucune variable (ni `use`, ni portée englobante) sont **mises en cache** et réutilisées entre les appels, évitant ainsi leur ré-instanciation à chaque fois.

Ces deux optimisations sont transparentes pour le code existant, à une exception près : [`ReflectionFunction::getClosureThis()`](https://www.php.net/manual/fr/reflectionfunctionabstract.getclosurethis.php) retournera `null` pour les closures désormais inférées comme statiques, ce qui pourrait introduire un changement de comportement pour du code existant (Breaking Change).

### Soyez explicite

En règle générale, lorsqu'une closure, ou une short closure, n'a pas besoin de `$this`, il est préférable de la déclarer `static`.
Cela rend l'intention **explicite**, prévient les captures involontaires et permet à l'objet d'être détruit dès que sa dernière référence explicite disparaît.

Avec PHP 8.6, ce comportement sûr deviendra le comportement par **défaut**, mais déclarer `static` reste utile pour documenter l'intention et garantir la compatibilité avec les versions antérieures.
