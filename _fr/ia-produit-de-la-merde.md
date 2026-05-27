---
layout: post-fr
title: "L'IA produit de la merde ?"
date: 2026-05-03
category: fr
lang: fr
---

*(Publié sur [LinkedIn](https://www.linkedin.com/pulse/lia-produit-de-la-merde-frederic-bouchery-t5gbe/) le 3 mai 2026.)*

Un développeur avec qui j'ai partagé les bancs de plusieurs conférences et dont je respecte l'avis a récemment commenté [mon précédent article](/fr/honte-de-votre-code) : *"Quand j'utilise un LLM, c'est de la merde en barres. Ça me génère un truc, et finalement je passe plus de temps à tout reprendre de zéro. C'est insupportable."*

Je comprends ce ras-le-bol. L'état actuel de l'IA générative produit souvent une étrange sensation : un code qui semble fonctionner au premier regard, mais qui s'effondre lamentablement à la première règle métier complexe. Le code généré devient **jetable**.

Mais d'où ça vient ? De l'outil ? Non. De la manière dont on l'utilise.

## Le mirage du "Prompt"

Beaucoup ont cru qu'il suffisait d'une **phrase magique** pour remplacer des mois de réflexion. *"Crée-moi une page de paiement."* C'est un simple *prompt*. Et c'est précisément là que le piège se referme.

Quelqu'un d'expérimenté sait que la valeur de son travail ne réside pas dans l'écriture d'une boucle for. L'informatique, c'est la gestion de l'implicite. Gérer le mécanisme de *retry* quand le réseau saute. Penser à l'idempotence pour ne pas débiter deux fois le client. Anticiper les *race conditions* sur les stocks. Logguer correctement l'action pour le RGPD.

Quand on limite notre métier à du "prompting" de surface (ce qu'on appelle naïvement le [vibe coding](https://x.com/karpathy/status/1886192184808149383)), on délègue tout cet implicite critique à un outil auquel on n'a rien dit. L'IA, livrée à elle-même face au vide de notre description, comble les trous par du générique. Le résultat ? **De la merde en barres.**

## Le retour de la spécification profonde

Le bon vieux Cycle en V avait un péché capital : l'**effet tunnel**. On spécifiait pendant des mois, on codait pendant des mois, et on découvrait les malentendus bien trop tard. Mais son vrai défaut n'était pas la rigueur de la spécification — c'était le délai de **feedback**.

Avec l'IA, ce délai s'effondre. On spécifie, la machine produit en quelques minutes, on valide. L'effet tunnel disparaît. Ce qui reste, c'est la vertu du Cycle en V : l'exigence d'un besoin exprimé, cadré et incroyablement riche.

Pour piloter une IA, il faut définir les entités, tracer les limites du système, et expliciter chaque invariant (*"Un prix de panier ne peut jamais être négatif"*).

Il ne s'agit pas de rédiger 300 pages de documents Word qui prendront la poussière. Il s'agit de s'approcher de l'essence même de notre métier : la **programmation intentionnelle**.

Dans les années 90, Charles Simonyi rêvait déjà de séparer l'Intention (le "quoi" métier) de l'Implémentation (la syntaxe) — c'est ce qu'il appelait la [Programmation Intentionnelle](https://en.wikipedia.org/wiki/Intentional_programming). À l'époque, les langages ne suivaient pas. Les LLM viennent littéralement de rendre cela possible. L'IA devient le compilateur d'intention universel. Mais elle exige que nous formulions cette intention avec précision. C'est ce que j'appelle l'**ingénierie de l'intention**.

## Le mythe de l'hyper-productivité

C'est fort de ce constat que j'entends l'histoire de cette entreprise qui fanfaronne : ses Product Owners n'arrivent plus à fournir assez de travail car les développeurs et développeuses vont "**trop vite**" grâce à l'IA. Franchement, cette entreprise ferait bien de s'inquiéter. Elle va amèrement regretter cette vitesse le jour où elle découvrira la taille du tas de merde avec lequel elle devra composer en production.

Remplacer le prompt "magique" par l'ingénierie de l'intention ne vous fera pas coder 100 fois plus vite. Cela peut même prendre **plus de temps qu'avant**.

Il serait trop facile d'en faire un discours moralisateur : *"Si le code IA est mauvais, c'est de votre faute, car vous avez mal spécifié."* La réalité est plus cruelle. Une spécification parfaite est extraordinairement **coûteuse**. Poussée à l'extrême, spécifier sans faille revient ni plus ni moins... à écrire le code mentalement. Et même avec un cadrage exceptionnel, l'IA reste faillible : l'hallucination algorithmique rôde toujours.

Au lieu de taper la syntaxe, on passe du temps en amont à modéliser, puis en aval à traquer les erreurs inévitables de la machine. Pas de bond magique.

Le vrai bénéfice n'est pas la vitesse brute, c'est la **qualité** de la conception. On devient infiniment plus exigeant sur l'architecture. Notre métier ne disparaît pas, il **mute**.

## Le vrai coût cognitif

Mais voilà, si cette méthode fonctionne à merveille techniquement, elle impose un coût caché colossal que peu avouent encore : la **fatigue cognitive**.

Pendant des décennies, taper au clavier la syntaxe d'un algorithme générait un état de [**flow**](https://fr.wikipedia.org/wiki/Flow_). C'était le moment "Eurêka" de celui ou celle qui trouve enfin la bonne abstraction. Aujourd'hui, on ne tape plus, on **supervise**. L'IA génère le bloc logique en 40 secondes. On inspecte, on lit, on s'assure que rien n'est de travers.

Et lire le code d'un autre (surtout quand cet autre est une machine) est la tâche psychologiquement la plus **épuisante** de notre métier. La création génère de la dopamine. La validation, elle, génère une [**fatigue décisionnelle**](https://en.wikipedia.org/wiki/Ego_depletion) violente. Nous sommes passés du statut de créateurs passionnés à celui d'**inspecteurs des fraudes algorithmiques**. La colère de ce confrère vient de là : on a remplacé la satisfaction brute de résoudre un problème par l'ingratitude de devoir **surveiller** une machine.

## Pilote, pas passager

Faut-il pour autant jeter l'outil ? Évidemment que **non**.

Mais il faut assumer l'évolution de la posture. La fatigue que j'ai décrite — cette ingratitude de superviser plutôt que de créer — elle est **réelle**. Elle n'est pas un bug du système. C'est le signe que le métier a **changé de nature**.

Un pilote de ligne ne souffre pas de ne pas construire l'avion. Il souffre si on lui retire les commandes. Notre combat, c'est précisément ça : **rester aux commandes**. Définir le cap, poser les contraintes, valider ce qui sort, et endosser la responsabilité de ce qui part en production.

La vraie valeur d'un développeur ou d'une développeuse en 2026, ce n'est plus la **maîtrise** d'une syntaxe ou d'un framework. C'est sa capacité à **cadrer l'intention** avec précision, et à garantir la cohérence du système que la machine a assemblé.

Le mythe du "pisseur de code" hyper-productif est **révolu**. L'ère de l'architecte de l'intention ne fait que **commencer**.

## Et les juniors dans tout ça ?

C'est ma vraie inquiétude. Aujourd'hui, piloter une IA correctement requiert l'expérience d'un développeur ou d'une développeuse senior. Résultat : les signaux d'un **recul de l'embauche des juniors** se multiplient. Logique à court terme. **Catastrophique** à long terme.

Les seniors partiront en retraite. Si on n'a pas formé de **relève**, personne ne saura plus concevoir un système sans en **déléguer l'intégralité** à une machine qu'on ne comprend plus.

On ne peut pas se contenter de former des "pisseurs de prompts". Il faut inventer une nouvelle trajectoire : des développeuses et développeurs qui apprennent à **concevoir** avant de générer, à **lire** avant d'écrire, à anticiper les cas limites avant de valider le code de la machine. **Des architectes de l'intention, formés dès le départ.**

C'est le vrai chantier qui s'ouvre. Et il commence **maintenant**.
