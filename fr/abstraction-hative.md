---
layout: default
title: DRY à tout prix : Le piège de l’abstraction prématurée
updated: 2024-11-27
category: posts_fr
---

## DRY à tout prix : Le piège de l’abstraction prématurée
(Publié le 27 nov 2024 - [English version](/en/hasty-abstraction))

Le principe "DRY" (Don't Repeat Yourself) est une pratique de développement qui vise à réduire la duplication de code en favorisant la réutilisation et l'abstraction.
Seulement, c'est une méthode qui peut conduire les développeurs à créer des abstractions trop tôt, avant même de comprendre pleinement les besoins et les exigences du projet.
Ces abstractions **hâtives** risquent donc de ne pas s'adapter aux besoins futurs et introduisent une complexité inutile. On en vient à *patcher* le code pour faire entrer des carrés dans des ronds.

### Étude de cas
Voyons un exemple pour mieux comprendre le problème : En lançant [PHPCPD](https://github.com/sebastianbergmann/phpcpd). le développeur a constaté que ces lignes apparaissaient deux fois dans son code, de façon pratiquement identique : 
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

Dans un premier cas, `$data` était passé à la construction d'une réponse json, et dans l'autre cas, `$data` était envoyé à un moteur de template pour ensuite être affiché en HTML.
Il y avait juste une petite différence dans la limite du nombre d'articles. Dans un cas, cela venait d'un paramètre utilisateur et dans l'autre, c'était la constante.

Il a tout naturellement déplacé ce code dans une méthode privée comme cela :
```php
private function articlesAsArray(int $id, array $tags, int $limit = self::ARTICLES_PER_PAGE): array {
    /// ....
}
```

Tout fonctionnait parfaitement jusqu'à ce qu'il doive gérer son cache client, dans sa réponse HTML. Pour cela, il doit connaître la date de modification la plus récente de la liste des articles.
Or, cette donnée, "`updatedAt`", n'est pas présente dans le tableau de résultat, car ce n'est pas une donnée qui appartient au groupe public.
Il serait tenté d'ajouter "`updatedAt`" au groupe public, mais cela voudrait dire que cette donnée se retrouverait dans la réponse Json, et il ne le souhaite pas.
Il crée donc un nouveau groupe "`self::FULL_DATED_GROUP`", puis il va modifier le code comme cela : 

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

Il peut donc, dans le contexte de réponse HTML, appeler la méthode "`$this->articlesAsArray($id, $tags, withFullDatedGroup: true)`", puis rechercher dans le résultat la date de mise à jour la plus récente.
Au passage, on remerciera PHP de supporter les paramètres nommés, car en lisant, on sait à quoi correspond ce booléen "true", mais surtout, on n'est pas obligé de passer la limite, qui peut conserver sa valeur par défaut.

### Le problème est-il réglé ?

Le code est fonctionnel, mais du point de vue de la conception et de la propreté du code, pas vraiment.

Le problème à la base, c'est que le développeur a été trop vite dans sa réflexion pour factoriser son code, il a repris le bloc répété dans son intégralité pour en faire une méthode.
Seulement, en réalité, ce ne sont pas un mais deux blocs distincts qui sont répétés.
Regardez bien, entre la requête au repository et la création du contexte, le développeur avait sauté une ligne pour plus de clarté.
Cela aurait dû lui mettre la puce à l'oreille : Il y a un bloc de code qui récupère la donnée et un bloc de code qui transforme la collection en un tableau de valeurs.
En fait, il aurait dû répartir son code dans deux méthodes séparées : 
```php
 private function queryArticles(int $id, array $tags, int $limit): Collection {
    //...
 }
 private function normalizeArticles(Collection $articles): array {
    //...
 }
```

Au passage, vous noterez que "`$limit`" n'a plus de valeur par défaut.
De manière générale, il n'est pas toujours nécessaire d'avoir recours à des valeurs par défaut.
Certes, cela réduit la quantité de code à écrire, surtout quand dans 90% des cas, on garde la valeur par défaut, mais cela conduit à avoir un code dont le comportement n'est pas explicite : À la lecture, la limite par défaut n’est pas immédiatement visible.

La séparation en deux méthode a pour conséquence de mieux séparer les responsabilités et d'avoir un code plus explicite : "`articlesAsArray`" n'était pas franchement explicite dans ce qu'il accomplissait.

Un autre bénéfice de cette séparation, c'est qu'au moment de faire évoluer son code, pour récupérer la date de dernière modification, il avait juste à ajouter un appel entre les deux :

 ```php
 $lastModificationDate = $this->getLastModificationDate($articles);
 ```

Pas besoin d'ajouter un nouveau groupe, et de venir extraire la date dans le tableau normalisé. Surtout si ce tableau contenait une date formatée qu'il fallait ré-interpréter.

### Conclusion

Mal employée, l'abstraction peut devenir un piège. Comme nous l'avons vu, céder trop vite à la tentation du "DRY" peut aboutir à un code rigide et difficile à maintenir.

C'est pour cette raison que j'encourage l'approche "WET" (Write Everything Twice) : on accepte temporairement une duplication, mais au troisième cas d’usage, on commence à réfléchir à une abstraction. Avec trois cas concrets, on dispose d’une meilleure compréhension des besoins et des variations.

La vraie valeur du code ne réside pas seulement dans sa concision ou son élégance apparente, mais dans sa capacité à s’adapter au changement.
Prendre le temps de réfléchir aux responsabilités, découpler les blocs logiques et éviter les optimisations prématurées permet non seulement de produire un code plus robuste, mais aussi d'améliorer sa lisibilité et sa compréhension plus tard.

En fin de compte, la clé est de savoir équilibrer simplicité, lisibilité et évolutivité, en gardant à l’esprit que l’abstraction est un moyen, et non une fin.
