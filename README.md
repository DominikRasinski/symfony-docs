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
