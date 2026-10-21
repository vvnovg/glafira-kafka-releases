# Połączenia

Połączenie to zapisany profil jednego klastra Kafka: adresy, ustawienia zabezpieczeń i hasła. Połączenia są wyświetlane w panelu **Połączenia** po lewej stronie. Można je porządkować w folderach i przenosić na inny komputer za pomocą eksportu i importu.

<a id="create"></a>

## Tworzenie połączenia

1. Kliknąć `+` ( **Nowe połączenie** ) w nagłówku panelu **Połączenia**. Aby utworzyć połączenie od razu w folderze, należy otworzyć menu folderu i wybrać polecenie **Nowe połączenie tutaj**.
2. W polu **Nazwa** wpisać nazwę. Jest ona wyświetlana w panelu i w tytułach kart.
3. W polu **Serwery bootstrap** wpisać jeden lub kilka adresów w formacie `host:port`, oddzielonych przecinkami, na przykład `kafka1:9092,kafka2:9092`. Wystarczy jeden osiągalny broker: klient sam wykrywa pozostałe.
4. W polu **Protokół** wybrać protokół i wypełnić dodane przez niego pola. Zob. [Protokoły zabezpieczeń](connections.md#security).
5. Opcjonalnie wybrać wartości w polach **Folder**, **Środowisko** i **Kolor** (kolor kropki obok nazwy).
6. Kliknąć przycisk **Przetestuj połączenie**, a następnie przycisk **Zapisz**.

Po zapisaniu formularz się zamyka i otwiera się przegląd klastra.

Dwa kolejne pola służą do przypadków szczególnych. Pole **Limit czasu żądania, ms** określa, jak długo czekać na odpowiedź brokera. Wartość domyślna to `30000` ms, minimalna — `1000`. Warto ją zwiększyć w wolnych sieciach i przy VPN. Pole **Dodatkowe właściwości librdkafka** przyjmuje dowolne ustawienia librdkafka w postaci `key=value`, po jednym w wierszu. Wiersze zaczynające się od `#` są ignorowane. Nie można tam podawać haseł ani kluczy: właściwości, których nazwy zawierają `password` lub `secret`, a także `sasl.jaas.config`, `ssl.key.pem` i `ssl.keystore.key` są odrzucane przy zapisie.

Aby później zmienić połączenie, należy kliknąć `✎` ( **Edytuj połączenie** ) obok jego nazwy. Aby je usunąć, należy otworzyć formularz, kliknąć przycisk **Usuń** i potwierdzić przyciskiem **Tak, usuń**.

<a id="test"></a>

## Testowanie połączenia

Przycisk **Przetestuj połączenie** sprawdza ustawienia z formularza krok po kroku i zatrzymuje się na pierwszym nieudanym kroku:

1. **Konfiguracja** — adresy, pliki certyfikatów, hasła i nazwa użytkownika SASL.
2. **DNS / TCP** — nazwy hostów są rozwiązywane, a połączenie TCP z co najmniej jednym serwerem bootstrap zostaje nawiązane.
3. **Kafka (TLS, SASL, metadane)** — handshake TLS i logowanie SASL kończą się powodzeniem, a broker zwraca metadane. Po pomyślnym zakończeniu krok pokazuje liczbę brokerów i topików.

Test używa wartości, które są aktualnie w formularzu, w tym wpisanych, ale niezapisanych haseł. Nic nie jest zapisywane ani zmieniane w klastrze. Jeśli krok się nie powiedzie, pod nim wyświetlany jest tekst błędu i szczegóły. Typowe błędy wyjaśniono w sekcji [Nie można się połączyć](troubleshooting.md#cannot-connect).

<a id="security"></a>

## Protokoły zabezpieczeń

Wartość w polu **Protokół** musi odpowiadać listenerowi brokera, z którym następuje połączenie. Właściwy protokół znają administratorzy Kafki.

- `PLAINTEXT` — bez szyfrowania i bez logowania. Typowy dla klastrów lokalnych i testowych.
- `SSL` — ruch jest szyfrowany przez TLS. Z certyfikatem klienta jest to mTLS.
- `SASL_PLAINTEXT` — logowanie nazwą użytkownika i hasłem, bez szyfrowania.
- `SASL_SSL` — logowanie nazwą użytkownika i hasłem przez TLS.

Po zmianie protokołu formularz pokazuje tylko pola, których ten protokół wymaga.

<a id="sasl"></a>

## SASL

Dla `SASL_PLAINTEXT` i `SASL_SSL` należy wypełnić trzy pola:

- **Mechanizm SASL** — `PLAIN`, `SCRAM-SHA-256` lub `SCRAM-SHA-512`. To klaster określa, który z nich działa.
- **Użytkownik**.
- **Hasło**. Hasło jest zapisywane w systemowym magazynie sekretów, a nie w profilu. Zob. [Hasła](connections.md#passwords).

SCRAM nigdy nie przesyła samego hasła. `PLAIN` przesyła je bez zmian, więc przez `SASL_PLAINTEXT` hasło jest przesyłane otwartym tekstem. Dla klastrów poza własnym komputerem warto w miarę możliwości używać `SASL_SSL`.

<a id="tls"></a>

## SSL i zaufany CA

Dla `SSL` i `SASL_SSL` formularz dodaje ustawienia TLS.

**Zaufany certyfikat CA** to certyfikat urzędu (CA), który podpisał certyfikaty brokerów. Jeśli system operacyjny już ufa temu CA, pole można pozostawić puste: używane są wtedy systemowe certyfikaty główne. W przeciwnym razie należy wpisać pełną ścieżkę do pliku lub kliknąć przycisk **Przeglądaj…**. Po wybraniu pliku pojawia się pole **Format CA**. Dla PKCS#12 i JKS jest też pole **Hasło magazynu CA**.

Opcja **Weryfikuj nazwę hosta** jest domyślnie włączona. Sprawdza, czy certyfikat brokera został wystawiony dla nazwy hosta, z którą następuje połączenie. Wyłączać ją należy tylko w środowiskach testowych, na przykład gdy brokery używają certyfikatów samopodpisanych z innymi nazwami.

<a id="mtls"></a>

## Certyfikat klienta (mTLS)

Jeśli brokery uwierzytelniają klientów certyfikatem, należy włączyć opcję **Certyfikat klienta (mTLS)**. Certyfikat i klucz wydają administratorzy Kafki. Następnie trzeba wypełnić:

- **Certyfikat klienta** — ścieżka do pliku certyfikatu.
- **Format certyfikatu klienta** — PEM, PKCS#12 lub JKS.
- Dla PEM: **Klucz klienta** — ścieżka do pliku klucza prywatnego, oraz **Hasło klucza (jeśli zaszyfrowany)**, jeśli klucz jest zaszyfrowany.
- Dla PKCS#12: **Hasło magazynu kluczy**.
- Dla JKS: **Hasło magazynu kluczy**, **Hasło klucza (jeśli inne)**, jeśli klucz ma własne hasło, oraz **Alias klucza**, jeśli magazyn zawiera kilka kluczy.

Ścieżki muszą być bezwzględne. Pliki są odczytywane przy każdym połączeniu, więc po odnowieniu certyfikatu wystarczy połączyć się ponownie. Aplikacja sprawdza, czy klucz pasuje do certyfikatu i czy certyfikat nie wygasł.

Typowy przypadek to klaster w Kubernetes, w którym sekret klienta zawiera `tls.crt`, `tls.key` i `ca.crt`. Należy zapisać je do plików, wybrać protokół `SSL`, w polu **Zaufany certyfikat CA** wskazać `ca.crt`, a `tls.crt` i `tls.key` użyć jako certyfikatu i klucza klienta PEM z pustymi hasłami.

<a id="certificate-formats"></a>

## Formaty certyfikatów

Glafira odczytuje trzy formaty. Format jest wybierany automatycznie na podstawie rozszerzenia pliku; warto go sprawdzić w polu formatu i w razie potrzeby zmienić.

- PEM — plik tekstowy zaczynający się od `-----BEGIN`. Rozszerzenia `.pem`, `.crt`, `.cer`, `.key`. W przypadku certyfikatu klienta klucz jest osobnym plikiem. Zaufany CA może być też binarnym certyfikatem DER.
- PKCS#12 — binarny magazyn z certyfikatem i kluczem. Rozszerzenia `.p12`, `.pfx`.
- JKS — magazyn kluczy Javy. Rozszerzenia `.jks`, `.keystore`, `.truststore`.

Dwa rodzaje plików trzeba najpierw przekonwertować:

- Magazyny JCEKS nie są obsługiwane. Należy przekonwertować je do PKCS#12:

```bash
keytool -importkeystore -srcstoretype JCEKS -srckeystore client.jceks -deststoretype PKCS12 -destkeystore client.p12
```

- Magazyny PKCS#12 zaszyfrowane starszymi algorytmami (RC2, 3DES) mogą się nie otwierać. Jeśli aplikacja zgłasza przestarzały algorytm, należy utworzyć plik ponownie z AES:

```bash
openssl pkcs12 -export -keypbe AES-256-CBC -certpbe AES-256-CBC -in tls.crt -inkey tls.key -out client.p12
```

<a id="passwords"></a>

## Hasła

Hasło SASL oraz hasła kluczy i magazynów są zapisywane w systemowym magazynie sekretów: w pęku kluczy (Keychain) w systemie macOS, w Menedżerze poświadczeń w systemie Windows, w Secret Service w systemie Linux. Nigdy nie trafiają do profilu połączenia, plików eksportu ani archiwum diagnostycznego.

Podczas edycji zapisanego połączenia pola haseł są puste i wyświetlają tekst **pozostaw puste, aby nie zmieniać**. Pozostawienie pola pustego zachowuje zapisane hasło, a wpisanie nowego je zastępuje. Aby usunąć zapisane hasło certyfikatu, należy zaznaczyć opcję **Usuń zapisane hasło** obok pola. Przy zapisie usuwane są hasła, których połączenie już nie używa, na przykład po zmianie protokołu na `PLAINTEXT`.

Jeśli aplikacja nie ma dostępu do magazynu sekretów, przy zapisie wyświetlany jest błąd. W systemie Linux należy sprawdzić, czy w sesji działa GNOME Keyring, KWallet lub inny dostawca Secret Service.

<a id="environments"></a>

## Środowiska i prod

Pole **Środowisko** oznacza połączenie: `local`, `dev`, `test`, `stage` lub `prod`. Środowisko jest wyświetlane jako kolorowa plakietka obok nazwy w panelu, dzięki czemu zawsze widać, z którym klastrem się pracuje.

`prod` dodaje ochronę. Każda zmiana w połączeniu prod, na przykład utworzenie topiku, wysłanie wiadomości, reset offsetów lub uruchomienie sesji konsumenta, wymaga wpisania nazwy połączenia. Usuwanie i czyszczenie w każdym połączeniu wymaga wpisania nazwy obiektu (usuwanie kilku topików — ich liczby), a w połączeniu prod trzeba wpisać obie nazwy. Te potwierdzenia pojawiają się w oknie aplikacji.

Nowe połączenie prod ma domyślnie włączoną opcję **Tylko do odczytu**. Jeśli trzeba wprowadzać zmiany, należy odznaczyć to pole.

<a id="read-only"></a>

## Tylko do odczytu

Opcja **Tylko do odczytu** blokuje wszystko, co zmienia klaster: tworzenie, zmienianie, czyszczenie i usuwanie topików, wysyłanie wiadomości, reset offsetów, usuwanie grup i uruchamianie sesji konsumenta. Przyciski tych działań są ukryte lub nieaktywne, a rdzeń aplikacji również odrzuca takie operacje. Przeglądanie i odczyt pozostają dostępne. Ikona kłódki obok nazwy w panelu oznacza połączenia tylko do odczytu.

Z tego trybu warto korzystać, gdy trzeba tylko obejrzeć dane, zwłaszcza we współdzielonych klastrach. Tryb tylko do odczytu to ochrona wewnątrz aplikacji: nie zastępuje uprawnień dostępu (ACL) w Kafce.

<a id="folders"></a>

## Foldery

Foldery pomagają porządkować połączenia, na przykład według projektów lub środowisk. Foldery można zagnieżdżać.

- Aby utworzyć folder na najwyższym poziomie, należy kliknąć `+📁` ( **Nowy folder** ) w nagłówku panelu. Aby utworzyć podfolder, należy otworzyć menu folderu i wybrać polecenie **Nowy folder**. Następnie należy wpisać nazwę i nacisnąć `Enter` albo nacisnąć `Esc`, aby anulować.
- Aby otworzyć menu folderu, należy kliknąć `⋯` ( **Więcej działań** ) w wierszu folderu lub kliknąć folder prawym przyciskiem myszy. Menu zawiera polecenia **Zmień nazwę**, **Usuń** oraz polecenia eksportu i importu.
- Usunąć można tylko pusty folder. Najpierw trzeba przenieść lub usunąć jego połączenia i podfoldery.
- Nazwy folderów muszą być unikalne w obrębie jednego poziomu i mieć najwyżej 100 znaków.

Aby przenieść połączenie, należy przeciągnąć je na folder albo na inne połączenie, aby umieścić je przed nim. Aby przenieść je na najwyższy poziom, należy upuścić je na nagłówek panelu lub na puste miejsce pod listą. Folder można też wybrać w polu **Folder** formularza połączenia; wartość **(poziom główny)** oznacza najwyższy poziom. Foldery przeciąga się w ten sam sposób. Upuszczenie folderu na górną krawędź innego folderu umieszcza go przed tym folderem. Folderu nie można przenieść do jego własnego podfolderu.

Kliknięcie nazwy folderu zwija go lub rozwija. Aplikacja zapamiętuje, które foldery są zwinięte. Pole **Filtruj połączenia** nad drzewem pozwala wyszukać połączenia i foldery według nazwy. Pasujące foldery są wyświetlane z całą zawartością.

<a id="export"></a>

## Eksport

Eksport zapisuje połączenia i foldery do pliku, na przykład aby udostępnić je współpracownikom lub przenieść na inny komputer.

1. Aby wyeksportować wszystko, należy otworzyć menu `⋯` w nagłówku panelu i wybrać polecenie **Eksportuj wszystko…**. Aby wyeksportować jeden folder z podfolderami, należy wybrać polecenie **Eksportuj folder…** w jego menu.
2. Wybrać miejsce zapisu pliku. Proponowana nazwa to `glafira-connections-<date>.glafira-connections.json` lub `<folder>.glafira-connections.json`.

Po zapisaniu pliku panel pokazuje, ile połączeń i folderów on zawiera.

Plik zawiera ustawienia połączeń i strukturę folderów. Nie zawiera haseł ani właściwości poufnych. Nie są do niego dołączane również pliki certyfikatów: plik przechowuje tylko ich ścieżki. Hosty, nazwy użytkowników i ścieżki są zapisane otwartym tekstem, dlatego plik należy udostępniać tylko osobom, które mogą je znać.

<a id="import"></a>

## Import

1. Otworzyć menu `⋯` w nagłówku panelu i wybrać polecenie **Importuj…**. Aby zaimportować bezpośrednio do folderu, należy wybrać polecenie **Importuj tutaj…** w jego menu.
2. Wybrać plik `.glafira-connections.json`. Otwiera się okno **Import połączeń**.
3. W polu **Importuj do** wybrać, dokąd trafią nowe foldery i połączenia. Wartość **(poziom główny)** oznacza najwyższy poziom.
4. Pozostawić włączoną opcję **Aktualizuj istniejące połączenia**, aby zaktualizować istniejące połączenia danymi z pliku. Wyłączyć ją, aby pozostawić je bez zmian.
5. Sprawdzić podgląd i kliknąć przycisk **Importuj** albo przycisk **Anuluj**, aby zamknąć okno bez zmian.

Podgląd odświeża się po każdej zmianie w oknie. Pokazuje sumy i rozwijane listy: **Nowe**, **Zostaną zaktualizowane**, **Bez zmian** i **Pominięte** z przyczyną pominięcia każdego połączenia.

Jak aplikacja dopasowuje plik do istniejących połączeń:

- Połączenia są dopasowywane według wewnętrznego ID, a nie nazwy. Połączenie uznaje się za istniejące, jeśli plik zawiera to samo połączenie: na przykład zostało wyeksportowane z tego komputera lub wcześniej zaimportowane z tego samego źródła.
- Aktualizowane połączenie otrzymuje ustawienia z pliku. Hasła i jego miejsce w drzewie zostają zachowane.
- Nazwy istniejących folderów nie są zmieniane, a same foldery nie są przenoszone. Nowe połączenia z pliku i tak do nich trafiają.
- Jeśli nowe połączenie ma taką samą nazwę jak inne w tym samym folderze, dodawany jest numer, na przykład `kafka-dev (2)`.
- Połączenia, których ta wersja Glafiry nie obsługuje lub które zawierają sekrety w dodatkowych właściwościach, są pomijane.

Podgląd ostrzega też, gdy pliki certyfikatów nie zostaną znalezione na tym komputerze. Hasła nie są zapisane w pliku, więc po imporcie panel informuje, ile połączeń ich wymaga. Należy otworzyć każde z tych połączeń, wpisać hasła i zapisać.
