# Rozwiązywanie problemów

W tym rozdziale wyjaśniono typowe błędy połączenia i to, co sprawdzić w przypadku każdego z nich. Pokazano też, gdzie znajdują się logi i co dołączyć do zgłoszenia do pomocy technicznej. W systemie macOS ten rozdział otwiera polecenie **Rozwiązywanie problemów** w menu Pomoc.

Komunikat o błędzie zawiera krótki tekst, a na większości ekranów także kod błędu w nawiasie, na przykład `TIMEOUT`. Po rozwinięciu sekcji **Szczegóły** widać oryginalny komunikat klienta Kafki (librdkafka). Często podaje on dokładny adres brokera i przyczynę.

<a id="cannot-connect"></a>

## Nie można się połączyć

Na początek należy użyć przycisku **Przetestuj połączenie** w formularzu połączenia, zob. [Testowanie połączenia](connections.md#test). Test zatrzymuje się na pierwszym nieudanym kroku, a krok wskazuje, gdzie szukać:

- **Konfiguracja** — problem leży w samym formularzu: adres bez portu, brakujący lub nieczytelny plik certyfikatu, błędne hasło certyfikatu. Błąd wskazuje pole. Pliki certyfikatów opisano w sekcji [Błędy TLS](troubleshooting.md#tls-errors).
- **DNS / TCP** — komputer nie może dotrzeć do serwerów bootstrap. Zob. [DNS i TCP](troubleshooting.md#dns-tcp).
- **Kafka (TLS, SASL, metadane)** — sieć działa, ale handshake TLS, logowanie SASL lub żądanie metadanych nie powiodło się. Zob. [Błędy TLS](troubleshooting.md#tls-errors), [Błędy SASL](troubleshooting.md#sasl-errors) i [Limity czasu](troubleshooting.md#timeouts).

Najczęstszą przyczyną jest protokół, który nie pasuje do portu. Każdy listener brokera ma własny port i protokół. Który port należy do którego protokołu, wiedzą administratorzy Kafki, zob. [Protokoły zabezpieczeń](connections.md#security).

Jeśli test przechodzi, ale odczyt lub wysyłanie się nie udaje, przyczyną są często adresy brokerów. Klient pobiera listę brokerów z serwera bootstrap, a następnie łączy się z każdym brokerem pod adresem, który ten broker ogłasza (`advertised.listeners`). Jeśli są to adresy wewnętrzne, na przykład nazwy hostów Dockera lub Kubernetes, nie są one osiągalne z komputera użytkownika. Adresy są wymienione w tabeli brokerów w przeglądzie klastra, zob. [Przegląd klastra](topics.md#cluster-overview). Należy poprosić administratorów o listener osiągalny z zewnątrz.

<a id="dns-tcp"></a>

## DNS i TCP

Komunikat **Żaden z serwerów bootstrap nie jest osiągalny przez TCP** oznacza, że aplikacja nie mogła nawiązać połączenia TCP z żadnym adresem z pola **Serwery bootstrap**. Każdy adres ma do 5 sekund. Szczegóły pokazują ostatni sprawdzany adres i przyczynę. `DNS` w szczegółach oznacza, że nie udało się rozwiązać nazwy hosta.

Należy sprawdzić:

- nazwę hosta i port, bez prefiksu protokołu, takiego jak `https://`;
- czy VPN jest włączony, jeśli klaster znajduje się w sieci prywatnej;
- czy zapora nie blokuje portu;
- w przypadku brokera w Dockerze — czy port jest opublikowany na hoście, na przykład `-p 9092:9092`.

Aby sprawdzić port poza aplikacją, można uruchomić `nc -vz <host> <port>` w systemie macOS lub Linux albo `Test-NetConnection <host> -Port <port>` w PowerShell w systemie Windows.

Komunikat **Nie udało się połączyć z brokerami** po pomyślnym kroku TCP zwykle oznacza, że broker zamknął połączenie lub później okazał się nieosiągalny. Warto też sprawdzić adresy brokerów, zob. [Nie można się połączyć](troubleshooting.md#cannot-connect).

<a id="tls-errors"></a>

## Błędy TLS

Błędy w kroku **Kafka (TLS, SASL, metadane)** podczas handshake'u TLS:

- **Broker przyjmuje pod tym adresem połączenia bez TLS — sprawdź port lub wybierz protokół bez SSL**.
- **Certyfikat brokera nie jest podpisany przez zaufany CA — sprawdź pole „Zaufany certyfikat CA”**. W polu **Zaufany certyfikat CA** należy wskazać plik CA, który podpisał certyfikaty brokerów. Ten plik mogą przekazać administratorzy. Przy pustym polu zaufane są tylko systemowe certyfikaty główne.
- **Nazwa hosta brokera nie zgadza się z jego certyfikatem**. Należy łączyć się przez nazwę, dla której wystawiono certyfikat, a nie przez adres IP lub alias. Wyłącznie w środowiskach testowych można wyłączyć opcję **Weryfikuj nazwę hosta**.
- **Broker odrzucił certyfikat klienta**. Broker wymaga certyfikatu klienta lub nie ufa używanemu certyfikatowi. Należy sprawdzić, czy opcja **Certyfikat klienta (mTLS)** jest włączona i czy certyfikat został wystawiony dla tego klastra. Zob. [Certyfikat klienta (mTLS)](connections.md#mtls).
- **Błąd połączenia TLS** — inny problem z TLS. Przyczynę widać po rozwinięciu sekcji **Szczegóły**.

Jeśli wybrano protokół bez SSL, a mimo to pojawia się błąd TLS, port najprawdopodobniej należy do listenera TLS. Szczegóły zawierają wtedy tekst `connecting to a SSL listener?`. Należy wybrać `SSL` lub `SASL_SSL` albo użyć portu listenera bez TLS.

Problemy z plikami certyfikatów są zgłaszane w kroku **Konfiguracja**, zanim aplikacja się połączy. Komunikat zaczyna się od nazwy pola: zaufany CA, certyfikat klienta lub klucz klienta. Typowe przypadki:

- Ścieżka nie jest bezwzględna lub pliku nie można odczytać. Należy wpisać pełną ścieżkę lub wybrać plik przyciskiem **Przeglądaj…**.
- **Klucz klienta: klucz jest zaszyfrowany — podaj hasło klucza**, **Klucz klienta: nieprawidłowe hasło klucza**, **Certyfikat klienta: nieprawidłowe hasło magazynu PKCS\#12** lub **Nieprawidłowe hasło magazynu JKS (lub plik jest uszkodzony)** — należy wpisać w formularzu właściwe hasło.
- **Klucz klienta nie pasuje do certyfikatu klienta** — plik klucza należy do innego certyfikatu.
- Certyfikat klienta wygasł lub nie jest jeszcze ważny — komunikat podaje datę. Należy poprosić o nowy certyfikat.
- Magazyn PKCS#12 używa przestarzałego algorytmu lub magazyn jest w formacie JCEKS. Plik należy przekonwertować zgodnie z opisem w sekcji [Formaty certyfikatów](connections.md#certificate-formats).

<a id="sasl-errors"></a>

## Błędy SASL

Komunikat **Uwierzytelnianie nie powiodło się** oznacza, że broker nie przyjął logowania SASL. Należy sprawdzić:

- nazwę użytkownika i hasło. Podczas edycji zapisanego połączenia puste pole hasła zachowuje zapisane hasło. W razie wątpliwości co do poprawności hasła należy wpisać je ponownie;
- pole **Mechanizm SASL**. Musi to być mechanizm, dla którego utworzono użytkownika. Poświadczenia SCRAM są tworzone osobno dla `SCRAM-SHA-256` i `SCRAM-SHA-512`, a użytkownicy `PLAIN` są konfigurowani niezależnie od nich;
- protokół: `SASL_SSL` i `SASL_PLAINTEXT` zwykle używają różnych portów.

**Nie ustawiono hasła SASL dla połączenia** — dla tego połączenia nie zapisano hasła, na przykład po imporcie. Należy otworzyć połączenie, wpisać hasło i zapisać. Zob. [Hasła](connections.md#passwords).

**Brak dostępu do systemowego magazynu sekretów** — aplikacja nie może odczytać hasła z systemowego magazynu sekretów. W systemie macOS należy zezwolić na dostęp, gdy pyta o to pęk kluczy. W systemie Linux należy sprawdzić, czy w sesji działa GNOME Keyring, KWallet lub inny dostawca Secret Service.

Komunikat **Brak uprawnień do wykonania operacji** oznacza coś innego: logowanie się powiodło, ale użytkownik Kafki nie ma uprawnień (ACL) do tej operacji, na przykład do odczytu topiku lub grupy. Należy poprosić administratorów o uprawnienia. Warto pamiętać, że Kafka nie zgłasza błędu dla topików, do których użytkownik nie ma uprawnienia Describe: po prostu nie ma ich na liście topików.

<a id="timeouts"></a>

## Limity czasu

Komunikat **Przekroczono limit czasu operacji** oznacza, że broker nie odpowiedział na czas. Typowe przyczyny to wolna sieć lub VPN, przeciążony klaster albo nieosiągalne adresy brokerów, zob. [Nie można się połączyć](troubleshooting.md#cannot-connect).

Problem z TLS lub SASL może wyglądać jak przekroczenie limitu czasu. Aplikacja próbuje znaleźć rzeczywistą przyczynę i pokazać ją zamiast tego komunikatu, ale nie zawsze jest to możliwe. Warto rozwinąć sekcję **Szczegóły**: komunikaty takie jak `SSL handshake failed` lub `SASL authentication error` wskazują przyczynę.

Jeśli sieć jest po prostu wolna, należy zwiększyć wartość w polu **Limit czasu żądania, ms** w formularzu połączenia. Wartość domyślna to `30000` ms. Ta wartość dotyczy żądań do brokerów, wysyłania wiadomości i odczytu. Test połączenia używa własnych limitów (5 sekund na każdy adres TCP i 10 sekund na krok Kafka), więc większa wartość nie zmienia testu.

<a id="read-only-blocked"></a>

## Działanie jest niedostępne lub odrzucone

Jeśli przycisk jest ukryty lub nieaktywny albo działanie zostało odrzucone, należy sprawdzić następujące przyczyny:

- Połączenie jest tylko do odczytu. Oznacza je ikona kłódki obok nazwy. Przyciski zmieniające klaster są ukryte lub nieaktywne. Jeśli takie żądanie mimo to dotrze do rdzenia aplikacji, na przykład po włączeniu trybu przy otwartym ekranie, błąd informuje, że połączenie jest tylko do odczytu. Aby wprowadzić zmiany, należy otworzyć połączenie, wyłączyć opcję **Tylko do odczytu** i zapisać. Zob. [Tylko do odczytu](connections.md#read-only).
- Połączenie jest typu prod. Zmiany wymagają wpisania nazwy połączenia. Dopóki nazwa się nie zgadza, działanie nie jest wykonywane. Zob. [Środowiska i prod](connections.md#environments).
- Grupa konsumentów ma aktywnych członków. Offsety można zresetować, a grupę usunąć tylko wtedy, gdy nie ma członków. Zob. [Reset offsetów](groups.md#reset-offsets).
- Ustawieniem zarządza organizacja. Zasady mogą ustawić język interfejsu i sprawdzanie aktualizacji, a tych ustawień nie można zmienić w aplikacji. Zob. [Ustawienia określone przez organizację](getting-started.md#managed-settings).
- Kafka odrzuciła operację z komunikatem **Brak uprawnień do wykonania operacji**. To kwestia uprawnień w klastrze, a nie ustawień Glafiry. Zob. [Błędy SASL](troubleshooting.md#sasl-errors).

<a id="logs"></a>

## Logi

Aplikacja zapisuje log swojej pracy i błędów klienta Kafki. Dokładny folder jest pokazany na ekranie **O programie** w pozycji **Logi:** Domyślnie:

- macOS: `~/Library/Logs/io.github.vvnovg.glafira.kafka`
- Windows: `%LOCALAPPDATA%\io.github.vvnovg.glafira.kafka\logs`
- Linux: `~/.local/share/io.github.vvnovg.glafira.kafka/logs`

Bieżący plik to `app.log`. Gdy osiągnie 5 MB, zostaje przemianowany na `app.1.log`, a starsze pliki przesuwają się do `app.2.log` i dalej. Aplikacja przechowuje do pięciu plików i usuwa najstarszy.

Hasła nie są zapisywane w logach. Zapisywane są adresy brokerów i treści błędów Kafki. Nie trzeba zbierać plików ręcznie: archiwum diagnostyczne je zawiera, zob. [Archiwum diagnostyczne](updates.md#diagnostics).

<a id="support"></a>

## Kontakt z pomocą techniczną

Przed skontaktowaniem się z pomocą techniczną należy użyć przycisku **Przetestuj połączenie** i zanotować, który krok się nie powiódł. Następnie trzeba wysłać zgłoszenie do osób, które zapewniają wsparcie dla Glafiry, na przykład do administratorów. Należy dołączyć:

- archiwum diagnostyczne, zob. [Archiwum diagnostyczne](updates.md#diagnostics). Zawiera wersję, build, logi i profile połączeń bez haseł;
- opis krok po kroku: co zostało zrobione, czego oczekiwano i co się stało;
- pełny tekst błędu wraz z zawartością sekcji **Szczegóły**. Ten tekst można zaznaczyć i skopiować;
- zrzut ekranu, jeśli problem dotyczy interfejsu;
- czas wystąpienia błędu, aby można go było znaleźć w logach.

Jeśli nie można utworzyć archiwum, należy podać wartości **Wersja** i **Build** z ekranu **O programie** i dołączyć pliki logów z folderu logów.
