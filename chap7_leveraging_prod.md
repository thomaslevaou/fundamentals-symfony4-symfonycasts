# Leveraging the prod Environment

L'app actuelle est par défaut en configuration "dev". 
Pour passer en configuration de prod, il suffit d'aller dans le fichier `.env` et de passer `APP_ENV` à `prod`.

En rechargeant la page, on peut alors voir que le Profiler est parti comme prévu.  

En production, le cache interne de Symfony n'est pas reconstruit systématiquement (ce qui m'a posé problème sur une 
évolution que je devais apporter au taff btw), encore une fois pour optimiser la rapidité de l'appli.  
C'est pourquoi, à chaque MEP, on doit vider le cache via la commande `./bin/console cache:clear`.  
(la commande `./bin/console` est capable de lire le .env et donc de savoir si on est en prod).  

En prod, les pages d'erreur sont très simples (HTML pur), pas le truc de débogage du Profiler de dev.  
On peut customiser les pages d'erreur (on pourra rechercher "Symfony error pages" simplement).  

Comme le cache n'est plus mis à jour systématiquement si on change "3 hours ago" en "4 hours ago" dans le fichier 
`templates/article/show.html.twig`, la page ne se remet plus directement à jour. Une fois de plus, pour résoudre ce 
problème, un simple `./bin/console cache:clear` suffit.  

Dans le dossier `/var/cache/`, on peut voir que deux dossiers sont présents: un gère le cache du dev, un le cache de la prod.  
La commande `cache:clear` vide les contenus de ces dossiers et recrée certains fichiers.  
La commande `./bin/console cache:warmup`  vide non seulement le contenu des dossiers, mais crée aussi tous les fichiers 
de cache dont Symfony aura besoin en prod. En exécutant cette commande lors d'un déploiement en production, les premières 
requêtes seront bien plus rapides.  

Si on veut utiliser le cache "apcu" en prod mais le cache "filesystem" en dev, on rappelle qu'il suffit de surcharger 
(override) cette clé dans le dossier associé de son environnement (`config/packages/dev ou prod`). 
