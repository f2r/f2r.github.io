---
layout: post-fr
title: "Je deviens un développeur Markdown"
date: 2026-05-27
category: fr
lang: fr
---

Je code depuis 1983. J'ai connu les cartes perforées, les terminaux à texte et la naissance du Web. Aujourd'hui, j'entends deux discours qui me fatiguent. D'un côté, les évangélistes du vibe coding qui vous expliquent que l'IA va remplacer le département tech. De l'autre, les puristes qui refusent de toucher à l'IA par fierté ou par peur qu'elle produise n'importe quoi.

Les deux se plantent.

Moi, je m'éclate comme jamais avec l'IA, mais ça ne veut pas dire que j'ai posé mon clavier. Mon centre de gravité a bougé. Aujourd'hui, je passe un temps fou à rédiger des spécifications, de l'architecture, des contraintes. J'écris du Markdown pour piloter les agents.

## La valeur n'a jamais été dans le volume de code produit

Soyons lucides : pisser de la ligne au kilomètre, c'est la partie la moins intéressante de notre métier. Ce qui compte, c'est la conception, l'architecture et la robustesse du système.

Je continue d'écrire du code. Souvent, je préfère poser moi-même une fonction ou une modification structurelle complexe, mais je ne le fais plus seul dans mon coin : je montre mon code à l'IA, je lui explique ma démarche, ma logique, et on repart de cette base. C'est un vrai dialogue technique.

Le reste du temps, je la traite comme une exécutante ultra-rapide. Je conçois, pose les invariants métier et lui délègue la frappe. Derrière, j'ai un regard critique impitoyable. Je relis, je valide et je rejette ce qui ne convient pas. Parfois, elle me surprend avec une approche élégante à laquelle je n'avais pas pensé. Parfois, elle s'emmêle les pinceaux et je la recadre.

Le danger du vibe coding, c'est-à-dire balancer un prompt flou et regarder le résultat tourner sans comprendre, c'est qu'il produit des architectures opaques. Si votre application tourne sur du code que personne dans l'équipe n'est capable d'expliquer, vous ne possédez plus votre produit, vous en êtes l'otage. Et le jour où l'éditeur augmente ses tarifs de 300 % ou qu'il est indisponible longtemps, vous faites quoi ?

## La confiance naît de la compréhension

La confiance ne peut s'établir que sur la compréhension, que ce soit dans la vie ou en ingénierie.

Ce n'est pas un hasard si tout un champ de recherche (l'explicabilité des réseaux de neurones) existe pour répondre à ce besoin : comprendre pourquoi un modèle produit certains résultats, pour pouvoir lui faire confiance ou l'ajuster.

Je suis développeur PHP. Quand l'IA génère du PHP, je le dissèque, je l'interroge, je le rejette si nécessaire. Mais quand elle produit du JavaScript ou du CSS (que je connais sans maîtriser), j'ai tendance à dire "ok, si tu veux". Et là, honnêtement, je ne sais pas si ce code respecte les standards : je fais un arbitrage conscient coût/risque. Si l'impact est suffisamment limité, je l'accepte tel quel. Sinon, je délègue la relecture à qui peut le juger : mon expert JS pour le JavaScript, mon intégratrice pour le CSS.

Le niveau de confiance que vous accordez au code généré doit être proportionnel à votre capacité à le critiquer. L'IA doit se plier aux grands principes d'architecture pour que le système reste intelligible pour un humain, pas l'inverse.

## Le retour de la spécification

Le cycle en V avait ses défauts (il était lourd, fastidieux, top-down), mais il avait une rigueur que l'on a trop vite oubliée. Les rôles étaient clairement séparés : la maîtrise d'ouvrage rédigeait les spécifications fonctionnelles (le "quoi"), la maîtrise d'œuvre traduisait en spécifications techniques (le "comment"), et les développeurs arrivaient après dans un long tunnel. Les tests d'acceptance validaient la conformité au besoin initial. Enfin, on essayait ...

L'agilité a simplifié tout ça. Le Product Owner exprime un besoin, souvent incomplet (ex: "l'utilisateur doit être authentifié", sans plus d'information), et les développeurs comblent les trous. On a gagné en vélocité, mais on a perdu la rigueur de l'analyse.

On pourrait objecter que les user stories, avec leurs critères d'acceptance, sont déjà une forme de test fonctionnel. C'est vrai, mais elles restent incomplètes, rédigées en langage naturel et surtout non exécutables. Le développeur devait les interpréter, les compléter et les traduire en vrais tests, et le plus souvent il ne le faisait pas, faute de temps. L'agilité avait la bonne intuition, elle s'est arrêtée à mi-chemin.

Avec l'IA, on peut réhabiliter cette rigueur, mais légère, itérative, portée par le développeur lui-même. Le Markdown est devenu le langage naturel de cette pratique : c'est le format des documentations, c'est lisible par l'IA, c'est versionnable. L'IA participe à l'affinage des specs par l'échange, exactement comme une conversation technique avec un collègue. Dans un article précédent, je parlais de programmation intentionnelle, dont l'écriture Markdown est aujourd'hui le langage naturel.

Attention cependant à un piège : si vous spécifiez seul dans votre coin avec l'IA, vous pouvez tomber dans un tunnel d'analyse à la place du tunnel d'implémentation. L'IA va vite, et on peut très rapidement produire une spec très précise... de la mauvaise chose.

C'est là que les tests d'acceptance deviennent un outil de communication, pas seulement de validation. Vous identifiez les trous dans le besoin du PO ("si je mets une authentification sans mot de passe, ça vous convient ?"), vous proposez, vous échangez. Puis vous faites relire une sélection de tests au PO pour confirmer que vous avez bien compris l'essentiel. Vous n'allez pas lui faire relire tous les tests, ce serait fastidieux, et le PO n'a pas vocation à se plonger dans les détails d'implémentation. Vous filtrez selon les deux registres hérités du cycle en V : le comportement fonctionnel du système (le "quoi"), vous le faites valider par le PO car c'est son domaine, et les choix techniques (le "comment", c'est-à-dire l'architecture, le moteur de BDD, le découpage en couches), vous les validez vous-même en tant qu'architecte, c'est votre responsabilité.

C'est exactement là que se construit la confiance entre un PO et un architecte. Le PO délègue parce qu'il sait que vous saurez distinguer ce qui mérite sa décision de ce que vous pouvez trancher seul. Si vous connaissez un peu son métier, vous pouvez prendre certaines décisions pour lui. C'est cette compétence de jugement qui fait la valeur d'un architecte.

Ce n'est plus de la documentation après-coup, c'est le pilotage.

## Les tests : du filet de sécurité à l'outil de pilotage

Les tests ont toujours eu deux rôles, mais pratiquement tout le monde n'en identifie qu'un.

Le premier, tout le monde le connaît : la régression. Le second est moins verbalisé : le design. Écrire un test avant le code vous force à concevoir l'API, les entrées, les sorties, la DX. Vous décidez comment on utilisera cette fonction avant d'écrire une seule ligne. Double effet : le code produit est naturellement plus testable, moins dépendant des mocks, parce que la contrainte de testabilité était posée dès le départ.

Choisir l'outil de test (PHPUnit, Playwright, Selenium, JMeter) force à répondre à une question précise : qu'est-ce qui constitue une preuve que ça fonctionne ? Cette clarté bénéficie directement à la maintenance et à l'évolution. Dans six mois, le test dit encore ce que le système est censé faire, et il reste exécutable.

Avec l'IA, un troisième rôle émerge : la vérification de conformité à la spec. Sans test, comment savoir si l'IA a suivi tous les points de votre spécification ? Si elle n'a pas halluciné discrètement sur un cas limite ? Vous ne pouvez pas relire mentalement chaque ligne et la comparer au cahier des charges. Les tests d'acceptance répondent à cette question de façon objective : pass ou fail, sans place pour l'interprétation ni pour les oublis silencieux.

Les tests viennent cristalliser le besoin : on ne les écrit pas pour vérifier du code existant, mais pour définir ce qu'on veut construire, et l'IA code ensuite pour les faire passer. C'est du TDD, décalé d'un niveau d'abstraction. En refonte ou en maintenance, les enjeux sont différents.

Tout logiciel a une espérance de vie en bonne santé. Sans IA, beaucoup de projets atteignent ce point de bascule en quelques années. Avec du vibe coding, il peut arriver bien plus tôt : la première demande d'évolution suffit parfois à révéler une architecture impossible à faire évoluer.

Avec une IA bien encadrée (specs précises, tests solides), on retrouve au moins ce qu'on avait avant et on débloque quelque chose qui était auparavant un suicide économique : la grande refonte devient envisageable. Arrêter MongoDB pour revenir à PostgreSQL, c'est faisable quand vous avez de quoi vérifier automatiquement que rien n'a bougé.

Une nuance importante : pour certaines migrations critiques, je ne fais pas confiance au LLM pour appliquer directement les modifications, alors je lui demande d'écrire le script ou l'outil de migration, pour que la stratégie reste sous mon contrôle.

## Monter d'un niveau d'abstraction

Le métier de développeur ne meurt pas, il change de nature.

On quitte l'ère de la syntaxe pour celle de la clarté conceptuelle. Si vous ne savez pas ce que vous voulez construire, si vous ne maîtrisez pas vos règles métier et vos patterns d'architecture, l'IA va juste vous aider à produire de la mauvaise qualité dix fois plus vite.

J'entends parfois des sociétés dire qu'elles n'ont plus besoin de développeurs parce que le PO va déléguer directement à une IA, ce qui me fait sourire. Les PO ne sont pas équipés pour combler tous les trous techniques d'une spec, et ils vont inévitablement déléguer à quelqu'un qui peut faire le pont entre le besoin métier et ce que la machine peut exécuter.

Ce rôle, c'est celui de l'architecte. La chaîne qui se dessine ressemble à ça :

> Product Owner > Architecte (boosté à l'IA) > Agent(s) IA

Une partie du métier se déplace de la production syntaxique vers la conception de systèmes. Ça demande plus de rigueur, pas moins. On écrit autant, mais ce ne sont plus des parenthèses et des indentations, ce sont de longues phrases et des raisonnements. Ce n'est pas une promesse de vélocité, c'est une promesse de qualité.

Mais cette montée en abstraction n'est pas accessible à tout moment d'une carrière. Cet instinct se forge dans des années à écrire du code, à faire des erreurs d'architecture, à déboguer de nuit des systèmes mal conçus : la capacité à cadrer l'intention avec précision, à sentir un invariant manquant, à identifier quand l'IA déraille sur un cas limite. Un développeur débutant qui délègue massivement à l'IA avant d'avoir acquis ces réflexes ne monte pas en abstraction : il court-circuite la phase où se construit le jugement. C'est la question qui me préoccupe le plus dans l'évolution actuelle du métier, bien davantage que la posture des seniors.

Oui, le Markdown est devenu mon langage de pilotage principal, mais je garde les mains dans le cambouis dès que la situation l'exige, et bizarrement, je n'ai jamais eu autant l'impression de faire mon métier de développeur.

---

La vraie compétence de cette ère, ce n'est pas de savoir parler aux machines, c'est d'avoir quelque chose à leur dire.
