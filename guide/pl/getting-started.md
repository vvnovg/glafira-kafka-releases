# Pierwsze kroki

Glafira for Apache Kafka to desktopowy klient klastrów Kafka. Pozwala przeglądać topiki i grupy konsumentów, czytać i wysyłać wiadomości oraz resetować offsety. W tym rozdziale opisano instalację, pierwsze uruchomienie, lokalną Kafkę do ćwiczeń i język interfejsu.

<a id="install-macos"></a>

## Instalacja w systemie macOS

Glafira działa w systemie macOS 11 lub nowszym. Instalator można pobrać ze strony wydań: https://github.com/vvnovg/glafira-kafka-releases/releases

Należy wybrać plik odpowiedni dla danego Maca: `aarch64` dla Apple silicon (M1 i nowsze), `x64` dla procesorów Intel.

1. Otworzyć plik `.dmg`.
2. Przeciągnąć Glafira for Apache Kafka do folderu Aplikacje.
3. Uruchomić aplikację z folderu Aplikacje lub z Launchpada.

Można też użyć instalatora `.pkg`. Instaluje on aplikację w `/Applications` dla wszystkich użytkowników.

Buildy beta nie są jeszcze podpisane przez Apple, dlatego macOS blokuje pierwsze uruchomienie. Aby na nie zezwolić:

1. Spróbować raz otworzyć aplikację i zamknąć ostrzeżenie.
2. Otworzyć Ustawienia systemowe → Prywatność i ochrona.
3. Przewinąć w dół do komunikatu o Glafira for Apache Kafka i kliknąć „Otwórz mimo to”. W razie potrzeby potwierdzić hasłem.

W systemie macOS 14 i starszych można zamiast tego kliknąć aplikację prawym przyciskiem myszy, wybrać „Otwórz”, a następnie ponownie „Otwórz”.

Jeśli to nie pomoże, należy jednorazowo usunąć flagę kwarantanny w Terminalu:

```bash
xattr -dr com.apple.quarantine "/Applications/Glafira for Apache Kafka.app"
```

Jeśli nie otwiera się sam pobrany plik `.dmg`, najpierw trzeba usunąć flagę z niego:

```bash
xattr -d com.apple.quarantine ~/Downloads/Glafira_for_Apache_Kafka_*.dmg
```

Aktualizacje instalowane z poziomu aplikacji nie wymagają tych kroków.

<a id="install-windows"></a>

## Instalacja w systemie Windows

Instalator `.msi` (`x64`) można pobrać ze strony wydań: https://github.com/vvnovg/glafira-kafka-releases/releases

1. Uruchomić plik `.msi`.
2. Potwierdzić instalację w oknie systemu Windows. Aplikacja jest instalowana dla wszystkich użytkowników komputera, dlatego potrzebne są uprawnienia administratora.
3. Uruchomić Glafira for Apache Kafka z menu Start.

Buildy beta nie są jeszcze podpisane. Jeśli SmartScreen zablokuje instalator, należy kliknąć „Więcej informacji”, a następnie „Uruchom mimo to”. Można też odblokować plik w PowerShell przed jego uruchomieniem:

```powershell
Unblock-File -Path "$env:USERPROFILE\Downloads\Glafira_for_Apache_Kafka_*_x64_en-US.msi"
```

<a id="install-linux"></a>

## Instalacja w systemie Linux

Buildy są dostępne dla `x86_64` w trzech formatach. Ze strony wydań należy pobrać format odpowiedni dla danej dystrybucji: https://github.com/vvnovg/glafira-kafka-releases/releases

- Debian, Ubuntu i pochodne: `sudo apt install ./<file>.deb`
- Fedora, RHEL i pochodne: `sudo dnf install ./<file>.rpm`
- Dowolna dystrybucja: nadać plikowi AppImage prawo wykonywania poleceniem `chmod +x <file>.AppImage` i uruchomić go.

Po zainstalowaniu pakietu aplikację uruchamia się z menu aplikacji lub poleceniem `glafira-kafka` w terminalu.

Glafira przechowuje hasła w Secret Service (na przykład GNOME Keyring lub KWallet). Jedna z tych usług musi działać w sesji użytkownika. W przeciwnym razie aplikacja nie może zapisywać haseł do połączeń.

<a id="first-run"></a>

## Pierwsze uruchomienie

Okno składa się z dwóch części. Po lewej stronie znajduje się panel **Połączenia**. Po prawej są karty: każdy otwarty ekran, na przykład lista topików lub Producer, otrzymuje własną kartę.

Dopóki nie ma żadnych połączeń, panel wyświetla krótką podpowiedź, a obszar kart proponuje dodanie połączenia. Aby zacząć:

1. Kliknąć `+` ( **Nowe połączenie** ) w nagłówku panelu **Połączenia**.
2. Wypełnić pola **Nazwa** i **Serwery bootstrap**, na przykład `localhost:9092`.
3. Kliknąć przycisk **Przetestuj połączenie**, aby sprawdzić ustawienia, a następnie przycisk **Zapisz**.

Po zapisaniu otwiera się przegląd klastra. Pod każdym połączeniem w panelu znajdują się linki: **Przegląd klastra**, **Topiki**, **Grupy konsumentów**, **Producer** i **Sesja konsumenta**. Wszystkie ustawienia połączenia opisano w rozdziale [Połączenia](connections.md#create).

Na dole panelu znajdują się linki: **Ustawienia**, **Wesprzyj projekt**, **Pomoc** i **O programie**. Znak `?` obok pola wyświetla krótką podpowiedź na jego temat. Klawisz `F1` w dowolnym miejscu okna otwiera rozdział podręcznika dotyczący bieżącego ekranu. W systemie macOS podręcznik jest dostępny także w menu Pomoc.

Jeśli wcześniej używano Kafka Explorer (poprzedniej nazwy aplikacji), przy pierwszym uruchomieniu Glafira kopiuje jego połączenia, szablony i historię wysyłania. Hasła również są kopiowane z systemowego magazynu sekretów. Istniejące dane nie są nadpisywane, a stare dane nie są usuwane. macOS może jednorazowo poprosić o dostęp do starych elementów pęku kluczy.

<a id="local-cluster"></a>

## Lokalna Kafka do ćwiczeń

Aby wypróbować aplikację bez współdzielonego klastra, można uruchomić Kafkę na własnym komputerze w Dockerze. Wystarczy jeden broker:

```bash
docker run -d --name kafka -p 9092:9092 apache/kafka:3.9.0
```

Następnie należy utworzyć połączenie z serwerem bootstrap `localhost:9092`, protokołem `PLAINTEXT` i środowiskiem `local`. Gdy broker nie jest już potrzebny, usuwa się go poleceniem `docker rm -f kafka`.

Podpowiedź w pustym panelu **Połączenia** wspomina o `docker compose -f dev/docker-compose.yml up -d`. Ten plik jest częścią kodu źródłowego Glafiry. Mając kod źródłowy, można tym poleceniem uruchomić broker z trzema listenerami:

- `localhost:9092` — `PLAINTEXT`.
- `localhost:9094` — `SASL_PLAINTEXT` z `PLAIN` (użytkownicy `tester` / `tester-secret` i `admin` / `admin-secret`) oraz SCRAM (`scram-user` / `scram-secret` dla `SCRAM-SHA-256` i `SCRAM-SHA-512`).
- `localhost:9095` — `SSL` z certyfikatami klienta (mTLS). Certyfikaty testowe znajdują się w `dev/certs/out`.

Ci użytkownicy i certyfikaty służą wyłącznie do testów lokalnych.

<a id="language"></a>

## Język interfejsu

Interfejs jest dostępny po angielsku, rosyjsku, niemiecku, francusku, hiszpańsku, włosku, polsku, portugalsku (Brazylia), chińsku (uproszczonym i tradycyjnym), japońsku i koreańsku. Wszystkie języki są bezpłatne.

Aby zmienić język:

1. Kliknąć link **Ustawienia** na dole lewego panelu.
2. Wybrać język z listy **Język interfejsu**.

Zmiana działa od razu, także w tym podręczniku i w menu macOS. Opcja **Język systemu** (domyślna) przyjmuje język systemu operacyjnego. Jeśli tego języka nie ma na liście, interfejs jest po angielsku.

<a id="managed-settings"></a>

## Ustawienia określone przez organizację

W firmie administratorzy mogą za pomocą zasad ustawić niektóre opcje dla wszystkich użytkowników. Tych opcji nie można zmienić w aplikacji. Zasady mogą:

- ustawić język interfejsu — lista języków na ekranie **Ustawienia** jest wtedy niedostępna, a obok widnieje informacja, że język ustawiła organizacja;
- wyłączyć sprawdzanie aktualizacji lub skierować je do wewnętrznego serwera lustrzanego.

Aby sprawdzić, czy obowiązują jakieś zasady, należy otworzyć ekran **O programie** i spojrzeć na pozycję **Zasady organizacji**. Jeśli zasady zawierają błąd, jest on tam wyświetlany. W sprawie zasad należy kontaktować się z administratorami. Aktualizacje opisano w rozdziale [Aktualizacje i diagnostyka](updates.md).
