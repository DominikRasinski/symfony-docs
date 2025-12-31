# Kontroler

Kontroler to funkcja której zadaniem jest przyjmowanie informacji z zapytań (`Request object`) i tworzenie lub zwracanie odpowiedzi (`Response object`).
Zwrócona odpowiedź może być pod dowolną postacią taką jak `strona HTML`, `JSON`, `XML`, `plik do pobrania`, `przekierowanie`, `404 error` lub cokolwiek innego. Kontroler jest odpowiedzialny za uruchamianie dowolnej logiki jakiej potrzebuje aplikacja.

## Podstawowy kontroler

Kontroler może być dowolną możliwą do wywołania logiką, jak na przykład funkcją, metodą lub obiektem. Ale zazwyczaj konwencja jest taka aby kontroler był metodą klasy, która będzie zarządzać jej metodami.

```php
namespace App\Controller;

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;

class HomeController extends AbstractController // <- klasa zarządzająca życiem kontrolerów
{

    #[Route('/')]
    public function home(): Response // <- kontroler
    {
        return $this->render(
               'home.html.twig', []
        );
    }   
}
```

Kontrolerem jest metoda `home` która istnieje w środku klasy `HomeController`

1. `namespace App\Controller;` - dodanie do przestrzeni nazw App\Controller pozawala na wykorzystanie importowania dodatkowych komponentów za pomocą `use`
2. `use Symfony\Component\HttpFoundation\Response;` - importowanie klasy pozwalającej na wykorzystanie obiektu `Response` dzięki wykorzystaniu `namespace`
3. `class HomeController extends AbstractController` - klasa `HomeController` zarządzające życiem kontrolerów wewnątrz niej
4. `#[Route('/')]` - definicja `routingu` pod którym będzie się uruchamiać dany kontroler
5. `public function home(): Response` - kontroler zwracający obiekt `Response` kidy zostanie przekazany request o stronę główną `/`

## Podstawowa klasa kontrolera i serwisu

Aby mieć możliwość korzystania z metod typu `helper` w kontrolerze należy rozszerzyć klasę przechowującą kontrolery o klasę `AbstractController`

```php
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;

class HomeController extends AbstractController
{
    //...
}
```
Dzięki temu, teraz kontrolery w klasie `HomeController` mogą korzystać z metod takich jak `$this->render()` i wiele innych

### Generowanie URL

Aby wygenerować nowy URL możemy skorzystać z metody `generateURL()` dla wskazanego `route`
```php
$url = $this->generateUrl('app_lucky_number', ['max' => 10]);
```

### Przekierowania

Aby stworzyć przekierowanie do kolejnej strony to należy skorzystać z metod `redirectToRoute()` oraz `redirect()`

Przykład:
```php
use Symfony\Component\HttpFoundation\RedirectResponse;
use Symfony\Component\HttpFoundation\Response;

// ...
public function index(): RedirectResponse
{
    // redirects to the "homepage" route
    return $this->redirectToRoute('homepage');

    // redirectToRoute is a shortcut for:
    // return new RedirectResponse($this->generateUrl('homepage'));

    // does a permanent HTTP 301 redirect
    return $this->redirectToRoute('homepage', [], 301);
    // if you prefer, you can use PHP constants instead of hardcoded numbers
    return $this->redirectToRoute('homepage', [], Response::HTTP_MOVED_PERMANENTLY);

    // redirect to a route with parameters
    return $this->redirectToRoute('app_lucky_number', ['max' => 10]);

    // redirects to a route and maintains the original query string parameters
    return $this->redirectToRoute('blog_show', $request->query->all());

    // redirects to the current route (e.g. for Post/Redirect/Get pattern):
    return $this->redirectToRoute($request->attributes->get('_route'));

    // redirects externally
    return $this->redirect('http://symfony.com/doc');
}
```

> **UWAGA** Metoda `redirect()` nie sprawdza celu przekierowania w żaden sposób. Co może prowadzić do nie spodziewanych problemów jak i ryzyka wycieku danych więcej informacji tutaj 📚 - https://cheatsheetseries.owasp.org/cheatsheets/Unvalidated_Redirects_and_Forwards_Cheat_Sheet.html

### Dodawania serwisów

Symfony posiada wielką ilość pomocnych klas i funkcjonalności nazwanych jako serwisy.

Aby dodać serwis do kontrolera można wykorzystać mechanizm `autowiring` który opiera się na metodzie `argument injections` czyli po prostu podaniu argumentu do parametru aby mieć dostęp do dowolnego serwisu.

Przykład dodania serwisu `LoggerInterface` pod postacią zmiennej `$logger`:

```php
use Psr\Log\LoggerInterface;
use Symfony\Component\HttpFoundation\Response;
// ...

#[Route('/lucky/number/{max}')]
public function number(int $max, LoggerInterface $logger): Response
{
    $logger->info('We are logging!');
    // ...
}
```

Aby sprawdzić jakie są dostępne serwisy możliwe do wpięcia możemy wykorzystać komendę w `cli`

```bash
php bin/console debug:autowiring
```
Która wyświetli wszystkie dostępne serwisy gotowe do wpięcia jako argumenty do kontrolerów