# Serwisy

Serwisy to tak naprawdę re-używalne obiekty pozwalające na łatwiejsze wykonanie danego zadania lub operacji. Zazwyczaj wszystko to co jest wykonywane przez aplikację jest nazywane jako serwis. Na przykład możliwość wysłania maila jest wykonywana przez serwis połaczenie się do bazy danych też jest wykonywane przez serwis i tak dalej...

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
    public function logElderBehavior(LoggerInterface $logger)
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