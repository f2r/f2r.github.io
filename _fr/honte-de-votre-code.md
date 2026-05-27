---
layout: post-fr
title: "Depuis quand avez-vous honte de votre code ?"
date: 2026-03-31
category: fr
lang: fr
---

*(Publié sur [LinkedIn](https://www.linkedin.com/pulse/depuis-quand-avez-vous-honte-de-votre-code-frederic-bouchery-oo8ce/) le 31 mars 2026.)*

J'ai une scène qui se répète dans mes revues de code. Un développeur présente son travail. Il explique les choix, l'architecture, les compromis. Et à un moment, il y a une hésitation. Un léger flottement. Puis la phrase sort, à voix basse, presque en s'excusant : *"Cette partie-là... c'est pas moi qui l'ai écrite. C'est l'IA."*

Comme s'il avouait quelque chose. D'où ça vient, cette honte ? De deux endroits, je crois.

Le premier, c'est intérieur. Pendant des décennies, la valeur d'un développeur s'est mesurée à sa capacité à mémoriser de la syntaxe, à résoudre des problèmes complexes à partir d'une page blanche, à forger chaque ligne à la force du poignet. C'est cette image qui a construit son identité professionnelle. Et quand l'IA automatise cette forge, certains y voient une perte : si je ne tape plus le code, est-ce que je suis encore légitime ? Est-ce que je mérite mon titre, mon salaire, la crédibilité que mes collègues me donnent ? Le développeur a l'impression d'avoir utilisé un cheat code, un raccourci honteux, comme si la valeur de son travail se mesurait au nombre de lignes tapées à la main.

Le second, c'est hiérarchique. La crainte, souvent infondée mais bien réelle, que le manager pense : "si l'IA fait ton boulot, pourquoi je te paie ?" Cette peur-là, elle ne se dit pas à voix haute. Elle se manifeste exactement comme dans la scène que je décrivais : une hésitation, des mots qui s'excusent.

Je vais être direct : cette honte me met hors de moi. Pas contre eux, contre le discours ambiant qui l'a installée.

## Ce que j'ai dit à mon équipe, clairement, une fois pour toutes

Je ne vous en voudrai **jamais** d'avoir utilisé une IA pour écrire du code.

Par contre, je vous en voudrai **beaucoup** si vous passez trois heures à écrire quelque chose qu'un agent aurait produit en quarante secondes. Ce n'est pas de l'héroïsme, c'est du gaspillage : de votre temps, de votre énergie, et franchement de votre cerveau qui mérite mieux que de taper du boilerplate.

*La question n'a jamais été qui a écrit les lignes. Elle a toujours été qui est responsable du code produit.*

## Votre code a toujours eu des milliers de parents

L'image du développeur solitaire qui forge chaque ligne a toujours été une fiction. J'ai dit un jour, lors d'une conférence sur le pragmatisme, que les développeurs étaient des généticiens. Le code qu'on écrit est un assemblage de fragments qui se transmettent de génération en génération. Parfois ce sont des chapitres entiers récupérés sur Stack Overflow. Souvent ce sont trois lignes glanées dans un projet open-source, un pattern vu dans un livre, une astuce montrée par un collègue. Le résultat n'a pas deux parents : il en a des milliers. Des milliers de développeurs dont vous n'avez jamais croisé le chemin et dont le code vit quand même dans le vôtre.

L'IA n'a pas changé ça. Elle a juste accéléré la transmission, et apparemment ça dérange.

## Pilote, pas passager

Une IA fera un super boulot si un super développeur l'accompagne. C'est un travail d'équipe, pas une délégation, pas une sous-traitance. Un binôme où l'un génère vite et l'autre pense juste. Et dans ce binôme, c'est vous qui êtes le pilote, pas parce que c'est une belle métaphore, mais parce que c'est vous qui signez, vous qui déployez, et vous qui répondez quand ça part en production à 3h du matin.

Une IA qui écrit du code sans qu'un développeur compétent accompagne le travail, c'est une source de dette technique silencieuse. Elle ne connaît pas votre contexte métier. Elle ne sait pas pourquoi vous avez fait ce choix d'architecture il y a six mois. Elle ne voit pas les implications sur les modules adjacents. Elle produit quelque chose qui s'exécute et qui semble correct, et ces deux qualités réunies sont exactement ce qui rend le problème invisible jusqu'au jour où ça explose.

Restez vigilants, même sur trois lignes qui ont l'air simples. Les LLMs suggèrent régulièrement des bibliothèques plausibles mais inexistantes. [Une étude universitaire de 2025](https://arxiv.org/abs/2406.10279) (Université du Texas, Oklahoma et Virginia Tech) portant sur 576 000 échantillons de code généré par 16 modèles différents a établi que 19,7 % des paquets recommandés n'existaient dans aucun registre public. Des attaquants enregistrent ces noms hallucinés pour y déposer du code malveillant : c'est le ["slopsquatting"](https://en.wikipedia.org/wiki/Slopsquatting) et c'est actif. Trois lignes d'import non vérifiées peuvent ouvrir une porte dérobée. La vigilance n'est pas de la méfiance paranoïaque. C'est le travail normal d'un ingénieur qui comprend ce qu'il commite.

## Plus on l'utilise, moins on lui fait confiance

Le [Stack Overflow Developer Survey 2025](https://survey.stackoverflow.co/2025/ai) (49 000 développeurs, 177 pays) est cité partout pour sa stat d'ouverture : 84 % des développeurs utilisent ou prévoient d'utiliser des outils IA, contre 76 % l'année précédente. Formidable. Adoption en hausse.

Ce qu'on cite beaucoup moins : dans le même rapport, la confiance dans les sorties de ces outils est tombée à 29 %. Elle était à 40 % l'année d'avant. Le sentiment favorable global est passé de plus de 70 % en 2023-2024 à 60 % en 2025. La frustration numéro un citée par 66 % des répondants : recevoir des solutions "presque correctes mais pas tout à fait". Frustration numéro deux : déboguer du code généré par IA prend plus de temps que de l'écrire soi-même (45 % des répondants).

Plus on utilise l'IA, moins on lui fait confiance. C'est exactement l'inverse de la courbe d'adoption normale d'une technologie.

[L'étude METR publiée en juillet 2025](https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/) va dans le même sens. Un essai contrôlé randomisé sur 16 développeurs expérimentés, 246 tâches réelles : avec accès aux meilleurs outils du marché ([Cursor Pro](https://www.cursor.com/) avec Claude 3.5/3.7), ces développeurs ont pris en moyenne 19 % de temps en plus que sans IA. Le détail le plus troublant : après l'expérience, ils croyaient encore avoir été 20 % plus rapides. La perception était systématiquement déconnectée de la réalité mesurée.

Ce résultat est contextualisé (grandes codebases matures, développeurs très seniors) et METR elle-même met en garde contre les surgénéralisations. Mais il pointe vers quelque chose de réel : le temps économisé à la génération se retrouve souvent dépensé ailleurs, en vérification, en correction, en intégration. Ce n'est pas un argument contre l'IA. C'est un argument pour rester les yeux ouverts.

## La question qu'on évite

Est-ce que les développeurs qui apprennent aujourd'hui en s'appuyant massivement sur l'IA vont acquérir les réflexes dont ils auront besoin quand l'outil sera en panne, déprécié, ou simplement mauvais sur leur problème spécifique ?

Je ne sais pas. Personne ne le sait encore, personne n'a suivi ces développeurs dans la durée. Mais c'est cette question qui m'inquiète vraiment, bien plus que la honte de présenter du code assisté. Un développeur qui s'excuse d'avoir utilisé l'IA, ça se corrige en cinq minutes de conversation. Un développeur qui ne sait plus lire du code qu'il n'a pas généré lui-même, c'est un problème structurel.

## Donc

Arrêtez de vous excuser. Ce code, c'est le vôtre, avec tout ce que ça implique de fierté et de responsabilité.

Utilisez les outils. Pilotez-les. Lisez ce qu'ils produisent. Comprenez-le assez pour l'expliquer à un collègue ou le défendre en revue. Et si vous ne pouvez pas faire ça, le problème n'est pas l'IA : c'est que vous n'avez pas vraiment fait votre travail.
