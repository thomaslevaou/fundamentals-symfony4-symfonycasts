# Creating Services !

Afin de garantir une meilleur réutilisabilité (et tests unitaires) du code, on va déplacer la partie gérant le cache
dans `ArticleController` vers un nouvelle classe.  
Cette nouvelle classe, qu'on va placer dans un nouveau dossier `src/Service`, va être appelée `MarkdownHelper`.  

Il suffit d'ajouter du code dans une fonction de cette classe pour qu'on ait notre premier `Service` : on rappelle une fois 
de plus qu'un Service est juste un classe qui fait des trucs.  
L'autowiring (pour rappel, c'est la capacité de Symfony à rechercher le service associé à une instance de classe, à n'importe quel endroit du code)
est réalisé dès la création de cette classe : lorsqu'on exécute la commande `./bin/console debug: autowiring`, la classe 
`MarkdownHelper` est directement visible.  
Donc concrètement, dans notre code d'ArticleController, on n'a pas besoin de faire un truc du style `new MarkdownHelper()`.  
Il suffit d'ajouter un troisième paramètre de la méthode show du type de service désiré, comme ci-dessous: 
```PHP
public function show($slug, MarkdownInterface $markdown, AdapterInterface $cache, MarkdownHelper $markdownHelper)
``` 

On peut alors remplacer le code ci-dessous: 
```PHP
$item = $cache->getItem('markdown_'.md5($articleContent));
        if (!$item->isHit()) {
            $item->set($markdown->transform($articleContent));
            $cache->save($item);
        }
$articleContent = $item->get();
```

par un simple : 
```PHP
$articleContent = $markdownHelper->parse($articleContent);
```

Pour passer au MarkdownHelper les paramètres nécessaires à son bon fonctionnement, on peut ajouter des paramètres 
dans sa méthode `parse`:
```PHP
public function parse(string $source, AdapterInterface $cache, MarkdownInterface $markdown): string
```

On doit alors ajouter ces arguments lors de l'appel de `parse()` dans `ArticleController`. On rappelle que l'autowiring
ne fonctionne que pour les actions de contrôleurs Symfony, pas pour les services.  

Cependant, le meilleur moyen de passer ces paramètres à notre nouveau service reste de passer par le constructeur.  
Pour ce faire, on va créer un constructeur dans MarkdownHelper, qui va être le suivant :

```PHP 
private $cache;
private $markdown;

public function __construct(AdapterInterface $cache, MarkdownInterface $markdown)
{
    $this->cache = $cache;
    $this->markdown = $markdown;
}
```

Ce qui veut concrètement dire que n'importe quelle classes utilisant un MarkdownHelper doit lui passer un objet cache 
et un objet markdown.  

On a ainsi une séparation entre les arguments génériques pour toute méthode de la classe (qui doivent passer par le constructeur)
et les arguments spécifiques à la méthode concernée (ici la string `$source`).  

Grâce à ça, on peut supprimer appels aux deux services `AdapterInterface` et `MarkdownInterface` à présent inutilisés
dans le code de ArticleController. Et ça suffit pour faire marcher le code ! 
En effet, pas besoin de faire un truc du type `new MarkdownHelper($cache, $markdown)` : lorsqu'on crée un service,
tous les arguments de son constructeur sont autowirés si les arguments sont dans un type listé parmi ceux 
de la commande `debug:autowiring`. Symfony est capable de reconnaître tout seul où sont les autres services à passer 
au constructeur, lors de l'initialisation du paramètre de `show`.

