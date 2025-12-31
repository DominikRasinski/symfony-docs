# Jak działa Routing w Symfony

Podczas gdy aplikacja odbiera zapytanie, to zapytanie uderza do odpowiedniej akcji kontrolera aby wygenerować odpowiednią odpowiedź. Konfiguracja `routingu` definiuje jaka ma zostać uruchomiona akcja dla każdego przychodzącego zapytania w postaci URL. Przy okazji konfiguracja routingu dostarcza `SEO friendly URLs` dla przykładu skonfigurowany URL będzie wyglądać tak `/read/intro-to-symfony` zamiast tak `index.php?article_id=57`.

## Tworzenie Routingów

Routingi można skonfigurować w oddzielnych plikach takich jak YAML, PHP albo jako dekorator do metody za pomocą atrybutów. Nie ma żadnej różnicy podczas wykorzystywania oddzielnych plików a atrybutów, to już jest zależne od projektu i ustaleń jakie zostały podjęte, choć zalecane jest wykorzystywanie atrybutów aby utrzymywać logikę w obrębie jednego pliku.

**UWAGA** - jeżeli nie korzystamy w Symfony Flex to wykorzystywanie atrybutów jako routingu NIE będzie działać. Ponieważ potrzebna jest konfiguracja przedstawiona poniżej

`# config/routes/attributes.yaml` - ścieżka gdzie plik musi się znaleźć aby atrybuty były poprawnie odczytywane
```yaml
controllers:
    resource:
        path: ../../src/Controller/
        namespace: App\Controller
    type: attribute

kernel:
    resource: App\Kernel
    type: attribute
```

> Jeżeli w jednym pliku będzie zdefiniowane kilka klas (kontrolerów) Symfony załaduje tylko routingi z pierwszej klasy ignorując pozostałe.

**Możliwe jest tworzenie routingów za pomocą plików konfiguracyjnych takich jak YAML,PHP ale na to nie będę poświęcać czasu** 📚 - symfony.com/doc/current/routing.html#creating-routes-in-yaml-or-php-files

### Wyłapywanie odpowiednich metod HTTP

Domyślnie routingi wyłapują każde zapytanie HTTP (`GET`, `POST`, `PUT`, itp.). Aby zapewnić wykorzystanie odpowiedniej metody w routingu należy skorzystać z parametru `methods` dzięki temu route będzie tylko odpowiadać na zapytanie z odpowiednią metodą.

**Przykład `routingu` który będzie tylko zwracać odpowiedź gdy zostanie zapytany wybraną `metodą`**:

```php
#[Route('/api/test-subjects/{id}', methods: ['GET', 'HEAD'])]
public function showSubject(int $id): Response
{
    // Zwrocenie obiektu testowe o podanym id w postaci JSON
}

#[Route('/api/test-subjects/{id}', methods: ['PUT'])]
public function editSubject(int $id): Response
{
    // Edytowanie wskazanego obiektu testowego
}
```

> Metoda `HEAD` pozwala na zwrócenie tylko nagłówków zapytania bez ciała (body). Umożliwia to określenie  czy dana końcówka API istnieje za pomocą tylko uzyskania statusu lub dowiedzenia się jakiego typu lub długości jest zawartość (Content-Length, Content-Type)

### Wyrażenia podczas zapytań

Podczas tworzenia `routingu` można skorzystać z warunków, które będą nam definiować dostęp do danego `routingu`. Aby skorzystać z warunków należy do routingu dodać parametr `condition`.

Warunki jakie jeszcze muszą zostać spełnione są zapisywane za pomocą `expression language syntax` który bazuje na składni wyrażeń z `Twig`
📚 - https://symfony.com/doc/current/reference/formats/expression_language.html

**Przykładowe zastosowanie `condition` w `routingu`**:

```php
#[Route(
    '/classified'
    name: 'classified'
    condition: "context.getMethod() in ['GET', 'HEAD'] and request.headers.get('User-Agent') matches '/firefox/i'",
)]
public function show(): Response
{
    // Zwróć obiekt nie widoczny dla osób, które korzystają z innej przeglądarki niż firefox
}
```

Wyrażenia również pozwalają na dostęp do parametrów które są zaszyte w routingu. Aby mieć możliwość na uzyskanie dostępu do takiego parametru wystarczy, że skorzystamy z tablicy `params[]` gdzie pomiędzy kwadratowymi nawiasami należy podać nazwę parametru z `routingu`

**Przykładowe zastosowanie tablicy `params[]` w routingu**:

```php
#[Route(
    '/declassified/{id}',
    name: 'declassified',
    condition: "params['id'] < 500"
)]
public function show(int $id): Response
{
    // Zwróć dane obiektu którego id jest mniejsze od 500
}
```

Parametr `condition` również posiada specjalnie przygotowane zmienne jakie dostarcza Symfony, one też mogą być wykorzystywane do określania warunków
- `context` - instancja RequestContext, która przetrzymuje podstawowe informacje na temat `route` który został odpytany
- `request` - Symfony request obiekt reprezentuje aktualny request
- `params` - tablica parametrów jakie zostały przekazane w aktualnym `route`

Symfony zezwala na skorzystanie również z funkcji takich jak te:
- `env(string $name)` - zwraca wartość zmiennej jaka jest używana przez Environment Variable Processors
- `service(string $alias)` - zwraca serwis podpięty jako warunek to `routingu`

Aby zadziałała funkcja `service(string $alias)` musimy zdefiniować na początku, serwis jakie będzie importować `AsRoutingConditionService` i dodawać do siebie jako atrybut alias odnoszący się do niego

**Przykład serwisu jaki będzie można potem wykorzystać w kontrolerze jako parametr**:

```php
use Symfony\Bundle\FrameworkBundle\Routing\Attribute\AsRoutingConditionService;
use Symfony\Component\HttpFoundation\Request;

#[AsRoutingConditionService(alias: 'route_checker')]
class RouteChecker
{
    public function check(Request $request): bool
    {
        // ...
    }
}
```

Jak posiadamy już serwis do którego możemy się odnieść za pomocą aliasu to możemy podać go w `route` jako parametr `condition`

**Przykład**:

```php
class ArchiveController 
{
    #[Route(condition: "service('route_checker).check(request)")]
    public function show(): Response
    {
        // Zwrócenie wartości archiwum jak route zostanie sprawdzony przez serwis
    }
}
```

### Route Parameters

Są dość rozbudowaną częścią zarzadzania `routami` ponieważ zezwalają na ich różne konfigurację, ale najważniejszą z nich jest możliwość skorzystania z `slug` w samym route.

Parametry są zapisywane w `route` za pomocą nawiasów klamrowych `{}`

**Przykład**:

```php
#[Route('/naytiba/{slug}', name: 'naytiba_show')]
public function show(string $slug): Response
{
    // Wykonanie kodu
}
```

Parametr `{slug}` jest równy dynamicznej części URL. Co oznacza, że nie trzeba zapisywać wszystkich możliwych kombinacji, ponieważ `slug` będzie automatycznie przechowywać dynamiczną część URL czyli `/naytiba/older-one` zostanie podzielony na część statyczną i część dynamiczną, a zmienna `slug` będzie przetrzymywać część dynamiczną w tym przypadku slug będzie równy 'older-one' i zostanie ona przekazana do metody `show`

#### Route może posiada wiele różnych parametrów

Routing może mieć wiele różnych parametrów z których można skorzystać, ale każdy parametr może zostać tylko raz wykorzystany w routingu.

**Przykład**:

Posiada poprawną liczbę parametrów, czyli nie ma duplikatów
```
/blog/posts-about-{category}/page/{pageNumber}
```

Posiada NIE poprawną liczbę parametrów, czyli jeden z parametrów już wykorzystanych się powtarza
```
/blog/posts-about-{category}/page/{pageNumber}/post/{pageNumber}
```

Dokumentacja opisująca większą ilość przypadków jak i możliwości odnośnie parametrów wykorzystywanych w routingu tutaj -> 📚 https://symfony.com/doc/current/routing.html#route-parameters

## Debugowanie Routingu

Aby sprawdzać co się dzieje z routingiem mamy dostępną metodą w `cli` symfony pod nazwą:

```bash
php bin/console debug:router
```
Wykonanie jej wyświetli wszystkie dostępne routingi w aplikacji posiada kilka pomocnych do działania parametrów takich jak:

### Aby wyświetlić routingi o podanych metodach, należy skorzystać z flagi --method=nazwa_metody_http

```bash
php bin/console debug:router --method=GET
```

```bash
php bin/console debug:router --method=ANY
```

### Również można sprawdzić konkretnie nas obchodzący routing poprzez podanie jego nazwy w `cli`

```bash
php bin/console debug:router app_lucky_number
```
Wyświetli podstawowe informacje gdzie taki routing się znajduje

### Sprawdzenie jaki route załapie podany URL

Można przetestować czy routing działa poprawnie poprzez podanie spodziewanego adresu URL jaki powinien zostać obsłużony przez controller i nową metodą podpiętą pod to zapytanie

```bash
php bin/console router:match /lucky/number/8
```

Komenda sprawdzi jaki `route` odpowiedział na ciąg znaków w tym przypadku `/lucky/number/8` ale to może być dowolny ciąg, ważne aby to był poprawny ciąg Http

## Generowanie URL

Aby generować URL'e należy podawać do zdefiniowanego `route` parametr `name` który musi być unikalny. Dzięki temu będzie można sprawnie wygenerować link URL do określonej akcji w odrębnym kontrolerze, serwisie. Jeżeli nie dodamy parametru `name` to symfony sam wygeneruje url na podstawie połaczenia nazwy kontrolera i wykonywanej akcji.

📚 - https://symfony.com/doc/current/routing.html#generating-urls-in-controllers