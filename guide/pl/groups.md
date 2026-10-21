# Grupy konsumentów

Grupa konsumentów to zbiór konsumentów, którzy wspólnie czytają topiki. Kafka przydziela każdą partycję jednemu członkowi grupy i przechowuje dla każdej partycji offset zatwierdzony przez grupę. Po ponownym uruchomieniu aplikacje kontynuują odczyt od tego offsetu.

Ekran **Grupy konsumentów** otwiera się z linku pod połączeniem w panelu **Połączenia**. Grupy jednego topiku są też dostępne na karcie **Grupy konsumentów** tego topiku. Zob. [Ekran topiku](topics.md#topic-details).

<a id="group-list"></a>

## Lista grup

Lista pokazuje wszystkie grupy konsumentów klastra. Kolumny:

- **Grupa** — ID grupy (`group.id`).
- **Stan** — stan zgłaszany przez Kafkę, na przykład `Stable` (członkowie czytają), `Empty` (brak członków), `PreparingRebalance` lub `CompletingRebalance` (partycje są rozdzielane na nowo) i `Dead` (grupa już nie istnieje).
- **Członkowie** — liczba aktywnych członków.
- `Assignor` — strategia rozdzielania partycji, na przykład `range`.
- **Lag** — łączny lag grupy. Zob. [Lag](groups.md#lag).
- **Topiki** — topiki, dla których grupa ma zatwierdzone offsety.

Tekst wpisany w polu **Szukaj grup** pokazuje grupy, których ID go zawiera; wielkość liter nie ma znaczenia. Lista stanów obok pokazuje tylko grupy w jednym stanie; wartość **Wszystkie stany** usuwa ten filtr. Licznik pokazuje, ile grup jest wyświetlanych, a ile jest łącznie.

Członkowie, assignor i lag są wczytywane tylko dla widocznych wierszy, w miarę przewijania. Do czasu ich wczytania komórka pokazuje `…`. Jeśli nie można ich wczytać, komórka pokazuje `—`; po najechaniu na nią wskaźnikiem widać przyczynę.

Przycisk **Odśwież** wczytuje ponownie listę i jej dane. Jeśli niektóre brokery nie odpowiedziały, ostrzeżenie informuje, że lista może być niekompletna.

<a id="lag"></a>

## Lag

Lag to liczba wiadomości, których grupa jeszcze nie przeczytała. Dla każdej partycji jest to koniec logu minus zatwierdzony offset. Lag nigdy nie jest pokazywany poniżej zera.

Lag zależy wyłącznie od zatwierdzonych offsetów. Aktywny konsument mógł przeczytać więcej, niż zatwierdził, więc jego rzeczywisty lag może być mniejszy. Gdy grupa ponownie zatwierdzi offsety, lag spadnie.

Partycja bez zatwierdzonego offsetu pokazuje **brak zatwierdzenia** i nie jest wliczana do sumy. Łączny lag grupy pokazuje `—`, gdy grupa nie ma żadnych zatwierdzonych offsetów (**brak zatwierdzeń**) lub gdy nie udało się odczytać offsetów którejś partycji (**lag nieznany: błąd offsetów partycji**).

Lag liczy offsety, a nie wiadomości. W topikach z kompaktowaniem lub transakcjami niektóre offsety nie zawierają widocznych wiadomości, więc grupie może zostać do przeczytania mniej wiadomości, niż pokazuje lag.

Wartości są migawką z chwili wczytania. Aby śledzić lag, należy otworzyć grupę i włączyć autoodświeżanie.

<a id="group-details"></a>

## Szczegóły grupy

Kliknięcie grupy na liście otwiera jej kartę. U góry podsumowanie pokazuje stan, liczbę członków i assignor.

- **Autoodświeżanie** — **Autoodświeżanie: wyłączone**, **co 5 s** lub **co 10 s**.
- **Odśwież** — natychmiastowe ponowne wczytanie.
- **Czytaj jako członek** — uruchamia [sesję konsumenta](consumer-sessions.md) w tej grupie. Formularz sesji otrzymuje ID grupy i topiki, dla których grupa ma zatwierdzone offsety. W połączeniach z włączoną opcją **Tylko do odczytu** przycisk jest nieaktywny; po najechaniu na przycisk widać przyczynę.

Tabela **Partycje** zawiera partycje, dla których grupa ma zatwierdzone offsety, oraz partycje przydzielone jej członkom. Łączny lag grupy jest wyświetlany nad tabelą. Kolumny to **Topik**, **Partycja**, `Committed` (zatwierdzony offset), `Log end` (końcowy offset partycji), **Lag** i **Członek** (client ID członka, który czyta partycję).

Tabela **Członkowie** zawiera aktywnych członków: `Client ID`, **Host**, `Member ID` i **Przypisanie** (partycje członka w postaci `topic:partition`). Grupa bez członków pokazuje komunikat **Brak aktywnych członków.**

Drugi wiersz zawiera przyciski **Zresetuj offsety** i **Usuń grupę**. Jest ukryty w połączeniach tylko do odczytu.

<a id="reset-offsets"></a>

## Reset offsetów

Reset offsetów przesuwa zatwierdzone offsety grupy. Grupa ponownie czyta wtedy wiadomości lub je pomija. Offsety można zresetować tylko dla grupy bez aktywnych członków (stan `Empty` lub `Dead`). W przeciwnym razie członkowie nadpisaliby nowe offsety własnymi zatwierdzeniami. Najpierw trzeba zatrzymać aplikacje grupy.

1. Na karcie grupy kliknąć przycisk **Zresetuj offsety**. Dopóki grupa ma członków, przycisk jest nieaktywny; po najechaniu na przycisk widać przyczynę.
2. W polu **Strategia** wybrać strategię. Zob. [Strategie resetu](groups.md#reset-strategies).
3. W polu **Zakres** wybrać zakres. Zob. [Zakres resetu](groups.md#reset-scope).
4. Kliknąć przycisk **Podgląd**.
5. Sprawdzić tabelę i kliknąć przycisk zastosowania. Pokazuje on liczbę partycji, które się zmienią.

Podgląd niczego nie zmienia. Dla każdej partycji pokazuje topik, partycję, **Bieżący** (zatwierdzony offset), **Nowy** (docelowy offset) oraz `Low / High` (pierwszy i końcowy offset partycji). Partycja, dla której nie da się obliczyć celu, jest oznaczona komunikatem **Brak docelowego offsetu — nie zostanie zastosowany** z przyczyną. Takie partycje są pomijane.

Stosowany jest tylko plan pokazany w podglądzie. Każda zmiana w formularzu ukrywa podgląd i trzeba ponownie kliknąć przycisk **Podgląd**. W połączeniu prod zastosowanie wymaga wpisania nazwy połączenia; w innych połączeniach następuje od razu.

Przed zastosowaniem aplikacja ponownie sprawdza, czy grupa nie ma członków. Po zastosowaniu sekcja **Wynik resetu** zawiera każdą partycję z nowym offsetem lub błędem.

Reset można też przygotować z poziomu wiadomości: zob. [Reset grupy do wiadomości](browse.md#reset-group-here). Przycisk **Zamknij** zamyka panel resetu.

<a id="reset-strategies"></a>

## Strategie resetu

Pole **Strategia** określa docelowy offset dla każdej partycji w zakresie:

- **Od początku (earliest)** — pierwszy offset, który jest jeszcze przechowywany. Grupa ponownie czyta wszystko, co topik przechowuje.
- **Od końca (latest)** — koniec partycji. Grupa pomija wszystko, co się nagromadziło, i czyta tylko nowe wiadomości.
- **Od daty i godziny** — pierwsza wiadomość zapisana w chwili podanej w polu **Data i godzina (czas lokalny)** lub później. W partycjach bez nowszych wiadomości celem jest koniec.
- **Konkretny offset** — wartość z pola **Offset**, liczba całkowita równa 0 lub większa, w każdej partycji. Jeśli partycja nie ma tego offsetu, celem jest jej pierwszy lub końcowy offset.
- **Przesunięcie o ±N** — przesuwa zatwierdzony offset o wartość z pola **Przesunięcie (±N)**. Liczba ujemna oznacza ponowny odczyt N wiadomości wstecz, dodatnia — pominięcie N wiadomości. Wynik nie wykracza poza pierwszy i końcowy offset. Partycje bez zatwierdzonego offsetu nie mogą zostać przesunięte i są pomijane.

Czas jest czasem lokalnym, z sekundami. Kafka wyszukuje offset według znaczników czasu wiadomości.

<a id="reset-scope"></a>

## Zakres resetu

Pole **Zakres** określa, które partycje są resetowane. Pozostałe partycje zachowują swoje offsety.

- **Cała grupa** — wszystkie partycje, dla których grupa ma zatwierdzone offsety. Jeśli grupa nie ma jeszcze zatwierdzonych offsetów, należy zamiast tego wybrać topiki.
- **Topiki** — wszystkie partycje podanych topików, także partycje bez zatwierdzonych offsetów. Nazwy oddziela się przecinkami, spacjami lub podziałami wierszy. W ten sposób można sprawić, by grupa zaczęła czytać nowy topik od wybranego miejsca.
- **Partycje** — tylko podane partycje. Należy wpisać je w polu **Partycje (topik:partycja)** w postaci `topic:partition`, oddzielone przecinkami lub spacjami, na przykład `orders:0, orders:3`.

<a id="delete-group"></a>

## Usuwanie grupy

Usunięcie grupy usuwa wszystkie jej zatwierdzone offsety. Usunąć można tylko grupę bez członków, w stanie `Empty`.

1. Na karcie grupy kliknąć przycisk **Usuń grupę**. Dopóki grupa ma członków, przycisk jest nieaktywny; po najechaniu na przycisk widać przyczynę.
2. Wpisać ID grupy w celu potwierdzenia. W połączeniu prod wpisać też nazwę połączenia.
3. Kliknąć przycisk **Usuń**.

Po usunięciu karta grupy się zamyka. Jeśli aplikacja z tym `group.id` zostanie ponownie uruchomiona, Kafka utworzy grupę od nowa. Bez zatwierdzonych offsetów aplikacja zaczyna od miejsca określonego ustawieniem `auto.offset.reset`: od początku lub od końca.

W połączeniach z włączoną opcją **Tylko do odczytu** przycisk jest ukryty.
