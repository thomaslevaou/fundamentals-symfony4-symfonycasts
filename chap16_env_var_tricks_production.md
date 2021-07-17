# Env Var Tricks & on Production

Comme on peut voir dans le code de `public/index.php`, si la variable `APP_ENV` est définie, le fichier `.env` n'est pas lu.  
Ca veut dire surtout qu'en prod, cette variable va devoir être définie autrement (une fois de plus parce que les valeurs 
sur un serveur de production seront différentes de celles du serveur de dev).  

Mais parfois, ça peut être une galère d'avoir un fichier de variables d'environnement en prod qui est fait proprement.  

Dans ce cas, une solution de secours consiste à avoir un fichier `.env` de dev local, et un fichier `.env` de prod.  
Si on applique une telle solution, il est alors fortement recommandé de rendre notre `.env` actuel spécifique au dev local. 
Ce qui est effectué via les commandes suivantes :
```bash
composer remove symfony/dotenv
composer require symfony/dotenv --dev
```
Cette solution n'est pas recommandée car le fait de parser un fichier `.env` n'est pas optimisé pour de la production (ce 
qui fait perdre un peu de perf au code en prod). Mais si on n'a pas le choix, on n'a pas le choix. 



Notons par ailleurs que si on a besoin de préciser le type d'une variable d'environnement (qui par défaut est une string)
dans un yaml, on doit préciser son type suivi de ":", comme par exemple ici :
```yaml
parameters:
    # roughly equivalent to "(int) getenv('DATABASE_PORT')"
    app.connection.port: '%env(int:DATABASE_PORT)%'
```
D'autres exemples sont disponibles ici : https://symfony.com/blog/new-in-symfony-3-4-advanced-environment-variables

D'autres préfixes sont aussi utilisables dans des variables d'environnement : 
- `resolve:` permet de récupérer des paramètres (`%foo%`) si leur valeur est dans une variable d'environnement
- `file:` retourne le contenu d'un fichier, si le chemin d'accès au fichier est contenu dans une variable d'environnement
- `base64:` fait un `base64_decode` de la valeur (pour rappel, c'est plus simple pour la transmission de sauts de ligne ou de caractères spéciaux)
- `constant:` permet de lire des constantes PHP
- `json:` fait un `json_decode`

On peut tout à fait enchaîner ces préfixes : rien n'empêche de faire `'%env(json:file:SECRETS_FILE)%'`, ce qui permet d'extraire
le contenu d'un fichier json.  
Il est possible de créer notre propre préfixe de variable d'environnement, où on écrirait notre propre logique.