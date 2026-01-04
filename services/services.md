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

TODO - sprawdzić czy wszystko jest opisane w tym temacie https://symfony.com/doc/current/service_container.html#fetching-and-using-services