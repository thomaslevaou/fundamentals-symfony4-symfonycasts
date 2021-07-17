# Autowiring aliases

(Suite de la fiche de ce chapitre qui avait été commencée à Label)

Pour que le slack soit reconnu par Symfony, il est important de le préciser dans `services.yaml`, dans la partie bind 
comme ci-dessous: 
```yaml
        bind:
            $markdownLogger: '@monolog.logger.markdown'
            $isDebug: '%kernel.debug%'
            Nexy\Slack\Client: '@nexy_slack.client'
```

En effet, dans la partie `bind:`, on peut bind par le nom de l'argument ou par la classe/interface.  
Et à partir de là, si on a un compte Slack (que je n'ai pas fait parce que flemme & ça a l'air payant),
on a un message Slack qui est envoyé.  

Histoire que tous les services du système soient concernés (pas ceux ajoutés juste via `services.yaml`), on peut désindenter 
`Nexy\Slack\Client` dans le fichier pour qu'il soit au même niveau que `defaults_:`:
```yaml 
        bind:
            $markdownLogger: '@monolog.logger.markdown'
            $isDebug: '%kernel.debug%'
   Nexy\Slack\Client: '@nexy_slack.client'
```

La seule différence concrète est qu'avec la commande `./bin/console debug:autowiring`, on est à présent sûr que le service 
`Nexy\Slack\Client`, va apparaître dans les résultats. Ce qui est en fait déjà le cas car j'ai une version assez récente 
du bundle, mais sur une plus vieille version du bundle comme dans le tuto on n'aurait pas vu le service sans ce déplacement 
dans `services.yaml`.  

Bref de cette manière on a créé un nouveau service ayant pour id `Nexy\Slack\Client`, mais cet id est juste un **alias**, c'est-à-dire 
un raccourci pour récupérer par auto-wiring, le service nexy_slack.client existant (ce qui permet de récupérer proprement 
un service Symfony 3 par exemple). Chaque classe enregistrée dans `src` est enregistrée automatiquement en tant que Service,
ce qui simplifie l'auto-wiring.


