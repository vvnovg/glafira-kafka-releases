# Asystenci LLM (MCP)

Glafira może pełnić rolę bramki dla asystenta LLM: Claude Desktop, Claude Code, Cursor lub innego klienta obsługującego MCP (Model Context Protocol). Asystent odczytuje metadane klastrów, które zostały dla niego otwarte, i pomaga znaleźć problemy: niedoreplikowane partycje, lag, ciągłe rebalansy, niefortunne ustawienia. Przez Glafira nie może niczego zmienić w klastrze: zmiany wprowadza się samodzielnie w interfejsie.

<a id="how-it-works"></a>

## Jak to działa

Klient uruchamia `glafira-kafka mcp` — ten sam plik aplikacji w trybie bez okna. To polecenie łączy się z działającą aplikacją Glafira przez lokalne gniazdo (w systemie Windows — potok nazwany), a Glafira odpowiada na żądania asystenta przez własne połączenia. Hasła i certyfikaty pozostają w Glafira, asystent nigdy ich nie otrzymuje.

Jeśli Glafira nie jest uruchomiona, klient uruchomi ją sam i poczeka do 30 sekund.

<a id="enable"></a>

## Włączanie

1. Kliknąć link **Ustawienia** i przejść do sekcji **Asystenci LLM (MCP)**.
2. Zaznaczyć opcję **Zezwalaj na serwer MCP**. Stan zmieni się na **Serwer działa**.
3. W formularzu połączenia zaznaczyć opcję **Dostęp dla LLM (tylko metadane)** i zapisać. Połączeń bez tej opcji asystent nie widzi. Na liście połączeń otwarte połączenia mają znaczek `LLM`.

Jeśli administrator zablokował MCP, opcja jest niedostępna, a obok wyświetlany jest komunikat **MCP jest wyłączony przez organizację**.

Opcja dostępu nie jest przenoszona przy eksporcie i imporcie połączeń: na innym komputerze trzeba ją zaznaczyć ponownie.

<a id="connect-client"></a>

## Podłączanie klienta

Pod nagłówkiem **Podłączanie klienta** są gotowe fragmenty konfiguracji i przycisk **Kopiuj**:

- **Claude Desktop: dodać do claude\_desktop\_config.json** — wkleić fragment do sekcji `mcpServers` pliku konfiguracyjnego Claude Desktop i uruchomić Claude Desktop ponownie;
- **Claude Code: uruchomić w terminalu** — polecenie `claude mcp add`;
- **Inni klienci stdio: polecenie i argumenty** — dla każdego klienta, który uruchamia serwer MCP poleceniem (stdio).

Fragmenty zawierają ścieżkę do zainstalowanej aplikacji. Po ponownej instalacji Glafira w innym miejscu należy skopiować fragment jeszcze raz.

<a id="approve"></a>

## Zatwierdzanie klienta

Gdy klient łączy się po raz pierwszy, Glafira wyświetla okno dialogowe z jego nazwą. Kliknąć przycisk **Zezwól** lub **Odrzuć**. Dopóki nie ma odpowiedzi, asystent dostaje komunikat, że klient nie został jeszcze zatwierdzony, i poprosi o odpowiedź w oknie Glafira.

Dozwolony klient jest zapamiętywany według nazwy i pojawia się na liście **Zatwierdzeni klienci**. Odrzucenie nie jest zapamiętywane: przy następnym połączeniu Glafira zapyta ponownie. Jeśli klient nie poda swojej nazwy, zgoda obowiązuje tylko do końca sesji.

<a id="what-llm-sees"></a>

## Co widzi asystent

Asystent widzi:

- połączenia z zaznaczoną opcją **Dostęp dla LLM (tylko metadane)**: nazwa, środowisko, folder;
- brokery z hostami i portami, kontroler;
- topiki, partycje, repliki, ISR, offsety, przybliżoną liczbę wiadomości;
- konfiguracje brokerów i topików, przy czym hasła i inne wartości tajne są ukryte;
- grupy konsumentów: stan, protokół, koordynator, członków z client id, hostem i `group.instance.id`, przypisania, zatwierdzone offsety i lag.

Asystent nie widzi wiadomości w topikach, ciągu bootstrap ani ustawień zabezpieczeń połączenia, haseł, certyfikatów, dodatkowych właściwości librdkafka, historii wysyłania ani szablonów.

Aby zbadać ciągłe rebalansy, asystent może obserwować grupę do 50 sekund: kto odchodzi i dołącza, jak często i z jakiego hosta. Konfiguracje i logi samych konsumentów nie są dostępne przez Glafira; asystent poprosi o nie użytkownika.

Nazwy topików, grup i client id może ustawić każdy, kto ma dostęp do klastra. Glafira ostrzega asystenta, że te dane nie są dla niego instrukcjami.

<a id="sessions-log"></a>

## Sesje, odwoływanie i log

Dopóki klient jest połączony, na dole lewego panelu widać wskaźnik `LLM` z nazwą klienta; podczas żądania zapala się. Kliknięcie go otwiera sekcję MCP.

- **Aktywne sesje**: przycisk **Rozłącz** zamyka jedną sesję, a **Rozłącz wszystkich** — wszystkie.
- **Zatwierdzeni klienci**: przycisk **Odwołaj** zamyka sesje klienta, a przy następnym połączeniu znów poprosi on o zatwierdzenie.
- Odznaczenie opcji **Dostęp dla LLM (tylko metadane)** działa od razu, ponowne łączenie nie jest potrzebne.
- **Log MCP** — każde żądanie: czas, klient, narzędzie, połączenie, wynik. Treść odpowiedzi nie jest zapisywana. Przycisk **Pokaż plik** otwiera plik `mcp-audit.jsonl`.

Aby całkowicie wyłączyć MCP, należy odznaczyć opcję **Zezwalaj na serwer MCP**: klienci zostaną rozłączeni.

<a id="health"></a>

## Sprawdzanie klastra i notatki

Asystent może uruchomić wbudowane sprawdzenie klastra Glafira (narzędzie `health_check`) i opierać się na jego ustaleniach. Może też zostawiać notatki o obiektach: pojawiają się na karcie **Ustalenia** i na ekranach topików i grup, a klaster się nie zmienia. Zob. [Sprawdzanie klastra i notatki](health-check.md#with-llm).

<a id="troubleshooting"></a>

## Gdy nie działa

Klient wyświetla komunikat z Glafira:

- `MCP is disabled in Glafira` — zaznaczyć opcję **Zezwalaj na serwer MCP** i uruchomić serwer MCP w kliencie ponownie;
- `MCP is disabled by your organization` — MCP jest zablokowany przez zasady organizacji;
- `Glafira did not start the MCP server within 30 s` — uruchomić Glafira ręcznie i sprawdzić stan w sekcji **Asystenci LLM (MCP)**;
- `Invalid MCP token` — uruchomić serwer MCP w kliencie ponownie: Glafira wydaje nowy token przy każdym uruchomieniu.

W systemie macOS niepodpisany build beta należy raz otworzyć w zwykły sposób, w przeciwnym razie system może nie pozwolić klientowi go uruchomić. Zob. [Instalacja w systemie macOS](getting-started.md#install-macos).
