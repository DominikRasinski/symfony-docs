# Serwisy

Serwisy to tak naprawdę re-używalne obiekty pozwalające na łatwiejsze wykonanie danego zadania lub operacji. Zazwyczaj wszystko to co jest wykonywane przez aplikację jest nazywane serwisem. Na przykład możliwość wysłania maila jest wykonywana przez serwis, połaczenie się do bazy danych też jest wykonywane przez serwis i tak dalej...

W Symfony serwisy są powiązane z specjalnym obiektem `Service container`. `Service container` skupia w sobie wszystkie serwisy dostępne w aplikacji i zarządza ich życiem.

## Ładowanie oraz używanie serwisów

Aby załadować serwis do kontrolera i mieć możliwość skorzystania z niego wewnątrz wskazanego kontrolera, należy nazwę serwisu dodać jako argument to kontrolera oraz odwołać się do niego w ciele kontrolera za pomocą nadanej mu zmiennej.

Przykład użycia serwisu do logowania:

```php
namespace App\Controller;

use Psr\Log\LoggerInterface;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

class NaytibaBehavior extends AbstractController
{
    #[Route('/elder')]
    public function logElderBehavior(LoggerInterface $logger) // Ładowanie serwisu za pomocą metody parametr injection
    {
        // code for checking elder naytiba behavior
        // return variable isMove
        if (isMove) {
            $logger->notice("Elder is moving!!!");
        } else {
            $logger->info("Elder is not moving");
        }
    }
}
```

Aby sprawdzić jakie są dostępne serwisy w aplikacji należy uruchomić komedę w CLI

```bash
php bin/console debug:autowiring
```

> Symfony posiada sporo serwisów w kontenerze i każdy z serwisów posiada unikatowy id jak `request_stack` lub `router.default`.
> Aby sprawdzić całą dostępną listę serwisów można skorzystać z komendy

```bash
php bin/console debug:container
```
> ale zazwyczaj komenda `autowiring` jest wystarczająca

## Tworzenie/Konfigurowanie serwisu w kontenerze

Serwisy można tworzyć samemu poprzez organizowanie własnego kodu. Dla przykładu to tworzenie kodu, który ma generować szczęśliwą wiadomość.

Serwis:

```php
// src/Service/MessageGenerator.php
namespace App\Service;

class MessageGenerator
{
    public function getHappyMessage(): string
    {
        $messages = [
            'You did it! You updated the system! Amazing!',
            'That was one of the coolest updates I\'ve seen all day!',
            'Great work! Keep going!',
        ];

        $index = array_rand($messages);

        return $messages[$index];
    }
}
```

Kontroler który będzie korzystać z przygotowanego serwisu:

```php
// src/Controller/ProductController.php
use App\Service\MessageGenerator; // załadowanie serwisu MessageGenerator
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

class ProductController extends AbstractController
{
    #[Route('/products/new')]
    public function new(MessageGenerator $messageGenerator): Response // wstrzyknięcie serwisu jako argumentu
    {
        // thanks to the type-hint, the container will instantiate a
        // new MessageGenerator and pass it to you!
        // ...

        $message = $messageGenerator->getHappyMessage();
        $this->addFlash('success', $message);
        // ...
    }
}
```

> W momencie dodania serwisu do kontrolera, to kontener serwisów stworzy obiekt `MessageGenerator` i instancję tego obiektu. 
> Ale jeżeli nie wywołamy serwisu, kontener serwisów nie stworzy jego instancji oszczędzając pamięć i moc obliczeniową.
> `MessageGenerator` w momencie tworzenia jest tworzony tylko raz i przechowywany wewnątrz kontenera serwisów i zawsze jest zwraca jego tylko instancja w momencie wywołania danego serwisu.

## Możliwości z serwisami

1. Limiting Services to a specific Symfony Environment - 📚 symfony.com/doc/current/service_container.html#limiting-services-to-a-specific-symfony-environment
2. Injecting Services/Config into a Service - https://symfony.com/doc/current/service_container.html#limiting-services-to-a-specific-symfony-environment

## Serwis korzystający z serwisu

`Serwisy` mają możliwość korzystania z innych serwisów. Można importować do jednego serwisu wiele innych serwisów z których można korzystać później w klasie w odpowiednich metodach

Aby dodać serwis do serwisu należy podczas tworzenia serwisu bazowego utworzyć specjalną funkcję php `__contruct` której za zadaniem jest konstruowanie oraz inicjalizowanie metod i zmiennych w klasie.

Przykład:

```php
// src/Service/MessageGenerator.php
namespace App\Service;

use Psr\Log\LoggerInterface;

class MessageGenerator
{
    public function __construct(
        private LoggerInterface $logger,
    ) {
    }

    public function getHappyMessage(): string
    {
        $this->logger->info('About to find a happy message!');
        // ...
    }
}
```

### Dodawanie wielu serwisów do jednego serwisu

Proces dodania kilku serwisów do jednego serwisu przebiega analogicznie jak podczas dodawania jednego serwisu. Należy w metodzie `__contruct` zainicjować serwisu i przypisać je do zmiennych.

Przykład:

```php
// src/Service/SiteUpdateManager.php
namespace App\Service;

use App\Service\MessageGenerator;
use Symfony\Component\Mailer\MailerInterface;
use Symfony\Component\Mime\Email;

class SiteUpdateManager
{
    public function __construct(
        // Inicjacja serwisów pod postacią zmiennych
        private MessageGenerator $messageGenerator,
        private MailerInterface $mailer,
    ) {
    }

    public function notifyOfSiteUpdate(): bool
    {
        $happyMessage = $this->messageGenerator->getHappyMessage();

        $email = (new Email())
            ->from('admin@example.com')
            ->to('manager@example.com')
            ->subject('Site update just happened!')
            ->text('Someone just updated the site. We told them: '.$happyMessage);

        $this->mailer->send($email);

        // ...

        return true;
    }
}
```

## Umożliwienie Autowire serwisom i parametrom które domyślnie nie są możliwe do łaczenia za pomocą Autowire

Niektóre z serwisów domyślnie nie są możliwe do przekazania za pomocą techniki `AutoWire` dlatego wymagane jest wymuszenie na nich takiej możliwości.

Parametry możemy pozyskać w kontrolerze za pomocą metody `getParameter` ale ta metoda jest tylko dostępna w kontrolerach. Jeżeli będziemy chcieli skorzystać z parametru na przykład w serwisie i spróbujemy się do niego dostać za pomocą metody `getParameter` to symfony zwróci nam błąd.

Przykład w kontrolerze:

```php
// src/Controller/MainController.php
class MainController extends AbstractController
{
    public function homepage(): Response
    {
        dd($this->getParameter('iss_location_cache_ttl'));
    }
}
```

Przykład przekazania parametru jako argument kontrolera:

Ten przykład zwróci bład, ponieważ symfony nie może argumentu `$issLocationCacheTtl` przypisać jako wartość za pomocą `Autowire`
```php
    public function homepage($issLocationCacheTtl,): Response 
    {
        // code...
    }
```

Aby dostać się do parametru bez wykorzystywania metody `getParameter` należy skorzystać z atrybutu `#[Autowire()]`

Przykład użycia atrybutu pozwalającego na wpięcie parametru za pomoca `AutoWire`

```php
use Symfony\Component\DependencyInjection\Attribute\Autowire; // wymagany import klasy
class MainController extends AbstractController
{
    public function homepage(
        #[Autowire(param: 'iss_location_cache_ttl')] //wywołanie atrybutu Autowire
        $issLocationCacheTtl, // poprawne przekazanie parametru jako argument, to samo można uzyskać w serwisie
    ): Response {
    }
}
```

### Wpięcie non-autowire serwisu

Istnieją serwisy które domyślnie nie mogą zostać poddane mechanizmowi `AutoWire` i również wymagają wykorzystania atrybutu `#[Autowire()]`

Przykład użycia:

```php
use Symfony\Component\DependencyInjection\Attribute\Autowire; // import wymaganej klasy
class MainController extends AbstractController
{
    public function homepage(
        #[Autowire(service: 'twig.command.debug')] // wykorzystanie atrybutu do 
        DebugCommand $twigDebugCommand,
    ): Response {
    }
}
```