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
