# Bazy danych

Aby projekt symfony posiadał poprawnie działającą `Doctrine` najpierw jest wymagane postawienie bazy danych obok. Oraz połaczenia jej z projektem, aby była możliwość poprawnego uruchamiania migracji.

TODO - dodać opis konfiguracji bazy danych
TODO - dodać opis tworzenia doctrine

## Doctrine

Jest to oddzielna aplikacja umożliwiająca na mapowanie obiektów `ORM` (Object Relational Mapper) dla PHP. Co przyczynia się do tego, że zamiast pisać kod SQL piszemy obiekty w kodzie PHP które są mapowane na zapytania SQL.

Główne funkcje jakie dostarcza `Doctrine` to:
- Doctrine ORM - mapowanie zapytań SQL na klay PHP (Entity)
- Doctrine DBAL - warstwa abstrakcji bazy danych do wykonywania zapytań SQL za pomocą PHP
- Migracje - wersjonowanie i aktualizacja schematu bazy danych
- DQL (Doctrine Query Language) - obiektowy język zapytań podobny do SQL