# Symfony

Uruchamianie projektów:
- `composer install`
- `symfony serve`

Jeżeli nie wstaje, to oznacza, że brakuje nam potrzebnych zależności, takich jak:
- `php`
- `symfony`
- `composer`

## Stworzenie pierwszej strony (szybki start)

Za renderowanie oraz ruch są odpowiedzialne kontrolery w symfony, to one sprawdzają jaki został podany routing i podejmują na jego podstawie odpowiednie działania jak zwrócenie w odpowiedzi strony, albo wartości JSON w przypadku end-pointa odpowiedzialnego za działanie API.

Kontrolery są przetrzymywane w ścieżce `./src/Controller/<Nazwa_kontrolera>`

Aby kontroler był dostępny pod daną ścieżką i umożliwiał na zwrócenie określonego typu danych, lub obiektu odpowiedzi należy w kontrolerze nad metodą która jest odpowiedzialna za określony wątek, działanie, zwrócenie odpowiedzi podać ścieżkę pod którą będzie się identyfikować.

Każdy kontroler jest klasą, którą możemy rozbudowywać za pomocą jej rozszerzanie o następne metody. Właśnie te metody będą wywoływane w momencie kiedy odpytamy daną ścieżkę. Zalecane dodanie ścieżki jest sa pomocą `atrybutu` nad metodą, jest to mechanizm wykorzystywany w `PHP` do przekazywania metadanych do kodu, bez ingerencji w sam kod, działa jak `dekorator`

Przykład dodania ścieżki w kontrolerze za pomocą atrybutu:

```php
class HomeController extends AbstractController
{

    #[Route('/')] // <- Atrybut definiujący ścieżkę home
    public function home(): Response // <- Metoda home która zostanie uruchomiona w momencie przejścia do głównego katalogu strony
    {
        return $this->render(
               'home.html.twig', []
        );
    }   
}
```

Aby kontroler mógł zwracać odpowiedź `Response` jako odpowiedź przeglądarki należy zaimportować klasę `Response` w głównej klasie kontrolera

```php
use Symfony\Component\HttpFoundation\Response;
```

Jeszcze potrzebujemy atrybut `Route`, który pozwoli nam na mapowanie URL pod konkretną metodę kontrolera.

```php
use Symfony\Component\Routing\Attribute\Route;
```

Dzięki temu importowi możemy skorzystać z tego zapisu `#[Route('/')]`

Aby skorzystać w kontrolera z renderowania template twig najpierw musimy zainstalować `twig`
`composer require twig`

Potem w kontrolerze należy importować `AbstractController` który udostępni nam metodę `render()` i nie tylko!

Przykładowe metody które udostępnia klasa `AbstractController`:
- json() - tworzenie odpowiedzi JSON
- redirectToRoute() - przekierowanie do innej routy
- createNotFoundException() - tworzenie wyjątku 404
- ge-Doctrine() - dostęp do Doctrine ORM
- getUser() - pobieranie zalogowanego użytkownika
- isGranted() - sprawdzanie uprawnień
- addFlash() - dodawanie komunikatów flash

Również aby mieć możliwość do korzystania z metod klasy `AbstractController` należy kontroler aktualnie tworzony rozszerzyć o dziedziczenie z klasy `AbstractController`

```php
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;

class HomeController extends AbstractController
{
    // code...
}
```

## Struktura podstawowego projektu 

### Główne foldery

- `config/` - folder przechowujący konfiguracje, tutaj można konfigurować routing, serwisy oraz zarządzanie paczkami
- `src/`  - folder przechowujący cały dostępny kod, tutaj jest pisana cała logika
- `templates/` - folder generowany automatycznie po zainstalowaniu pakietu `twig` w nim można odnaleźć wszystkie pliki `twig`

### Pozostałe foldery

- `bin/` - folder przechowujący plik `bin/console` oraz wszystkie mniej ważne pliki wykonywalne
- `var/` - folder przechowujący wszystkie automatycznie tworzone pliki. Typu `cache` (var/cache/) oraz `logi` (var/log/)
- `vendor/` - folder przeznaczony do przechowywania zewnętrznych paczek/bibliotek instalowanych za pomocą `Composer package manager`
- `public/` - główny folder do przechowywania wszystkich publicznie dostępnych plików, takich jak obrazki wykorzystywane na stronie albo regulaminy itp.

## Ważne kwestie działania FrameWorku

- [`Routing`](routing/Routings.md)