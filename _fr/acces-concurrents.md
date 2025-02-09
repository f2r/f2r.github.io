---
layout: post-fr
title: "Jouons avec les accès concurrents"
date: 2025-02-09
category: fr
lang: fr
---

# Jouons avec les accès concurrents
(Publié le 9 fév 2025 - [English version](/en/concurrent-access))

Je voulais évoquer avec vous le problème des accès concurrents et comment résoudre cela avec PHP et MySQL.

Imaginons que vous ayez un traitement un peu *long* à faire.
Pour éviter que les requêtes HTTP ne prennent trop de temps à répondre, vous décidez de créer une table "`tasks`" dans laquelle vous venez insérer le travail que vous voulez faire, puis vous avez un processus externe qui va venir voir s'il y a des tâches à effectuer au moyen de cette table.

Une fois le travail terminé, la ligne est supprimée, et une nouvelle requête vérifie s'il reste des tâches à traiter.
Et ainsi de suite.
Voici un exemple de code illustrant cette approche:
```php
<?php
$cnx = new mysqli('localhost', 'root', 'root', 'test');

while ($cnx->error === '') {
    $task = $cnx->query('SELECT id, data FROM tasks LIMIT 1')->fetch_assoc();
    if ($task === null) {
        sleep(1);
        continue;
    }
    
    var_dump(json_decode($task['data'], true));
    sleep(10);// Tâche longue

    $cnx->query("DELETE FROM tasks WHERE id = " . $task['id']);
}
```

## Observabilité

La solution est intéressante, mais elle manque d'observabilité.
À moins de suivre les logs dans la console, il est difficile de savoir quelle tâche est en cours de traitement par exemple, et on perd l'historique de ce qu'il s'est passé.
À ce sujet, je vous invite à regarder la vidéo de [Smaine Milianni sur l'observabilité](https://www.youtube.com/watch?v=BwoJYLPXUoQ).

On va donc ajouter un champ "`state`" pour indiquer l'état d'une tâche, et on stockera aussi quelques informations statistiques sur l'exécution de la tâche.
Enfin, au lieu de supprimer la ligne, on va juste changer son état.

Voici à quoi pourrait ressembler cette table :

| Champ   | Type               |
| ------- | ------------------ |
| id      | INTEGER auto-inc   |
| state   | TINYINT            |
| task    | JSON               |
| stats   | JSON               |

Le champ state correspond à ceci :
 - 0 : En attente
 - 1 : En cours de traitement
 - 2 : Traité

```php
<?php
$cnx = new mysqli('localhost', 'root', 'root', 'test');

while ($cnx->error === '') {
    $task = $cnx->query('SELECT id, data FROM tasks WHERE state = 0 LIMIT 1')->fetch_assoc();
    if ($task === null) {
        sleep(1);
        continue;
    }
    $cnx->query("UPDATE tasks SET state = 1 WHERE id = " . $task['id']);
    $time = microtime(true);
    sleep(10); // tâche longue
    $stats = json_encode(['duration' => round((microtime(true) - $time) * 1000)]);
    $cnx->query("UPDATE tasks SET state = 2, stats = '{$stats}' WHERE id = " . $task['id']);
}
```

## Parallélisation

La solution fonctionne bien, jusqu'au jour où le volume de tâches augmente et nécessite l'exécution de plusieurs processus en parallèle.

Et là, quand on commence à travailler avec des processus en parallèle, on entre dans l'univers des accès concurrents.

Même si PHP est très performant, entre le moment où on fait la requête `SELECT` et le moment où s'exécute l'`UPDATE`, il peut s'écouler quelques microsecondes, voire même quelques millisecondes si la base de données est un peu chargée.

Dans ce très court intervalle, un autre processus peut lancer à son tour la requête `SELECT`, et comme l'état n'a pas encore changé, au final, on récupère le même identifiant.
La tâche sera donc exécutée deux fois.
La probabilité que ce cas se produise est assez faible, mais si une double exécution peut générer des dysfonctionnements critiques, comme le double envoi d'un colis par exemple, il ne doit pas être négligé.

## Mécanisme de réservation

Une solution intéressante consiste à réserver une ligne dans la table avant d'y accéder.
Cela consiste à créer un identifiant aléatoire de type UUID et on lance un `UPDATE` avec cet identifiant.
En quelque sorte, on inverse la solution précédente : D'abord un `UPDATE` et ensuite un `SELECT`:
```sql
UPDATE tasks SET reservationId = '{$uuid}', state = 1 WHERE state = 0 LIMIT 1
```
Puis on fait :
```sql
SELECT id, data FROM tasks WHERE reservationId = '{$uuid}'
```

Avec cette technique, il y a toujours un risque d'accès concurrent au niveau de la base de données, car deux `UPDATE` lancés strictement en même temps, pourraient tous les deux sélectionner la même ligne avec l'état `0` au moment de leur exécution.

Certes, cette probabilité de collision est bien plus faible qu'avec la technique précédente, mais elle n'est pas nulle.

Toutefois, nous n'avons pas de risque de lancer deux fois la même tâche, car le premier identifiant de réservation est écrasé par le second.
Quand on va venir chercher les informations de la tâche avec le premier identifiant, on ne le trouvera pas dans la table.

À ce moment-là, il faudra relancer une nouvelle réservation.

## Verrouillage

Si l'on souhaite absolument éviter les accès concurrents, il est nécessaire d'utiliser des mécanismes de verrouillage.

Dans notre cas, on va revenir sur le code précédent où nous faisions un premier `SELECT` avant l'`UPDATE`, mais nous allons utiliser à la place un [`SELECT FOR UPDATE`](https://dev.mysql.com/doc/refman/8.4/en/innodb-locking-reads.html).

Cette requête permet de faire une lecture et en même temps, de verrouiller la ligne sélectionnée pour éviter qu'une autre requête `SELECT FOR UPDATE` prenne la même ligne.
Voici un exemple simplifié, avec une pause de 5 secondes entre le `SELECT` et l'`UPDATE` pour permettre de tomber dans un accès concurrent :
```php
<?php
$cnx = new mysqli('localhost', 'root', 'root', 'test');
$task = $cnx->query('SELECT id, data FROM tasks WHERE state = 0 LIMIT 1 FOR UPDATE')->fetch_assoc();
echo "Found task {$task['id']}. Sleep 5s\n";
sleep(5);
echo "UPDATE state to 1\n";
$cnx->query("UPDATE tasks SET state = 1 WHERE id = " . $task['id']);
echo "SLEEP 3s\n";
sleep(3);
echo "UPDATE state to 2\n";
$cnx->query("UPDATE tasks SET state = 2 WHERE id = " . $task['id']);
```

Si vous lancez ce script dans deux terminaux en même temps, en supposant qu'il y a deux tâches en attente, les 2 sorties seront identiques, car ils vont traiter tous les deux la même tâche :
```
Found task 1. Sleep 5s
UPDATE state to 1
SLEEP 3s
UPDATE state to 2
```
## Transaction 

En fait, cela n'a pas fonctionné, car le `SELECT FOR UPDATE` doit être dans une transaction.
Une fois corrigé, cela donne : 
```php
<?php
$cnx = new mysqli('localhost', 'root', 'root', 'test');
$cnx->begin_transaction();
$task = $cnx->query('SELECT id, data FROM tasks WHERE state = 0 LIMIT 1 FOR UPDATE')->fetch_assoc();
if ($task === null) die("Not found\n");
echo "Found task {$task['id']}. Sleep 5s\n";
sleep(5);
echo "UPDATE state to 1\n";
$cnx->query("UPDATE tasks SET state = 1 WHERE id = " . $task['id']);
$cnx->commit();
echo "SLEEP 3s\n";
sleep(3);
echo "UPDATE state to 2\n";
$cnx->query("UPDATE tasks SET state = 2 WHERE id = " . $task['id']);
```

Maintenant, quand vous lancez le script dans deux terminaux séparés, vous constatez que le deuxième attend que la requête `SELECT` se termine et retourne l'identifiant de la seconde tâche juste au moment où le premier déclenche son `commit`.

**Attention** : Vous pouvez rencontrer une erreur du type : `Deadlock found when trying to get lock`. 
C'est lié au fait que les transactions tentent d'accéder aux ressources, mais dans un ordre différent.

En effet, depuis le début, la requête de sélection ne spécifie pas d'ordre de classement, et une base de données ne stocke pas forcement les données dans l'ordre où elle les reçoit, car si on supprime une ligne, le moteur pourrait réutiliser la place laissée libre.

D'un point de vue fonctionnel, ce n'est déjà pas une bonne chose, car on pourrait se retrouver à lancer une tâche qui vient tout juste d'être insérée alors que d'autres sont en attente depuis plus longtemps.

Et d'un point de vue technique, l'absence d'un ordre de classement va générer des erreurs quand on souhaite utiliser des verrous.

Nous avons donc résolu le problème d'accès concurrent, mais nous avons un autre problème.

Imaginons que le traitement de la tâche génère une anomalie fatale et que le script s’interrompt brutalement.
La tâche qui était en cours de traitement ne sera pas relancée quand on exécutera à nouveau notre script", car le champ `state` est à "1" ("En cours de traitement").

Comme nous avons une transaction, nous allons pouvoir l'utiliser en déclenchant le `commit` après le changement d'état à "2" ("traité").
Ainsi, si le script s'interrompt brutalement, la transaction n'étant pas validée, elle sera annulée, et l'état reviendra à "0" ("En attente").

On peut également ajouter un `try/catch` pour lancer un `rollback` en cas de problème.

## Ne pas attendre le relâchement du verrou

C'est parfait, mais nous voilà confrontés à un nouveau problème : le second script va attendre le `commit` du premier pour terminer sa requête de sélection, ce qui n'est vraiment pas souhaité, car on en revient à séquentialiser les traitements, hors si on lance plusieurs script, c'est justement pour pouvoir paralléliser.

Pour résoudre cela, il y a un mot clef très simple [`SKIP LOCKED`](https://dev.mysql.com/doc/refman/8.4/en/innodb-locking-reads.html#innodb-locking-reads-nowait-skip-locked).
La requête devient `SELECT id, data FROM tasks WHERE state = 0 ORDER BY id LIMIT 1 FOR UPDATE SKIP LOCKED`.

Vous noterez l'usage du `ORDER BY` qui devient essentiel dans ce genre de requête avec des verrous.
```php
<?php
$cnx = new mysqli('localhost', 'root', 'root', 'test');

while ($cnx->error === '') {
    $cnx->begin_transaction();
    $task = $cnx
        ->query('SELECT id, data FROM tasks WHERE state = 0 ORDER BY id LIMIT 1 FOR UPDATE SKIP LOCKED')
        ->fetch_assoc()
    ;
    if ($task === null) {
        sleep(1);
        continue;
    }
    $cnx->query("UPDATE tasks SET state = 1 WHERE id = " . $task['id']);
    $time = microtime(true);
    sleep(10); // tâche longue
    $stats = json_encode(['duration' => round((microtime(true) - $time) * 1000)]);
    $cnx->query("UPDATE tasks SET state = 2, stats = '{$stats}' WHERE id = " . $task['id']);
    $cnx->commit();
}
```

## Lecture de transaction non terminée

Cependant, il nous reste un dernier problème à régler, que nous n'avons pas encore évoqué, en rapport avec l'observabilité : La récupération de la liste des tâches en cours de traitement.

Pour résoudre notre problème d'observabilité, nous avons un état "En cours de traitement", et le script suivant permet de lister les tâches dont le champ `state` est à "1" :
```php
<?php
$cnx = new mysqli('localhost', 'root', 'root', 'test');
$result = $cnx->query('SELECT id FROM tasks WHERE state = 1 ORDER BY id')->fetch_all(MYSQLI_ASSOC);
echo "Tâche(s) en cours de traitement :\n";
foreach ($result as $row) {
    echo " - {$row['id']}\n";
}
```
Seulement, on ne voit jamais les tâches en cours de traitement.
Ce qui est parfaitement normal, puisque nous avons ouvert une transaction, et tant qu'elle n'a pas été validée, on ne peut pas voir le changement d'état.
Au final, on ne peut voir dans la table, que des tâches dans l'état "0" ou "2", mais jamais à "1".

Pour pouvoir lire les changements non validés dans une transaction, il va falloir ajouter une directive avant de faire notre `SELECT` pour indiquer que l'on souhaite lire les transactions non-validées, ce qui donne : 
```php
<?php
$cnx = new mysqli('localhost', 'root', 'root', 'test');
$cnx->query('SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED');
$result = $cnx->query('SELECT id FROM tasks WHERE state = 1 ORDER BY id')->fetch_all(MYSQLI_ASSOC);
echo "Tâche(s) en cours de traitement :\n";
foreach ($result as $row) {
    echo " - {$row['id']}\n";
}
```

## Restez pragmatiques

L'utilisation de verrous dans une base de données, bien que puissante, peut introduire des complexités supplémentaires et des risques de comportements indésirables, tels que les deadlocks ou la séquentialisation involontaire des traitements.
Il est donc essentiel de rester pragmatique et de bien évaluer la criticité de vos besoins avant de mettre en place de tels mécanismes.

Dans certains cas, une solution plus simple, comme la technique de réservation ou même l'acceptation d'un faible risque de double exécution, peut suffire.
Tout dépend du contexte et des conséquences potentielles d'une éventuelle collision.
Par exemple, si une double exécution n'entraîne pas de dysfonctionnement critique, il peut être préférable d'opter pour une approche moins restrictive, tout en gardant un œil sur les performances et l'observabilité du système.

En résumé, avant de vous lancer dans l'implémentation de verrous ou d'autres mécanismes complexes, posez-vous les bonnes questions :
Quel est le niveau de tolérance aux erreurs de mon système ?
Quelles sont les conséquences d'une double exécution ou d'un accès concurrent non contrôlé ?
En fonction des réponses, vous pourrez choisir la solution la plus adaptée, en équilibrant simplicité, performance et fiabilité.