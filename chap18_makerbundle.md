# MakerBundle

On va installer un bundle appelé Maker via la commande `composer require maker --dev`, `maker` étant un alias pour 
`symfony/maker-bundle`.  
On ne va pas utiliser directement ce service dans le code comme avec les autres services. Ici, Maker va ajouter 
pas mal de commandes, visibles via `/bin/console` dans la section "make".  

Par exemple, en appliquant la commande `./bin/console make:command`, la console va nous demander un nom pour la 
fenêtre de commande. Si on entre par exemple `article:stats`, le code nous crée alors une nouvelle commande 
`ArticleStatsCommand` dans le dossier `src/Command/` créé pour l'occasion.
Si on retape `./bin/console`, on peut voir que la section `article` a été crée, ainsi que sa première commande 
`article:stats`. Cette commande `./bin/console article:stats` est directement utilisable (bien qu'elle ne fait qu'afficher 
un simple message pour le moment).  

Comme on a deviné quand on travaillait sur Trëmma, les classes présentes dans le dossier `Command/` héritent de la classe 
Command. Le fait de repérer les classes de la classe `Command` comme des commandes s'appelle de l'**auto-configuration**
(auto-configure). Elle est en place car le paramètre "autoconfigure" dans `services.yaml > services: > _defaults: > autoconfigure:`
vaut true.  

