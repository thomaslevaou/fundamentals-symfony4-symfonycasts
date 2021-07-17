# KnpMarkdownBundle & its Services

Pour illustrer le concept de bundle, on va installer un process qui parse une page en MarkDown, appelé
**KnpMarkdownBundle**. On l'installe via la commande `composer require knplabs/knp-markdown-bundle`.  
KnpMarkdownBundle étant un bundle, il contient des classes PHP et une config qui va ajouter un ou plusieurs 
services à notre application.  

On peut donc voir que le `bundles.php` a bien été mis à jour comme convenu. 
En exécutant un `./bin/console debug:autowiring`, on peut voir qu'on peut utiliser deux interfaces 
différentes pour utiliser le même service. On va ici utiliser MarkdownInterface.  

On l'utilise via l'ajout dans la méthode `show` suivant: 
```PHP
public function show($slug, MarkdownInterface $markdown)
{
    $comments = [
        'I ate a normal rock once. It did NOT taste like bacon!',
        'Woohoo! I\'m going on an all-asteroid diet!',
        'I like bacon too! Buy some from my site! bakinsomebacon.com',
    ];

    $articleContent = <<<EOF
Spicy **jalapeno bacon** ipsum dolor amet veniam shank in dolore. Ham hock nisi landjaeger cow,
lorem proident [beef ribs](https://baconipsum.com/) aute enim veniam ut cillum pork chuck picanha. Dolore reprehenderit
labore minim pork belly spare ribs cupim short loin in. Elit exercitation eiusmod dolore cow
turkey shank eu pork belly meatball non cupim.
EOF;
    $articleContent = $markdown->transform($articleContent);

    return $this->render('article/show.html.twig', [
        'title' => ucwords(str_replace('-', ' ', $slug)),
        'articleContent' => $articleContent,
        'slug' => $slug,
        'comments' => $comments,
    ]);
}
```

Notons que Twig échappe automatiquement par défaut les bouts de code : si une balise HTML est présente dans la variable passée à Twig, 
ce dernier en échappe automatiquement les caractères.  
Pour demander à Twig de faire compiler le HTML de la variable, on doit alors ajouter un filtre `|raw` : `{{ articleContent|raw }}`.  
Le service de MarkdownInterface a alors pu traduire le code Markdown en HTML sur la page, comme attendu.




