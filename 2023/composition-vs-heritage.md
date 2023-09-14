# Composition vs Héritage

Il y a quelques années j'ai commencé à me former à la programmation fonctionnelle. Depuis j'utilise de plus en plus le principe de composition mais je ne me suis jamais formalisé _pourquoi_.

Cet article est ma tentative de condenser mes années d'expérience sur ce sujet.

## Héritage

PHP comme beaucoup d'autres langages orientés objet permet de faire de l'héritage. Le principe est qu'une classe (`B`) peut _étendre_ une autre classe (`A`). La classe enfante (`B`) expose à un utilisateur toutes ses méthodes `public` mais également celles de son parent (`A`). En interne `B` a accès à toutes les méthodes `public` et `protected` de `A`, elle peut soit les utiliser comme n'importe quelle méthode définies dans `B` soit les réimplementer.

Ce système d'héritage est souvent enseigné via l'exemple d'un `Chat` ou un `Chien` qui hérite d'une classe `Animal`. Il montre comment réutiliser les définitions (méthodes ou propriétés) qui sont communes entre les chats et les chiens via une classe (souvent `abstract`) définissant un animal.

D'un certains point de vue cet exercice est logique puisque les chats et les chiens sont bien des animaux. Mais il nous a surtout appris à utiliser l'héritage pour réutiliser du code. Dans nos applications on retrouve cette réutilisation de code dans les services (les classes qui _font_ des choses).

Quand je réfléchis au système d'héritage j'ai toujours l'exemple du package [`innmind/http-transport`](https://packagist.org/packages/innmind/http-transport) en tête. Il s'agit d'un client HTTP qui permet :
- de faire des appels HTTP via [cURL](https://fr.wikipedia.org/wiki/CURL)
- de suivre les redirections HTTP
- d'utiliser un [Circuit Breaker](https://en.wikipedia.org/wiki/Circuit_breaker_design_pattern)
- de faire de l'[Exponential Backoff](https://en.wikipedia.org/wiki/Exponential_backoff)
- de logguer les appels

Un utilisateur pourrait vouloir agencer ces fonctionnalités de différentes manières. Des exemples simples seraient soit le client cURL tout seul, logguer les appels, logguer les redirections, etc... (en pseudo-code cela ressemblerait à `curl`, `log(curl)`, `redirection(log(curl))`, `log(redirection(curl))`, etc...).

Lorsqu'une classe est chargée en mémoire dans PHP elle ne peut plus changer. Pour offrir à l'utilisateur toutes les possibilités d'usages il faudrait implémenter toutes les chaines d'héritage (par exemple `LogRedirectionCurl` qui étends `RedirectionCurl` qui étend `Curl`).

Sur ces 5 fonctionnalités on a `4!` permutations, soit **`24`** classes. Et si l'utilisateur souhaite rajouter une fonctionnalité alors l'ensemble des permutations est `5!`, soit **`120`** classes.

Bien entendu l'ensemble des permutations n'est pas forcément utile. Mais le problème fondamentale de l'héritage ce situe ici de mon point de vue.

Au plus vous essayez de faire coexister des fonctionnalités via de l'héritage au plus votre code devient complexe, et de manière exponentielle de surcroit.

_Aparté_ : dans l'écosystème PHP il est de plus en plus populaire d'utiliser des [Traits](https://www.php.net/manual/fr/language.oop5.traits.php) pour "composer" des fonctionnalités. Cette approche n'a rien à voir avec la composition et souffre des mêmes problèmes que l'héritage (même si amoindri, mais apporte aussi son lot dédié de problèmes) puisque les permutations doivent être figées dans une classe.

## Composition

La composition fait souvent référence à la composition de fonctions en programmation fonctionnelle. En programmation objet, comme PHP, ce système est connu sous le nom d'[injection de dépendances](https://fr.wikipedia.org/wiki/Injection_de_dépendances).

L'objectif de cette approche est de créer des classes simples (ayant qu'une seule responsabilité) et d'agencer les instances de ces classes localement en fonction du besoin.

Si on reprend l'exemple du client HTTP introduit au dessus et qu'on souhaite utiliser différentes permutations on obtient ce genre de code :

```php
$logger = new Logger;
$curl = new Curl;
$logCurl = new Log($curl, $logger);
$logRedirectionCurl = new Log(new Redirection($curl), $logger);
// etc...
```

Là où avec l'héritage on devait créer une classe pour une permutation, ici on ne fait que de l'injection d'objet dans d'autres objets. Et leur usage est interchangeable puisque les classes implémentent la même interface.

L'intérêt de cette approche est qu'il est tout à fait possible d'utiliser partout dans l'application notre instance `$logCurl`, et vouloir suivre les redirections seulement à un endroit de notre application. Par composition en faisant `new Redirection($logCurl)` à l'endroit voulu on obtient l'effet souhaité sans impacter le reste de l'application. A noter que cet exemple serait extrêmement compliqué à mettre en place via de l'héritage.

L'utilisateur de ce système est libre de rajouter une fonctionnalité sans avoir à prendre en compte les autres fonctionnalités. Il y a donc moins de code et de tests à écrire.

Là où une permutation des fonctionnalités nécessite plusieurs classes via de l'héritage, avec la composition cela nécessite la création d'un objet.

## SOLID

Cet [acronyme](https://fr.wikipedia.org/wiki/SOLID_(informatique)) représente 5 principes de conceptions introduits par Robert C. Martin et sont des références de la programmation objet.

Dans le contexte de cet article les 2 premières lettres nous importent le plus : S pour _Single responsibility principle_ et O pour _Open/Closed principle_.

_Single responsibility principle_ dit qu'une classe ne devrait faire qu'une seule chose.

Hors l'héritage enfreind ce principe puisque comme démontré au dessus la combinaison de fonctionnalités est représentée par une classe enfante qui essaie de faire plusieurs choses. A l'inverse la composition pousse dans cette direction pour permettre à un utilisateur  d'agencer des objets au gré de ses besoins.

_Open/Closed principle_ dit qu'une classe devrait être ouverte à l'extension mais fermée à la modification.

Là encore l'héritage enfreind ce principe puisqu'une classe enfante est libre de réimplémenter une méthode de la classe parente. Et une fois de plus la composition pousse dans cette direction via des classes simples et c'est à l'utilisation des instances que l'utilisateur peut les agencer pour obtenir le comportement qu'il souhaite.

## Final

En conclusion la différence entre ces 2 approches pourrait se résumer via :
- l'héritage en se voulant ouvert engendre une complexité croissante qui finit par fermer les possibilités,
- la composition en utilisant des classes fermées garde une complexité constante et ouvre les possibilités.

(Au passage, l'utilisation du mot clé `final` sur les classes est le meilleur moyen d'empêcher l'héritage et d'imposer la composition)

---

Vous pouvez [laisser vos commentaires](https://github.com/Baptouuuu/blog/discussions/14) dans le channel de discussion dédié à cet article.

Vous avez trouvé des fautes ? N'hésitez pas à ouvrir une [Pull Request](https://github.com/Baptouuuu/blog/pulls).
