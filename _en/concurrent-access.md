---
layout: post-en
title: "Exploring Concurrent Access Handling"
date: 2025-02-09
category: en
lang: en
---

# Exploring Concurrent Access Handling
(Published on Feb 9, 2025 - [Version française](/fr/acces-concurrents))

In this article, I’ll discuss concurrent access and how to manage it using PHP and MySQL.

Imagine you have a process that takes some time to complete.
To prevent HTTP requests from timing out, you decide to create a "`tasks`" table where you insert jobs to be processed asynchronously.
An external process then monitors this table for pending tasks, executes them, and removes the corresponding row before checking for new tasks.
This cycle continues indefinitely.

Here's a simple implementation:
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
    sleep(10); // Long task

    $cnx->query("DELETE FROM tasks WHERE id = " . $task['id']);
}
```

## Observability

While this solution is functional, it lacks observability.
Without monitoring console logs, it becomes challenging to track the currently processed task or review execution history.
For a deeper dive into observability, check out [Smaine Milianni’s video on observability](https://www.youtube.com/watch?v=BwoJYLPXUoQ) (French).

To improve this, we can introduce a "`state`" field to track task status and store execution metadata instead of deleting rows.
The updated table structure looks like this:

| Field   | Type               |
| ------- | ------------------ |
| id      | INTEGER auto-inc   |
| state   | TINYINT            |
| task    | JSON               |
| stats   | JSON               |

The "state" field follows this convention:
 - 0: Pending
 - 1: In progress
 - 2: Processed

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
    sleep(10); // Long task
    $stats = json_encode(['duration' => round((microtime(true) - $time) * 1000)]);
    $cnx->query("UPDATE tasks SET state = 2, stats = '{$stats}' WHERE id = " . $task['id']);
}
```

## Parallelization

This approach works well until the task volume grows, necessitating multiple processes running in parallel.

And that’s when we enter the world of **concurrent access**.

Although PHP is highly performant, there can be a delay of a few microseconds or even milliseconds between executing the `SELECT` query and the `UPDATE`, especially when the database is under load.
In this very short delay, another process could select the same task before the update occurs.
The probability of this happening is quite low, but this results in the task being executed twice, which can be problematic for certain operations (e.g., duplicate shipments).

## Using a reservation ID

A common approach is the reservation technique, which involves generating a random UUID and assigning it to a task via an `UPDATE`.
In a way, we reverse the previous approach: First an `UPDATE`, followed by a `SELECT`: 
```sql
UPDATE tasks SET reservationId = '{$uuid}', state = 1 WHERE state = 0 LIMIT 1
```
Then we perform:
```sql
SELECT id, data FROM tasks WHERE reservationId = '{$uuid}'
```
This method reduces the risk significantly but doesn’t eliminate it entirely.
Two concurrent UPDATE queries might still select the same task before the state is updated.
Admittedly, the probability of collision is much lower than with the previous technique, but it is not zero.
However, only one execution will succeed, as the first reservation ID will be overwritten, preventing duplicate processing.
When we try to retrieve the task information with the first ID, it won’t be found in the table.
At that point, a new reservation will need to be made.

## Locking

To completely prevent concurrent selection, we can employ locking mechanisms.
In our case, we’ll revisit the previous code where we did a `SELECT` before the `UPDATE`, but we’ll use a `SELECT FOR UPDATE` instead.
This query enables us to read and simultaneously lock the selected row, preventing another `SELECT FOR UPDATE` from accessing the same row.
Here’s a simplified example, with a 5-second pause between the `SELECT` and the `UPDATE` to facilitate concurrent access:
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

If you run this script in two terminals at the same time, assuming there are two pending tasks, both outputs will be identical, as they will both process the same task ID:
```
Found task 1. Sleep 5s
UPDATE state to 1
SLEEP 3s
UPDATE state to 2
```
## Transaction 

However, this didn’t work as expected because the `SELECT FOR UPDATE` must be executed within a transaction.
Once corrected, it looks like this:
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

Now, when running the script in two separate terminals, you’ll observe that the second terminal waits for the `SELECT` query to complete and retrieves the ID of the second task only after the first terminal triggers its `commit`.

Warning: With MySQL, you might encounter an error like: `Deadlock found when trying to get lock`.
This is related to the fact that transactions are trying to access resources but in a different order.
Indeed, the selection query does not specify an order, and a database does not necessarily store data in the order it receives it, as deleting a row could lead the engine to reuse the freed space.
From a functional perspective, this is already not ideal, as we might end up launching a task that was just inserted while others have been waiting longer.
And from a technical perspective, the lake of order will generate errors when using locks.

We’ve solved the concurrent access problem, but we have another issue.
Imagine that the task processing generates a fatal error and the script abruptly stops.
The task that was being processed will not be relaunched when we run our script again, because the `state` field is set to "1" ("In progress").
Since we have a transaction, we can use it by triggering the `commit` after changing the state to "2" ("Processed").
This way, if the script stops abruptly, the transaction not being validated will be canceled, and the state will revert to "0" ("Pending").
We can also add a `try/catch` to trigger a `rollback` in case of an issue.

## Avoiding Wait Times

While this solution is effective, it introduces a new issue: the second script must wait for the first one’s `commit` to complete its selection query.
This effectively reverts us to sequential processing, which defeats the purpose of running multiple scripts for parallelization.

To solve this, there’s a very simple keyword: `SKIP LOCKED`.
The query becomes `SELECT id, data FROM tasks WHERE state = 0 ORDER BY id LIMIT 1 FOR UPDATE SKIP LOCKED`.
Note the use of `ORDER BY`, which becomes essential in this kind of query with locks.
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
    sleep(10); // Long task
    $stats = json_encode(['duration' => round((microtime(true) - $time) * 1000)]);
    $cnx->query("UPDATE tasks SET state = 2, stats = '{$stats}' WHERE id = " . $task['id']);
    $cnx->commit();
}
```

## Reading Uncommitted Transactions

However, there’s one final issue we need to address, related to observability: retrieving the list of tasks currently being processed.

To solve our observability issue, we have a state "In progress," and the following script lists tasks where the `state` field is set to "1":
```php
<?php
$cnx = new mysqli('localhost', 'root', 'root', 'test');
$result = $cnx->query('SELECT id FROM tasks WHERE state = 1 ORDER BY id')->fetch_all(MYSQLI_ASSOC);
echo "Task(s) being processed:\n";
foreach ($result as $row) {
    echo " - {$row['id']}\n";
}
```
However, we never see the tasks being processed.
This is expected behavior since the transaction remains uncommitted, preventing us from observing the state change.
In the end, we can only see tasks in the table with state "0" or "2," but never "1."

To be able to read uncommitted changes in a transaction, we need to add a directive before our `SELECT` to indicate that we want to read uncommitted transactions, which gives:
```php
<?php
$cnx = new mysqli('localhost', 'root', 'root', 'test');
$cnx->query('SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED');
$result = $cnx->query('SELECT id FROM tasks WHERE state = 1 ORDER BY id')->fetch_all(MYSQLI_ASSOC);
echo "Task(s) being processed:\n";
foreach ($result as $row) {
    echo " - {$row['id']}\n";
}
```

## Stay Pragmatic

While database locks are powerful, they can introduce complexities and risks, such as deadlocks or unintended sequentialization of processes.
Therefore, it’s essential to stay pragmatic and carefully evaluate the criticality of your needs before implementing such mechanisms.

In some scenarios, a simpler approach—such as the reservation technique or tolerating a low risk of double execution—may be sufficient.
It all depends on the context and the potential consequences of a collision.
For example, if double execution doesn’t lead to critical issues, it might be preferable to opt for a less restrictive approach, while keeping an eye on system performance and observability.

In conclusion, before implementing locks or other complex mechanisms, consider the following:
What is your system’s error tolerance? 
What are the implications of double execution or unmanaged concurrent access?
Based on the answers, you can choose the most suitable solution, balancing simplicity, performance, and reliability.
