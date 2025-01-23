---
layout: post-fr
title: "Arrêtez d'utiliser les pseudo-types"
date: 2025-01-20
category: fr
lang: fr
---

# Arrêtez d'utiliser les pseudo-types
(Publié le 20 jan 2025 - [English version](/en/stop-using-pseudo-types))

En 2011, pour la sortie de [PHP 5.4](https://www.php.net/ChangeLog-5.php#PHP_5_4), le pseudo type `callable` a été introduit par la [RFC: Callable](https://wiki.php.net/rfc/callable), et en 2016, un autre pseudo type, `iterable`, a été introduit en [PHP 7.1.0](https://www.php.net/ChangeLog-7.php#7.1.0) par la [RFC: Iterable](https://wiki.php.net/rfc/iterable).

Les pseudo-types ne sont pas des vrais types comme `string` ou `array`.
Ce sont des ensembles de types avec une logique de validation particulière.
Voyons le pseudo-type `callable` pour mieux comprendre.

## Le pseudo-type `callable`

En PHP 4, la fonction [`call_user_func()`](https://www.php.net/call_user_func) a été introduite pour permettre d'appeler une fonction à partir de son nom sous forme d'une chaîne de caractères.
Il y avait également une fonction `call_user_method()`, mais elle a été rapidement dépréciée en PHP 4.1 et supprimée en PHP 7.0, car `call_user_func()` acceptait également un tableau où le premier élément prenait un objet et le second le nom de la méthode.
```php
<?php
$format = [new Datetime('2025-01-30'), 'format'];
echo call_user_func($format, 'd/m/Y');
// Résultat : 30/01/2025
```
[Try this code](https://3v4l.org/Fi9Ma)

La version PHP 4.0.6 a introduit la fonction [`is_callable()`](https://www.php.net/is_callable), permettant de vérifier si un paramètre est appelable."
En effet, une chaîne ou un tableau peuvent être appelables, mais seulement dans certaines conditions.
```php
<?php
$create = 'Datetime::createFromFormat';
$invoke = [new stdClass, '__invoke'];

var_dump(is_callable($create), is_callable($invoke));
// Résultat:
// bool(true)
// bool(false)
```
[Try this code](https://3v4l.org/K5PMD)

Je vous suggère la lecture de l'article de mon ami Damien Seguy : [How to call a method in PHP](https://www.exakat.io/en/call-a-method-in-php/).

Pour qu'une chaîne soit appelable, il faut qu'elle contienne le nom d'une fonction ou d'une méthode existante.
Ainsi, `trim` est appelable, mais `unknown` ne le sera pas si vous n'avez pas de fonction portant ce nom.

Attention, certains noms ne seront pas appelables, tels que `isset`, `empty`, `echo` ou `include`.
`exit` et `die` n'étaient pas appelable avant PHP 8.4, mais elles le sont désormais : [PHP RFC: Transform exit() from a language construct into a standard function](https://wiki.php.net/rfc/exit-as-function)

Petite lecture intéressante : [A Look At PHP’s isset()](https://medium.com/@liamhammett/a-look-at-phps-isset-df64df7158ab).

Bref, au moment de l'exécution, on ne peut pas savoir si une chaîne est appelable ou non.

C'est en mars 2012, avec la sortie de [PHP 5.4](https://www.php.net/releases/5_4_0.php), que la logique de la fonction `is_callable()` a été déplacée pour introduire le pseudo-type `callable`.
C'est pour cela que l'on parle de pseudo-type : `callable` n'est pas un type, c'est une union des types `Closure`, `string` et `array` avec une logique pour vérifier, lors de l'exécution, qu'un paramètre est bien appelable.
À la compilation, à moins que ce soit une `Closure`, on ne peut pas savoir si une fonction ou une classe existera lors de sa validation.
C'est d'ailleurs pour cette raison qu'il n'est pas possible d'utiliser `callable` pour typer les propriétés d'une classe : [PHP RFC: Typed Properties 2.0, Supported Types](https://wiki.php.net/rfc/typed_properties_v2#supported_types).

## Le pseudo-type (ou pas) `iterable`

En PHP 7.1, le pseudo-type `iterable` a été introduit.
Il permettait de vérifier que l'on pouvait itérer dessus en validant qu'il implémentait le type `array` ou l'interface `Traversable`.
C'est avec la sortie de [PHP 8.2](https://www.php.net/ChangeLog-8.php#PHP_8_2) que `iterable` a perdu son statut de pseudo-type pour devenir une union de types `Traversable|array`.

Pour mieux comprendre cette différence, regardez le comportement de ce code en PHP 8.1 et en 8.2 : 
```php
<?php

function foo(): iterable {
    return [];
}

function bar(): iterable|bool {
    return [];
}

echo (new ReflectionFunction('foo'))->getReturnType()->__toString();
echo ", ";
echo (new ReflectionFunction('bar'))->getReturnType()->__toString();
```
[Try this code](https://3v4l.org/JGM9D)

- PHP 8.1.0 : `iterable, iterable|bool`
- PHP 8.2.0 : `iterable, Traversable|array|bool`

Ce changement interne a été introduit par cette pull request: [Convert iterable into an internal alias for Traversable|array](https://github.com/php/php-src/pull/7309).

Seulement, l'interface `Traversable` est un peu particulière et pourrait presque s'assimiler à un pseudo-type également, car c'est une interface qui n'est pas implémentable mais qui est étendue par les interfaces Iterator et IteratorAggregate.
En fait, le type `iterable` aurait dû être une union de types : `Iterator|IteratorAggregate|array`

## Typage de données avec `iterable`

Le type `iterable` ne devrait pas être utilisé pour typer un retour de fonction ou de méthode, car cela peut générer de la confusion si la donnée est mal utilisée.
Il n'est pas rare de voir des développeurs et des développeuses peu rigoureux et rigoureuses qui ne se limitent pas à itérer sur le résultat d'une fonction, mais qui veulent manipuler celui-ci.
En constatant, par exemple, que la donnée retournée est un tableau, illes peuvent se laisser tenter par l'utilisation d'un `count` afin de savoir s'il y a des valeurs pour afficher un texte alternatif.
Seulement, ce type pourrait évoluer et retourner, plus tard, un itérateur. 
Exemple:
```php
<?php
class A {
    public function get(): iterable {
        return [17];
    }
}

class B extends A {
    public function get(): iterable {
        yield 42;
    }
}

echo (new A)->get()[0]; // 17
echo (new B)->get()[0]; // Fatal error
```
[Try this code](https://3v4l.org/l9NY2)

Alors, oui, je vous l'accorde, c'est une erreur grossière, et l'usage d'un bon analyseur statique comme [PHPStan](https://phpstan.org/) ou [Exakat](https://www.exakat.io/en/) ne doit pas permettre ce genre de chose.
Seulement, dites-vous que les développeurs et les développeuses font des erreurs, surtout quand illes débutent.

Pour gérer correctement cette variation de résultat, on doit donc tester si la donnée retournée est un tableau (`is_array`) ou un itérateur (`instanceof Iterator`), ou alors utiliser une fonction comme [`iterator_to_array()`](https://www.php.net/iterator_to_array), comme ceci:

```php
<?php
// ...
echo iterator_to_array((new B)->get())[0];
```
[Try this code](https://3v4l.org/LVOQM)

Cette solution fonctionne, mais peut avoir des conséquences sur la performance de votre code, surtout si l'usage d'un [Generator](https://www.php.net/generator) a été justement prévu pour pallier l'occupation mémoire d'un résultat de grande taille.

Une bonne approche [Defensive programming](https://en.wikipedia.org/wiki/Defensive_programming) consiste donc à limiter les possibilités d'utilisation, en proposant soit un tableau, soit un itérateur.
Personnellement, j'ai tendance à privilégier les itérateurs par l'intermédiaire des générateurs.

Quant au typage de paramètres dans une fonction ou une méthode, utiliser `iterable` indiquera que votre implémentation va seulement itérer sur votre donnée, et vous pouvez donc passer un tableau ou un itérateur indifféremment.
Cela va donc améliorer la Developer eXperience (DX).

## Typage de données avec `callable`

Autant l'usage de `iterable` est discutable, autant `callable` l'est moins.
Déjà, parce qu'il n'est pas possible de l'utiliser pour typer une propriété.
Ensuite, parce qu'il autorise les développeuses et les développeurs à définir l'appel sous la forme d'une chaîne, et cette forme pose énormément de problèmes quand on fait de l'analyse statique ou juste quand on fait une recherche sur l'usage d'une méthode, par exemple.

Même si les IDE, aujourd'hui, font beaucoup de progrès, il peut être difficile pour eux de trouver les usages d'une fonction quand celle-ci est définie dans une variable sous la forme d'une chaîne de caractères.

De plus, avec les *[first class callables](https://www.php.net/manual/en/functions.first_class_callable_syntax.php)*, vous n'avez plus d'excuse pour définir vos fonctions de rappel sous cette forme :
```php
<?php
$data = array_map(trim(...), [' x', 'z  ']);
```
[Try this code](https://3v4l.org/gpCav)

Il est donc préférable d'utiliser la [classe `Closure`](https://www.php.net/closure) quand vous souhaitez définir des fonctions de rappel.
Si vous avez une version antérieure à PHP 8.1, et que vous ne pouvez donc pas utiliser les first class callables, je vous suggère d'utiliser `Closure::fromCallable()`:
```php
<?php
$data = array_map(
    Closure::fromCallable('trim'),
    [' x', 'z  ']
);
```
[Try this code](https://3v4l.org/7tiOX)

### Typage des propriétés avec des Closures

Un point de détail quand on utilise des propriétés du type `Closure`, c'est qu'on ne peut pas appeler la closure de cette façon : 

```php
<?php

class A {
    public function __construct(
        private Closure $callback
    ) {}
    
    public function show(): void {
        $this->callback();
    }
}
```
[Try this code](https://3v4l.org/AKZ5d)

Cela génère une erreur `Fatal error: Uncaught Error: Call to undefined method A::callback()`.
Ce qui est plutôt logique, car il devient difficile de faire la différence entre un appel de méthode et une propriété invocable.

Il faut donc appeler la propriété de cette façon : `($this->callback)();` ([Try this code](https://3v4l.org/b6Rf3)).

Vous pouvez aussi faire comme ceci : `call_user_func($this->callback);` ([Try this code](https://3v4l.org/HT8QR)).

Mais il y a une autre méthode intéressante : la classe `Closure` dispose d'une méthode `__invoke`. Vous pouvez donc écrire : `$this->callback->__invoke();` ([Try this code](https://3v4l.org/iZ0VR)).
Je trouve cette écriture séduisante car elle est très explicite : "On invoque la closure".

## Conclusion

Les pseudo-types comme `callable` et `iterable` peuvent sembler pratiques à première vue, mais ils introduisent des ambiguïtés et rendent le code plus difficile à analyser.

Privilégier l'usage de types clairs et explicites permet non seulement d'améliorer la lisibilité du code, mais aussi de réduire les risques d'erreurs.
Par exemple, remplacer `callable` par `Closure` facilite la recherche de méthodes dans un projet et optimise les performances du compilateur.
De même, utiliser des générateurs au lieu d'un simple type `iterable` permet de mieux gérer les grands ensembles de données tout en garantissant une meilleure Developer eXperience.

De plus, si vous utilisez des analyseurs statiques comme PHPStan (vous devriez si ce n'est pas le cas), completez vos types `Closure` avec un maximum d'information : [callable typehint](https://phpstan.org/writing-php-code/phpdoc-types#callables).

En résumé, évitez les pseudo-types lorsque vous le pouvez.
Utiliser des types précis renforce la robustesse du code et permet d'exploiter pleinement les outils modernes d'analyse statique comme PHPStan.
Cela vous aidera à produire du code plus maintenable et performant — un objectif que tout développeur devrait viser.