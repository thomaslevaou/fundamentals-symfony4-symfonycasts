# services.yaml & the Amazing Bind

Si la plupart des services sont chargés automatiquement par Symfony, on a vu que pour charger des services faits par
nous-mêmes, on doit les insérer dans le fichier `config/services.yaml`.  

La structure de la partie `services:` de ce fichier, commune à tout nouveau projet Symfony, est la suivante :

- la partie `_defaults:` définit les valeurs de config qui seront appliquées sur tous les services enregistrés dans ce fichier.
 Par exemple, `autowire: true` dans cette partie implique que tout service enregistré dans ce fichier aura son autowiring activé.
Si besoin, on peut surcharger la config par défaut dans un service, en allant dans la partie associée à ce service dans 
ce même fichier.
  
- le contenu de la partie `App\` permet de rendre toutes les classes au sein de `src/` disponibles en tant que services du container.
Un service est instancié uniquement s'il est appelé quelque part dans le code, et un service est instancié maximum une fois
à chaque requête (pour des raisons de perf). Ca veut notamment dire que si une requête demande par exemple un `MarkdownHelper`
à plusieurs endroits du code, celui-ci ne sera instancié qu'une seule fois et c'est cette même instance qui servira à tous ces endroits en même temps.
Le `exclude:` est là juste pour exclure certains fichiers du container de services (ce qui donne des perf légèrement meilleurs en dev ici). 
Cette partie de `services.yaml` s'appelle **service auto-registration**.
  
- Et puis c'est à la fin de cette partie qu'on peut configurer nos propres services, comme réalisé dans le chapitre précédent.
Notons juste que dans notre cas, App\Service\MarkdownHelper n'est pas le nom de la classe du service, mais son service id
(dont les valeurs dans ce cas précis sont égales). Lorsqu'on exécute la commande `./bin/console debug:container --show-private`,
on peut voir que les services issus de l'auto-registration ont un id égal au nom de la classe. Tandis que les autres 
ont un id différent de leur nom de classe.
  

Au final, au lieu de faire la config du chapitre précédent pour configurer notre `MarkdownHelper`, on peut ajouter 
simplement ce morceau de yaml au sein de la partie `_defaults:`:
```yaml
        bind:
            $markdownLogger: '@monolog.logger.markdown'
```

(en remplaçant évidemment `$logger` par `$markdownLogger` dans `MarkdownHelper`).

En gros ça veut dire que dans N'IMPORTE quel service, si la variable s'appelle `markdownLogger`, alors on lui passe directement 
le service de notre channel custom `monolog.logger.markdown`.
C'est intéressant parce que potentiellement le type de Service peut alors potentiellement ne plus être de type `MarkdownHelper`,
contrairement à la config du chapitre précédent. Ce serait intéressant de voir si cette config est prioritaire pour un service 
qui n'a rien à voir avec LoggerInterface et les channels de son Monolog associé (à mon avis oui, vérifier à l'occasion).
