# Aktualizacje i diagnostyka

W tym rozdziale opisano, jak aktualizować aplikację, ekran **O programie**, archiwum diagnostyczne i jak wesprzeć projekt.

<a id="updates"></a>

## Aktualizacje

Przy uruchomieniu aplikacja jednorazowo sprawdza, czy jest dostępna nowa wersja. Jeśli tak, powiadomienie nad kartami podaje jej numer. Przycisk **Szczegóły** otwiera ekran **O programie**. Jeśli sprawdzenie się nie powiedzie, na przykład bez dostępu do internetu, żaden komunikat nie jest wyświetlany.

Aby sprawdzić ręcznie:

1. Otworzyć ekran **O programie** na dole lewego panelu.
2. W sekcji **Aktualizacje** kliknąć przycisk **Sprawdź aktualizacje**.
3. Jeśli jest dostępna nowa wersja, ekran pokazuje jej numer i informacje o wydaniu. Kliknąć przycisk **Zainstaluj i uruchom ponownie**.

Aplikacja pobiera aktualizację, sprawdza jej podpis, instaluje ją i uruchamia się ponownie. Pobieranie przerwane przez problem z siecią jest automatycznie ponawiane, łącznie do trzech prób. Jeśli wszystkie trzy próby się nie powiodą, aplikacja zgłasza błąd **Pobieranie aktualizacji zostało przerwane — sprawdź połączenie sieciowe i spróbuj ponownie**. Jeśli weryfikacja podpisu się nie powiedzie, aplikacja zgłasza błąd **Podpis aktualizacji nie przeszedł weryfikacji — aktualizacja nie została zainstalowana**.

Instalacja może wymagać hasła administratora, ponieważ aplikacja jest zainstalowana dla wszystkich użytkowników komputera: instalatorem `.pkg` w systemie macOS, instalatorem `.msi` w systemie Windows lub pakietem `.deb` albo `.rpm` w systemie Linux. Użytkownicy bez uprawnień administratora nie mogą aktualizować takiej instalacji.

Jeśli trwa odczyt wiadomości, działa sesja konsumenta lub trwa wysyłka wsadowa, aplikacja najpierw pyta o zgodę i podaje, ile sesji jest aktywnych. Przycisk **Zatrzymaj sesje i zaktualizuj** zatrzymuje je i kontynuuje, a przycisk **Anuluj** rezygnuje z aktualizacji. Po aktualizacji połączenia, ustawienia i inne dane są zachowywane.

Aktualizacje są pobierane ze strony wydań w GitHub. Jeśli sprawdzenie kończy się błędem **Nie udało się sprawdzić aktualizacji**, należy sprawdzić sieć lub serwer proxy. Komunikat **Aktualizacje nie są skonfigurowane w tym buildzie** oznacza, że ten build został przygotowany bez klucza aktualizacji, na przykład jako build deweloperski lub testowy. W takim przypadku nowe wersje należy pobierać ze strony wydań: https://github.com/vvnovg/glafira-kafka-releases/releases

Organizacja może za pomocą zasad wyłączyć sprawdzanie aktualizacji lub skierować je do wewnętrznego serwera lustrzanego. Gdy sprawdzanie jest wyłączone, po kliknięciu przycisku **Sprawdź aktualizacje** sekcja **Aktualizacje** pokazuje komunikat **Aktualizacjami zarządza organizacja**. Podobnie jak **Aktualizacje nie są skonfigurowane w tym buildzie**, ten komunikat pojawia się dopiero po kliknięciu; sprawdzenie przy uruchomieniu w tych przypadkach niczego nie pokazuje.

<a id="about"></a>

## O programie

Ekran **O programie** otwiera się linkiem na dole lewego panelu. W systemie macOS jest też dostępny w menu Pomoc jako **O programie**.

Ekran pokazuje krótki opis aplikacji oraz:

- **Wersja** — numer wersji, oraz **Build** — identyfikator kodu źródłowego, z którego zbudowano aplikację. Oba należy podać w zgłoszeniu do pomocy technicznej.
- **System** — system operacyjny i architektura procesora.
- **Licencja** — Apache 2.0; link otwiera tekst licencji w przeglądarce.
- **Zasady organizacji** — **ustawione** lub **brak**. Błędy w zasadach są wymienione poniżej.
- **Dane:** — folder z profilami połączeń, szablonami, historią i ustawieniami.
- **Logi:** — folder z plikami logów. Zob. [Logi](troubleshooting.md#logs).

Na tym ekranie znajdują się też sekcje **Aktualizacje** i **Diagnostyka**.

<a id="diagnostics"></a>

## Archiwum diagnostyczne

Archiwum diagnostyczne zbiera to, czego pomoc techniczna potrzebuje, aby zrozumieć problem. Aby je utworzyć:

1. Otworzyć ekran **O programie**.
2. W sekcji **Diagnostyka** kliknąć przycisk **Zbierz diagnostykę**. W systemie macOS można też wybrać polecenie **Zbierz diagnostykę…** w menu Pomoc.
3. Wybrać miejsce zapisu pliku. Proponowana nazwa to `diagnostics-<date>.zip`.

Po zapisaniu archiwum ekran pokazuje jego ścieżkę. Plik ZIP zawiera:

- `info.json` — wersję i build aplikacji, system operacyjny, ścieżki folderów danych i logów oraz stan zasad;
- `connections.json` — profile połączeń: nazwy, środowiska, serwery bootstrap, protokoły, nazwy użytkowników SASL, ścieżki plików certyfikatów, dodatkowe właściwości i limity czasu;
- `logs/` — pliki logów aplikacji.

Archiwum nie zawiera haseł: są one przechowywane w systemowym magazynie sekretów, a dodatkowe właściwości z hasłami i sekretami są odrzucane przy zapisie połączenia. Nie są do niego dołączane również pliki certyfikatów. Nazwy hostów, nazwy użytkowników i ścieżki plików są zapisane otwartym tekstem, a logi zawierają adresy brokerów z komunikatów błędów Kafki. Przed wysłaniem archiwum poza firmę warto je przejrzeć.

Aplikacja sama niczego nie wysyła. Archiwum należy samodzielnie dołączyć do zgłoszenia do pomocy technicznej, zob. [Kontakt z pomocą techniczną](troubleshooting.md#support).

<a id="support"></a>

## Wesprzyj projekt

Glafira jest bezpłatna, a jej kod źródłowy jest otwarty na licencji Apache 2.0. Darowizny pozwalają ją rozwijać. Pozycja **Wesprzyj projekt** u dołu lewego panelu, na ekranie **O programie** oraz, w systemie macOS, w menu Pomoc otwiera stronę wsparcia w przeglądarce: https://github.com/vvnovg/glafira-kafka-releases/blob/main/SUPPORT.md
