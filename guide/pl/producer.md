# Producer

Ekran **Producer** wysyła wiadomości do Kafki: pojedynczo z formularza lub wiele naraz. Otwiera się go z poziomu połączenia w panelu **Połączenia** albo przyciskiem **Wyślij wiadomość** na karcie topiku; przycisk wypełnia pole topiku. Każde połączenie ma własną kartę Producer.

W połączeniach z włączoną opcją **Tylko do odczytu** ekran Producer otwiera się z ostrzeżeniem **Połączenie jest tylko do odczytu — wysyłanie jest niedostępne.** Przyciski wysyłania są ukryte, ale szablony i historię nadal można przeglądać.

<a id="send-message"></a>

## Wysyłanie wiadomości

1. W polu **Topik** wpisać topik. Pole podpowiada topiki klastra, bez topików wewnętrznych.
2. Opcjonalnie w polu **Partycja** wpisać numer partycji. Gdy pole pokazuje wartość **auto**, partycja jest wybierana według klucza: wiadomości z tym samym kluczem trafiają do tej samej partycji, tak jak w kliencie Java. Wiadomości bez klucza trafiają do losowej partycji.
3. Wypełnić pola **Klucz** i **Wartość** oraz wybrać ich formaty. Zob. [Formaty klucza i wartości](producer.md#key-value-formats).
4. Opcjonalnie dodać nagłówki.
5. Kliknąć przycisk **Wyślij**.

Po udanym wysłaniu wiersz pod formularzem pokazuje partycję, offset i czas wiadomości. Przy `acks` ustawionym na `0` broker nie zwraca offsetu i zamiast niego wyświetlany jest znak `—`.

Jeśli numer partycji jest zbyt duży lub broker nie potwierdzi wiadomości w limicie czasu połączenia, aplikacja pokazuje błąd. Nieistniejący topik również powoduje błąd, chyba że broker tworzy topiki automatycznie (`auto.create.topics.enable`). Przy wyłączonym automatycznym tworzeniu ten błąd może pojawić się dopiero po około 30 s (limit czasu połączenia). Błędy w formularzu, takie jak niepoprawny JSON lub nieznany symbol zastępczy, są wyświetlane, zanim cokolwiek zostanie wysłane.

W połączeniu prod wysłanie wymaga wpisania nazwy połączenia i potwierdzenia przyciskiem **Wyślij do prod**. Zob. [Środowiska i prod](connections.md#environments).

Przycisk **Wyczyść formularz** opróżnia formularz i przywraca domyślne ustawienia producenta. Topik pozostaje.

<a id="key-value-formats"></a>

## Formaty klucza i wartości

Pola **Format klucza** i **Format wartości** określają, jak tekst z pola jest zamieniany na bajty:

- `string` — tekst bez zmian, w UTF-8.
- `json` — tekst jest sprawdzany pod kątem poprawności JSON i wysyłany tak, jak został wpisany, bez przeformatowania.
- `hex` — dwie cyfry szesnastkowe na bajt, na przykład `48656c6c6f`. Spacje i podziały wierszy są ignorowane.
- `base64` — bajty w standardowym Base64.
- `null` — całkowity brak klucza lub wartości. Pole jest puste, a wpisanie tekstu przełącza format na `string`.

Domyślnie klucz ma format `null`, a wartość — `json`. Aby wysłać wiadomość z kluczem, wystarczy go wpisać: format zmieni się na `string`. Pusty ciąg znaków też jest kluczem, a nie brakiem klucza: jeśli wybrać `string` i zostawić pole puste, wszystkie takie wiadomości będą miały ten sam hash i trafią do jednej partycji.

Wiadomość z wartością `null` to tombstone. W topiku z `cleanup.policy=compact` sygnalizuje Kafce, że należy usunąć starsze wiadomości z tym samym kluczem.

<a id="headers"></a>

## Nagłówki

Nagłówki to pary nazwa–wartość wysyłane razem z wiadomością. Przycisk **Dodaj nagłówek** dodaje wiersz, w którym należy wypełnić kolumny **Nazwa**, **Wartość** i **Format**. Przycisk `×` usuwa wiersz.

Format wartości nagłówka to `string` (tekst w UTF-8) lub `hex` (bajty jako cyfry szesnastkowe). Wiersze z pustą nazwą nie są wysyłane. Kafka dopuszcza kilka nagłówków o tej samej nazwie i wszystkie są wysyłane.

Symbole zastępcze działają w wartościach nagłówków, ale nie w ich nazwach.

<a id="placeholders"></a>

## Symbole zastępcze

Symbole zastępcze są zastępowane wartościami przy każdym wysłaniu. Działają w kluczu, wartości i wartościach nagłówków. Ich lista jest wyświetlana pod nagłówkami:

- `{{uuid}}` — nowy losowy UUID, na przykład `0b6f6f2c-3c1e-4a8e-9a54-2f0d6f1d7c21`.
- `{{now}}` — bieżący czas UTC w formacie ISO 8601 z milisekundami, na przykład `2026-10-15T12:30:00.000Z`.
- `{{nowMillis}}` — bieżący czas w milisekundach od 1970-01-01 UTC.
- `{{seq}}` — numer kopii w partii, licząc od 0. Przy pojedynczym wysłaniu wynosi `0`.
- `{{randomInt 1 100}}` — losowa liczba całkowita z przedziału między dwiema liczbami, włącznie z nimi.
- `{{randomString 8}}` — losowy ciąg liter łacińskich i cyfr o podanej długości, maksymalnie 100000.
- `{{env NAME}}` — wartość zmiennej środowiskowej `NAME`. Jeśli nie jest ustawiona, nic nie jest wysyłane i wyświetlany jest błąd.

Spacje wewnątrz nawiasów klamrowych są dozwolone. Nieznany symbol zastępczy lub brak `}}` zatrzymuje wysyłanie z błędem. Aby wysłać `{{` jako tekst, należy napisać `\{{`.

W formacie `json` poprawność JSON jest sprawdzana po podstawieniu symboli zastępczych. Symbole zastępcze, które dają tekst, wymagają cudzysłowów: `{"id": "{{uuid}}"}`. Liczby ich nie wymagają: `{"n": {{seq}}}`.

`{{env NAME}}` odczytuje środowisko procesu Glafiry. Jeśli Glafira jest uruchamiana z Docka lub menu Start, może brakować zmiennych ustawionych tylko w profilu terminala.

Gdy wiadomość pochodzi z [otwierania na ekranie Producer](browse.md#open-in-producer) lub z historii, każde `{{` jest w niej zapisane jako `\{{`. Wiadomość jest wtedy wysyłana dokładnie w takiej postaci, w jakiej była, bez żadnych podstawień.

<a id="templates"></a>

## Szablony

Szablon zapisuje cały formularz: topik, partycję, klucz, wartość, nagłówki, formaty i ustawienia producenta. Symbole zastępcze są zapisywane tak, jak zostały wpisane, i podstawiane przy każdym wysłaniu.

Aby zapisać szablon, należy kliknąć przycisk **Zapisz jako szablon**, w polu **Nazwa szablonu** wpisać nazwę i kliknąć przycisk **Zapisz**. Topik może być pusty. Domyślnie szablon należy do tego połączenia. Zaznaczenie opcji **Dla wszystkich połączeń** pokazuje go we wszystkich połączeniach; takie szablony są oznaczone na liście dopiskiem **(wszystkie połączenia)**.

Aby użyć szablonu, należy wybrać go z listy **Szablon**. Formularz zostaje zastąpiony szablonem. Przed wysłaniem formularz można edytować; szablon się nie zmienia.

Zapisanie zawsze tworzy nowy szablon. Aby zmienić istniejący, należy go wczytać, zmienić formularz, zapisać pod nową nazwą i usunąć stary. Aby usunąć wybrany szablon, należy kliknąć przycisk **Usuń szablon** i potwierdzić przyciskiem **Usuń**.

Szablony połączenia są usuwane razem z połączeniem.

<a id="history"></a>

## Historia

Sekcja **Historia (ostatnie 100)** zawiera 100 ostatnich pojedynczych wysłań tego połączenia, od najnowszych. Wiadomości z wysyłek wsadowych nie są dodawane. Każdy wiersz pokazuje czas, topik, partycję i offset oraz początek klucza.

Historia przechowuje wiadomość w takiej postaci, w jakiej została wysłana, z już podstawionymi symbolami zastępczymi. Przycisk **Wyślij ponownie** wczytuje ją do formularza razem z ustawieniami producenta. Nic nie jest wysyłane, dopóki nie zostanie kliknięty przycisk **Wyślij**. Kopia jest wysyłana dokładnie tak, jak została zapisana: na przykład ponownie wysyłany jest ten sam UUID.

Klucz, wartość lub wartość nagłówka dłuższe niż 64 KB są zapisywane tylko częściowo. Dla takich wierszy przycisk **Wyślij ponownie** jest nieaktywny.

Historia jest przechowywana na tym komputerze otwartym tekstem, łącznie z wartościami `{{env NAME}}`. Jeśli ma to znaczenie, nie należy wysyłać sekretów przez ekran Producer. Historia połączenia jest usuwana razem z połączeniem.

<a id="batch-copies"></a>

## N kopii formularza

Sekcja **Wysyłka wsadowa** pod formularzem wysyła wiele wiadomości w jednym przebiegu. W trybie **N kopii formularza** wysyła formularz podaną liczbę razy.

1. Wypełnić formularz jak do pojedynczego wysłania.
2. Wybrać tryb **N kopii formularza** i w polu **Kopie** wpisać liczbę od 1 do 1000000.
3. Opcjonalnie ustawić pole **Szybkość**. Zob. [Szybkość wysyłania](producer.md#batch-rate).
4. Kliknąć przycisk **Uruchom**.

Symbole zastępcze są podstawiane osobno dla każdej kopii. `{{seq}}` to numer kopii, od 0 do N−1, więc każda wiadomość może otrzymać własny klucz, na przykład `order-{{seq}}`. Błędy symboli zastępczych są sprawdzane na pierwszej kopii przed rozpoczęciem przebiegu.

W trakcie przebiegu panel pokazuje, ile wiadomości wysłano, a ilu nie udało się wysłać. Na koniec pokazuje stan **gotowe**. Błędy są wyświetlane z numerem kopii, liczonym od 1. Lista pokazuje pierwsze 20 błędów; jeśli jest ich więcej, uwaga pod listą podaje łączną liczbę.

Przycisk **Anuluj** zatrzymuje przebieg. Wiadomości, które jeszcze nie opuściły aplikacji, nie są wysyłane, a panel pokazuje ich liczbę. Aplikacja czeka do 2 sekund, aż broker potwierdzi już wysłane wiadomości. Wiadomości, które nadal nie są potwierdzone, liczą się jako błędy, ponieważ nie wiadomo, czy zostały zapisane. Zamknięcie karty Producer również zatrzymuje przebieg.

W połączeniu prod przebieg wymaga wpisania nazwy połączenia i potwierdzenia przyciskiem **Uruchom na prod**.

<a id="batch-file"></a>

## Wysyłanie z pliku

W trybie **z pliku** każdy wiersz lub rekord pliku staje się wiadomością. Obsługiwane pliki to JSON Lines (`.jsonl`, `.ndjson`) i CSV (`.csv`).

1. W polu **Topik** w formularzu powyżej wpisać topik. Wszystkie wiadomości trafiają do tego topiku.
2. Wybrać tryb **z pliku** i kliknąć przycisk **Wybierz plik…**.
3. Kliknąć przycisk **Uruchom**.

Klucz, wartość, nagłówki i partycja z formularza nie są używane, a symbole zastępcze nie są podstawiane. Obowiązują ustawienia producenta z formularza.

W JSON Lines każdy wiersz to jeden obiekt JSON, na przykład `{"key": "k1", "value": {"id": 1}, "headers": {"source": "test"}, "partition": 0}`. Wszystkie pola są opcjonalne:

- `key` i `value` — ciąg JSON jest wysyłany jako jego tekst, bez cudzysłowów. Każda inna wartość JSON, na przykład obiekt lub liczba, jest wysyłana jako tekst JSON. `null` lub brak pola oznacza brak klucza lub wartości.
- `headers` — obiekt z wartościami tekstowymi.
- `partition` — numer partycji. Bez niego partycja jest wybierana według klucza.

Puste wiersze są pomijane. Plik musi być w UTF-8.

W CSV pierwszy wiersz zawiera nazwy kolumn. Kolumna `value` jest wymagana; kolumny `key`, `partition` i `header.NAME` są opcjonalne, na przykład `header.source`. Pusta komórka `key` oznacza brak klucza, a pusta komórka nagłówka — brak tego nagłówka. Pusta komórka `value` wysyła pustą wartość, a nie `null`, więc z CSV nie można wysłać tombstone.

Plik jest odczytywany strumieniowo, więc łączna liczba nie jest znana z góry i jest wyświetlana jako `—`. Wiersz lub rekord większy niż 16 MB, niepoprawny JSON lub ujemny numer partycji liczy się jako błąd dla danego wiersza, a przebieg jest kontynuowany od następnego wiersza. Błędy są wyświetlane z numerem wiersza.

<a id="batch-rate"></a>

## Szybkość wysyłania

Pole **Szybkość** ogranicza partię do podanej liczby wiadomości na sekundę. Należy wpisać liczbę całkowitą równą 1 lub większą. Puste pole oznacza wartość **bez ograniczeń**.

Wiadomości są rozkładane równomiernie w czasie: przy 10 wiadomościach na sekundę jedna wiadomość wychodzi mniej więcej co 100 ms. Limit pozwala uniknąć przeciążenia współdzielonego klastra lub symulować stałe obciążenie.

Bez limitu aplikacja wysyła tak szybko, jak broker przyjmuje wiadomości. Szybkość dotyczy obu trybów wysyłki wsadowej.

<a id="producer-settings"></a>

## Ustawienia producenta

**Ustawienia producenta** obowiązują przy pojedynczych wysłaniach i w partiach. Są zapisywane w szablonach i historii.

`acks` określa, ile replik musi potwierdzić zapis:

- `all` — domyślnie. Leader czeka na wszystkie zsynchronizowane repliki. To najbezpieczniejszy wybór. Jeśli zsynchronizowanych replik jest mniej, niż wymaga ustawienie topiku `min.insync.replicas`, zapis kończy się błędem.
- `1` — potwierdza tylko leader. Szybciej, ale wiadomość może zostać utracona, jeśli leader ulegnie awarii, zanim followery ją skopiują.
- `0` — producent nie czeka na potwierdzenie. Wiadomości mogą zostać utracone, a offset nie jest zwracany.

Pole **kompresja** kompresuje partie wiadomości: `none` (domyślnie), `gzip`, `snappy`, `lz4` lub `zstd`. Kompresja oszczędza ruch sieciowy i miejsce na dysku kosztem pewnego obciążenia procesora. Często wybierane są `zstd` i `lz4`. Przy pojedynczej małej wiadomości zysk jest niewielki.

Opcja **idempotencja** ustawia `enable.idempotence`. Zapobiega duplikatom, gdy klient ponawia wysłanie po błędzie sieci. Wymaga `acks` ustawionego na `all`; przy innych wartościach aplikacja pokazuje błąd i niczego nie wysyła.
