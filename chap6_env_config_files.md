# Explore! Environments & Config Files 

Pour que notre page listant des articles fonctionne correctement, on a besoin de préciser certaines 
configurations. Par exemple, on doit préciser: 
- où écrire les fichiers de log, est-ce qu'ils doivent tous être écrits ou juste les erreurs
- l'identifiant et le mot de passe de la bdd
- Vaut-il mieux afficher une grosse page d'exception (pratique en dev), ou une page plus pro pour une production
- etc.

Symfony gère ce genre de configurations via son système d'**environments**. Dans le dossier `/config/packages`, on peut 
y voir trois dossiers `dev/`, `test/` et `prod/` qui sont les environments par défaut proposés à la création d'un projet 
Symfony.  
Jusqu'à maintenant, on a utilisé la configuration de développement: les erreurs étaient clairement affichées, les logs 
toutes affichées en commandes, et la web debug toolbar clairement accessible.  

L'environment de production utilise une configuration plus rapide, affichant seulement les logs d'erreur, et cachant 
des informations techniques sur les pages d'erreur.  

Le fichier `/public/index.php` s'appelle le `front controller`. Ca veut dire qu'il s'agit du premier fichier qui est exécuté 
pour chaque page.  
Ce fichier recherche une variable d'environnement appelée `APP_ENV`. Les variables d'environnement (ici) sont juste une manière 
de stocker des valeurs de configuration. A ne pas confondre avec les environments Symfony qui n'ont rien à voir !  

On peut voir que la variable `APP_ENV` est définie dans le fichier `.env` à la racine du projet, et qu'elle vaut actuellement 
`dev`.  
En retournant dans le fichier `public/index.php`, on peut voir que la variable `$env` prélève la valeur issue du
fichier `.env` et la passe au premier argument du constructeur de la classe `Kernel` ligne 35.  
Cette classe `Kernel` est déclarée dans le fichier `src/Kernel.php`.

Sur les nouveaux projets aujourd'hui, la logique liée à `APP_ENV` dans `public/index.php` a été déplacée dans un fichier appelé `config/boostrap.php`.


Dans cette classe `Kernel`, on peut y voir une méthode `registerBundles`, qui va créer des classes associées aux bundles, mais
si l'environnement associé correspond à l'environnement listé dans le fichier  `config/bundles.php`.  
Par exemple, le `DebugBundle` ne sera instancié qu'en dev et test. Pour information, `yield` dans une boucle for c'est
comme un `return`, sauf qu'il n'arrête pas la boucle for mais créer un objet itérable avec chacun des résultats 
des yields appelés dans la boucle.  

L'autre méthode intéressante de la classe `Kernel` est `configureContainer`, qui est là en gros pour configurer les services.
C'est cette méthode qui permet de configurer les fichiers de log, ou la connexion à la base de données.  
Les deux premières lignes de la méthode sont juste de l'optimisation interne.  
Ensuite, des fichiers de configuration sont appelés via l'attribut `CONFIG_EXTS` qui sert à charger tout fichier 
php, xml, yaml ou yml comme présenté à la ligne 15 du fichier.  
La méthode charge d'abord tous les fichiers de config présents à la racine du dossier `packages/`. Ensuite, les fichiers 
du sous-répertoire associé à l'environnement actuel sont chargés. Tout fichier présent dans le sous-dossier associé à
l'environnement surcharge le fichier de config de la racine de `packages/`.  
Par exemple, dans le fichier `framework.yaml` à la racine du dossier `packages/`, la valeur à 
`strict_requirements` vaut `null` (qui s'écrit `~` en YAML). Mais dans le fichier `routing.yaml` du dossier `dev/`,
la valeur de `strict_requirements` vaut `true`, ce qui surcharge la précédente valeur du dossier parent. On peut s'en rendre 
compte en tapant en console la commande `./bin/console debug:config framework`, où on peut bien voir que la valeur 
de `framework: > router: > strict_requirements:` est bien `true`.  
On peut bien voir que les noms des fichiers de config ne sont pas importants (il ne sont pas pris en compte dans le 
code de `Kernel`), mais que c'est bien la clé racine du fichier de config (ici `framework` par exemple, utilisée dans 
les deux fichiers `framework.yaml` et `routing.yaml`) qui l'est.  

Donc en fait, on pourrait même rassembler tous les fichiers d'un dossier de config en un seul `my_big_old_config_file.yaml`,
et le résultat serait le même. On ne le fait pas juste pour rendre le code plus simple à lire.  

Dans la suite du code de `configureContainer`, on peut voir que le fichier suivant appelé est le fichier `services.yaml`
à la racine du dossier `/config`. Ensuite, le fichier spécifique à l'environnement utilisé, comme `services_test.yaml`
par exemple, est chargé.
 
La méthode `configureRoutes` fonctionne sur un principe similaire: elle charge les routes globales, puis les routes 
associées à l'environment actuellement utilisé avant de charger le fichier `routes.yaml` à la racine du dossier `config`.

On rappelle que, comme on peut le voir une fois de plus ici, Symfony est juste un ensemble de services et de routes.
