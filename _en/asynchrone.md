---
layout: post-en
title: "Asynchronous Programming in PHP"
date: 2025-05-25
category: en
lang: en
---

(Published on June 3, 2025 - [Version française](/fr/asynchrone))

## Asynchronous Programming in PHP

PHP's traditional execution model is **synchronous**, which means that each instruction is executed in the **order** in which it appears in the code.
This isn't a problem in itself, as it's often simpler to think synchronously.

When you asked a PHP developer to create a paginated display with SQL, they will make a first SQL query to count the total number of results, then a second one to retrieve the results for the current page.
The total number of results is necessary for creating pagination links: *first page, next page, last page, etc*.

While the SQL server processes the first count query, the PHP server **waits**, and once the response is received, it will process the second one.

> Yes, there's a method to retrieve both results in a single query, but that's not the topic of this article, stay focused.

We can see in this pagination example a potential for **optimization**, by starting to process the second query **while** the SQL server is processing the first.
But be careful, we don't display the pagination links until we've displayed datas; therefore, even if the count query is finished, we must wait for displaying the other one, first.

Thus, managing asynchronous operations involves not only handling **parallel** tasks, but also controlling the **order** in which responses are processed.

There are many situations where executing code asynchronously is necessary, most commonly for **input/output** (I/O) operations such as HTTP requests, database queries, file access, or launching external processes.

## Is PHP Asynchronous?

To know if PHP is "*asynchronous*," we first need to understand what "*being asynchronous*" means.
"*Asynchronous*" means: not occurring at the **same time**.
When an operation takes time, instead of waiting for it to finish, we do something else, and we will **resume** when the operation is complete.
The core of asynchronicity, therefore, is that an operation is **non-blocking**.

We often tend to confuse *asynchronicity* and *parallelism*.

To illustrate this, think of asynchronicity like a cook who fills a pot with water, places it on the stove, and lights the burner.
While the water heats up, the cook chops vegetables.
Once the water is boiling and the vegetables are ready, cooking can begin.

With parallelism, imagine there are **two cooks**: while one chops the vegetables, the other heats the water.
Once the vegetables are ready and the water is boiling, the first cook takes over to start the actual cooking.

With this parallelism, we save time because we start chopping vegetables **while** the first cook is putting water in the pot and lighting the burner.
However, in both cases, we do something else while the water is boiling.

> Concretely, our cooks are the machine's CPU/GPU.

And now, if we look at PHP's capabilities, we see that since 2002, with the release of [PHP 4.3](https://www.php.net/ChangeLog-4.php#4.3.0), a major feature was introduced: [Streams](https://www.php.net/manual/en/book.stream.php).
And it's particularly the use of the [`stream_set_blocking()`](https://www.google.com/search?q=%5Bhttps://www.php.net/manual/en/function.stream-set-blocking.php%5D\(https://www.php.net/manual/en/function.stream-set-blocking.php\)) and [`stream_select()`](https://www.google.com/search?q=%5Bhttps://www.php.net/manual/en/function.stream-select.php%5D\(https://www.php.net/manual/en/function.stream-select.php\)) functions that brought PHP into the era of asynchronous programming.

```php
<?php
$h = fopen(__FILE__, 'r');
stream_set_blocking($h, false);
$content = '';
while (!feof($h)) {
    $read = array($h);
    $write = $except = null;
    // We check if there's anything to read and wait a maximum
    // of 1000 µs. Never "0" to avoid high CPU consumption
    $ready = stream_select($read, $write, $except, 0, 1000);

    if ($ready === 0) {
        // There's nothing to read, we wait a bit
        // or do something else...
        usleep(1000);
        continue;
    }
    $chunk = fgets($h, 1024);
    if ($chunk !== false) {
        $content .= $chunk;
    }
}

fclose($h);

echo $content;
```

> Warning: This illustrative code is intentionally simplistic and does not handle errors, for example.

Instead of calling `usleep(1000)`, we could perform other operations, like reading another file or making an HTTP request to a different server.
However, if your filesystem is fast, you might never actually hit a noticeable waiting time.
To observe meaningful delays, you'd typically need to work with slower filesystems or other latency-prone I/O operations.

PHP has technically supported asynchronous programming for 23 years. Yet, until fairly recently, it was commonly said that PHP is **not an asynchronous language**. Why?
Because supporting asynchronicity isn’t just about launching non-blocking operations, it's also about providing mechanisms to **manage waiting times** effectively.

This is where [coroutines](https://en.wikipedia.org/wiki/Coroutine) come into play.
A coroutine is a special kind of function that can pause its execution at certain points and resume later, preserving its state in the meantime.

In June 2013, with the release of [generators in PHP 5.5](https://www.php.net/releases/5_5_0.php), developers began to repurpose their use to act like coroutines.

```php
<?php
$generator = (function() {
    $count = 3;
    echo "Start\n";
    while(true) {
        yield; // suspend the function (the generator)
        echo "Are there results?\n";
        $count--;
        if ($count === 0) {
            return; // We received the results, we stop
        }
    }
})();

$generator->current(); // Initiates processing
do {
    echo "Do something else\n";
    $generator->next(); // Restart the function (resume at "yield")
} while ($generator->valid()); // Has the function finished?
echo "End\n";
```

> [Test this code on 3v4l.org](https://3v4l.org/1Juph)

It was with the release of [version 8.1](https://www.php.net/releases/8_1_0.php) that PHP took a real turn towards asynchronicity with the addition of [fibers](https://wiki.php.net/rfc/fibers) as a technical basis for true coroutines.

```php
<?php
$fiber = new Fiber(function() {
    $count = 3;
    echo "Start\n";
    while(true) {
        Fiber::suspend(); // suspend the fiber
        echo "Are there results?\n";
        $count--;
        if ($count === 0) {
            return; // We received the results, we stop
        }
    }
});

$fiber->start(); // Initiates processing
do {
    echo "Do something else\n";
    $fiber->resume(); // Restart the fiber
} while (!$fiber->isTerminated()); // The fiber has terminated
echo "End\n";
```

> [Test this code on 3v4l.org](https://3v4l.org/YrDZ0)

You’ll notice that the code has changed very little compared to the generator-based version.

While PHP had low-level asynchronous capabilities since version 4.3, the advent of PHP 8.1 with Fibers marks a **decisive step**.
Fibers provide powerful and ergonomic native tools for asynchronous programming, making it significantly more **natural**.

## Event Loop

Now that we know how to **interrupt** a coroutine and perform non-blocking processing, we need to manage **multiple** tasks in parallel, because a single asynchronous operation isn't very useful.

When we talk about *parallelism*, we often think of *threads*, which offer natural isolation between processes and can leverage multiple CPU cores, making them very interesting for intensive calculations.

However, parallelism, and more specifically, *multi-threading*, is more complex to implement, harder to debug, and introduces risks such as deadlocks and concurrent memory access issues.

For these reasons, another pattern is often preferred in the web world, where the number of simultaneous connections can be *very high*: the **EventLoop**.

The EventLoop is an infinite loop that monitors a queue of events (the arrival of a result, for example), and processes them **sequentially**, one at a time.

So, we will add our tasks to be done in this queue, then we will start the loop.

But how do we tell it what to do with the result of our operations?
It's quite *simple*; we will indicate a **callback function** that it will call when the result is **available**.

> Note: The Event Loop shown in this code is fictional, but it accurately represents the way most Event Loops operate.

```php
<?php
$loop = EventLoop::get();
$loop->addReadStream('file.txt', function(string $data) {
    echo "Data read: {$data}";
});
echo "Starting EventLoop\n";
$loop->run();
```

This code should display the following result:

```
Starting EventLoop
Data read: <some data from file.txt>
```

With reading 2 files, it could look like this:

```php
<?php
$loop = EventLoop::get();
$loop->addReadStream('/dev/cdrom/file1.txt', function(string $data) {
    echo "Data 1 read: {$data}";
});
$loop->addReadStream('/dev/fb0/file2.txt', function(string $data) {
    echo "Data 2 read: {$data}";
});
echo "Starting EventLoop\n";
$loop->run();
```

Depending on the performance of the reading media, we might see the following display:

```
Starting EventLoop
Data 2 read: <some data from floppy>
Data 1 read: <some data from CDRom>
```

Now, if we need to chain asynchronous operations, we end up with a *callback hell* (or *pyramid of doom*): callbacks nested within each other.

```php
<?php
$loop = EventLoop::get();
$loop->addReadStream('file.txt', function(string $data) {
    EventLoop::get()->defer(function() use ($data) {
        return compressData($data);
    }, function ($compressedData) {
        EventLoop::get()->addWriteStream(
            'http://foo', 
            $compressedData,
            function (Response $response) {
                echo "Data sent\n";
            });
    });
});
echo "Starting EventLoop\n";
$loop->run();
```

And it's even more complicated and **unreadable** if we add error handling to this.

```php
<?php
$loop = EventLoop::get();
$loop->addReadStream('file.txt', function(string $data) {
    EventLoop::get()->defer(function() use ($data) {
        return compressData($data);
    }, function ($compressedData) {
        EventLoop::get()->addWriteStream(
            'http://foo',
            $compressedData,
            function (Response $response) {
                echo "Data sent\n";
            }, function ($error) {
                echo "Error sending data: {$error}";
            });
    }, function ($error) {
        echo "Compression error: {$error}";
    });
}, function ($error) {
    echo "Error reading file: {$error}";
});
echo "Starting EventLoop\n";
$loop->run();
```

## Promises

To make reading easier and better manage asynchronicity, it can be interesting to use **promises**.

This concept was introduced in the 1980s in languages like Multilisp, but it was really in 2009 that the first implementations appeared in JavaScript in libraries like Dojo, Q, or jQuery.Deferred.

What is a promise? It's an object that contains the result of a process, present or **future**.
It's a bit like being told:

> "I won't give you the result of your processing immediately, but I promise to give it to you later, in this object."

Here's an example:

```php
<?php
$promise = new Promise(function ($resolve, $reject) {
    echo "Starting promise\n";
    $resolve("Hello, world!");
});
```

If we run this code, we see "`Starting promise`" displayed, but where is "`Hello, world!`"? And why call "`$resolve()`"?

Actually, for that, we need to use the "`then()`" method with... a **callback function**.

```php
<?php
$promise = new Promise(function ($resolve, $reject) {
    echo "Starting promise\n";
    $resolve("Hello, world!");
});

$promise->then(
    function ($value) {
        echo "Promise result: $value\n";
    }
);
```

This will display:

```
Starting promise
Promise result: Hello, world!
```

If we hadn't resolved the promise, nothing would have happened.

This code would just display the promise start message:

```php
<?php
$promise = new Promise(function ($resolve, $reject) {
    echo "Starting promise\n";
});

$promise->then(
    function ($value) {
        echo "Promise result: $value\n";
    }
);
```

Concretely, when the promise is resolved, the callback function in `then()` will be executed. And this will happen if, for example, the promise contains... a coroutine, which after a long process will receive its result and call `$resolve()`.

For this, we'll add an EventLoop, which would look like this:

```php
<?php
$loop = EventLoop::get();

$promise = new Promise(function ($resolve, $reject) use ($loop) {
    echo "Starting promise\n";
    $loop->addTimer(1, function () use ($resolve) {
        echo "Resolving promise\n";
        $resolve("Hello, World!");
    });
});

$promise->then(
    function ($value) {
        echo "Result: $value\n";
    }
);

$loop->run();
```

For this code to work, we use an *asynchronous timer*, which allows the promise to be resolved after 1 second.
This produces the following output:

```
Starting promise
Resolving promise
Result: Hello, World!
```

At this point, you must be wondering what the point of promises is in all this.
Let's go back to when we talked about **callback hell**.

With promises, it becomes possible to write our code like this:

```php
<?php
readFileAsync('file.txt')
    ->then(function ($data) {
        return compressDataAsync($data);
    })
    ->then(function ($compressedData) {
        return sendDataAsync('http://foo', $compressedData);
    })
    ->catch(function ($error) {
        echo "Error: {$error}\n";
    });
```

The "`readFileAsync()`" function returns a promise that uses the EventLoop to allow it to be resolved when it has the result.

`compressDataAsync()` and `sendDataAsync()` also return promises.

Finally, `catch()` allows handling errors raised throughout the chain.
Because **yes**, now we no longer have callbacks **nested** within callbacks, but a **chain** of callbacks.

You also have the option of returning a value in your callback, and in this case, this value is transformed into an **immediately resolved** promise with your value.
And obviously, if you don't return anything, it will be a promise resolved with the value `NULL`.

Finally, if you need to handle errors at different stages, the `then()` method accepts a **second parameter**, which is again a callback in case of rejection (error):

```php
<?php
readFileAsync('file.txt')
    ->then(
        function ($data) {
            return compressDataAsync($data);
        },
        function ($error) {
            echo "Error reading file: {$error}\n";
        }
    )
    ->then(function ($compressedData) {
        return sendDataAsync('http://foo', $compressedData);
    })
    ->catch(function ($error) {
        echo "Error: {$error}\n";
    });
```

However, note that if the error callback returns a value (or doesn't explicitly throw an exception), the subsequent `then()` will receive a resolved promise.

Therefore, you must return an errored promise or `throw` an exception.

And this is perhaps one of the common pitfalls when managing errors in a `then(onResolve, onReject)`; you have to handle **all** errors in the subsequent `then()` calls.
In the code above, we will call `sendDataAsync()` with `$compressedData` containing `NULL`.

## Which package to choose?

If you search for "[promise](https://packagist.org/?query=promise)" on Packagist, you'll notice that there are 4 packages that seem to stand out.

### Guzzle/promises and php-http/promise

The number of downloads for [guzzle/promises](https://github.com/guzzle/promises) is far ahead of the others, but this is also because it's directly used by the very popular HTTP client [Guzzle/Guzzle](https://github.com/guzzle/guzzle).

If you use this package, it might not be necessary to choose another one, as it's quite complete.

The problem is that **Guzzle/Promises** was initially designed to handle **asynchronous HTTP requests**, and for this, it uses an internal EventLoop that it doesn't expose, which makes it harder to integrate other types of Input/Output like [asynchronous Mysqli queries](https://www.php.net/manual/en/mysqli.reap-async-query.php) or [processes](https://www.php.net/manual/en/function.proc-open.php).

It's somewhat the same for the [php-http/promise](https://github.com/php-http/promise) package, which is also dedicated to HTTP requests.

### ReactPHP and Amp

Two important candidates remain: [react/promise](https://github.com/reactphp/promise) and [amphp/amp](https://github.com/amphp/amp).

**ReactPHP** offers a simple and performant implementation of the [JavaScript Promises/A+ standard](https://promisesaplus.com/) (yes, promises were initially a standard that emerged in the JavaScript language, hadn't we told you?).

On its side, **Amp** doesn't quite implement promises: there is no `then()` in version 3.0. However, it implements another mechanism called `Futures`, designed to be "*awaited*" (`await()`) within coroutines implemented with a generator or a fiber.

So, on one hand, you have management by **promise chains**, and on the other, management focused on **coroutines**.

If you've already used promises in JavaScript, it might be simpler to use **ReactPHP**; otherwise, **Amp's** coroutine management allows for **simpler** reading that is closer to our "*synchronous*" PHP practices.

But whether it's ReactPHP or Amp, you will need an **EventLoop**.

ReactPHP offers a "[react/event-loop](https://github.com/reactphp/event-loop)" package, while Amp suggests using [revolt/event-loop](https://github.com/revoltphp/event-loop), which was initiated by the Amp team to unify the asynchronous PHP ecosystem around a **standard** modern event loop.
Revolt is interoperable with ReactPHP via an adapter.

### So, what do I choose?

If you want to use the "**promises**" pattern, there's no contest, you should turn to React/Promises.

But on the other hand, Amp offers a different way of writing, which might seem more "*natural*" to some, and I think you should test both to see which one suits you best.

However, for the EventLoop, I invite you to lean towards **Revolt**, whose unifying ambition could pay off in the medium term.

Finally, there might be an argument that could help you choose: Amp v3 uses **PHP 8.1** fibers, which is not the case for ReactPHP, which can run perfectly well on an old **PHP 7.1**.

> Post-Script: We haven't discussed the **testability** of asynchronous development as this will be the subject of a future article.