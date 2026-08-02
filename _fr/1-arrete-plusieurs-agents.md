---
layout: post-fr
title: "J'ai arrêté de lancer plusieurs agents."
date: 2026-08-02
category: fr
lang: fr
permalink: /fr/arrete-plusieurs-agents.html
description: "Attendre n'est pas gaspiller. Je comblais chaque seconde de génération en ouvrant un agent de plus, jusqu'à devenir le seul point par lequel passaient toutes leurs questions. Ce que ça m'a coûté, et ce que j'ai décidé d'arrêter."
---

# J'ai arrêté de lancer plusieurs agents
(Publié le 2 août 2026 - [English version](/en/stopped-running-several-agents))

Hier après-midi, j'avais quatre agents en train de travailler et je passais mon temps à répondre à leurs questions, jusqu'au moment où je n'ai plus su lequel attendait quoi.

Ma façon de travailler avec l'IA n'a rien d'improvisé, et je l'ai décrite en détail le jour où [je suis devenu un développeur Markdown](/fr/developpeur-markdown). Je passe désormais l'essentiel de mon temps à spécifier, je fais écrire des tests d'acceptance raccords avec la spécification, et je ne lance l'implémentation en autonomie complète qu'une fois ces deux étapes tenues. L'agent part alors pour quelques minutes, parfois plusieurs dizaines, sans avoir besoin de moi, sauf que je me retrouve devant un terminal qui tourne, les bras croisés, et le réflexe qui suit est d'ouvrir un autre agent sur une autre tâche, puis un autre.

Personne ne m'a demandé de faire ça, et c'est bien ce qui me gêne, parce que cette dérive n'est pas le prix d'un mauvais cadrage : elle se nourrit du temps libre qu'un bon cadrage me rend.

Le plus embarrassant, c'est que je connais la théorie sur la limitation de l'en-cours (la [loi de Little](https://fr.wikipedia.org/wiki/Th%C3%A9orie_des_files_d%27attente#Loi_de_Little)), et le "stop starting, start finishing" fait partie du décor depuis longtemps. Je n'ai simplement jamais été bon pour les appliquer, la gestion des tickets n'a jamais été mon point fort, et il a suffi que les agents arrivent dans mon shell pour que ce défaut se mette à coûter cher.

Ce qui suit est mécanique, puisque chaque agent finit par buter sur une décision qu'il ne peut pas prendre seul, donc il s'arrête et il me pose une question. Avec quatre agents, je me retrouve d'astreinte sans l'avoir décidé, à dépiler des questions une par une sans jamais voir la file se vider.

Ce qui m'a vidé, c'est cette pression continue de quatre interlocuteurs qui attendent une réponse de moi pour avancer, une charge que je n'aurais jamais acceptée de la part d'une équipe humaine.

## Le goulot avait déménagé sans que je le remarque

La [théorie des contraintes](https://fr.wikipedia.org/wiki/Th%C3%A9orie_des_contraintes) est impitoyable : optimiser une étape en amont du goulot ne produit rien en sortie, ça accumule juste du stock devant la contrainte.

Dans un atelier de meubles où la découpe sort 100 planches par heure, le ponçage à la main en traite 10 et la peinture en absorbe 100, la production plafonne à 10 meubles par heure, et cette réalité physique se fout complètement des deux postes rapides. Si j'achète une découpeuse laser qui monte à 1 000 planches par heure, la sortie ne bouge pas d'un meuble, mais l'atelier se retrouve noyé sous 990 planches qui occupent les allées et se voilent en attendant leur tour. Voilà exactement à quoi ressemble du stock devant la contrainte.

## Dans le logiciel, le stock est invisible

Personne ne trébuche sur les planches. Toute ma vie, ma contrainte a été l'écriture, c'est-à-dire taper de la syntaxe et pondre du code, et l'IA a plié le problème, si bien que 500 lignes ne me coûtent aujourd'hui plus rien à produire. La contrainte a immédiatement sauté sur la seule ressource qui ne passe pas à l'échelle, à savoir, moi avec ma capacité de lecture et d'arbitrage.

Générer dix fois plus vite ne débloque donc rien, ça empile juste du stock devant moi : des branches ouvertes partout, des PR qui moisissent et une journée entière passée à arbitrer des propositions que je n'ai pas le temps de comprendre. J'ai fabriqué un embouteillage dont je suis le seul carrefour.

Quand on gave un poste déjà saturé, on déclenche une panne : sur un serveur, les requêtes finissent en timeout, et sur un humain, ça s'appelle de la [fatigue décisionnelle](/fr/ia-produit-de-la-merde). J'en parlais déjà en constatant que lire du code produit par une machine est la tâche la plus épuisante du métier, et ça finit en bugs qu'on ne voit plus passer.

## Quatre agents qui remontent tous vers moi

Ajouter des développeurs à un projet en retard le retarde encore plus, tout le monde connaît la [loi de Brooks](https://fr.wikipedia.org/wiki/Loi_de_Brooks), et la mécanique est simple : le coût de coordination grimpe plus vite que la production. Sauf qu'avec quatre agents lancés en parallèle, une partie du travail se recouvre forcément, et chaque conflit qui demande une décision remonte vers moi, ce qui fait de moi le bus de données de ma propre équipe augmentée.

Pendant que l'un refactorise le module d'authentification, l'autre écrit un composant contre l'ancienne signature et le troisième monte une dépendance qui casse les types du premier. Comme ils travaillent tous dans le même dépôt, chacun finit par remarquer des fichiers qu'il n'a pas touchés. Git sait fusionner du texte et les agents savent se rattraper seuls la plupart du temps, mais chaque rattrapage leur donne une raison de plus de venir me demander mon avis, et c'est ainsi que le coût de coordination se paie en interruptions.

Les lancer sur des projets différents supprime les conflits sans régler le problème, parce que chaque question m'oblige alors à recharger un projet entier dans ma tête : sur un même dépôt, je mélange les choses, et sur des dépôts séparés, chaque arbitrage devient plus lent. Dans les deux cas, tout remonte vers le même cerveau.

## Le temps qu'ils me rendent arrive en miettes

Quatre agents ne me libèrent pas un bloc propre de deux heures, ils me crachent quarante fenêtres de trois minutes.

Or on ne conçoit pas une architecture par tranches de 180 secondes, pas plus qu'on ne fait de veille utile, et le temps haché ne produit rien d'autre que du scroll. C'est pour ça que la liste des bonnes résolutions ne tient jamais, celle où l'on se dit "pendant que l'agent bosse, je réponds à mes 548 emails en attente de lecture, je fais ma veille et je mets le board à jour", parce qu'elle suppose des plages de temps continues et que quatre agents en parallèle ne m'en laissent aucune.

La nuance est décisive, parce que la longueur de ces plages dépend entièrement de la façon dont j'ai découpé le travail : une tâche unique, spécifiée et couverte par ses tests d'acceptance, me rend ce temps d'un seul bloc, alors que quatre tâches lancées en parallèle me rendent exactement la même durée en confettis. Le problème n'a jamais été la durée totale, c'est la taille du plus grand morceau continu.

Il faut aussi se méfier de la revue de code pendant ce temps-là, parce que relire le code d'une tâche pendant qu'un agent en produit une autre reste du context switching, juste avec un agent en moins. Elle mobilise la même ressource que tout le reste : ma capacité de lecture et d'arbitrage. La revue de code, c'est du développement.

## Intervenir tout de suite coûte trois secondes

Combler chaque seconde de génération en ouvrant une autre tâche, c'est du productivisme managérial mal placé, le même qui reproche à un opérateur de rester debout devant une machine qui tourne.

Suivre l'agent sur la tâche que je viens de lui confier est une autre affaire, puisque je reste dans le même contexte, et c'est même la seule activité qui tienne vraiment dans une fenêtre de trois minutes. C'est là que je vois l'absurdité en direct, quand l'agent s'engage dans une impasse, quand il importe un package de 50 Mo pour une seule fonction utilitaire ou quand il glisse en douce un mock bancal pour faire passer un test au vert.

Quand je regarde l'agent travailler, il m'arrive de l'interrompre en plein vol parce que je le vois partir dans une mauvaise direction, alors qu'avec plusieurs agents, je remonte le fil de travail en scrollant vers le haut, et je découvre après coup qu'il a perdu du temps pour rien. Le faire revenir en arrière n'est pas long, mais tout ce qu'il a produit entre-temps est à jeter, alors qu'une intervention en direct m'aurait coûté trois secondes. Encore faut-il regarder au bon moment, parce que mon agent affiche sa réflexion et ses appels d'outils pendant qu'il travaille, puis replie tout une fois la tâche terminée : pour s'apercevoir qu'il s'est trompé, il faudrait rouvrir le fil de raisonnement, ce qu'on ne fait pratiquement jamais, et la dérive passée inaperçue se paie bien plus tard, en bug. C'est très exactement la [vigilance](/fr/honte-de-votre-code) dont je disais qu'elle n'a rien d'une méfiance paranoïaque puisqu'elle est le travail normal de quelqu'un qui comprend ce qu'il commite. Ça ne fait pas pour autant de la surveillance un projet de journée, parce que ces dérives relèvent de ma chaîne de validation, c'est-à-dire des tests d'acceptance et de tous les contrôles automatiques qui bloquent une livraison sans mon avis.

## Spécifier plus gros, pas prompter plus vite

Passer sa journée collé au terminal à valider des tâches de trente secondes, c'est du babysitting d'agent, et le levier se trouve ailleurs, dans la taille du périmètre délégué.

Une tâche mal cadrée de deux minutes m'oblige à rester devant, alors qu'une tâche au périmètre large, avec des contraintes claires et des critères d'acceptance rédigés avant, me rend un bloc continu. Le coût d'entrée est réel, car cette tâche-là, il faut l'écrire avant, et c'est précisément le travail qu'on fuit en lançant quatre agents sur quatre demandes floues.

## Quand le multi-agent tiendra debout

Quand je décrivais [l'architecture multi-agents](/fr/llm-probabilite-orchestration), le chef d'orchestre était un processus qui délègue à des agents spécialisés et indépendants, et c'est précisément là que j'ai dévié, puisque dans mon terminal ce chef d'orchestre, c'était moi, à la main. Le multi-agent tiendra debout le jour où il n'aura plus besoin de moi pour valider, et il s'écroule tant que je reste la seule boucle. Quand ma chaîne de validation rejettera les dérives sans passer par mon cerveau, le goulot se déplacera encore et le parallélisme redeviendra une option défendable, si bien que la bonne question porte sur la charge qu'elle encaisse sans moi.

Ceux qui tiennent dix agents sans finir en bouillie n'ont pas plus d'endurance que moi, ils ont juste une meilleure chaîne de validation, et c'est un autre sujet.

En attendant, j'ai arrêté d'en lancer plusieurs, et je ne prétendrai pas que ce soit confortable, parce que la tentation revient chaque fois que l'agent part pour un quart d'heure et qu'un terminal qui tourne pendant que je ne fais rien ressemble à du gaspillage.

Attendre n'est pas gaspiller. Mon incapacité à tolérer une inactivité visible est la cause réelle de ma fatigue.

Sauf que résister ne fait pas disparaître le vide, ça le rend seulement disponible, et il faut bien le remplir avec quelque chose : mettre le projet en ordre, écrire ce qui n'est consigné nulle part, trancher ce que je remettais à plus tard. Ce sont exactement les tâches où je n'ai jamais été bon, et je n'ai plus l'excuse du manque de temps pour continuer à les éviter.

La théorie des contraintes appelle ça élever la contrainte, et c'est la réponse à l'objection du goulot qu'on affame : le temps que je ne passe plus à dépiler des questions sert à augmenter la capacité du seul poste de l'atelier qui ne s'achète pas.

Reste à savoir si je vais m'y tenir, ou si je vais me trouver une bonne raison d'ouvrir un cinquième terminal.
