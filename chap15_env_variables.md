# Environment Variables

Dans `nexy_slack.yaml`, l'URL d'accès à Slack est codée en dur, et on sait que c'est mal.  
Le fait d'utiliser des **variables d'environnement** est le meilleur moyen, recommandé par Symfony, pour passer 
des données sensibles.  

Dans `public/index.php`, on peut voir que le code lit déjà la variable `APP_ENV`, via la variable `$_SERVER`.  
Pour appeler des variables d'environnement dans un fichier yaml, on va faire appel au paramètre `env()`. Si son 
paramètre correspond à une variable d'environnement, celle-ci est appelée à la place. Exemple ci-dessous:
```yaml
nexy_slack:
  endpoint: '%env(SLACK_WEBHOOK_ENDPOINT)%'
```

L'appel au variables d'environnement en prod va être différent en fonction du serveur qu'on utilise (Docker, Nginx, Apache, etc). 
Mais en développement, la variable `APP_ENV` va prélever les données dans le fichier `.env`, à la racine du projet.

Lorsqu'on va ajouter des Recipes, celles-ci vont mettre automatiquement le fichier `.env` à jour avec leurs variables 
d'environnement nécessaires si besoin. Mais ça ne nous empêcher pas de mettre à jour ce fichier nous-mêmes si besoin.  
Et c'est ce qu'on va faire en ajoutant dans ce fichier notre propre bloc "Custom vars":
```bash
### CUSTOM VARS
SLACK_WEBHOOK_ENDPOINT=https://hooks.slack.com/services/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
### END CUSTOM VARS
```

On peut alors voir cette nouvelle variable d'environnement dans la liste des variables d'environnement, affichable 
via la commande `./bin/console about`.  

Dans les projets Symfony récents, le fichier d'environnement `.env` va contenir des valeurs sensibles mais pas secrètes.
Pour avoir un fichier contenant uniquement des valeurs spécifiques à notre poste (et qui ne sera pas commité), on peut 
créer un fichier `.env.local`, qui sera automatiquement ignoré par git, et dans lequel on pourra mettre nos données sensibles.  

Mais dans ce projet ici, on a aussi un fichier `.env.dist`, qui n'est pas lu par Symfony, mais qui sera lu par Git.  
Ca permet de savoir à un nouveau développeur quelles seront les variables d'environnement à ajouter.


