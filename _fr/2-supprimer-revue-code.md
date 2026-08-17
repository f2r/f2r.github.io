---
layout: post-fr
title: "Ceux qui ont supprimé la revue de code ont raison."
date: 2026-08-17
category: fr
lang: fr
published: true
permalink: /fr/supprimer-revue-code.html
description: "Des développeurs sont arrivés sur ma MR avec une IA qui faisait la revue à leur place. Ceux qui ont carrément supprimé la revue sont presque plus honnêtes, parce qu'ils ont vu qu'elle avait perdu la course. Reste à savoir ce qu'ils ont mis à la place, et la plupart n'ont rien mis."
---

# Ceux qui ont supprimé la revue de code ont raison
(Publié le 17 août 2026 - [English version](/en/removed-code-review))

Ils ont raison sur le constat, mais pour de mauvaises raisons, et les conséquences ne ressemblent pas à ce qu'ils imaginent.

Le raccourci se répand vite : le code sort en trois secondes, donc les garde-fous passent pour de la bureaucratie, la PR devient un frein, la revue par un pair une politesse coûteuse et le test d'intégration un luxe de frileux. On disait "ça compile, donc ça fonctionne" pour se moquer des collègues trop sûrs d'eux, mais j'en ai croisé assez pour savoir que beaucoup s'arrêtaient sincèrement là, et l'IA vient de rendre leur méthode présentable : ça compile, les tests générés sont au vert, donc go MEP.

La moitié de ce raisonnement mérite d'être défendue, et l'autre d'être enterrée.

## La revue humaine a perdu la course, et c'est arithmétique

Un relecteur attentif traite [quelques centaines de lignes par heure](https://smartbear.com/learn/code-review/best-practices-for-peer-code-review/), et au-delà de ce rythme, son taux de détection des défauts s'effondre, une limite documentée depuis les [inspections de code des années 70](https://en.wikipedia.org/wiki/Fagan_inspection) que l'IA n'a évidemment pas repoussée. Un agent produit ce volume en quelques secondes, si bien que le code s'écrit désormais cent fois plus vite qu'il ne se relit, et aucune équipe ne comble un écart pareil en s'organisant mieux.

Il ne reste alors que deux issues : soit votre équipe relit réellement, et la revue devient [le nouveau goulot](/fr/arrete-plusieurs-agents), avec des PR qui moisissent cinq jours pendant que trois autres se périment derrière, soit elle fait semblant. C'est là que ça devient grave, parce qu'une revue de complaisance est pire que pas de revue du tout : sans revue, tout le monde sait que le risque existe, alors qu'avec une approbation, on fabrique de l'assurance, et deux relecteurs qui scrollent une PR de 900 lignes en quatre minutes ne produisent rien d'autre qu'une trace d'audit.

J'ai vu le théâtre se moderniser dans ma propre équipe, où le volume de code produit a explosé pendant que la relecture détaillée s'amenuisait, faute de pouvoir suivre : des développeurs passent une IA pour faire leur passe de revue, et certains arrivent sur ma MR avec un agent qui relit à leur place. Je leur ai dit en réunion que faire la revue avec une IA, c'est lui faire aveuglément confiance, et que puisque c'est déjà elle qui produit le code, autant arrêter de s'emmerder avec la revue : ça compile, donc ça marche, go prod. Ce que je leur reprochais, c'était d'approuver sans avoir jugé : leur agent rend un avis qui change à chaque exécution, personne n'en répond, et l'approbation reste signée d'un humain qui n'a rien regardé.

Supprimer volontairement la revue a même une vertu que le théâtre n'aura jamais : l'absence assumée met l'équipe au pied du mur, et il faut bien trouver les outils et les méthodes qui permettent de mettre en production sans cette phase. Faire semblant supprime cette pression sans supprimer le risque : le code part en production sans avoir été regardé, et comme la case est cochée, personne ne construira jamais les outils qui manquent.

Ceux qui ont supprimé la revue ont vu ça, et sur le constat, ils ont raison.

## Ils se sont trompés sur ce que la revue attrapait

L'erreur n'est pas la suppression, c'est le vide laissé derrière, et pour le voir, il faut se demander honnêtement ce que la revue attrapait en pratique, plutôt que ce qu'elle était censée attraper sur le papier.

La syntaxe, le style et les erreurs de type étaient déjà automatisés bien avant l'IA, par le compilateur, le check style ou l'analyse statique selon les langages. Ce que la revue attrapait, c'est un écart d'intention, celui qui se manifeste par "attends, pourquoi tu passes par là ?", par "ce cas-là, il arrive quand exactement ?", par "ce nom promet une chose et la méthode en fait une autre" ou par "on avait décidé l'inverse il y a trois mois, tu étais en congé".

Rien de tout ça n'est vérifiable par un compilateur, et rien de tout ça n'a disparu avec l'IA. C'est même pire, puisqu'un modèle produit du code plausible, bien formé, cohérent avec lui-même et parfois complètement à côté de l'intention, c'est-à-dire exactement le profil de défaut que la revue humaine attrapait, et il est devenu beaucoup plus fréquent. Supprimer la revue sans rien mettre à la place retire donc le seul contrôle qui portait sur la seule chose que la machine ne vérifie pas.

## Le compilateur ignore votre modèle économique

Le scénario type se rejoue partout sous des habillages différents. L'agent sort une fonction de remboursement au code propre, aux types alignés et aux tests de surface au vert, sauf qu'elle ne verrouille pas l'état de la commande en base avant d'appeler l'API de paiement. Le compilateur s'en fout, le linter aussi, et aucun des deux ne sait que rembourser deux fois le même client est un problème, si bien que les dix minutes de lecture économisées se paieront en réconciliation de tables et en post-mortem.

Rien là-dedans ne condamne l'IA : le scénario dit seulement où doit porter la vérification.

## Le continuous deployment est l'automatisation de la rigueur

Pousser en force parce que ça compile se réclame souvent du continuous deployment, et c'est un contresens, parce que les équipes qui déploient dix fois par jour n'ont pas retiré les vérifications : elles les ont toutes automatisées, des tests au déploiement progressif jusqu'au rollback. Chez elles, la vitesse et le contrôle ont grandi ensemble, le second étant la condition de la première, si bien que supprimer la validation en invoquant la vitesse, c'est en faire beaucoup moins que les autres en gardant le vocabulaire. Accélérer la génération sans renforcer la validation raccourcit surtout le délai entre une intention floue et un incident de production.

## Ce qui doit remplacer la revue

Le blocage humain manuel est effectivement absurde à cette cadence, et ceux qui veulent restaurer le comité d'architecture de trois heures ou la PR qui attend cinq jours se battent contre l'arithmétique. Ce qui remplace la revue doit donc être automatique, déterministe, et porter sur l'intention plutôt que sur la forme.

Ça commence par des critères d'acceptance dérivés d'une spécification écrite avant le code, jamais après, parce qu'un critère rédigé après l'implémentation ne fait que décrire l'implémentation. Ça continue avec des contraintes d'architecture exécutables, où le build casse quand un contrôleur tape directement dans la base, parce qu'une convention dans un wiki ne bloque rien alors qu'un test qui échoue bloque tout. Ça passe par des linters métier, ceux que personne n'écrivait parce qu'ils coûtaient des jours de travail et qu'ils en coûtent maintenant quelques heures, ce qui en fait probablement le gain le plus sous-estimé de toute cette histoire. Et ça se termine par des tests d'intégration qui ne repassent jamais par un modèle à l'exécution, parce qu'un test qui interroge un LLM à chaque exécution n'est pas un test, c'est un avis.

Le point commun de ces quatre briques, c'est qu'elles rendent le même verdict à chaque exécution, et qu'aucune ne dépend de la disponibilité d'un humain un vendredi à 17h.

## La part qui restera humaine

Une partie du travail ne s'automatise pas, et prétendre le contraire serait malhonnête : décider si la fonctionnalité est la bonne, arbitrer une direction d'architecture ou repérer qu'une spec en contredit une autre écrite il y a six mois demande un humain, et ça en demandera longtemps. Simplement, cet humain n'a plus rien à faire au niveau du diff, parce qu'il travaille désormais un cran au-dessus. La revue de code survit en changeant d'étage : elle devient une revue de spécification, portée sur un artefact beaucoup plus court et beaucoup plus dense.

La revue faisait d'ailleurs autre chose qu'attraper des défauts : elle formait ceux qui la pratiquaient, dans les deux sens, et c'est le seul de ses services pour lequel je n'ai rien à proposer. Où se formeront ceux qui n'ont pas encore leurs années de cambouis quand plus personne ne lira de diff, je ne le sais pas.

Cette chaîne de validation, je l'ai construite brique par brique sur mon propre projet, et elle m'a emmené plus loin que je ne l'assumais au départ, puisque aujourd'hui je n'ai pas relu 1% du code qui en sort. C'est le sujet du prochain article.

Reste la question désagréable, pour ceux qui ont supprimé la revue pour aller plus vite : cette lecture de 900 lignes qu'ils ont supprimée, ils l'ont remplacée par quoi ? Dans votre équipe, la revue a-t-elle été supprimée, ou déplacée ? Si la réponse est supprimée, cherchez ce qui vérifie encore l'intention aujourd'hui, et si vous ne trouvez rien, vous n'avez pas gagné en vélocité, vous avez juste retiré les freins.
