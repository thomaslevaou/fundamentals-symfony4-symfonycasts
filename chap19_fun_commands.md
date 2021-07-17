# Fun with Commands

Dans une description de commande :
- la méthode `addArgument()` permet d'ajouter des arugments : `./bin/console article:stats argument`
- la méthode `addOption()` permet d'ajouter des options, chacune précédée par un `--` : `./bin/console article:stats --option`

Pour `addArgument()`, la constante `InputArgument::REQUIRED` permet de préciser que la commande est obligatoire.  
Pour `addOption()`, la constante `InputOption::VALUE_REQUIRED` permet de dire que l'option devra avoir obligatoirement une valeur,
par exemple `--option=qqch`. Le 5ème argument de `addOption()` permet de préciser une valeur par défaut.  

Notre méthode `configure()` peut donc contenir le code suivant : 
```PHP
    protected function configure()
    {
        $this
            ->setDescription('Returns some article stats')
            ->addArgument('slug', InputArgument::REQUIRED, 'The article\'s slug')
            ->addOption('format', null, InputOption::VALUE_REQUIRED, 'The output format', 'text')
        ;
    }
```

Si on exécute à présent `./bin/console article:stats --help`, on peut afficher les descriptions des options & arguments 
qu'on a placées dans `configure()` dans la console. Pour n'importe quelle commande, l'option `--help` fera ce job. 
Cette option présente aussi toutes les autres options communes à toutes les commandes.  

Comme déjà remarqué sur Trëmma, le traitement de la commande est exécuté dans la méthode `execute()`.  

Pour afficher des résultats en console, on va utiliser une instance de l'objet `SymfonyStyle`, ici appelée `$io`.  

On peut appliquer le code suivant dans `execute()` :

```PHP
protected function execute(InputInterface $input, OutputInterface $output)
    {
        $io = new SymfonyStyle($input, $output);
        $slug = $input->getArgument('slug');

        $data = [
            'slug' => $slug,
            'hearts' => rand(10,100),

        ];

        switch ($input->getOption('format')) {
            case 'text':
                $io->listing($data);
                break;
            case 'json':
                $io->write(\GuzzleHttp\json_encode($data));
                break;
            default:
                throw new Exception('What kind of crazy format is that ?!');
        }
    }
```

Ce qui permet d'avoir un premier résultat dans la commande `./bin/console article:stats khaaaaaaan`.
Comme prévu, l'ajout du paramètre `--format=json` permet d'afficher le résultat au format json.  


Histoire d'afficher plus élégamment le code dans le `case 'text'`, on va utiliser la méthode `table()` de `SymfonyStyle`:
```PHP
            case 'text':
//                $io->listing($data);
                $rows = [];
                foreach ($data as $key => $val) {
                    $rows[] = [$key, $val];
                }
                $io->table(['Key', 'Value'], $rows);
```

Avec SymfonyStyle, on peut afficher des trucs comme des questions, des barres de progression, etc.

Les commandes sont évidemment également des services ! Si on a besoin d'un service dans une commande, il suffit de l'ajouter 
dans son constructeur comme on a toujours fait jusqu'à présent.