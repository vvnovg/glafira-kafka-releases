# Klaster i topiki

W tym rozdziale opisano przegląd klastra, listę topików, tworzenie i usuwanie topików, ekran topiku i konfigurację topiku. Pod każdym połączeniem w panelu **Połączenia** znajdują się linki **Przegląd klastra** i **Topiki**. Każdy z nich otwiera się we własnej karcie.

<a id="cluster-overview"></a>

## Przegląd klastra

Ekran **Przegląd klastra** pokazuje stan klastra w chwili wczytania. Przycisk **Odśwież** wczytuje go ponownie.

- `Cluster ID` — identyfikator klastra.
- **Kontroler** — ID brokera, którego klaster zgłasza jako kontroler. W klastrach KRaft klienci nie komunikują się bezpośrednio z kontrolerami: klaster zgłasza tu losowy broker, a nie aktywny kontroler, i zgłaszany broker może się zmieniać między odświeżeniami.
- **Brokery** — liczba brokerów.
- **Topiki (wewnętrzne)** — wszystkie topiki, w nawiasie liczba topików wewnętrznych.
- **Partycje** — liczba partycji we wszystkich topikach.
- `Under-replicated` — partycje, w których nie wszystkie repliki są zsynchronizowane (w ISR). Zwykle oznacza to, że broker nie działa lub nie nadąża.
- `Offline` — partycje bez leadera. Nie można do nich zapisywać ani z nich czytać.

Niezerowa wartość w `Under-replicated` lub `Offline` jest wyróżniona.

Poniżej znajduje się tabela brokerów z kolumnami `ID` i **Adres**. Broker z pola **Kontroler** jest oznaczony etykietą **kontroler**. W klastrach KRaft jest to ten sam losowy broker, a nie aktywny kontroler. Po kliknięciu brokera pod tabelą wyświetla się jego konfiguracja. Konfigurację brokera można tylko przeglądać. Pole **Szukaj parametru** pozwala znaleźć parametr według nazwy, a opcja **Tylko zmienione** ukrywa parametry z wartościami domyślnymi. Ikona kłódki oznacza parametry tylko do odczytu.

<a id="topic-list"></a>

## Lista topików

Ekran **Topiki** zawiera listę topików klastra. Kliknięcie wiersza otwiera topik we własnej karcie.

Kolumny to **Nazwa**, **Partycje**, `RF` (współczynnik replikacji), **Wiadomości (≈)**, `cleanup.policy` i **Niedorepl.** (liczba niedoreplikowanych partycji). Liczba wiadomości jest przybliżona: to różnica między końcowym a początkowym offsetem partycji. Kompaktowanie i znaczniki transakcji sprawiają, że rzeczywista liczba jest mniejsza. Liczby wiadomości i `cleanup.policy` są wczytywane tylko dla wierszy, do których przewinięto listę.

Elementy paska narzędzi:

- **Szukaj (lub /regex/)** — wpisanie części nazwy wyszukuje topiki; wielkość liter nie ma znaczenia. Aby wyszukać wyrażeniem regularnym, należy umieścić je między ukośnikami, na przykład `/^orders\..*/`.
- **Wewnętrzne** — pokazuje topiki wewnętrzne, takie jak `__consumer_offsets`. Są oznaczone etykietą **(wewnętrzny)**.
- Lista autoodświeżania — **Autoodświeżanie: wyłączone**, **co 10 s** lub **co 30 s**.
- **Odśwież** — natychmiast wczytuje ponownie listę i liczby wiadomości.

Obok paska narzędzi aplikacja pokazuje, ile topików pasuje do wyszukiwania w stosunku do łącznej liczby. W połączeniach z włączoną opcją **Tylko do odczytu** przyciski **Utwórz topik** i **Zaznacz** są ukryte.

<a id="create-topic"></a>

## Tworzenie topiku

1. Na liście topików kliknąć przycisk **Utwórz topik**. Nad listą otwiera się formularz **Nowy topik**.
2. W polu **Nazwa topiku** wpisać nazwę. Może zawierać litery łacińskie, cyfry, `.`, `_` i `-`, maksymalnie 249 znaków. Formularz ostrzega, jeśli nazwa zawiera jednocześnie `.` i `_`: nazwy metryk Kafki dla takich topików mogą kolidować z innymi topikami.
3. W polu **Partycje** ustawić liczbę partycji. Partycje są czytane równolegle, a kolejność jest gwarantowana tylko w obrębie jednej partycji. Liczbę można później zwiększyć, ale nigdy zmniejszyć.
4. W polu **Współczynnik replikacji** ustawić, ile kopii każdej partycji przechowują brokery. Wartość nie może być większa niż liczba brokerów, którą formularz pokazuje pod polem. Wartość domyślna to 3 lub liczba brokerów, jeśli jest ich mniej. `-1` oznacza wartość domyślną brokera.
5. W polu **Profil** wybrać profil:
   - **delete (domyślnie)** — stare wiadomości są usuwane według czasu.
   - `compact` — przechowywana jest tylko najnowsza wartość dla każdego klucza.
   - `compact + delete` — oba sposoby.
   - **testowy (retencja 1 h)** — wiadomości są przechowywane przez godzinę.
6. Opcjonalnie dodać **Dodatkowe parametry (key=value)**, po jednym w wierszu, na przykład `retention.ms=86400000`. Zastępują one ustawienia profilu. Wiersze zaczynające się od `#` są ignorowane.
7. Kliknąć przycisk **Utwórz**.

Gdy formularz jest poprawnie wypełniony, pokazuje równoważne polecenie `kafka-topics.sh`. Można je skopiować, na przykład aby utworzyć taki sam topik w innym klastrze.

W połączeniu prod utworzenie trzeba potwierdzić, wpisując nazwę połączenia. Po utworzeniu topiku otwiera się jego karta.

<a id="delete-topics"></a>

## Usuwanie topików

Aby usunąć jeden topik, należy go otworzyć i kliknąć przycisk **Usuń topik**. Następnie trzeba wpisać nazwę topiku w celu potwierdzenia i kliknąć przycisk **Usuń**. Karta topiku się zamyka.

Aby usunąć kilka topików naraz:

1. Na liście topików kliknąć przycisk **Zaznacz**. Obok nazw pojawiają się pola wyboru.
2. Zaznaczyć topiki. Topików wewnętrznych nie można zaznaczyć.
3. Kliknąć przycisk usuwania. Pokazuje on liczbę zaznaczonych topików.
4. Sprawdzić listę nazw, wpisać ich liczbę w celu potwierdzenia i kliknąć przycisk **Usuń**.

Przycisk **Anuluj zaznaczenie** ukrywa pola wyboru.

W połączeniu prod oba sposoby usuwania wymagają także wpisania nazwy połączenia.

Usunięcia nie można cofnąć: dane zostają utracone. Jeśli niektóre topiki nie zostaną usunięte, na przykład z powodu braku uprawnień, aplikacja wyświetla ich listę z przyczyną. Pozostałe topiki są usuwane.

<a id="topic-details"></a>

## Ekran topiku

Karta topiku pokazuje liczbę partycji, współczynnik replikacji i przybliżoną liczbę wiadomości. Przyciski u góry:

- **Odśwież** — ponownie wczytuje dane topiku.
- **Wyślij wiadomość** — otwiera ekran **Producer** z tym topikiem. Zob. [Producer](producer.md).
- **Czytaj jako grupa** — uruchamia sesję konsumenta dla tego topiku. Zob. [Sesje konsumenta](consumer-sessions.md).

W połączeniach z włączoną opcją **Tylko do odczytu** oba przyciski są nieaktywne.

Drugi wiersz zawiera działania zmieniające topik. Jest ukryty w połączeniach tylko do odczytu i dla topików wewnętrznych.

- **Dodaj partycje** — należy wpisać nową łączną liczbę i kliknąć przycisk **Zwiększ**. Liczby partycji nie można zmniejszyć. Po zwiększeniu wiadomości z tym samym kluczem mogą trafiać do innej partycji, więc kolejność według klucza zostaje zaburzona.
- **Wyczyść** — usuwa wszystkie wiadomości do bieżącego końca wybranych partycji. Sam topik pozostaje. Należy zaznaczyć partycje, wpisać nazwę topiku i kliknąć przycisk **Wyczyść partycje**. Czyszczenie działa tylko wtedy, gdy `cleanup.policy` zawiera `delete`; w przeciwnym razie przycisk jest nieaktywny, a obok wyświetlana jest przyczyna. W topiku `compact,delete` czyszczenie usuwa też najnowsze wartości kluczy.
- **Usuń topik** — zob. [Usuwanie topików](topics.md#delete-topics).

W połączeniu prod każde z tych działań wymaga także wpisania nazwy połączenia.

Karty poniżej:

- **Wiadomości** — przeglądanie wiadomości. Zob. [Przeglądanie wiadomości](browse.md).
- **Partycje** — tabela z każdą partycją: **Leader**, **Repliki**, `ISR` (repliki zsynchronizowane), początkowy (`Low`) i końcowy (`High`) offset oraz liczba wiadomości. Partycja bez leadera pokazuje wartość **brak**.
- **Konfiguracja** — parametry topiku. Zob. [Konfiguracja topiku](topics.md#topic-config).
- **Grupy konsumentów** — grupy konsumentów, które mają zatwierdzone offsety dla tego topiku, z kolumnami **Stan**, **Partycje z zatwierdzonymi offsetami** i **Lag**. Kliknięcie grupy ją otwiera. Zob. [Grupy konsumentów](groups.md).

<a id="topic-config"></a>

## Konfiguracja topiku

Karta **Konfiguracja** zawiera wszystkie parametry topiku w trzech kolumnach:

- **Parametr** — nazwa parametru. Nazwy pogrubione są ustawione jawnie: w topiku lub w konfiguracji brokera.
- **Wartość** — bieżąca wartość. Wartości poufne są ukryte jako `••••••`.
- **Źródło** — skąd pochodzi wartość. Na przykład `DynamicTopic` oznacza wartość ustawioną w tym topiku, `StaticBroker` i `DynamicBroker` pochodzą z konfiguracji brokera, a `Default` to wartość domyślna Kafki.

Pole **Szukaj parametru** pozwala znaleźć parametr według nazwy, a opcja **Tylko zmienione** pokazuje tylko parametry pogrubione. Ikona kłódki oznacza parametry tylko do odczytu. Ikona `?` obok nazwy objaśnia parametr.

<a id="edit-config"></a>

## Zmiana konfiguracji

1. Na karcie **Konfiguracja** kliknąć przycisk **Edytuj**. W połączeniach z włączoną opcją **Tylko do odczytu** ten przycisk jest ukryty.
2. Zmienić wartości w kolumnie **Wartość**. Parametrów tylko do odczytu i poufnych nie można edytować. Aby usunąć własną wartość topiku, należy kliknąć **przywróć wartość domyślną**: obowiązuje wtedy wartość brokera. Pusta wartość jest niedozwolona.
3. Kliknąć przycisk **Przejrzyj zmiany**. Aplikacja pokazuje każdą zmianę jako stara wartość → nowa wartość; **domyślnie** oznacza przywrócenie wartości domyślnej.
4. Kliknąć przycisk **Zastosuj** albo przycisk **Wstecz**, aby kontynuować edycję.

Do momentu zastosowania nic nie zmienia się w klastrze. Przycisk **Odrzuć zmiany** anuluje wszystkie zmiany. Dopóki są niezapisane zmiany, przycisk **Zamknij edytor** jest nieaktywny. W połączeniu prod zmiany trzeba potwierdzić, wpisując nazwę połączenia.

Wysyłane są tylko zmienione parametry. Pozostałe parametry topiku pozostają bez zmian. Broker sprawdza wartości i odrzuca nieprawidłowe z błędem.

<a id="config-parameters"></a>

## Parametry topiku

Topiki Kafki mają dziesiątki parametrów. Ikony `?` w tabeli konfiguracji objaśniają każdy z nich. Najczęściej potrzebne są te:

- `cleanup.policy` — co dzieje się ze starymi danymi. `delete` usuwa stare segmenty według czasu lub rozmiaru. `compact` przechowuje tylko najnowszą wartość dla każdego klucza. `compact,delete` robi jedno i drugie.
- `retention.ms` — jak długo wiadomości są przechowywane, w milisekundach. Wartość domyślna to `604800000` (7 dni); `-1` przechowuje je bezterminowo. Działa, gdy `cleanup.policy` zawiera `delete`.
- `retention.bytes` — maksymalny rozmiar każdej partycji, po przekroczeniu którego usuwane są stare segmenty. Domyślne `-1` oznacza brak limitu rozmiaru.
- `segment.ms` i `segment.bytes` — kiedy broker rozpoczyna nowy segment logu. Retencja i kompaktowanie działają na całych segmentach, więc wiadomość może pozostać dłużej niż `retention.ms`, dopóki jej segment nie zostanie zamknięty.
- `min.insync.replicas` — ile replik musi potwierdzić zapis przy `acks=all`. Zwykle 2 przy współczynniku replikacji 3.
- `max.message.bytes` — największy batch, jaki broker przyjmuje dla tego topiku. Wartość domyślna to około 1 MB. Producenci muszą dopuszczać taki sam rozmiar (`max.request.size`).
- `compression.type` — jak broker przechowuje wiadomości. `producer` zachowuje kompresję producenta.
- `message.timestamp.type` — `CreateTime` (czas ustawiony przez producenta) lub `LogAppendTime` (czas zapisania wiadomości przez broker). Ten czas jest wyświetlany podczas przeglądania wiadomości.
- `delete.retention.ms` — jak długo znaczniki usunięcia (wiadomości z wartością null) pozostają w topiku kompaktowanym. Wartość domyślna to 1 dzień.

Niektóre parametry istnieją tylko w nowszych wersjach Kafki, a niektóre są przestarzałe. Ikona `?` informuje o tym tam, gdzie ma to znaczenie. Parametry tiered storage, takie jak `remote.storage.enable`, działają tylko wtedy, gdy klaster je obsługuje.
