---
layout: default
title: "DRY at all costs: The trap of premature abstraction"
updated: 2024-11-27
category: en
---

## DRY at all costs: The trap of premature abstraction
(Published on Nov 27, 2024 - [Version française](/fr/abstraction-hative))

The 'DRY' (Don't Repeat Yourself) principle is a software development practice that aims to reduce code duplication by encouraging reuse and abstraction.
However, it’s a principle that can lead developers to create hasty abstractions too early, before fully understanding a project’s needs.
Such **hasty** abstractions may fail to adapt to future requirements and can introduce unnecessary complexity. We end up *patching* the code to fit squares into rounds.

### Case Study
Let's look at an example to better understand the problem: By running [PHPCPD](https://github.com/sebastianbergmann/phpcpd), the developer noticed that these lines appeared twice in the code, almost identically:
```php
$criteria = new ArticleCriteria()
    ->byId($id)
    ->isPublished()
    ->associatedToTags(...$tags)
;
$modifiers = new QueryModifiers
    ->orderBy(ArticleField::PublishedDate->value, Order::DESC)
    ->limit(self::ARTICLES_PER_PAGE)
    ->useUnbuffered()
;
$entities = $this->repository->query($criteria, $modifiers);

$context = new ObjectNormalizerContextBuilder()
    ->withGroups(self::PUBLIC_VIEW_GROUP)
    ->toArray()
;
$data = $this->serializer->normalize($entities, 'json', $context);
foreach ($data as $key => $values) {
    $data[$key]['slug'] = null;
    if (isset($values['label']) && trim($values['label']) !== '') {
        $data[$key]['slug'] = $this->slugger
            ->slug($values['label'])
            ->lower()
        ;
    }
}
```

In one case, `$data` was passed to build a JSON response, while in the other, it was passed to a template engine for rendering in HTML.
The only difference lay in the number of articles processed. In one case, it came from a user parameter, and in the other, it was a constant.

He naturally moved this code into a private method like this:
```php
private function articlesAsArray(int $id, array $tags, int $limit = self::ARTICLES_PER_PAGE): array {
    /// ....
}
```

Everything worked perfectly until he had to manage his client cache in his HTML response. For this, he needs to know the most recent modification date of the articles collection.
However, this data, "`updatedAt`", is not present in the result array because it's not data that belongs to the public group.
He would be tempted to add "`updatedAt`" to the public group, but that would mean this data would appear in the JSON response, which he doesn't want.
So he creates a new group "`self::FULL_DATED_GROUP`", and then modifies the code like this:

```php
private function articlesAsArray(int $id, array $tags, int $limit = self::ARTICLES_PER_PAGE, bool $withFullDatedGroup = false): array {

    /// ....
    $groups = [self::PUBLIC_VIEW_GROUP];
    if ($withFullDatedGroup === true) {
        $groups[] = self::FULL_DATED_GROUP;
    }
    $context = new ObjectNormalizerContextBuilder()
        ->withGroups($groups)
        ->toArray()
    ;
    /// ....

}
```

He can now, in the HTML response context, call the method "`$this->articlesAsArray($id, $tags, withFullDatedGroup: true)`", and then search the result for the most recent update date.
By the way, we can thank PHP for supporting named parameters, because when reading, we know what this "true" boolean is, but most importantly, we're not forced to pass the limit, which can keep its default value.

### Is the Problem Solved?

The code is working, but from a design and code cleanliness perspective, not really.

The fundamental problem is that the developer was too quick in his thinking to factorize his code, taking the repeated block in its entirety to make a method.
Actually, these are not one but two distinct blocks that are repeated.
Look closely, between the repository query and the context creation, the developer had skipped a line for clarity.
This should have been a hint: There's a code block that retrieves the data and a code block that transforms the collection into an array of values.
In fact, he should have distributed his code into two separate methods:
```php
 private function queryArticles(int $id, array $tags, int $limit): Collection {
    //...
 }
 private function normalizeArticles(Collection $articles): array {
    //...
 }
```

Notice that "`$limit`" no longer has a default value.
Generally, it's not always necessary to use default values.
Certainly, it reduces the amount of code to write, especially when in 90% of cases, the default value is kept, but it leads to code whose behavior is not explicit: When reading, the default limit is not immediately visible.

The separation into two methods has the consequence of better separating responsibilities and having more explicit code: "`articlesAsArray`" was not really explicit in what it was accomplishing.

Another benefit of this separation is that when evolving his code to retrieve the last modification date, he only needed to add a call between the two:

 ```php
 $lastModificationDate = $this->getLastModificationDate($articles);
 ```

No need to add a new group, and to extract the date from the normalized array. Especially if that array contained a formatted date that would need to be re-interpreted.

### Conclusion

Poorly used, abstraction can become a trap. As we've seen, yielding too quickly to the "DRY" temptation can result in rigid and hard-to-maintain code.

This is why I advocate for the "WET" (Write Everything Twice) approach: temporarily accept duplication, but on the third use case, start thinking about an abstraction. With three concrete cases, we have a better understanding of needs and variations.

The true value of code lies not only in its conciseness or apparent elegance, but in its ability to adapt to change.
Taking the time to reflect on responsibilities, decoupling logical blocks, and avoiding premature optimizations allows not only producing more robust code, but also improving its readability and understanding later.

Ultimately, the key is to balance simplicity, readability, and scalability, keeping in mind that abstraction is a means, not an end.