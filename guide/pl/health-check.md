# Sprawdzanie klastra i notatki

Glafira sprawdza klaster zestawem wbudowanych reguł i pokazuje znalezione problemy obok obiektów: w drzewie połączeń, na listach topików i grup oraz na ekranach topiku i grupy. Asystent LLM może uzupełnić te ustalenia własnymi notatkami. Sprawdzenie tylko odczytuje klaster i niczego w nim nie zmienia.

<a id="run"></a>

## Uruchamianie sprawdzenia

Należy otworzyć **Przegląd klastra** lub **Ustalenia** połączenia i kliknąć przycisk **Sprawdź klaster**. Sprawdzenie trwa do 45 sekund: Glafira odczytuje brokery, partycje, konfiguracje topików (do 2000 topików) i grupy konsumentów (do 500). Jeśli topików lub grup jest więcej, czegoś nie udało się odczytać albo upłynęło 45 sekund, wynik jest oznaczony jako **sprawdzono częściowo**, a ustalenie **Niepełne sprawdzenie** pokazuje, czego nie odczytano.

Glafira sama nie odpytuje klastra: sprawdzenie uruchamia się tylko po kliknięciu przycisku albo na prośbę asystenta LLM. Wynik jest przechowywany do następnego sprawdzenia.

<a id="rules"></a>

## Co jest sprawdzane

- Brokery i partycje: brak kontrolera, niedostępne i niedoreplikowane partycje, klaster z jednym brokerem, nierówny rozkład leaderów, leaderzy poza preferowaną repliką.
- Trwałość topików: współczynnik replikacji 1, `min.insync.replicas` większe niż współczynnik replikacji, `min.insync.replicas` równe współczynnikowi replikacji, `min.insync.replicas` równe 1, nieczysty wybór leadera.
- Konfiguracja brokerów: automatyczne tworzenie topików, domyślny współczynnik replikacji 1, domyślne `min.insync.replicas` 1, ważne ustawienia, które różnią się między brokerami.
- Grupy konsumentów: puste grupy z lagiem, grupy w trakcie rebalansu, członkowie bez partycji, lag, który wzrósł od poprzedniego sprawdzenia.

<a id="severity"></a>

## Waga i środowisko

Każde ustalenie ma wagę: **Krytyczne**, **Ostrzeżenie** lub **Informacja**. Zależy ona od środowiska połączenia. Na przykład współczynnik replikacji 1 na prod jest krytyczny, na stage jest ostrzeżeniem, a na dev, test i local nie jest pokazywany. Środowisko ustawia się w formularzu połączenia.

<a id="tab"></a>

## Karta „Ustalenia”

Na karcie **Ustalenia** ustalenia są pogrupowane według reguł; każde wyjaśnia problem i co zrobić. Można je filtrować według kryteriów **Waga**, **Źródło** i **Obiekt** albo wyszukiwać po nazwie. Nazwa obiektu otwiera jego ekran.

Znaczek z liczbą w drzewie połączeń i na listach topików i grup pokazuje, ile ustaleń i otwartych notatek LLM o wadze **Krytyczne** lub **Ostrzeżenie** dotyczy obiektu. Po najechaniu na znaczek widać ich tytuły.

<a id="hide"></a>

## Ukrywanie ustaleń

Jeśli ustalenie jest oczekiwane dla konkretnego obiektu, na przykład testowego topiku ze współczynnikiem replikacji 1 na prod, należy kliknąć przycisk **Ukryj dla tego obiektu**. Ukryte ustalenie nie jest liczone w znaczkach i pozostaje ukryte przy kolejnych sprawdzeniach. Aby zobaczyć ukryte, należy zaznaczyć opcję „Pokaż ukryte”; tam jest też przycisk **Przywróć**.

<a id="notes"></a>

## Notatki LLM

Asystent LLM połączony przez MCP (zob. [Asystenci LLM (MCP)](llm-mcp.md#how-it-works)) może zostawić notatkę o klastrze, brokerze, topiku lub grupie. Notatki pojawiają się w sekcji **Notatki LLM** i na ekranach obiektów, podpisane nazwą klienta. Tekst jest pokazywany bez zmian, bez formatowania i linków.

- **Rozwiązane** i **Nieaktualne** zamykają notatkę; zamknięte są przechowywane w zwiniętej sekcji.
- **Usuń** usuwa notatkę, a **Usuń wszystkie notatki LLM** usuwa wszystkie notatki połączenia.
- Notatka jest usuwana automatycznie 7 dni po utworzeniu. Połączenie może mieć najwyżej 200 otwartych notatek.

<a id="with-llm"></a>

## Razem z asystentem LLM

Asystent uruchamia to samo sprawdzenie narzędziem `health_check` i widzi te same ustalenia, w tym ukryte, oznaczone jako ukryte. Jeśli ostatnie sprawdzenie ma mniej niż 5 minut, asystent dostaje jego wynik bez ponownego odpytywania klastra. Można na przykład poprosić: „Sprawdź stan klastra i zostaw notatki o najważniejszych problemach”.
