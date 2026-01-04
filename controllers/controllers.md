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

### Generowanie kontrolerów za pomocą CLI

Jeżeli projekt ma zainstalowany `Symfony maker` to jest możliwość korzystania z wbudowanych komend CLI do generowania już gotowych klas

Instalacja `Symfony maker` 📚 - https://symfony.com/bundles/SymfonyMakerBundle/current/index.html

Komenda umożliwiająca wygenerowanie nowy kontroler

```bash
php bin/console make:controller nazwa_nowego_kontrolera
```

Jeszcze jest możliwość wygenerowania całego `CRUD` na podstawie `Doctrine entity`. Aby móc wygenerować `CRUD` najpierw musimy posiadać skonfigurowany `Doctrine ORM` 📚 - https://symfony.com/doc/current/doctrine.html

Komenda umożliwiająca utworzenie `CRUD` na podstawie `Doctrine entity`:

```bash
php bin/console make:crud Product
```

## Obiekt request jako argument kontrolera

Zdarza się, że aplikacja będzie musiała odczytać parametry, pobrać głowę request'u albo uzyskać dostęp do przesłanego pliku. Takie informacje są przechowywane w obiekcie `Symfony Request object`. Aby uzyskać dostęp do takiego obiektu w kontrolerze należy go dodać jako argument do kontrolera oraz zaimportować klasę `Request`.

```php
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
// ...

public function index(Request $request): Response
{
    $page = $request->query->get('page', 1);

    // ...
}
```

Istnieje możliwość automatycznego mapowania danych jakie są przechowywane w obiekcie `Request` za pomocą atrybutu `MapQueryParameter` i przekazania ich do kontrolera.

Przykład takiego mapowania może polegać na wysłaniu zapytania o następującej wartości:

`https://example.com/dashboard?firstName=John&lastName=Smith&age=27`

Zapytanie możemy mapować w taki sposób:

```php
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\HttpKernel\Attribute\MapQueryParameter;

// ...

public function dashboard(
    // Automatyczne wyciągnięcie parametrów z zapytania pasujących do nazwy zmiennej
    #[MapQueryParameter] string $firstName, //firstName=John
    #[MapQueryParameter] string $lastName, //lastName=Smith
    #[MapQueryParameter] int $age, //age=27`
): Response
{
    // ...
}
```

Istnieje więcej możliwości mapowania zapytań tutaj jest więcej opisanych 📚 - https://symfony.com/doc/current/controller.html#automatic-mapping-of-the-request

## Zarządzanie sesją

Kontroler ma możliwość do zarządzania sesją użytkownika. Przykładem zarządzania sesją użytkownika jest wykorzystanie `$this->addFlash()` - metoda odpowiedzialna za wyświetlenie wiadomości typu `flash` automatycznie znika z sesji po wyświetleniu.

```php
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
// ...

public function update(Request $request): Response
{
    // ...

    if ($form->isSubmitted() && $form->isValid()) {
        // do some sort of processing

        $this->addFlash(
            'notice',
            'Your changes were saved!'
        );
        // $this->addFlash() is equivalent to $request->getSession()->getFlashBag()->add()

        return $this->redirectToRoute(/* ... */);
    }

    return $this->render(/* ... */);
}
```

Więcej na temat sesji tutaj 📚 - https://symfony.com/doc/current/session.html#session-intro

## Request i Response obiekt

Obiekt `Request` jest przekazywany do kontrolera w momencie dodania go jako parametru.

Klasa `Request` posiada metody jaki i własności dostępne publicznie, które potrafią zwracać informacje na temat aktualnie uzyskanego requestu.

Przykładowe metody dostępne w klasie `**Request**`

```php
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;

public function index(Request $request): Response
{
    $request->isXmlHttpRequest(); // is it an Ajax request?

    $request->getPreferredLanguage(['en', 'fr']);

    // retrieves GET and POST variables respectively
    $request->query->get('page');
    $request->getPayload()->get('page');

    // retrieves SERVER variables
    $request->server->get('HTTP_HOST');

    // retrieves an instance of UploadedFile identified by foo
    $request->files->get('foo');

    // retrieves a COOKIE value
    $request->cookies->get('PHPSESSID');

    // retrieves an HTTP request header, with normalized, lowercase keys
    $request->headers->get('host');
    $request->headers->get('content-type');
}
```

Tak samo jak klasa `Request`, klasa `Response` posiada metody i właściwości publicznie dostępne. Jednak obiekt response jest typu `ResponseHeaderBag` obiekt tego typu posiada metody umożliwiające na pozyskanie lub ustawienie `response headers`.
Nazwa nagłówka (Header) jest normalizowana co w rezultacie nazwa `Content-Type` jest równa nazwą `content-type` lub `content_type`

Kontrolery w Symfony mają wymóg zby zwracać obiekt `Response`

```php
use Symfony\Component\HttpFoundation\Response;

// creates a simple Response with a 200 status code (the default)
$response = new Response('Hello '.$name, Response::HTTP_OK);

// creates a CSS-response with a 200 status code
$response = new Response('<style> ... </style>');
$response->headers->set('Content-Type', 'text/css');
```

Różne obiekty `Response` są zawarte aby zwracać różne odpowiedzi w zależności od typu takiej odpowiedzi.

Podstawowe typy to:

1. Uzyskiwanie dostępu do wartości konfiguracji
2. Zwracanie JSON Response
3. Stremowanie pliku jako obiektu Response

1. Uzyskiwanie dostępu do wartości konfiguracji, aby uzyskać wartość dowolnego parametru konfiguracji z kontrolera, jest używana do tego pomocnicza metoda `getParameter()`

Przykład:

```php
// ...
public function index(): Response
{
    $contentsDir = $this->getParameter('kernel.project_dir').'/contents';
    // ...
}
```

2. Zwracanie JSON Response, aby zwrócić odpowiedź JSON z kontrolera, jest używana pomocnicza metoda `json()`, która zwróci obiekt typu `JsonResponse` który koduje dane automatycznie

Przykład:

```php
use Symfony\Component\HttpFoundation\JsonResponse;
// ...

public function index(): JsonResponse
{
    // returns '{"username":"jane.doe"}' and sets the proper Content-Type header
    return $this->json(['username' => 'jane.doe']);

    // the shortcut defines three optional arguments
    // return $this->json($data, $status = 200, $headers = [], $context = []);
}
```

3. Stremowanie pliku jako obiektu Response, aby stremować plik do przeglądarki z kontrolera używana jest pomocnicza funkcja `file()`

Przykład:

```php
use Symfony\Component\HttpFoundation\BinaryFileResponse;
// ...

public function download(): BinaryFileResponse
{
    // send the file contents and force the browser to download it
    return $this->file('/path/to/some_file.pdf');
}
```

Funkcja `file()` również dostarcza dodatkowe argumenty aby mieć opcje to konfiguracji zachowania

Przykład:

```php
use Symfony\Component\HttpFoundation\File\File;
use Symfony\Component\HttpFoundation\ResponseHeaderBag;
// ...

public function download(): BinaryFileResponse
{
    // load the file from the filesystem
    $file = new File('/path/to/some_file.pdf');

    return $this->file($file);

    // rename the downloaded file
    return $this->file($file, 'custom_name.pdf');

    // display the file contents in the browser instead of downloading it
    return $this->file('invoice_3241.pdf', 'my_invoice.pdf', ResponseHeaderBag::DISPOSITION_INLINE);
}
```

Wiecej informacji na temat samych obiektów `Response` oraz `Request` tutaj: 📚 - https://symfony.com/doc/current/components/http_foundation.html#request