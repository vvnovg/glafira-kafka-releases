# Sesje konsumenta

Sesja konsumenta czyta topiki jako rzeczywisty członek grupy konsumentów.

<a id="what-is"></a>

## Czym jest sesja

[Przeglądanie](browse.md) czyta wiadomości bez dołączania do grupy i niczego nie zmienia w klastrze. Sesja działa inaczej: dołącza do grupy o podanym `group.id`, otrzymuje partycje od Kafki i może zatwierdzać offsety. Służy do czytania tego, co czytałaby grupa, do przesuwania offsetów grupy przez zatwierdzanie lub do obserwowania rebalansu grupy.

Sesja wpływa na klaster. Gdy dołącza lub odchodzi, Kafka rozdziela partycje grupy na nowo, a pozostali członkowie przechodzą rebalans. Zatwierdzenia zmieniają offsety grupy, z których będą korzystać jej aplikacje.

Sesje nie są dostępne w połączeniach z włączoną opcją **Tylko do odczytu**: karta pokazuje komunikat **Sesje konsumenta są niedostępne dla połączeń tylko do odczytu: dołączenie do grupy zmienia jej stan.** Jeśli połączenie stanie się tylko do odczytu w trakcie działania sesji, sesja opuszcza grupę.

<a id="start-session"></a>

## Uruchamianie sesji

Sesję można otworzyć na trzy sposoby:

- link **Sesja konsumenta** pod połączeniem w panelu **Połączenia** — pusty formularz;
- przycisk **Czytaj jako grupa** na karcie topiku — topik jest już wpisany;
- przycisk **Czytaj jako członek** na karcie grupy — grupa i topiki, dla których ma zatwierdzone offsety, są już wpisane.

Za każdym razem otwiera się nowa karta, więc kilka sesji może działać jednocześnie.

W sekcji **Grupa i topiki**:

- `group.id` — grupa, do której należy dołączyć. Jeśli pole pozostanie puste, aplikacja zaproponuje nową nazwę, na przykład `glafira-kafka-anna-1a2b3c`. Nowa grupa nie ma zatwierdzonych offsetów.
- **Topiki** — nazwy topików oddzielone przecinkami lub spacjami. Można też zaznaczyć opcję **topiki według regex** i wpisać wyrażenie w polu **Wyrażenie regularne topików**, na przykład `orders\..*`. Wyrażenie jest dopasowywane od początku nazwy: jeśli brakuje `^`, jest dodawany. Pasujące topiki utworzone później są dodawane w ciągu kilku minut (przy odświeżeniu metadanych); powoduje to rebalans.
- `auto.offset.reset` — gdzie zacząć w partycjach, dla których grupa nie ma zatwierdzonego offsetu: **od końca (latest)** (domyślnie) lub **od początku (earliest)**. Tam, gdzie grupa ma zatwierdzony offset, odczyt jest kontynuowany od niego.
- **autozatwierdzanie** — automatyczne zatwierdzanie odczytanych offsetów. Domyślnie wyłączone. Zob. [Zatwierdzanie offsetów](consumer-sessions.md#commits).
- `isolation.level` — `read_committed` (domyślnie) pokazuje tylko wiadomości z zatwierdzonych transakcji. `read_uncommitted` pokazuje też wiadomości z otwartych i przerwanych transakcji.

W sekcji **Format** należy wybrać wartości w polach **Format klucza** i **Format wartości**, tak jak przy przeglądaniu. Zob. [Formaty](browse.md#formats). Pole **Bufor, wiadomości** określa, ile wiadomości przechowuje tabela, od 1 do 100000; domyślnie 5000.

Filtry działają tak jak przy przeglądaniu. Zob. [Filtry](browse.md#filters).

Następnie należy kliknąć przycisk **Dołącz do grupy**. W połączeniu prod aplikacja prosi o wpisanie nazwy połączenia i potwierdzenie przyciskiem **Potwierdź**. Podczas działania sesji formularza nie można zmieniać.

<a id="reading"></a>

## Odczyt

Wiersz stanu obok przycisków pokazuje **łączenie…**, a potem **w grupie**. Pokazuje też, ile wiadomości odczytano, ile pasuje do filtrów i ile jest w tabeli. Jeśli połączenie z brokerem zostanie przerwane, sesja sama łączy się ponownie i informuje o tym w wierszu stanu.

Wiadomości pojawiają się w tabeli w miarę napływania, a sesja czyta nowe wiadomości, dopóki nie zostanie zatrzymana. Gdy tabela się zapełni, najstarsze wiadomości są usuwane. Kliknięcie wiersza pokazuje szczegóły wiadomości, tak jak przy przeglądaniu. Zob. [Szczegóły wiadomości](browse.md#message-details). Zaznaczenie `UTC` pokazuje czas w UTC.

Sesja czyta tylko partycje, które przydzieliła jej Kafka. Jeśli grupa ma innych członków, czytają oni pozostałe partycje. Jeśli grupa ma więcej członków niż partycji, sesja może nie otrzymać żadnej partycji.

Przycisk **Rozłącz** opuszcza grupę, a jej partycje otrzymują pozostali członkowie. Zamknięcie karty sesji również opuszcza grupę. Po zakończeniu sesji wiersz stanu pokazuje **rozłączono**.

Jeśli topik nie istnieje, sesja zatrzymuje się z błędem i pokazuje nazwy do sprawdzenia.

<a id="commits"></a>

## Zatwierdzanie offsetów

Zatwierdzenie (commit) zapisuje offset dla grupy. Następny członek, który czyta partycję, lub ta sama aplikacja po ponownym uruchomieniu zaczyna od tego offsetu.

Z włączoną opcją **autozatwierdzanie** sesja zatwierdza odczytane offsety w tle co kilka sekund, a także jeszcze raz przy opuszczaniu grupy. Wiadomości ukryte przez filtry liczą się jako odczytane.

Bez autozatwierdzania sesja sama niczego nie zatwierdza. Służą do tego przyciski:

- **Zatwierdź do zaznaczonej** — zatwierdza offset następujący po zaznaczonej wiadomości, tylko w jej partycji. Grupa będzie kontynuować od następnej wiadomości. Jeśli zaznaczona wiadomość jest starsza niż bieżący zatwierdzony offset, offset przesuwa się wstecz.
- **Zatwierdź wszystkie odczytane** — zatwierdza bieżącą pozycję w każdej przydzielonej partycji. Obejmuje to wiadomości ukryte przez filtry oraz wiadomości już usunięte z tabeli.

Po zatwierdzeniu wiersz pokazuje zatwierdzone offsety w postaci `topic:partition → offset`. W połączeniu prod każde zatwierdzenie wymaga wpisania nazwy połączenia.

Jeśli sesja nie odpowie na czas, aplikacja ostrzega, że nie wiadomo, czy zatwierdzenie zostało zastosowane. Należy wtedy sprawdzić offsety na karcie grupy. Zob. [Szczegóły grupy](groups.md#group-details).

<a id="pause-partitions"></a>

## Wstrzymywanie partycji

Panel na prawo od tabeli zawiera partycje przydzielone sesji w postaci `topic:partition`. Przycisk **Wstrzymaj** obok partycji wstrzymuje jej odczyt, a przycisk **Wznów** go wznawia. Wstrzymana partycja jest oznaczona jako **wstrzymana**.

Wstrzymanie nie opuszcza grupy, więc nie ma rebalansu. Partycja pozostaje przydzielona sesji, a inni członkowie nie czytają jej, dopóki jest wstrzymana. Wstrzymanie niczego nie zmienia w klastrze i nie wymaga potwierdzenia.

Po rebalansie nowo przydzielone partycje nie są wstrzymane. Gdy sesja opuszcza grupę, wszystkie wstrzymania się kończą.

<a id="rebalance-log"></a>

## Dziennik rebalansów

**Dziennik rebalansów** pod listą partycji rejestruje każdą zmianę przydziału, od najnowszych. Każdy wpis ma czas i jeden z trzech rodzajów: partycje przydzielone sesji, partycje jej odebrane lub błąd rebalansu z jego treścią.

Rebalans następuje, gdy członek dołącza do grupy lub ją opuszcza, gdy zmieniają się subskrybowane topiki lub gdy członek przestaje odpowiadać na czas. Aby go zaobserwować, należy uruchomić drugą sesję z tym samym `group.id` w innej karcie: partycje zostaną podzielone między dwie sesje i oba dzienniki to pokażą.

Dziennik przechowuje ostatnie 200 wpisów. Przed pierwszym rebalansem pokazuje komunikat **Nie było jeszcze rebalansów.**

<a id="limits"></a>

## Ograniczenia

- Sesje nie są dostępne w połączeniach z włączoną opcją **Tylko do odczytu**.
- Sesji nie można uruchomić od wybranego offsetu. Aby ustawić początek, należy najpierw zresetować offsety grupy: zob. [Reset offsetów](groups.md#reset-offsets). Reset wymaga grupy bez członków, więc przed resetem trzeba rozłączyć sesje tej grupy.
- Tabela przechowuje maksymalnie 100000 wiadomości. Starsze wiadomości są usuwane, ale nadal liczą się jako odczytane.
- Bardzo duże klucze i wartości są wyświetlane tylko częściowo, tak jak przy przeglądaniu.
- Nie należy dołączać do grupy działającej aplikacji bez wyraźnej potrzeby. Sesja zabiera partycje aplikacji, a aplikacja ich nie przetwarza, dopóki trzyma je sesja. Zatwierdzenia z sesji, w tym automatyczne, przesuwają offsety aplikacji.
