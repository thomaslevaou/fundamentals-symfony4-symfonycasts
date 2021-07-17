# The Cache Service

Via un `./bin/console debug:autowiring`, on peut constater en haut de la liste la présence d'un service 
`CacheItemPoolInterface`, lui-même alias de `cache.app` au même titre que `AdapterInterface` plus bas 
dans cette liste. En fait, l'id interne du service de cache de Symfony s'appelle cache.app : si on a
deux services avec le même id interne, ça veut dire que les deux types d'objets correspondent en 
réalité exactement au même objet Symfony.  

Là on va utiliser ce service de cache pour mettre en cache la conversion en MarkDown qu'on fait dans `ArticleController`.  
Ce service implémente l'interface de cache standard de PHP, appelée PSR-6 si un jour on a besoin de creuser plus
sur ce sujet.  

On implémente le service de cache via le code suivant :
```PHP
public function show($slug, MarkdownInterface $markdown, AdapterInterface $cache)
{
    $comments = [
    ...
EOF;

    $item = $cache->getItem('markdown_'.md5($articleContent));
    if (!$item->isHit()) {
        $item->set($markdown->transform($articleContent));
        $cache->save($item);
    }
    $articleContent = $item->get();
    
    ...
}
```

Dans le Profiler de Symfony, on peut alors voir dans la section `Cache` qu'une section **Pools** apparait. 
Les Pools correspondent à un système de cache dont la plupart sont utilisés en interne par Symfony.  

On peut voir dans cette section qu'on a eu 1 "Miss" et 2 appels (1 en lecture, 1 en écriture) au cache.  

En rechargeant la page, on peut voir dans le profiler qu'à ce deuxième affichage, on a bien "Hit" le cache.  

Si on modifie le contenu "markdownisé", on voit que le cache se met à jour car la clé utilisée pour stocker le cache 
change si le texte change (`'markdown_'.md5($articleContent)`). Le Profiler montre alors comme attendu un nouveau miss
du cache de l'app, puis un nouvel accès en écriture et en lecture.