# debug:container & Cache Config

On rappelle que tous les services vivent dans un objet appelé container.  
De ce fait, chaque service dispose de son propre identifiant interne.  
En fait, dans le bundle KnpMarkdownBundle, le paramètre `service` qu'on a modifié pour l'exemple 
correspond à un id de service dans le container.  

Donc dans la méthode `show()` de `ArticleController`, le service qui est passé via le paramètre `MarkdownInterface`
dépend du paramètre `service` de KnpMarkdownBundle.  
Et donc ça veut aussi dire que suite à notre dernier changement de config, si on fait maintenant 
`./bin/console debug:autowiring`, l'alias de `MarkdownInterface` est à présent `markdown.parser.light` (alors que 
ce n'était pas le cas avec la config par défaut ofc).  

Mais à vrai dire, la commande `debug:autowiring` n'affiche pas tous les services du container.  
Par contre, si on fait `./bin/console debug:container --show-private`, là on a la liste complète de tous les services 
du container. L'id du service est à gauche, le chemin d'accès vers la classe sur la droite.  
En vrai, on n'utilisera directement en général qu'une petite partie de tous ces services. Les plus importants sont affichés 
via `debug:autowiring`.  

Si on dumpe notre `$cache` juste avant son `getItem`, on peut voir qu'il s'agit d'une instance de `TraceableAdapter`, 
dont le `pool` est un `FilesystemAdapter`. On peut donc supposer que ce cache est stocké dans un système de fichiers,
dont le répertoire associé est donné via son attribut `directory`: `/var/cache/dev/pools/VXCE+hl0w9`.  

Le service de cache est fourni par le bundle `FrameworkBundle`, lui-même fourni directement par l'application Symfony.  

La configuration du cache est donc accessible et modifiable dans `config/packages/framework.yaml`, dans sa partie associée 
`cache:`. Rappelons qu'on peut avoir une vue plus enrichie de cette configuration via la commande 
`./bin/console config:dump framework`.  
Avec la commande `./bin/console debug:config framework`, on peut avoir accès à la configuration actuelle du bundle.  
Dans la partie cache, on peut y voir affichées 6 configurations actuellement en place. On ne les voit pas dans 
`framework.yaml` parce que ce sont les valeurs par défaut.  
Via le paramètre `app` du cache, on peut voir que son système actuel est un système de fichiers 
(`cache.adapter.filesystem`). Dans `framework.yaml`, on peut modifier la valeur associée à `app` pour que celle-ci 
devienne `cache.adapter.apcu`, ce dernier étant toujours un service (déjà dans le container) pour rappel.  
Un nouveau dump du cache suffit pour voir que le système de cache a changé suite à la modification de paramètre. En fait mon Symfony local affiche une erreur d'inaccessibilité de apcu mais osef, c'est sûrement dû à une incompatibilité 
a priori de APCU sur Debian mais j'ai trop la flemme d'adapter la config si ce n'est pas demandé pour le prochain chapitre.  

Si on fait une faute de frappe dans un paramètre, Symfony saura nous le signaler.