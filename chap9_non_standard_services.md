# Using Non-Standard Services: Logger Channels

On va ajouter un peu de logs dans notre nouveau service MarkdownHelper.  
Pour ce faire, il suffit d'ajouter un troisième argument du type `LoggerInterface` dans le constructeur de notre MarkdownHelper,
qu'on affecte à son attribut associé dans la classe:
```PHP 
public function __construct(AdapterInterface $cache, MarkdownInterface $markdown, LoggerInterface $logger)
```
PhpStorm a un raccouci pour ajouter le nouvel attribut: lorsque le curseur est sur `$logger`, on peut faire `Alt + entrée`,
puis cliquer sur `initialize fields`, ce qui permet la création du nouvel attribut et son initialisation dans le constructeur.  

Après avoir ajouté un code un peu random dans la fonction `parse()` juste pour vérifier le bon fonctionnement des logs :
```PHP
if (stripos($source, 'bacon') !== false) {
    $this->logger->info('They are talking about bacon again!');
}
```

On peut alors vérifier le bon fonctionnement sur la page en allant dans le Profiler, puis dans Logs: le message sera alors visible 
parmi les différents log messages.  

On sait que via la commande `./bin/console debug:autowiring`, le LoggerInterface a pour identifiant interne `monolog.logger`.  
De ce fait, on peut avoir plus d'infos sur ce service en exécutant la commande `./bin/console debug:container monolog.logger`
(bon ça n'apporte pas grand-chose de plus qu'un simple `dump()`, mais c'est bien de connaître cette commande quand même).  

On peut filtrer la liste des services du container affiché dans `./bin/console debug:container` en ajoutant tout simplement 
un mot filtre derrière, par exemple ici `./bin/console debug:container --show-private log`.  
On peut alors voir six résultats du type `monolog.logger.(qqch)`.  
En gros pour faire des logs, Symfony utilise une bibliothèque appelée `Monolog`. Et `Monolog` dispose de **Channels**,
qui sont en gros des catégories. On peut donc avoir plusieurs loggers, chacun ayant son nom (via son channel) unique.  
On peut alors s'amuser à écrire chaque channel de logs dans des fichiers de logs différents par exemple.  
A l'endroit des logs, le Profiler affiche le Channel ("request", "app", "event" par exemple).  

On peut voir plus de détails sur la config de la bibliothèque `Monolog` dans `config/dev/monolog.yaml`:  
```yaml
monolog:
    handlers:
        main:
            type: stream
            path: "%kernel.logs_dir%/%kernel.environment%.log"
            level: debug
            channels: ["!event"]
```
On peut voir que les fichiers de logs sont sauvés (en env de dev) dans un fichier `dev.log`, mais sauf pour les Monolog 
ayant un Channel "events" (via la précision apportée `["!event"]`).  

En fait, dans `Monolog` on va même pouvoir créer notre propre channel `markdown` où on inscrira des logs qu'on souhaite.  
Pour ce faire, on a créé un fichier "monolog.yaml" dans `config/packages` (pour que notre channel spécifique soit le même 
en dev et en prod), où on commence juste par préciser l'existence de notre nouveau channel:
```yaml
monolog:
  channels: ['markdown']
```

En vidant le cache puis en faisant un nouveau `./bin/console debug:container log`, on peut y avoir notre nouveau 
service de log issu du nouveau channel : `monolog.logger.markdown`.  
Ensuite, on va vouloir stocker nos logs de "markdown" dans un fichier spécifique. C'est pourquoi, on va ajouter
une nouvelle clé `markdown_logging` dans `dev/monolog.yaml`, avec le contenu suivant : 
```yaml
monolog:
    handlers:
        main:
            type: stream
            path: "%kernel.logs_dir%/%kernel.environment%.log"
            level: debug
            channels: ["!event"]
        markdown_logging:
            type: stream
            path: "%kernel.logs_dir%/markdown.log"
            level: debug
            channels: ["markdown"]
```

Pour préciser que notre nouvelle log sera de channel "markdown", on va devoir préciser à la fin de `config/services.yaml` que
les variables `$logger` du type `MarkdownHelper` seront de channel `markdown`, ce qui se traduit par (toujours au sein de la clé `services:`) :
```yaml 
    App\Service\MarkdownHelper:
        arguments:
            $logger: '@monolog.logger.markdown'
```
Notons qu'on précise que `$logger` va correspondre à un identifiant de service, et pas une string, via un '@'. 

En rechargeant ensuite la page, on peut voir que les logs sont bien affichées dans notre dossier 'markdown.log', qui s'il n'existe pas, 
est créé.

