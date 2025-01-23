---
layout: post-en
title: "Stop using Pseudo-Types"
date: 2025-01-20
category: en
lang: en
---
# Stop Using Pseudo-Types
(Published on Jan 20, 2025 - [Version française](/fr/arretez-les-pseudo-types))

In 2011, with the release of [PHP 5.4](https://www.php.net/ChangeLog-5.php#PHP_5_4), the pseudo-type `callable` was introduced via the [RFC: Callable](https://wiki.php.net/rfc/callable).
In 2016, another pseudo-type, `iterable`, was added in [PHP 7.1.0](https://www.php.net/ChangeLog-7.php#7.1.0) through the [RFC: Iterable](https://wiki.php.net/rfc/iterable).

Pseudo-types are not true types like `string` or `array`.  
They are sets of types with specific validation logic.
Let’s look at the `callable` pseudo-type for a better understanding.

## The `callable` Pseudo-Type

In PHP 4, the [`call_user_func()`](https://www.php.net/call_user_func) function was introduced to call a function by its name as a string.  
There was also a `call_user_method()` function, but it was quickly deprecated in PHP 4.1 and removed in PHP 7.0 because `call_user_func()` could also accept an array where the first element was an object, and the second was the method name.

```php
<?php
$format = [new Datetime('2025-01-30'), 'format'];
echo call_user_func($format, 'd/m/Y');
// Output: 30/01/2025
```
[Try this code](https://3v4l.org/Fi9Ma)

PHP 4.0.6 introduced the [`is_callable()`](https://www.php.net/is_callable) function, which checks if a parameter is callable.  

In fact, a string or an array can be callable, but only under certain conditions.

```php
<?php
$create = 'Datetime::createFromFormat';
$invoke = [new stdClass, '__invoke'];

var_dump(is_callable($create), is_callable($invoke));
// Output:
// bool(true)
// bool(false)
```
[Try this code](https://3v4l.org/K5PMD)

I suggest reading the article by my friend Damien Seguy: [How to call a method in PHP](https://www.exakat.io/en/call-a-method-in-php/).

For a string to be callable, it must contain the name of an existing function or method.  
For example, `trim` is callable, but `unknown` will not be if no such function exists.

Note that certain names are not callable, such as `isset`, `empty`, `echo`, or `include`.  
`exit` and `die` were not callable before PHP 8.4 but are now callable: [PHP RFC: Transform exit() from a language construct into a standard function](https://wiki.php.net/rfc/exit-as-function).

An interesting read: [A Look At PHP’s isset()](https://medium.com/@liamhammett/a-look-at-phps-isset-df64df7158ab).

In short, at runtime, you cannot always determine whether a string is callable.

In March 2012, with the release of [PHP 5.4](https://www.php.net/releases/5_4_0.php), the logic of the `is_callable()` function was extended to introduce the `callable` pseudo-type.  
This is why it is called a pseudo-type: `callable` is not a type; it is a union of the types `Closure`, `string`, and `array` with logic to verify, at runtime, whether a parameter is callable.  
At compile time, unless it’s a `Closure`, you cannot know if a function or class will exist during validation.  
This is also why it is not possible to use `callable` for typed properties: [PHP RFC: Typed Properties 2.0, Supported Types](https://wiki.php.net/rfc/typed_properties_v2#supported_types).

## The (Not So) Pseudo-Type `iterable`

In PHP 7.1, the `iterable` pseudo-type was formally introduced.  
It allowed validation of whether a value was iterable by checking if it implemented the type `array` or the interface `Traversable`.  
With the release of [PHP 8.2](https://www.php.net/ChangeLog-8.php#PHP_8_2), `iterable` lost its pseudo-type status and became a union type: `Traversable|array`.

To better understand this difference, observe the behavior of the following code in PHP 8.1 and 8.2:

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

- PHP 8.1.0: `iterable, iterable|bool`
- PHP 8.2.0: `iterable, Traversable|array|bool`

This internal change was introduced by this pull request : [Convert iterable into an internal alias for Traversable array](https://github.com/php/php-src/pull/7309).

However, the `Traversable` interface is somewhat unique and could almost be considered a pseudo-type, as it is an interface that cannot be implemented directly but is extended by the `Iterator` and `IteratorAggregate` interfaces.  
In fact, the `iterable` type should have been a union of types: `Iterator|IteratorAggregate|array`.

## Data Typing with `iterable`

The `iterable` type should not be used to type a function or method return, as this can create confusion if the data is misused.  
For example, developers might be tempted to manipulate the returned value upon discovering it is an array.
They might use `count` to check if there are values to display alternative text.
However, this type could evolve later to return an iterator.  

Example:

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

This is a blatant mistake, but a good static analyzer like [PHPStan](https://phpstan.org/) or [Exakat](https://www.exakat.io/en/) should catch this.
Just keep in mind that developers make mistakes, especially when they're starting out.

To handle this variation in results correctly, you need to check whether the returned data is an array (`is_array`) or an iterator (`instanceof Iterator`), or use a function like [`iterator_to_array()`](https://www.php.net/iterator_to_array):

```php
<?php
// ...
echo iterator_to_array((new B)->get())[0];
```
[Try this code](https://3v4l.org/LVOQM)

This solution works but can impact performance, especially if a [Generator](https://www.php.net/generator) is used to optimize memory usage for large results.

A good [Defensive Programming](https://en.wikipedia.org/wiki/Defensive_programming) approach is to limit the possibilities by offering either an array or an iterator.  
Personally, I tend to prefer iterators via generators.

When typing parameters in a function or method, using `iterable` indicates that your implementation will only iterate over the data.
Thus, you can pass either an array or an iterator interchangeably, improving the Developer Experience (DX).

## Data Typing with `callable`

While the use of `iterable` is disputable, `callable` is less so.  
This is partly because it cannot be used to type a property and partly because it allows developers to define the call as a string, which creates numerous problems for static analysis or even just for searching the usage of a method, for instance.

Even though IDEs have made significant progress, it can be challenging for them to find function usages when the function is defined in a variable as a string.

The introduction of *[first-class callables](https://www.php.net/manual/en/functions.first_class_callable_syntax.php)* simplifies callback handling.
You no longer have an excuse to define your callbacks like this:

```php
<?php
$data = array_map(trim(...), [' x', 'z  ']);
```
[Try this code](https://3v4l.org/gpCav)

It is therefore better to use the [`Closure`](https://www.php.net/closure) class when defining callbacks.  
If you are using a version prior to PHP 8.1 and cannot use first-class callables, I suggest using `Closure::fromCallable()`:

```php
<?php
$data = array_map(
    Closure::fromCallable('trim'),
    [' x', 'z  ']
);
```
[Try this code](https://3v4l.org/7tiOX)

### Typing Properties with Closures

One detail when using properties of type `Closure` is that you cannot call the closure like this:

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

This generates an error: `Fatal error: Uncaught Error: Call to undefined method A::callback()`.  
This makes sense since it is difficult to distinguish between a method call and an invokable property.

You must call the property like this: `($this->callback)();` ([Try this code](https://3v4l.org/b6Rf3)).

You can also use: `call_user_func($this->callback);` ([Try this code](https://3v4l.org/HT8QR)).

Another interesting approach is that the `Closure` class provides an `__invoke` method.
You can write: `$this->callback->__invoke();` ([Try this code](https://3v4l.org/iZ0VR)).  
I find this syntax appealing because it is very explicit: “We are invoking the closure.”

## Conclusion

Pseudo-types like `callable` and `iterable` may seem convenient at first glance, but they introduce ambiguities and make code harder to analyze.

Favoring clear and explicit types not only improves code readability but also reduces the risk of errors.  
For example, replacing `callable` with `Closure` makes it easier to search for methods within a project and optimizes compiler performance.  
Similarly, using generators instead of a simple `iterable` type allows for better handling of large datasets while ensuring a better Developer Experience.

Additionally, if you use static analyzers like PHPStan (and you should if you don’t), complement your `Closure` types with as much information as possible: [callable typehint](https://phpstan.org/writing-php-code/phpdoc-types#callables).

In summary, avoid pseudo-types whenever possible.  
Using precise types strengthens code robustness and maximizes the potential of modern tools like PHPStan.
This will help you produce more maintainable and efficient code — an objective every developer should strive for.
