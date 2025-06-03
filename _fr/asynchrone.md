---
layout: post-fr
title: "La programmation asynchrone en PHP"
date: 2025-05-25
category: fr
lang: fr
---

(Published on June 3, 2025 - [Version française](/en/asynchrone))

## La programmation asynchrone en PHP

Le modèle d'exécution traditionnel de PHP est **synchrone**, ce qui signifie que chaque instruction est exécutée dans l'**ordre** dans lequel elles apparaissent dans le code.
Cela n'est pas en soi un problème, car il est souvent plus simple de penser de manière synchrone.

Quand on demande à un développeur PHP de faire un affichage paginé en SQL, il va faire une première requête SQL pour compter le nombre total de résultats, puis une deuxième pour récupérer les résultats de la page courante.
Le nombre de résultats total étant nécessaire à la création des liens de pagination : *première page, page suivante, dernière page, etc*.

Pendant que le serveur SQL traite la première requête de comptage, le serveur PHP **attend**, et une fois la réponse reçue, il va traiter la deuxième.
> Oui, il existe une méthode pour récupérer les 2 informations en une seule requête, mais ce n'est pas l'objet de cet article, restez concentrés

On peut voir dans cet exemple de pagination, un potentiel d'**optimisation**, en commençant à traiter la deuxième requête **pendant** que le serveur SQL traite la première.
Mais attention, on n'affiche pas les liens de pagination tant que l'on n'a pas affiché les résultats, par conséquent, même si la requête de comptage est terminée, on doit attendre l'affichage de l'autre.

Ainsi, gérer des opérations asynchrones ne se limitent pas à exécuter des tâches en **parallèle**, mais aussi à gérer l'**ordre** de traitement des réponses.

Il existe de nombreux cas où vous pourriez avoir besoin d'exécuter du code de manière asynchrone, et c'est très souvent lié à des opérations d'**entrée/sortie** (I/O), comme des requêtes HTTP, des accès à des bases de données, des lectures/écritures de fichiers ou le lancement de processus externes.

## PHP est-il asynchrone ?

Pour savoir si PHP est "*asynchrone*", il faudrait déjà comprendre ce que veut dire "*être asynchrone*". 
"*Asynchrone*" veut dire : qui ne se produit pas en **même temps**.
Quand une opération prend du temps, au lieu d'attendre la fin, on va faire autre chose, et on **reprendra** quand l'opération sera terminée.
Le cœur de l'asynchronisme, c'est donc le fait qu'une opération est **non-bloquante**.

On a souvent tendance à confondre l'*asynchronisme* et le *parallélisme*.

Pour imager cela, on peut voir l'asynchronisme comme un cuisinier qui va mettre de l'eau dans une casserole, la mettre sur la gazinière puis allumer le feu.
Pendant que l'eau bout, il découpe des légumes.
Quand les légumes sont découpés et que l'eau est chaude, il lance la cuisson.

Avec le parallélisme, on a **2 cuisiniers** : pendant que l'un découpe, un second s'occupe de faire chauffer l'eau.
Quand les légumes seront découpés et l'eau chaude, c'est le premier cuisinier qui s'occupera de la cuisson.

Avec ce parallélisme, on gagne du temps car on commence à découper les légumes **pendant** que le premier cuisinier met de l'eau dans la casserole et allume le feu.
Par contre, dans les 2 cas, on fait autre chose pendant que l'eau bout.

> Concrètement, nos cuisiniers sont les CPU/GPU de la machine.

Et maintenant, si on regarde les capacités de PHP, on constate que depuis 2002, avec la sortie de [PHP 4.3](https://www.php.net/ChangeLog-4.php#4.3.0), une fonctionnalité majeure a été introduite : [Les streams](https://www.php.net/manual/en/book.stream.php).
Et c'est l'usage plus particulièrement de la fonction [`stream_set_blocking()`](https://www.php.net/manual/en/function.stream-set-blocking.php) et [`stream_select()`](https://www.php.net/manual/en/function.stream-select.php) que PHP est entré dans l'ère de la programmation asynchrone.

```php
<?php
$h = fopen(__FILE__, 'r');
stream_set_blocking($h, false);
$content = '';
while (!feof($h)) {
    $read = array($h);
    $write = $except = null;
    // On regarde s'il y a des choses à lire et on attend au
    // maximum 1000 µs. Jamais "0" pour éviter une sur-consommation
    // CPU
    $ready = stream_select($read, $write, $except, 1000);

    if ($ready === 0) {
        // Il n'y a rien à lire, on attend un peu
        // ou on fait autre chose ...
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
> Attention, ce code d'illustration est volontairement simpliste et ne gère pas, par exemple, les erreurs.

A la place du `usleep(1000)`, on pourrait faire d'autres opérations, comme par exemple lire un autre fichier, ou même faire une requête HTTP à un autre serveur. Par contre, si vous avez un filesystem rapide, vous ne devriez jamais rentrer dans le temps d'attente. Il faut vraiment travailler avec des filesystem lents ou d'autres types d'entrée/sortie.

23 ans aujourd'hui que PHP permet de faire de la programmation asynchrone, pourtant, il y a encore quelques années, on disait que PHP **n'était pas un langage asynchrone**, pourquoi ?

Parce que faire de l'asynchronisme, ce n'est pas juste lancer un traitement non bloquant, c'est aussi avoir des mécanismes pour **gérer ces temps d'attente**.

C'est là qu'entre en jeu les [coroutines](https://en.wikipedia.org/wiki/Coroutine).
Une coroutine est une fonction qui peut être suspendue puis reprise plus tard.

En juin 2013, avec la sortie des [générateurs en PHP 5.5](https://www.php.net/releases/5_5_0.php), des développeurs ont commencé à détourner leur usage pour les utiliser comme des coroutines.

```php
<?php
$generator = (function() {
    $count = 3;
    echo "Début\n";
    while(true) {
        yield; // on suspend la fonction (le générateur)
        echo "Il y a des résultats ?\n";
        $count--;
        if ($count === 0) {
            return; // On a reçu les résultats, on s'arrête
        }
    }
})();

$generator->current(); // Initie le traitement
do {
    echo "Faire autre chose\n";
    $generator->next(); // On relance la fonction (on reprend au "yield")
} while ($generator->valid()); // La fonction s'est terminée ?
echo "Fin\n";
```
> [Testez ce code sur 3v4l.org](https://3v4l.org/Ogjug)

C'est avec la sortie de la [version 8.1](https://www.php.net/releases/8_1_0.php) que PHP a pris un vrai tournant vers l'asynchronisme avec l'ajout des [fibers](https://wiki.php.net/rfc/fibers) comme base technique pour des vraies coroutines.

```php
<?php
$fiber = new Fiber(function() {
    $count = 3;
    echo "Début\n";
    while(true) {
        Fiber::suspend(); // on suspend la fiber
        echo "Il y a des résultats ?\n";
        $count--;
        if ($count === 0) {
            return; // On a reçu les résultats, on s'arrête
        }
    }
});

$fiber->start(); // Initie le traitement
do {
    echo "Faire autre chose\n";
    $fiber->resume(); // On relance la fiber
} while (!$fiber->isTerminated()); // La fiber s'est terminée
echo "Fin\n";
```
> [Testez ce code sur 3v4l.org](https://3v4l.org/tiap5)

Vous remarquerez que le code a à peine changé par rapport à l'utilisation des générateurs.

Si PHP disposait de capacités asynchrones bas niveau depuis la version 4.3, l'avènement de PHP 8.1 avec les Fibers marque une **étape décisive**.
Ces dernières fournissent des outils natifs puissants et ergonomiques pour la programmation asynchrone, la rendant significativement plus **naturelle**.

## Event Loop

Maintenant que l'on sait **interrompre** une coroutine, et effectuer du traitement non-bloquant, il va falloir gérer **plusieurs** traitement en parallèle, car un seul traitement asynchrone, cela n'a pas grand intérêt.

Quand on parle de *parallélisme*, on pense souvent aux *threads* qui offrent une isolation naturelle entre les processus et peuvent exploiter plusieurs cœurs CPU, les rendant très intéressant pour du calcul intensif.

Seulement, le parallélisme, et plus précisément, le *multi-threading* est plus complexe à mettre en œuvre, plus difficile à déboguer et présente des risques de deadlocks et d'accès concurrent à la mémoire.

C'est pour ces raisons qu'un autre pattern est privilégié dans le monde du web, où le nombre de connexions simultanées peut être *très élevé* : l'**EventLoop**.

L'EventLoop est une boucle infinie qui surveille une file d'événements (l'arrivée d'un résultat par exemple), et les traite un par un de manière **séquentielle**.

On va donc ajouter nos traitements à faire dans cette file, puis on va démarrer la boucle.

Seulement, comment lui dire ce qu'il faut faire du résultat de nos traitements ?
C'est assez *simple*, on va lui indiquer une **fonction de rappel** (callback en anglais) qu'elle appellera quand le résultat sera **disponible**.

> Remarque : L'EventLoop présenté dans ce code est fictif, mais représentatif du fonctionnement de la plupart des EventLoop.

```php
<?php
$loop = EventLoop::get();
$loop->addReadStream('file.txt', function(string $data) {
    echo "Données lues : {$data}";
});
echo "Démarrage de l'EventLoop\n";
$loop->run();
```

Ce code devrait afficher le résultat suivant : 
```
Démarrage de l'EventLoop
Données lues : <some data from file.txt>
```

Avec la lecture de 2 fichiers, cela pourrait donner :
```php
<?php
$loop = EventLoop::get();
$loop->addReadStream('/dev/cdrom/file1.txt', function(string $data) {
    echo "Données 1 lues : {$data}";
});
$loop->addReadStream('/dev/fb0/file2.txt', function(string $data) {
    echo "Données 2 lues : {$data}";
});
echo "Démarrage de l'EventLoop\n";
$loop->run();
```

En fonction de la performance des supports de lecture, nous pourrions avoir l'affichage suivant :
```
Démarrage de l'EventLoop
Données 2 lues : <some data from floppy>
Données 1 lues : <some data from CDRom>
```

Maintenant, si on doit enchaîner des opérations asynchrones, on se retrouve avec un *callback hell* (ou *pyramid of doom*) : des callbacks imbriquées les unes dans les autres.

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
echo "Démarrage de l'EventLoop\n";
$loop->run();
```

Et c'est d'autant plus compliqué et **illisible**, si on ajoute à cela la gestion des erreurs.

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
                echo "Erreur d'envoi des données: {$error}";
            });
    }, function ($error) {
        echo "Erreur de compression: {$error}";
    });
}, function ($error) {
    echo "Erreur de lecture du fichier: {$error}";
});
echo "Démarrage de l'EventLoop\n";
$loop->run();
```

## Les promises

Pour rendre la lecture plus facile et mieux gérer l'asynchronisme, il peut être intéressant d'utiliser les **promesses** (ou "promises" en anglais).

Ce concept a été introduit dans les années 80 dans des langages comme Multilisp, mais c'est vraiment en 2009 que les premières implémentations sont apparues dans Javascript dans des bibliothèques comme Dojo, Q ou jQuery.Deferred.

Une promesse, c'est quoi ? C'est un objet qui contient le résultat d'un traitement, présent ou **futur**.
C'est un peu comme si on vous disait : 
> "Je ne vous donne pas immédiatement le résultat de votre traitement, mais je vous promet de vous le donner plus tard, dans cet objet."

Voici un exemple : 
```php
<?php
$promise = new Promise(function ($resolve, $reject) {
    echo "Lancement de la promesse\n";
    $resolve("Hello, world!");
});
```
Si on lance ce code, on constate l'affichage de "`Lancement de la promesse`", mais où est "`Hello, world!`" ? Et pourquoi appeler "`$resolve()`" ?

En fait, pour cela, il faut utiliser la méthode "`then()`" avec ... une **fonction de rappel**.

```php
<?php
$promise = new Promise(function ($resolve, $reject) {
    echo "Lancement de la promesse\n";
    $resolve("Hello, world!");
});

$promise->then(
    function ($value) {
        echo "Résultat de la promesse : $value\n";
    }
);
```

Ce qui va afficher : 
```
Lancement de la promesse
Résultat de la promesse : Hello, world!
```

Si nous n'avions pas résolu la promesse, Il ne se serait rien passé.

Ce code afficherait juste le message de lancement de la promesse : 
```php
<?php
$promise = new Promise(function ($resolve, $reject) {
    echo "Lancement de la promesse\n";
});

$promise->then(
    function ($value) {
        echo "Résultat de la promesse : $value\n";
    }
);
```

Concrètement, quand la promesse sera résolue, la fonction de rappel dans `then()` sera exécutée. Et cela arrivera si par exemple, la promesse contient ... une coroutine, qui après un long traitement va recevoir son résultat et appeler `$resolve()`.

Pour cela, on va ajouter une EventLoop ce qui donnerait : 

```php
<?php
$loop = EventLoop::get();

$promise = new Promise(function ($resolve, $reject) use ($loop) {
    echo "Lancement de la promesse\n";
    $loop->addTimer(1, function () use ($resolve) {
        echo "Résolution de la promesse\n";
        $resolve("Hello, World!");
    });
});

$promise->then(
    function ($value) {
        echo "Résultat : $value\n";
    }
);

$loop->run();
```

Pour que ce code fonctionne, on utilise un *timer asynchrone*, qui permet de résoudre la promesse après 1 seconde.
Ce qui donne l'affichage : 

```
Lancement de la promesse
Résolution de la promesse
Résultat : Hello, World!
```

Là, vous devez vous demander quel est l'intérêt des promesses dans tout ça.
Revenons au moment où on vous parlait du **callback hell**.

Avec les promesses, il devient possible d'écrire notre code de cette façon : 

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
        echo "Erreur : {$error}\n";
    });
```

La fonction "`readFileAsync()`" retourne une promesse qui utilise l'EventLoop pour permettre de la résoudre quand elle aura le résultat.

`compressDataAsync()` et `sendDataAsync()` retourne également des promesses.

Enfin, le `catch()` permet de gérer les erreurs remontées dans l'ensemble de la chaîne.
Car **oui**, désormais, nous n'avons plus des callbacks **imbriquées** dans des callbacks, mais une **chaîne** de callbacks.

Vous avez aussi la possibilité de retourner une valeur dans votre callback, et dans ce cas, cette valeur est transformée en une promesse **imédiatement résolue** avec votre valeur.
Et bien évidemment, si vous ne retournez rien, ça sera une promesse résolue avec la valeur `NULL`.

Enfin, si vous devez gérer des erreurs aux différentes étapes, la méthode `then()` accepte un **second paramètre**, qui est à nouveau une callback en cas de rejet (erreur) : 

```php
<?php
readFileAsync('file.txt')
    ->then(
        function ($data) {
            return compressDataAsync($data);
        },
        function ($error) {
            echo "Erreur de lecture du fichier: {$error}\n";
        }
    )
    ->then(function ($compressedData) {
        return sendDataAsync('http://foo', $compressedData);
    })
    ->catch(function ($error) {
        echo "Erreur : {$error}\n";
    });
```

Par contre, il faut noter que si la callback d'erreur retourne une valeur (ou pas de `return` du tout), dans le `then()` suivant, vous allez récupérer une promesse résolue.

Il faut, par conséquent, retourner une promesse en erreur ou faire un `throw`.

Et c'est peut-être l'un des pièges courant quand on gère les erreurs dans un `then(onResolve, onReject)`, il faut gérer **toutes** les erreurs dans les `then()` suivants.
Dans le code ci-dessus, on va faire un `sendDataAsync()` avec `$compressedData` qui contient `NULL`.

## Quel package choisir ?

Si vous cherchez "[promise](https://packagist.org/?query=promise)" sur packagist vous constaterez qu'il y a 4 packages qui semblent se distinguer.

### Guzzle/promises et php-http/promise

Le nombre de téléchargements de [guzzle/promises](https://github.com/guzzle/promises) est loin devant les autres, mais c'est aussi parce qu'il est directement utilisé par le très populaire client HTTP [Guzzle/Guzzle](https://github.com/guzzle/guzzle).

Si vous utilisez ce package, il n'est peut-être pas utile d'en choisir un autre, car il est assez complet.

Le problème, c'est que **Guzzle/Promises** a été conçu au départ pour gérer les **requêtes HTTP asynchrones**, et pour cela, il utilise une EventLoop interne qu'il n'expose pas, ce qui rend plus difficile l'intégration d'autres types d'Entrées/Sorties comme les [requêtes asynchrones de Mysqli](https://www.php.net/manual/en/mysqli.reap-async-query.php) ou les [processus](https://www.php.net/manual/en/function.proc-open.php).

C'est un peu la même chose pour le package [php-http/promise](https://github.com/php-http/promise) qui, lui aussi, est dédié au requêtage HTTP.

### ReactPHP et Amp

Reste deux candidats importants qui sont [react/promise](https://github.com/reactphp/promise) et [amphp/amp](https://github.com/amphp/amp).

**ReactPHP** offre une implémentation simple et performante du [standard javascript Promises/A+](https://promisesaplus.com/) (oui, les promises sont au départ un standard qui a émergé dans le langage Javascript, on ne vous l'avait pas dit ?).

De son côté, **Amp** n'implémente pas tout à fait les promises : il n'y a pas de `then()` dans la version 3.0, par contre, il implémente une autre mécanique qui sont les `Futures`, conçues pour être "*attendus*" (`await()`) au sein de coroutines implémentées avec un générateur ou une fiber.

Vous avez donc d'un côté une gestion par **chaîne de promesses**, et de l'autre une gestion axé sur les **coroutines**.

Si vous avez déjà utilisé des promesses en javascript, il peut être plus simple d'utiliser **ReactPHP**, sinon la gestion par coroutine d'**Amp** permet une lecture plus **simple** et plus proche de nos pratiques PHP "*synchrones*".

Mais que ce soit ReactPHP ou Amp, il vous faudra une **EventLoop**.

ReactPHP propose un package "[react/event-loop](https://github.com/reactphp/event-loop)" tandis qu'Amp propose d'utiliser [revolt/event-loop](https://github.com/revoltphp/event-loop) qui a été initié par l'équipe d'Amp pour unifier l’écosystème PHP asynchrone autour d’un **standard** de boucle d’événements moderne.
Revolt est interopérable avec ReactPHP via un adaptateur.

### Et donc, je choisis quoi ?

Si vous souhaitez utiliser le pattern "**promises**", il n'y a pas photo, vous devez vous tourner vers React/Promises.

Mais de l'autre côté, Amp offre une écriture différente, qui pourrait sembler, pour certains, plus "*naturelle*", et je pense que vous devriez tester les deux pour voir celle qui vous convient le plus.

Par contre, pour l'EventLoop, je vous invite à vous orienter vers **Revolt** dont la volonté fédératrice pourrait payer à moyen terme.

Enfin, il y a peut-être un argument qui pourrait vous aider à choisir : Amp v3 utilise les fibers de **PHP 8.1**, ce qui n'est pas la cas de ReactPHP qui peut parfaitement tourner sur un vieux **PHP 7.1**. 

> Post-Scriptum : On n'a pas abordé la **testabilité** d'un développement asynchrone car cela fera l'object d'un prochain article.
