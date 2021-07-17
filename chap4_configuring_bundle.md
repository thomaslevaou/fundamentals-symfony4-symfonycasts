# Configuring a Bundle

Si on dumpe la variable `$markdown`, on peut voir qu'elle semble composée entre autres d'un tableau de features 
qu'on peut activer ou désactiver.  

D'une manière plus générale, on peut afficher la config par défaut d'un service (ici avec KnpMarkdownBundle) en YAML 
via la commande `./bin/console config:dump KnpMarkdownBundle`. Parfois, le nom des paramètres est explicite, parfois 
il est nécessaire de comparer le résultat de cette commande avec la doc du Bundle pour bien comprendre.  

Dans la doc de KnpMarkdownBundle, on peut voir que le paramètre `service` du `parser` affiché en console peut être modifié
selon certaines valeurs affichées sur la doc.

Pour modifier ce paramètre, on crée un fichier dans `/config/packages` appelé `knp_markdown.yaml`. Comme indiqué dans la 
doc du bundle, on insère dans ce fichier le contenu suivant : 
```yaml
knp_markdown:
  parser:
    service: markdown.parser.light
```

Une fois ce fichier modifié, il est recommandé de vider le cache de Symfony via un `./bin/console cache:clear`.  
D'une manière générale, il est recommandé de vider le cache de Symfony à chaque ajout d'un fichier de config.  

Le but d'un bundle étant de fournir des services, le but de la configuration d'un bundle est de changer le comportement de ses 
services (changer une classe, ou des arguments passés à un service, etc).  

Suite au changement de configuration, on peut voir dans le dump du `$markdown` qu'il est à présent une instance de `Light` 
et non de `Max`.  

Notons que le nom du fichier de config (`knp_markdown.yaml`) n'est pas important, car Symfony charge automatiquement 
le contenu de tous les fichiers dans `packages/` quels qu'en soient leurs noms. Le plus important est la clé "racine" 
(root key) du YAML, à savoir `knp_markdown:`. Toute configuration sous cette clé root sera passée au bundle KnpMarkdownBundle.
Chaque config a une clé root définissant le bundle qui configuré associé.  
