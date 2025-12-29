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