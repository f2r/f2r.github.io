---
layout: post-en
title: "Understanding Dependency Injection"
date: 2024-12-16
category: en
lang: en
---

### Understanding Dependency Injection  
(Published on Dec 12, 2024 – [Version française](/fr/injection-de-dependance))  

In the world of development, the use of modern frameworks has become commonplace.  
These tools provide practical and efficient solutions for building applications, standardizing development approaches, and reducing code complexity.  
One of their major strengths lies in their ability to effectively manage dependencies between various components, particularly through dependency injection containers.  
Dependency injection, now so common, is built on fundamental principles that predate the frameworks themselves.  

Why is dependency injection essential?  
What problems does it solve, and how can it improve our code?  
Let’s explore these questions with a concrete example.  

---

### The Problem  

Let’s consider the following code:  

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

Here, the `Renderer` class relies on the `ContentManagement` class to function, it **depends** on it.  
While this isn’t inherently problematic in such a simple example, it introduces a limitation: the `Renderer` class is difficult to test.  
Testing would require creating test files, and because the file path is hardcoded in the `ContentManagement` class, issues could arise between the test files and the actual content files.  

---

### A Quick Fix with Pass-Through Variables  

To address the file path issue, we could modify the code like this:  

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

This approach is often used by developers who want a quick solution.  
Here, we added a patch to fix the testability issue, but it increases entropy.  
Although the testability seems improved, we’ve actually tightened the coupling between classes because `Renderer` still depends on `ContentManagement`, and now both are bound to a file system.  

Looking closer at `Renderer`, it merely acts as a pass-through for the `$path` variable without adding any real value.  

Whenever you find yourself using pass-through variables, it might indicate that something is being done incorrectly.  

---

### Dependency Injection  

To solve this, let’s separate the creation of the `ContentManagement` class from the `Renderer` class, and inject `ContentManagement` into `Renderer`.  
This is called [Inversion of Control](https://en.wikipedia.org/wiki/Inversion_of_control) (IoC): shifting the control of instantiation to another part of the program.  
A specific implementation of this is [Dependency Injection](https://en.wikipedia.org/wiki/Dependency_injection) (DI).  

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

Now, `Renderer` still depends on `ContentManagement`, but there’s no pass-through variable.  
`ContentManagement` is configured separately, and `Renderer` becomes agnostic to how the content is managed.  
This adheres to the [Single Responsibility Principle](https://en.wikipedia.org/wiki/Single_responsibility_principle): `Renderer` focuses solely on rendering.  

We can now change the code’s behavior using inheritance:  

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

With dependency injection, we no longer create instances directly inside our classes, eliminating the need for `new`. However, as we’ll see later, this isn’t always automatic.  

In the previous example, instantiation happens in the application code just before calling the render method.  
However, the process of creating objects can often become quite complex.  
In such cases, we can use another class to handle this task: **factories**.  

Here’s an example:  

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

$contentManagement = (new ContentManagementFactory())->makeFor('foo');  

echo new Renderer($contentManagement)->render();  
```

Now, you might be thinking: “Wait, didn’t we just reintroduce a tightly coupled class?”  
And you’d be right. However, in this architecture, using the factory is optional, you can still create an instance of a class manually when writing tests.  

Factories are facilitators: they centralize the tedious object creation code, which may include injecting sub-dependencies at multiple levels.  

That said, factories are typically specialized for creating objects within a specific domain.  
They are strictly limited to instantiation and should never participate in the execution of processes.  

At a higher level, you’ll encounter **dependency injection containers**.  
These containers handle the creation of most objects, including factories, based on configuration files.  
In the `ApiContentManagement` example above, configuration details like the API URL or token would be stored in a configuration file, and the injection container would use this information to build the objects.  

### Behavior Injection

When practicing dependency injection, it doesn't mean that we stop depending on other classes. Instead, we reduce coupling between classes by allowing a dependency's behavior to be substituted with another one.

Moreover, using inheritance for this substitution is not a great approach either. It’s better to rely on interfaces for such cases. Here's an example:

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

Notice how the concrete class is declared `final` to emphasize that inheritance is not promoted in this object-oriented architecture.

At this point, we no longer depend on a class but rather on a class behavior, which is what we inject.
In the end, your classes should contain no `new` keywords, and the parameters in your constructors should exclusively consist of interfaces to aggregate behaviors.

### Not Everything Should Be Injected

When we say "there should be no `new`," that's not entirely true, because there are many cases where injection is not appropriate.

For example, you might use *[value objects](https://acairns.co.uk/posts/primitive-obsession)* (VO), *[data transfer objects](https://en.wikipedia.org/wiki/Data_transfer_object)* (DTO), events, exceptions, and generally any transient or short-lived objects.

One of the key benefits of dependency injection is the ability to substitute behaviors, which makes them significantly more testable.

However, some class compositions are intentionally designed this way to break down the code into smaller pieces, making it easier to read and maintain. When you have a long method, you refactor it into a series of private sub-methods to make it more digestible. The same principle applies to classes: you subdivide your large class into several smaller classes and move parts of your code into them. These classes often have no purpose outside the context of the class they belong to, and dependency injection adds no value in these cases.

In fact, you might even add the `@internal` tag or use a static analyzer like [Deptrac](https://deptrac.github.io/deptrac/) to limit interactions with your internal classes. One day, PHP might offer the ability to adapt [class visibility](https://wiki.php.net/rfc/namespace-visibility) or introduce [friend classes](https://wiki.php.net/rfc/friend-classes).

You need to strike a balance. While injection provides flexibility to evolve your code, excessive or inappropriate use can make it complex and harder to understand.

Stay pragmatic!
