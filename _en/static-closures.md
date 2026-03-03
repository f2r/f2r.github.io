---
layout: post-en
title: "Why use static closures?"
date: 2026-03-03
category: en
lang: en
---

## Why use static closures?
(Published on March 3, 2026 - [Version française](/fr/closures-statiques))

In PHP, we use [closures](https://www.php.net/closure) more and more, in dependency injection, middleware, collection callbacks, and also in asynchronous processing, as I wrote in my article "[Asynchronous Programming in PHP](/en/asynchrone)" as a callback tool.

However, they have a behaviour that can be surprising: any closure created inside an instance method automatically carries a reference to the current object, even if it does **not** use `$this`.
This behaviour can have unexpected consequences on object lifetimes and generate memory leaks if you aren't careful..

To understand why, we first need to understand how PHP manages memory.
Unlike languages such as Java, which rely on a garbage collector to free memory in a deferred manner, PHP uses **reference counting** (Anyway, to be honest, PHP actually has a [garbage collector](https://www.php.net/manual/en/features.gc.php) for cyclic references, but that’s a whole other story).

When you assign a variable, its content must be stored in memory, and when the variable is no longer used, the memory can be freed. When you write this:
```php
<?php
$a = 'Hello';
$b = $a;
```
PHP will not create a second memory space for the variable `$b`, it will simply indicate that it points to the same memory space as `$a`.
If you then assign a new value to `$a`, say `"Hi"`, a new memory space is allocated and `$a` points to it, while `$b` continues to point to the old space.
Now, if you assign `NULL` to `$b`, the memory space that contained `"Hello"` is no longer referenced by any variable, and can therefore be freed.
To do this, PHP maintains a reference count, and if that count drops to zero, the space is freed.

### The lifecycle of an object

With an object, when the reference count drops to 0, before freeing the memory, if the class defines a [`__destruct`](https://www.php.net/manual/en/language.oop5.decon.php#language.oop5.decon.destructor) method, it is called:

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
echo "End\n";
```

```
Construct
Destruct
End
```

The object is not assigned to any variable: its counter drops to zero immediately after the constructor is called, and `__destruct` is invoked right after.

On the other hand, if the object is assigned to a variable, destruction is deferred:

```php
<?php
$foo = new Foo();
echo "End\n";
```

```
Construct
End
Destruct
```

As long as `$foo` points to the object, the counter stays at one. Destruction only occurs at the end of the script, once all variables are freed. To force early destruction, simply release the variable explicitly by assigning a new value or calling `unset()`:

```php
<?php
$foo = new Foo();
echo "Before release\n";
$foo = null;
echo "After release\n";
```

```
Construct
Before release
Destruct
After release
```

### A closure keeps the object alive

Let us see what happens with a class `Bar` that defines a `getCallback()` method, which itself returns a closure reading the `$this->id` property:

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

echo "Before releasing the object\n";
$bar = null;
echo "After releasing the object\n";

echo $getId() . "\n";

echo "End\n";
```

```
Construct
Before releasing the object
After releasing the object
foo
End
Destruct
```

The object is not destroyed when we assign `null` to the variable `$bar`, because the closure accesses `$this->id`, so it constitutes a reference to the object. The counter does not drop to zero as long as the closure exists, that is, until the end of the script. If we had reassigned `$getId`, the call to `__destruct` would have occurred earlier, since releasing the variable also released the reference to `$this`.

### Even without $this, the object stays alive

What happens if we do not use `$this` in the closure?

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

echo "Before releasing the object\n";
$bar = null;
echo "After releasing the object\n";
$callback = null;
echo "End\n";
```

```
Construct
Before releasing the object
After releasing the object
Destruct
End
```

The object is still kept alive, because even though we do not use `$this`, the closure still references the object — PHP automatically binds `$this` to any closure created in an instance method, whether it uses it or not, whether it is empty or not.
The closure therefore always carries a reference to the object, invisible when reading the code.

Of course, if the closure is created inside a static method, there is no reference to `$this`, and destruction occurs at the moment the variable is released:
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

echo "Before releasing the object\n";
$bar = null;
echo "End\n";
```

```
Construct
Before releasing the object
Destruct
End
```

### The static closure

The `static` keyword applied to a closure explicitly forbids it from being bound to `$this`.
PHP then no longer stores any reference to the object, even implicitly.

```php
// ...
public function getCallback(): Closure {
    return static function(): void {};
}
// ...
```

```
Construct
Before releasing the object
Destruct
End
```

And if we need to retrieve the value of a property in the closure, we can use `use` like this:

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

This time, PHP will destroy the object right after the variable is released, because the closure retains no reference to it.

If you attempt to use `$this` inside a static closure, PHP will throw an error:

```php
return static function(): string {
    return $this->id; // Error: Using $this when not in object context
};
```

The PHP engine thus protects you from accidental capture.

### Short closures

Short closures (`fn() =>`) offer a more concise syntax and capture variables from the enclosing scope **automatically**, without `use`.
But they share the same behaviour as regular closures with respect to `$this`:

```php
public function getCallback(): Closure {
    return fn(): string => $this->id;
}
```

Here, `$this` is captured implicitly, just as with an ordinary closure. The object stays alive as long as the closure exists.

The `static` keyword also applies to short closures. Variables from the enclosing scope are still captured automatically, but `$this` is no longer captured:

```php
public function getCallback(): Closure {
    return static fn(): string => $this->id; // Error: Using $this when not in object context
}
```

To pass the value without capturing the object, simply extract it beforehand:

```php
public function getCallback(): Closure {
    $id = $this->id;
    return static fn(): string => $id;
}
```

The variable `$id` is captured by value, `$this` is no longer involved, and the object can be freed as soon as its explicit reference disappears.

### What PHP 8.6 will change

The [Closure Optimizations RFC](https://wiki.php.net/rfc/closure-optimizations), under vote for PHP 8.6 at the time of writing, addresses precisely this behaviour.
It introduces **automatic inference**: if a closure makes no use of `$this`, PHP will make it static on its own, without the developer having to write it.

Our example with a closure using `use ($id)` or the short closure `fn(): string => $id` would therefore no longer capture the object implicitly once this RFC is adopted.

The RFC goes even further with a second optimisation: static closures that capture no variables (neither `use`, nor enclosing scope) are **cached** and reused between calls, avoiding their re-instantiation every time.

These two optimisations are transparent for existing code, with one exception: [`ReflectionFunction::getClosureThis()`](https://www.php.net/manual/en/reflectionfunctionabstract.getclosurethis.php) will return `null` for closures now inferred as static, which could introduce a behaviour change for existing code (Breaking Change).

### Be explicit

As a general rule, when a closure — or a short closure — does not need `$this`, it is preferable to declare it `static`.
This makes the intention **explicit**, prevents involuntary captures, and allows the object to be destroyed as soon as its last explicit reference disappears.

With PHP 8.6, this safe behaviour will become the **default**, but declaring `static` remains useful for documenting intent and guaranteeing compatibility with earlier versions.
