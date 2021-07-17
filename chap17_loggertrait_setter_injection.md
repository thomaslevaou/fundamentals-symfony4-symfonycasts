# Bonus! LoggerTrait & Setter Injection 

Encore une fois, on a écrit notre code de message Slack directement dans la méthode `show()` de `ArticleController`, 
alors qu'il vaut mieux l'écrire dans notre propre Service pour que le code soit plus facilement réutilisable sur long 
terme. D'où le fait que maintenant, le code lié à Slack va être placé dans notre nouveau Service 
`src/Service/SlackClient.php`, via le code suivant :
```PHP

use Nexy\Slack\Client;

class SlackClient
{
    private $slack;

    public function __construct(Client $slack)
    {
        $this->slack = $slack;
    }

    public function sendMessage(string $from, string $message)
    {
        $slackMessage = $this->slack->createMessage()
            ->from($from)
            ->withIcon(':ghost:')
            ->setText($message);
        $this->slack->sendMessage($slackMessage);
    }
} 
```

Ce qui allège énormément le code dans `ArticleController` :
```PHP
    public function show($slug, MarkdownHelper $markdownHelper, SlackClient $slack)
    {
        if ($slug === 'khaaaaaan') {
            $slack->sendMessage('Khan', 'Ah, Kirk, my old friend...');
        }
    ...
```

Maintenant, supposons que dans notre nouveau service, on souhaite ajouter des messages de log. En soi on pourrait le 
faire en ajoutant une instance de LoggerInterface dans le constructeur qu'on appelle ensuite au moment voulu.  
Mais on va ici répondre à la demande en auto-wirant la dépendance via une **setter injection**. C'est assez pratique 
quand on veut avoir des dépendances optionnelles (genre on voudrait par exemple ici que le SlackClient marche même 
si on n'a pas de LoggerInterface).  

La setter injection s'implémente de la manière suivante : 
```PHP
use Nexy\Slack\Client;
use Psr\Log\LoggerInterface;

class SlackClient
{
    /**
     * @var LoggerInterface|null
     * Soit l'attribut est une instance de LoggerInterface, soit ce ne sera rien 
     */
    private $logger;

    ...
    // L'annotation @required ci-dessous est nécessaire pour que la méthode soit appelée à l'intialisation de l'objet, et ainsi le logger pourra être auto-wiré
    /**
     * @required 
     */
    public function setLogger(LoggerInterface $logger)
    {
        $this->logger = $logger;
    }

    public function sendMessage(string $from, string $message)
    {
        if ($this->logger) {
            $this->logger->info('Beaming a message to Slack!');
        }
    ...
    }
}
```

Quand on a besoin régulièrement d'un service, on peut utiliser un **Trait**. Il s'agit d'une sorte de "sous-service" qu'on peut 
"injecter" dans d'autres services, pour que celui qui l'utilise puisse utiliser directement ses méthodes.  
C'est assez pratique pour ne mettre qu'à un seul endroit des fonctionnalités présentes dans plusieurs services.  

Pour mettre en place ce trait, on va créer un nouveau dossier `/src/Helper`, dans lequel on va créer une nouvelle classe PHP
dont le template sera "Trait" et qu'on va appeler `LoggerTrait`. Son code va être le suivant : 
```PHP
use Psr\Log\LoggerInterface;

trait LoggerTrait
{

    /**
     * @var LoggerInterface|null
     */
    private $logger;


    /**
     * @required
     */
    public function setLogger(LoggerInterface $logger)
    {
        $this->logger = $logger;
    }

    public function logInfo(strin $message, array $context = [])
    {
        if ($this->logger) {
            $this->logger->info($message, $context);
        }
    }
}
```

En conséquence, on peut alléger le message dans `SlackClient` de la manière suivante : 
```PHP

class SlackClient
{
    use LoggerTrait;
    private $slack;

    public function __construct(Client $slack)
    {
        $this->slack = $slack;
    }

    public function sendMessage(string $from, string $message)
    {
        $this->logInfo('Beaming a message to Slack', [
            'message' => $message
        ]);
    ...
    }
}
```

Et grâce au deuxième paramètre de `logInfo`, un "context" est affichable dans le Profiler de Symfony.