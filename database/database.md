# Bazy danych

Aby projekt symfony posiadał poprawnie działającą `Doctrine` najpierw jest wymagane postawienie bazy danych obok. Oraz połaczenia jej z projektem, aby była możliwość poprawnego uruchamiania migracji.

## Doctrine

Jest to oddzielna aplikacja umożliwiająca na mapowanie obiektów `ORM` (Object Relational Mapper) dla PHP. Co przyczynia się do tego, że zamiast pisać kod SQL piszemy obiekty w kodzie PHP które są mapowane na zapytania SQL.

Główne funkcje jakie dostarcza `Doctrine` to:
- Doctrine ORM - mapowanie zapytań SQL na klay PHP (Entity)
- Doctrine DBAL - warstwa abstrakcji bazy danych do wykonywania zapytań SQL za pomocą PHP
- Migracje - wersjonowanie i aktualizacja schematu bazy danych
- DQL (Doctrine Query Language) - obiektowy język zapytań podobny do SQL

### Instalacja

W kursie Symnfonycast jest pokazana możliwość postawienia bazy danych za pomocą jednej komendy

```bash
composer require doctrine
```
Wychodzi na to, że jest to już nie obsługiwana komenda i należy przejść ręczną konfigurację za pomocą instrukcji z samego Symfony czyli na początku należy wykonać takie operacje

```bash
composer require symfony/orm-pack
composer require --dev symfony/maker-bundle
```

Poczym należy skonfigurować ustawienia bazy danych w pliku `.env`

Jeżeli baza danych na której będzie stać aplikacja jest zwykłym plikiem czyli `SqlLite` należy pamiętać o tym, że na nie posiada komendy `create` dlatego podczas próby uruchomienia komendy `php bin/console doctrine:database:create` zostanie zwrócony błąd, że ta platforma nie obsługuje takiej operacji. I to jest normalne baza `SqlLite` jest tylko plikiem i zostanie utworzona w momencie jak zostanie użyta.

#### Dalsze kroki

Po skonfigurowaniu bazy danych z jakiej chcemy korzystać należy wykonać następujące kroki:

1. Utworzenie klasy Entity dla tabeli
   1. Klasa jest tworzona za pomocą komendy `symfony console make:entity poczym podążamy za wskazówkami
   2. jeżeli model na którego podstawie chcemy utworzyć tabelę posiada niestandardowe typy, należy skorzystać z znaku `?` aby wyświetlić więcej typów, i aby dodać na przykład typ `enum` to zostaniemy poproszeni o jego pełną nazwę czyli `App\Model\TypEnum`
2. Po utworzeniu klasy `Entity` warto zwalidować czy `schema` bazy danych jest poprawna za pomocą komendy `symfony console doctrine:schema:validate`
3. Baza danych nadal będzie nie równa póki nie zostanie zrobiona migracja
   1. Utworzenie migracji `symfony console make:migration`
   2. Sprawdzenie czy jakie migracje są zakolejkowane `symfony console doctrine:migrations:list`
   3. Uruchomienie migracji `symfony console doctrine:migrations:migrate`
4. Baza danych jest gotowa!

## Korzystanie z bazy danych SqlLite

Aby skorzystać z bazy danych i posiadać dane na niej należy na początku uruchomić komendę

1. Uruchomić dostępne migracje aby powstała tabela
```bash
symfony console doctrine:migrations:migrate
```
2. Uruchomić `fixture` który wypełni bazę danych informacjami
```bash
symfony console doctrine:fixtures:load
```

Aby korzystać z bazy danych SqlLite należy wejść do folderu `var` i z poziomu `cli` wpisać komendę
```bash
sqlite3 .open <nazwa_bazy_danych>
```
Domyślna nazwa bazy danych w projekcie symfony to `data_dev.db`

**Uwaga** podczas pomyłki należy skasować cały plik bazy danych aby mieć pewność, że ID będzie się pokrywać z ID jakie jest wpisane do fixture. Niby odpalenie ponowne `fixture` robi purge na bazie danych ale to i tak zostawia poprzednią inkrementacje bazy.

## Entity

Jest to klasa odwzorowująca tabele z jaką chcemy się komunikować szybkim sposobem na stworzenie klasy `entity` jest wykonanie polecenia w CLI

```bash
make:entity
```

Klasa jest pośrednikiem pomiędzy kodem, a bazą danych. Dzięki klasie `entity` `doctrine` wie jak może działać na bazie danych, ponieważ klasa `entity` dostarcza schematu jak wygląda dana tabela.

### Pobieranie danych z bazy danych za pomocą `Doctrine` przy użyciu szablony `entity`

Aby mieć możliwość pobrania danych z bazy danych najpierw warto stworzyć cały proces za pomocą `Doctrine` oraz `Entity` i upewnić się czy migracje zostały wykonane poprawnie, jeżeli tak to można przejść do pobierania danych z bazy do samej aplikacji

Podczas wykonywania operacji `AppFixture::load()` (altess-levoire/altess-levoire/src/DataFixtures/AppFixtures.php) do tej operacji jest przekazywany parameter jako menadżer w przypadku tej aplikacji jest menadżer pośredniczący w operacjach na bazie danych to `Doctrine`. Dlatego teraz aby dobrać się do danych umieszczonych w bazie danych za pomocą `fixture` należy skorzystać z `EntityManagerInterface`

```php
//altess-levoire/altess-levoire/src/Controller/HomeController.php
class HomeController extends AbstractController
{

    #[Route('/')]
    public function home(EntityManagerInterface $em): Response // autowire EntityManagerInterface pod postacią zmiennej $em
    {

        $nayiba =  $em->createQuery('SELECT n FROM App\Entity\Naytiba n')->getResult(); //wykonanie surowego zapytania na bazie danych

        $selectedNaytiba = $nayiba[array_rand($nayiba)]; //przypisanie wyniku zapytania do zmiennej

         // wyświetlenie zapytania w szablonie
        return $this->render(
               'home.html.twig', [
                'selectedNaytiba' => $selectedNaytiba,
               ]
        );
    }   
}
```

Możliwe jest wykorzystywanie samych zapytań SQL, ale lepszym pomysłem będzie wykorzystanie zapytania na poziomie `klasy` aby z tego skorzystać należy wybrać `createQueryBuilder()`. Umożliwia wykonanie zapytań takich samych jak za pomocą `CreateQuery()` ale nie obciąża nas możliwością popełnienia literówek podczas wpisywania ścieżki do klasy `entity` z jakiej chcemy pobrać dane.

```php
//altess-levoire/altess-levoire/src/Controller/HomeController.php

    #[Route('/v2')]
    public function homeV2(EntityManagerInterface $em): Response
    {
        $naytiba = $em->createQueryBuilder()
        ->select('s')
        ->from(Naytiba::class, 's') //klasa to App\Entity\Naytiba aby przez przypadek nie podać modelu bo nie zadziała
        ->getQuery()
        ->getResult();

        $selectedNaytiba = $naytiba[array_rand($naytiba)];

        return $this->render(
               'home.html.twig', [
                'selectedNaytiba' => $selectedNaytiba,
               ]
        );
    }
```

Możliwe jest pobieranie danych za pomocą kryteriów takich jak: 
1. `findAll()` - pobiera wszystkie rekordy
2. `findBy(['field' => 'value'])` - pobiera według kryteriów
3. `findOneBy(['field' => 'value'])` - pobiera jeden rekord według kryteriów
4. `find()` - pobieranie pojedynczej encji na podstawie klucza głównego (primary key)

Wszystkie te metody są dostarczana za poprzez `EntityManagerInterface` który sam tworzy podstawowe zapytania, ale trzeba pamiętać, że jeżeli element nie zostanie znaleziony to zostanie zwrócona wartość `null`

```php
//altess-levoire/altess-levoire/src/Controller/NaytibaController.php
class NaytibaController extends AbstractController
{
    #[Route('/naytiba/{id<\d+>}', name: 'naytiba_show')]
    public function show(int $id, EntityManagerInterface $em): Response
    {
        $naytiba = $em->find(Naytiba::class, $id); //pobranie danych za pomocą klucza głównego przy wykorzystaniu metody find() z EntityManagerInterface

        return $this->render (
            'naytiba.html.twig', ['findNaytiba' => $naytiba]
        );
    }
}
```