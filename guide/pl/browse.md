# Przeglądanie wiadomości

Wiadomości przegląda się na karcie **Wiadomości** topiku. Przeglądanie nie dołącza do grupy konsumentów i niczego nie zatwierdza, więc klaster się nie zmienia. Działa też w połączeniach z włączoną opcją **Tylko do odczytu**. Aby czytać jako członek grupy, należy użyć [sesji konsumenta](consumer-sessions.md).

<a id="start-reading"></a>

## Rozpoczęcie odczytu

1. Otworzyć topik z listy **Topiki**. Jako pierwsza otwiera się karta **Wiadomości**.
2. W polu **Od** wybrać, od którego miejsca zacząć. Zob. [Pozycja początkowa](browse.md#start-position).
3. Opcjonalnie ustawić formaty i filtry.
4. Kliknąć przycisk **Czytaj**.

Wiadomości pojawiają się w tabeli w miarę napływania. Wiersz obok przycisków pokazuje, ile wiadomości przeskanowano, ile pasuje do filtrów i ile jest w tabeli. Pokazuje też stan: **odczytywanie…**, **osiągnięto limit**, **osiągnięto koniec topiku** lub **zatrzymano**.

Bez `Live tail` odczyt kończy się na końcu topiku w stanie z chwili kliknięcia przycisku **Czytaj** albo wcześniej, po osiągnięciu wartości z pola **Limit**. Limit liczy wiadomości pasujące do filtrów. Może wynosić od 1 do 100000; duże wartości spowalniają odczyt.

Z włączonym `Live tail` odczyt nie kończy się na końcu topiku: nowe wiadomości są dodawane w miarę ich zapisywania. Gdy wszystkie stare wiadomości zostaną odczytane, stan pokazuje **oczekiwanie na nowe wiadomości…**. Tabela przechowuje ostatnie wiadomości do wysokości limitu i odrzuca najstarsze.

Przycisk **Zatrzymaj** zatrzymuje odczyt i zachowuje tabelę. Podczas odczytu przycisk **Czytaj** zmienia się w przycisk **Uruchom ponownie**: uruchamia odczyt od nowa z bieżącymi ustawieniami i czyści tabelę. Jeśli połączenie z brokerem zostanie przerwane, aplikacja sama łączy się ponownie i informuje o tym w wierszu stanu.

Wiadomości z różnych partycji są w tabeli przemieszane. Kafka zachowuje kolejność tylko w obrębie jednej partycji. Kolumna `P` pokazuje partycję, a `Offset` — pozycję w niej. Zaznaczenie `UTC` pokazuje czas w UTC zamiast czasu lokalnego.

<a id="start-position"></a>

## Pozycja początkowa

Pole **Pozycja początkowa** określa, gdzie w każdej partycji zaczyna się odczyt:

- **Ostatnie N** — ostatnie N wiadomości. N jest dzielone równo między partycje, więc przy 100 wiadomościach i 4 partycjach aplikacja pobiera do 25 z każdej.
- **Od początku** — od najstarszej wiadomości, która jest jeszcze przechowywana.
- **Od offsetu** — od tego samego offsetu w każdej partycji. Jeśli partycja nie ma takiego offsetu, odczyt zaczyna się od jej początku lub końca. Ten tryb jest najbardziej przydatny przy jednej partycji.
- **Od daty i godziny** — od pierwszej wiadomości ze znacznikiem czasu równym podanemu czasowi lokalnemu lub późniejszym. W partycjach bez nowszych wiadomości odczyt zaczyna się od końca.
- **Ostatnie X minut** — to samo, licząc wstecz od teraz.

Pole z tekstem **partycje: wszystkie** ogranicza odczyt do wybranych partycji. Należy wpisać numery partycji oddzielone przecinkami, na przykład `0,2`. Puste pole oznacza odczyt wszystkich partycji.

<a id="filters"></a>

## Filtry

Filtry pokazują tylko pasujące wiadomości. Aplikacja i tak czyta wszystkie wiadomości od pozycji początkowej i sprawdza każdą z nich, więc filtry nie przyspieszają odczytu. Muszą być spełnione wszystkie wypełnione pola.

- **Klucz zawiera** i **Wartość zawiera** — tekst w kluczu lub wartości.
- `regex` — traktuje te pola oraz wartość nagłówka jako wyrażenia regularne.
- `Aa` — uwzględnia wielkość liter. Domyślnie wielkość liter nie ma znaczenia.
- **Nagłówek** — wiadomość ma nagłówek o dokładnie takiej nazwie. Pole **Wartość nagłówka** sprawdza też jego wartość.
- `JSONPath` — warunek dotyczący wartości JSON, na przykład `$.order.status == "FAILED"`. Bez `==` lub `!=` ścieżka musi po prostu istnieć, na przykład `$.error`. Wiadomości, których wartość nie jest poprawnym dokumentem JSON, nie pasują.
- **Czas od** i **Czas do** — znacznik czasu wiadomości mieści się w tym zakresie, w czasie lokalnym. Oba końce są uwzględniane.

Filtry sprawdzają tekst wyświetlany w tabeli, po zdekodowaniu. Jeśli format wartości to `hex`, filtr wartości przeszukuje tekst szesnastkowy. Filtry są stosowane po kliknięciu przycisku **Czytaj**; po ich zmianie trzeba odczytać ponownie.

Filtr nie przesuwa pozycji początkowej. Na przykład **Ostatnie N** z filtrem znajduje dopasowania tylko wśród ostatnich N wiadomości. Aby przeszukać cały topik, należy zacząć od pozycji **Od początku**.

<a id="formats"></a>

## Formaty

Pole **Format** określa, jak dekodowane są klucze i wartości, osobno dla pozycji **klucz** i **wartość**. Zmienia się tylko sposób wyświetlania, a nie dane.

- `auto` — domyślnie. Poprawne obiekty i tablice JSON są wyświetlane jako JSON, inny tekst UTF-8 jako ciąg znaków, dane binarne jako hex.
- `string` — tekst w UTF-8. Bajty, które nie są poprawnym UTF-8, są zastępowane znakiem `�`.
- `json` — JSON z wcięciami w szczegółach wiadomości. Jeśli wartość nie jest poprawnym dokumentem JSON, szczegóły pokazują błąd.
- `hex` — każdy bajt jako dwie cyfry szesnastkowe.
- `base64` — bajty w Base64.

Szczegóły pokazują, jaki format został faktycznie użyty, na przykład `json`, gdy `auto` wykrył JSON. Wartości nagłówków są wyświetlane jako tekst; binarne wartości nagłówków — jako `0x` i ciąg szesnastkowy. Brakujący klucz lub brakująca wartość są wyświetlane jako `null`.

Bardzo duże klucze i wartości są wyświetlane tylko częściowo, mniej więcej pierwszy 1 MB tekstu. Dotyczy to także `hex` i `base64`. Szczegóły wiadomości pokazują wtedy ostrzeżenie.

<a id="message-details"></a>

## Szczegóły wiadomości

Kliknięcie wiersza otwiera szczegóły wiadomości obok tabeli. U góry są partycja, offset i czas. Poniżej:

- **Wartość** i **Klucz** — pełny tekst z formatem i rozmiarem w bajtach. JSON jest wyświetlany z wcięciami. Przycisk **Kopiuj** kopiuje tekst.
- **Nagłówki** — tabela nazw i wartości nagłówków.

Kolumny tabeli to `P`, `Offset`, **Czas**, **Klucz**, **Wartość**, **Nagłówki** (ich liczba) i **Rozmiar** (rozmiar klucza i wartości w bajtach). Czas to znacznik czasu wiadomości. W zależności od ustawienia topiku `message.timestamp.type` ustawia go producent lub broker.

<a id="export-messages"></a>

## Kopiowanie wiadomości

Glafira nie zapisuje przeglądanych wiadomości do pliku. Aby wynieść wiadomość poza aplikację, należy ją skopiować:

- **Kopiuj wszystko jako JSON** kopiuje całą wiadomość jako JSON: topik, partycję, offset, znacznik czasu, klucz, wartość, nagłówki i rozmiar. Klucz i wartość są dołączane w postaci zdekodowanej, z formatem.
- Przycisk **Kopiuj** obok pozycji **Wartość** lub **Klucz** kopiuje tylko ten tekst.

Skopiowany tekst ma format wybrany do odczytu. Aby dokładnie skopiować dane binarne, należy odczytać je jako `hex` lub `base64`. Bardzo duże klucze i wartości są kopiowane tylko częściowo, tak jak są wyświetlane, nawet w `hex` i `base64`. Zob. [Formaty](browse.md#formats).

<a id="open-in-producer"></a>

## Otwieranie na ekranie Producer

Przycisk **Wyślij ponownie** otwiera wiadomość na ekranie **Producer** tego samego połączenia. Formularz otrzymuje topik, klucz, wartość, nagłówki i formaty wiadomości. Wybór partycji pozostaje po stronie Kafki. Symbole zastępcze w tekście wiadomości nie są podstawiane: kopia jest wysyłana w takiej postaci, w jakiej została odczytana.

Nic nie jest wysyłane od razu. Należy sprawdzić formularz, w razie potrzeby go zmienić i wysłać z ekranu Producer. Zob. [Producer](producer.md).

Przycisk jest nieaktywny w dwóch przypadkach, a obok wyświetlana jest przyczyna:

- klucz lub wartość zostały wyświetlone tylko częściowo, ponieważ są zbyt duże;
- tekst został zdekodowany ze stratami, ponieważ nie jest poprawnym UTF-8. Aby wysłać dokładną kopię, należy odczytać go jako `hex` lub `base64`.

W połączeniach z włączoną opcją **Tylko do odczytu** przycisk jest ukryty.

<a id="reset-group-here"></a>

## Reset grupy do wiadomości

Polecenie **Zresetuj grupę do tego miejsca…** przygotowuje reset offsetów tak, aby grupa konsumentów zaczęła czytać od tej wiadomości.

1. W szczegółach wiadomości kliknąć **Zresetuj grupę do tego miejsca…**.
2. Wybrać grupę z listy. Wyświetlane są tylko istniejące grupy.
3. Kliknąć przycisk **Przejdź do resetu**.

Otwiera się karta grupy z wypełnionym formularzem resetu. Pole **Strategia** ma wartość **Konkretny offset** z offsetem wiadomości. Pole **Zakres** ma wartość **Partycje** z topikiem i partycją wiadomości. Na razie nic się nie zmienia: należy kliknąć przycisk **Podgląd**, aby zobaczyć nowe offsety, a następnie zastosować reset w tym miejscu. Grupa nie może mieć aktywnych członków. Zob. [Grupy konsumentów](groups.md).

W połączeniach z włączoną opcją **Tylko do odczytu** przycisk jest ukryty.
