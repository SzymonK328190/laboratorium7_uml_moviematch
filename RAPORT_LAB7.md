# Laboratorium nr 7 — Modelowanie systemu MovieMatch za pomocą diagramów UML

**Przedmiot:** Inżynieria oprogramowania (2026–2027), UWM w Olsztynie  
**Projekt:** MovieMatch — aplikacja mobilna (mobile-first) do indywidualnego i grupowego dobierania filmów (zakres **MVP v1.0**)  
**Technologia diagramów:** PlantUML (źródła w [`puml/`](puml/), eksport w [`png/`](png/) oraz wektorowe [`pdf/`](pdf/))

---

## Spis rezultatów laboratorium

| Nr | Wymagany element (wg instrukcji Lab 7) | Pliki w repozytorium |
|---|---|---|
| 1 | **Model kontekstowy (Context Diagram)** — granice systemu i otoczenie | [`pdf/01_context_diagram.pdf`](pdf/01_context_diagram.pdf) / [`png/01_context_diagram.png`](png/01_context_diagram.png) |
| 2 | **Activity Diagram** — wybrany kluczowy proces z MVP | [`pdf/02_activity_diagram.pdf`](pdf/02_activity_diagram.pdf) / [`png/02_activity_diagram.png`](png/02_activity_diagram.png) |
| 3 | **Sequence Diagram** — interakcje w tym samym procesie | [`pdf/03_sequence_diagram.pdf`](pdf/03_sequence_diagram.pdf) / [`png/03_sequence_diagram.png`](png/03_sequence_diagram.png) |
| 4 | **State Diagram** — cykl życia wybranego obiektu (`Grupa`) | [`pdf/04_state_diagram.pdf`](pdf/04_state_diagram.pdf) / [`png/04_state_diagram.png`](png/04_state_diagram.png) |
| 5 | **Class Diagram** — podstawowa struktura produktu (6 klas, relacje, liczności) | [`pdf/05_class_diagram.pdf`](pdf/05_class_diagram.pdf) / [`png/05_class_diagram.png`](png/05_class_diagram.png) |
| 6 | **Weryfikacja spójności oraz pytania i decyzje do wyjaśnienia (PO)** | Sekcje 2, 3 i 4 niniejszego raportu |
| 7 | **Link do Product Backlogu z oznaczoną pierwszą funkcjonalnością** | Sekcja 5 niniejszego raportu |

---

## 1. Wybrany kluczowy scenariusz MVP i podsumowanie 5 modeli

Jako kluczowy scenariusz z zakresu **MVP v1.0** (wspólny dla diagramu czynności i diagramu przebiegu) wybrano rdzeń wyróżniający aplikację MovieMatch:  
**„Sesja grupowa i grupowe dobieranie filmu” (`FR-07`, `FR-06`, `FR-13`, `FR-10`, User Stories 4 i 5)** — z uwzględnieniem uwierzytelniania (`FR-02`, `FR-03`) oraz wymagań niefunkcjonalnych (algorytm wektorowy ML, HTTPS, czas odpowiedzi ≤ 3 s).

### 1.1. Model kontekstowy (Context Diagram) — [`01_context_diagram`](pdf/01_context_diagram.pdf)
- **Wewnątrz granicy systemu (Aplikacja mobilna + Backend, MVP v1.0):** zarządzanie kontami i hashowanie haseł (`FR-02`, `FR-03`), ankieta początkowa (`FR-01`), ocenianie filmów w skali `1.0–10.0` i ranking (`FR-04`, `FR-08`), silnik rekomendacji ML operujący na wektorach filmów i preferencji (`FR-10`, `FR-11`), obsługa sesji grupowych (tymczasowych i stałych), dołączanie linkiem oraz synchronizacja widoku pokoju (`FR-06`, `FR-07`, `FR-13`, `US-5 AC2`).
- **Aktorzy (poza granicą):** `Gość`, `Użytkownik`, `Uczestnik sesji grupowej` (specjalizacja `Użytkownika`), `Administrator grupy` (specjalizacja `Użytkownika`).
- **Otoczenie i systemy zewnętrzne (bez szczegółów API):**
  1. **Urządzenie mobilne (Smartfon / Tablet — iOS / Android)** — środowisko klienckie uruchamiające aplikację mobilną, obsługujące deep-linki i komunikację HTTPS (`NFR Kompatybilność`, `NFR Prywatność`, `NFR Obciążenie`).
  2. **Zewnętrzny komunikator (np. Messenger, SMS, WhatsApp)** — kanał poza aplikacją, przez który użytkownik samodzielnie udostępnia wygenerowany w MovieMatch link zaproszenia (`US-4 AC1`).
  3. **Zewnętrzna baza filmów (np. TMDB)** — źródło katalogu filmów, gatunków i metadanych zasilających wektory ML *(do potwierdzenia z Product Ownerem)*.
  4. **System kopii zapasowych (Backup Storage)** — zewnętrzny magazyn przyjmujący automatyczny, pełny backup bazy danych co najmniej raz na dobę (`NFR Niezawodność`).

### 1.2. Diagram czynności (Activity Diagram) — [`02_activity_diagram`](pdf/02_activity_diagram.pdf)
- Przedstawia przepływ procesu w 3 pasach odpowiedzialności (`Administrator grupy`, `System MovieMatch`, `Uczestnik sesji grupowej`) od zainicjowania grupy w aplikacji mobilnej (`FR-07`), przez wygenerowanie linku i jego ręczne przekazanie przez zewnętrzny komunikator (`US-4 AC1`), po zatwierdzenie tytułu, możliwość dobrania kolejnego filmu (np. maraton filmowy) i wygasanie grupy tymczasowej po krótkim czasie nieaktywności.
- Zawiera jawne odwołania do wymagań i kryteriów akceptacji (`FR-02`, `FR-03`, `FR-06`, `FR-07`, `FR-10`, `FR-13`, `US-4 AC1, AC3–AC4`, `US-5 AC1–AC3`, `NFR Algorytm`).
- Modeluje **3 ścieżki alternatywne oraz dwie pętle**:
  1. **Alt. 1 — Gość (użytkownik niezalogowany):** przekierowanie do logowania/rejestracji przed dołączeniem do pokoju (`FR-02`, `FR-03`).
  2. **Alt. 2 — Usunięcie uczestnika przez Administratora:** dezaktywacja członkostwa i pominięcie jego ocen oraz wektora preferencji w dobieraniu (`FR-13`, `US-4 AC3–AC4`).
  3. **Alt. 3 — Brak idealnego dopasowania wektorowego ML:** tryb awaryjny proponujący film o najwyższej średniej ocen wewnątrz grupy (`US-5 AC3`).
  4. **Pętla akceptacji i pętla maratonu:** możliwość odrzucenia zaproponowanego tytułu i wylosowania kolejnego, a po wyborze filmu — opcja ponownego dobierania kolejnego tytułu w tej samej sesji (zarówno dla grup stałych, jak i tymczasowych).

### 1.3. Diagram przebiegu (Sequence Diagram) — [`03_sequence_diagram`](pdf/03_sequence_diagram.pdf)
- Opisuje **ten sam scenariusz** z perspektywy chronologicznej wymiany komunikatów (`[01]`–`[55]`) pomiędzy 6 liniami życia: `Administrator grupy`, `Uczestnik (Użytkownik)`, `Aplikacja Mobilna (Klient HTTPS)`, `Serwis Grup (Grupa)`, `Silnik ML (PropozycjaFilmu)` oraz `Baza Danych`.
- Udostępnienie linku zaproszenia odbywa się bezpośrednio między aktorami poza aplikacją (np. przez Messenger/SMS — `[07]`), a dołączenie następuje po otwarciu deep-linku w aplikacji mobilnej (`[08]`).
- Zawiera 5 faz oraz fragmenty złożone UML:
  - `alt` (użytkownik zalogowany vs niezalogowany Gość — `401`),
  - `opt` (opcjonalne usunięcie członka grupy przez Administratora — `aktywne = false`),
  - `loop` (dobieranie do akceptacji propozycji, powtarzalne dla kolejnych filmów np. w maratonie),
  - `alt` wewnątrz silnika ML (idealne dopasowanie wektorowe `WEKTOR_ML` vs tryb awaryjny `SREDNIA_OCEN`),
  - `opt` zamykania grupy **tymczasowej** po upływie krótkiego czasu nieaktywności (`timeout`), z odświeżaniem `ostatniaAktywnoscO` przy każdej akcji w pokoju.

### 1.4. Diagram stanu (State Diagram) — [`04_state_diagram`](pdf/04_state_diagram.pdf)
- Modeluje cykl życia kluczowego obiektu dziedzinowego **`Grupa`** (sesja grupowa) z użyciem stanu złożonego `Aktywna Grupa` oraz 7 spójnych stanów:
  - `UTWORZONA` → `OCZEKUJACA` → `W_TRAKCIE_DOBIERANIA` ↔ `PROPOZYCJA_WYGENEROWANA` → `FILM_WYBRANY`,
  - przejście powrotne `FILM_WYBRANY` → `OCZEKUJACA` dla kolejnych dobierań (np. maraton filmowy) dostępne dla grup typu **`STALA`** oraz **`TYMCZASOWA`** (`FR-07`),
  - stan końcowy `ZAKONCZONA (Wygasła)` osiągalny dla grup typu **`TYMCZASOWA`** poprzez zdarzenie czasowe `after(czasNieaktywnosci)` (`FR-07`),
  - stan końcowy `USUNIETA (Anulowana)` osiągalny z dowolnego stanu aktywnego poprzez zdarzenie `usunGrupe() [Administrator]` (`FR-13`).

### 1.5. Diagram klas (Class Diagram) — [`05_class_diagram`](pdf/05_class_diagram.pdf)
- Zawiera dokładnie **6 kluczowych klas dziedzinowych** (zgodnie z zaleceniem 3–6 klas):
  1. `Uzytkownik` — konto, zahaszowane hasło, stan ankiety początkowej, ulubione gatunki, wektor preferencji ML, znacznik `usunietyO` (24h).
  2. `Film` — tytuł, gatunki, rok produkcji, wektor cech ML (`wektorCech`).
  3. `Ocena` — wartość w skali `1.0..10.0` z dziesiętnymi, daty dodania i modyfikacji (sortowanie i edycja/usuwanie z rankingu).
  4. `Grupa` — nazwa, typ (`TYMCZASOWA` / `STALA`), kod zaproszenia, status spójny 1:1 ze State Diagramem, `utworzonaO` oraz `ostatniaAktywnoscO` (do kontroli wygasania grupy tymczasowej).
  5. `CzlonkostwoGrupy` — rola (`ADMINISTRATOR` / `UCZESTNIK`), data dołączenia, flaga `aktywne` (wykluczanie preferencji usuniętych członków).
  6. `PropozycjaFilmu` — tryb (`INDYWIDUALNY` / `GRUPOWY`), wynik dopasowania, metoda (`WEKTOR_ML` / `SREDNIA_OCEN`), status propozycji.
- Zawiera pełne liczności (`1`, `0..1`, `0..*`, `1..*`), kompozycję (`Grupa *-- CzlonkostwoGrupy`), agregację (`Grupa o-- PropozycjaFilmu`) oraz ograniczenia `{xor}` i unikalności pary `(Uzytkownik, Film)`.

---

## 2. Weryfikacja spójności modeli z wymaganiami (Matryca śledzenia)

| Wymaganie / NFR | Model kontekstowy | Activity | Sequence | State (`Grupa`) | Class Diagram |
|---|:---:|:---:|:---:|:---:|:---:|
| **FR-01** Początkowa ankieta (v1.0 Must*) | Granica systemu, relacja `Użytkownik` | — *(poza wybranym scenariuszem)* | — | — | `Uzytkownik.ankietaWypelniona`, `ulubioneGatunki`, `wypelnijAnkiete()` |
| **FR-02** Rejestracja konta | Aktor `Gość` → System | Alt. 1 (przekierowanie Gościa) | `alt` Gość → logowanie/rejestracja | — | `Uzytkownik.email {unique}`, `hasloHash` |
| **FR-03** Logowanie użytkownika | Aktor `Użytkownik` → System | Weryfikacja zalogowania | Weryfikacja `tokenSesji`, `401` | Guard `[zalogowany]` przy `dolaczDoGrupy()` | `Uzytkownik.email`, `hasloHash` |
| **FR-04** Ocenianie filmów (`1.0–10.0`) | Granica systemu, przepływ ocen | Pobranie ocen do ML | Pobranie ocen z DB | Wykorzystanie w `W_TRAKCIE_DOBIERANIA` | `Ocena.wartosc {1.0..10.0}`, `Uzytkownik.wystawOcene()` |
| **FR-06** Dołączanie do grupy (1 kliknięcie) | `Użytkownik`, `Zewnętrzny komunikator` | Otwarcie linku zaproszenia i dodanie do pokoju | Faza 1 (`[07]`) i Faza 2 (`[08]`–`[16]`) | `OCZEKUJACA`: `dolaczDoGrupy()` | `Grupa.dodajCzlonka()`, `CzlonkostwoGrupy` |
| **FR-07** Tworzenie grupy (tymczasowa / stała) | `Administrator` → System | Start + maraton + wygasanie tymczasowej po nieaktywności | Faza 1 (`[01]`–`[06]`) i Faza 5 (`opt` timeout) | `UTWORZONA`, maraton `[STALA/TYMCZASOWA]`, `after(czasNieaktywnosci)` → `ZAKONCZONA` | `Grupa.typ`, `kodZaproszenia`, `ostatniaAktywnoscO` |
| **FR-08** Ranking filmów (sortowanie, edycja, usuwanie) | Granica systemu, `Użytkownik` | — *(poza wybranym scenariuszem)* | — | — | `Ocena.dataDodania`, `edytujOcene()`, `usunOcene()` |
| **FR-10** Dobieranie grupowo filmu (`US-5 AC1–AC3`) | `Uczestnik` ↔ System | Pętla ML + Alt. 3 (średnia) + synchronizacja | Faza 4 (`[23]`–`[42]`) i Faza 5 | `W_TRAKCIE_DOBIERANIA` → `PROPOZYCJA_WYGENEROWANA` → `FILM_WYBRANY` | `PropozycjaFilmu` (`GRUPOWY`, `WEKTOR_ML`/`SREDNIA_OCEN`), `Grupa.dobierzFilmGrupowo()` |
| **FR-11** Dobieranie indywidualnie filmu | `Użytkownik` ↔ System | — *(poza wybranym scenariuszem)* | — | — | `PropozycjaFilmu` (`INDYWIDUALNY`), `Uzytkownik.dobierzFilmIndywidualnie()` |
| **FR-13** Zarządzanie grupą (`US-4 AC3–AC4`) | `Administrator` ↔ System | Alt. 2: wyrzucenie i pominięcie ocen | Faza 3 `opt` (`[17]`–`[22]`) + filtr `aktywne=true` | `usunCzlonka()`, `zmienNazwe()`, przejście `usunGrupe()` → `USUNIETA` | `Grupa.usunCzlonka()`, `zmienNazwe()`, `usunGrupe()`, `CzlonkostwoGrupy.aktywne` |
| **NFR** Kompatybilność i Prywatność (HTTPS, 24h) | `Urządzenie mobilne` (HTTPS) | Aplikacja mobilna | `Aplikacja Mobilna (Klient HTTPS)`, `[HTTPS]` | — | `Uzytkownik.usunietyO [0..1]` |
| **NFR** Bezpieczeństwo (hashowanie haseł) | Zakres wewnątrz granicy systemu | — | — | — | `Uzytkownik.hasloHash` |
| **NFR** Obciążenie (≤ 3 s, do 10 użytkowników) | Etykieta kanału `Urządzenie mobilne` ↔ System | — | Komunikat `[40]` (≤ 3 s) | — | — |
| **NFR** Niezawodność (backup ≥ 1×/dobę) | `System kopii zapasowych` (Backup Storage) | — | — | — | — |
| **NFR** Algorytm ML (wektory filmów i preferencji) | Silnik ML w granicy + `Zewnętrzna baza filmów` | Obliczenie wspólnego profilu ML | Linia życia `Silnik ML` (`[27]`–`[37]`) | Aktywność `do /` w `W_TRAKCIE_DOBIERANIA` | `Film.wektorCech`, `Uzytkownik.wektorPreferencji` |

---

## 3. Wykryte niespójności i rekomendowane poprawki w dokumentacji wymagań (Lab 2–6)

Poniżej zestawiono rozbieżności wykryte podczas modelowania oraz uwagi zgłoszone przez zespół do samodzielnego naniesienia w dokumencie wymagań:

1. **Platforma docelowa — aplikacja mobilna zamiast webowej (`NFR Kompatybilność`, str. 3):**
   - Obecnie w NFR widnieje zapis: *„Interfejs aplikacji webowej musi być w pełni responsywny i poprawnie skalować się na urządzeniach mobilnych (smartfony, tablety)”*, podczas gdy docelowo produkt ma być **w pierwszej kolejności aplikacją mobilną** (Android/iOS). Należy poprawić NFR Kompatybilność (wskazać wspierane systemy mobilne i wersje oraz ewentualnie, czy wersja webowa jest opcjonalna/poza MVP).
2. **Sposób zapraszania do grupy — link przez zewnętrzny komunikator vs powiadomienia w aplikacji (`FR-06`, `US-4 AC1–AC2`, str. 1 i 4):**
   - `US-4 AC2` mówi o *„powiadomieniu dla zaproszonych znajomych umożliwiającym dołączenie jednym kliknięciem”*, a `FR-06` wspomina o *„grupie znajomych”*, mimo że system znajomych (`FR-05`) jest dopiero w `v1.2 (Could)`. W MVP v1.0 aplikacja nie wysyła powiadomień — generuje link zaproszenia, który użytkownik wysyła np. przez Messengera/SMS, a kliknięcie linku otwiera aplikację mobilną (deep-link). Należy rozdzielić w `US-4` zachowanie dla MVP (udostępnianie linku) od późniejszych powiadomień push dla znajomych (`v1.2`).
3. **Brak definicji cyklu życia grupy tymczasowej i stałej (`FR-07`, str. 1):**
   - W opisie `FR-07` zapisano jedynie *„Tymczasowej lub stałej”*, nie precyzując różnicy. Należy dopisać, że grupa **tymczasowa** służy również do maratonów filmowych (można dobrać wiele filmów w jednej sesji) i zamyka się automatycznie po **krótkim czasie nieaktywności** (oraz podać konkretną wartość timeoutu, np. 30–60 min), natomiast grupa **stała** pozostaje otwarta do momentu usunięcia przez Administratora (`FR-13`).
4. **Sprzeczny status `FR-01` (Wypełnianie początkowej ankiety) w obrębie dokumentu (str. 1–2 i 7–8):**
   - Tabela wymagań (str. 1) i dziennik zmian pkt 2 (str. 8) wskazują **`Must / v1.0`**, jednak brakuje `FR-01` na liście MUST w „Wyjaśnieniu MoSCoW” (str. 2–3), a sekcja 6 (str. 8) wciąż informuje, że w wersji 1.0 pominięto *„wypełnianie początkowej ankiety w wersji 1.1”*.
5. **NFR „Prywatność” (usunięcie konta z 24-godzinnym oknem na cofnięcie) bez odpowiadającego FR ani User Story (str. 3):**
   - Jeśli funkcja ma wejść do MVP v1.0, trzeba dopisać brakujące FR i User Story; jeśli nie — wyraźnie oznaczyć ją jako etap późniejszy.
6. **Literówka w numerze wymagania w dzienniku zmian (str. 8, pkt 3):**
   - Zapisano *„Zmiana FR-11, dodanie opisu grup Tymczasowe i stałe”*, podczas gdy zmiana dotyczy **`FR-07` (Tworzenie grupy)** — `FR-11` to dobieranie indywidualne.
7. **Niespójne nazewnictwo w `US-3` oraz zakres backupu w NFR „Niezawodność” (str. 3 i 4):**
   - W `US-3` pozostało słowo *„po loginie”* mimo zmiany w `FR-05` na *„nazwę użytkownika / kod znajomego”*.
   - W NFR „Niezawodność” wymieniono jedynie *„oceny użytkowników i listy znajomych”* (które wejdą dopiero w `v1.2`), pomijając konta, wektory preferencji oraz grupy z MVP v1.0.

---

## 4. Pytania i decyzje biznesowe do wyjaśnienia z Product Ownerem

| ID | Obszar / Model | Pytanie do Product Ownera | Wpływ na implementację i modele |
|---|---|---|---|
| **Q-01** | Context, Class | **Skąd pochodzi katalog filmów i ich metadane?** Czy integrujemy zewnętrzne API (np. TMDB), czy korzystamy z własnego statycznego zbioru danych przygotowanego dla modelu ML? | Zależność zewnętrzna w Modelu kontekstowym oraz zasilanie `Film.wektorCech`. |
| **Q-02** | Activity, Sequence (`FR-10`, `US-5 AC3`) | **Jak definiujemy „idealne dopasowanie” w dobieraniu grupowym?** Jaki próg podobieństwa wektorów (np. cosine similarity ≥ T) decyduje o wyborze ścieżki głównej ML zamiast trybu awaryjnego (najwyższa średnia ocen w grupie)? | Implementacja warunku rozgałęzienia `alt` w `Silniku ML`. |
| **Q-03** | Activity, Sequence (`FR-10` vs `FR-11`) | **Czy w dobieraniu grupowym wykluczamy filmy już ocenione przez członków grupy?** W `FR-11` (`US-9 AC3`) filmy ocenione przez użytkownika są wykluczane; czy w `FR-10` tryb główny ML ma pomijać filmy obejrzane przez wszystkich, przez kogokolwiek, czy ich nie wykluczać? | Logika filtrowania kandydatów w `wyznaczRekomendacjeGrupowa()`. |
| **Q-04** | Activity, Sequence, State (`FR-10`) | **Kto w grupie może uruchomić losowanie, odrzucić propozycję i zaakceptować wybrany film?** Każdy uczestnik sesji grupowej, wyłącznie Administrator grupy, czy wymagana jest zgoda/głosowanie wszystkich? | Kontrola uprawnień w `Serwisie Grup` oraz strażniki przejść w `State Diagram`. |
| **Q-05** | State, Sequence, Class (`FR-07`, `FR-13`) | **Jakie są dokładne progi cyklu życia grupy?** (a) Ile dokładnie wynosi krótki czas nieaktywności (`timeout`) zamykający grupę tymczasową (np. 30 min, 1 h, 3 h)? (b) Czy grupowe dobieranie wymaga min. 2 aktywnych uczestników? (c) Co dzieje się z grupą stałą po odejściu Administratora? | Parametr `timeout` (`ostatniaAktywnoscO`), strażnik `[aktywni >= 2]` i obsługa opuszczenia grupy. |
| **Q-06** | Class, `FR-01` (`US-2 AC2`) | **Jaka jest minimalna wymagana liczba gatunków w początkowej ankiecie?** Dokument mówi o „minimalnej wymaganej liczbie ulubionych gatunków”, bez podania konkretnej liczby (np. 3). | Walidacja formularza ankiety początkowej i inicjalizacja `Uzytkownik.wektorPreferencji`. |
| **Q-07** | Class, `FR-08` (`US-8 AC3`) | **Czy edycja lub całkowite usunięcie oceny w rankingu natychmiast przelicza wektor preferencji użytkownika?** (`US-7 AC3` precyzuje to tylko dla wystawienia oceny). | Wywołanie `zaktualizujWektorPreferencji()` po `Ocena.edytujOcene()` i `Ocena.usunOcene()`. |
| **Q-08** | NFR Prywatność, Class | **Czy funkcja usuwania konta (z 24-godzinnym oknem na cofnięcie) wchodzi w zakres MVP v1.0?** Jeśli tak — należy dopisać odpowiednie FR i User Story do backlogu. | Zakres sprintów MVP v1.0 i harmonogram trwałego usuwania danych. |
| **Q-09** | Sequence (`US-5 AC2`), NFR Obciążenie | **Jaki mechanizm synchronizacji pokoju grupowego w aplikacji mobilnej wybieramy?** Przy maks. 10 jednoczesnych użytkownikach i wymogu ≤ 3 s: WebSocket/SSE (natychmiastowy push) czy cykliczny polling? | Architektura komunikacji w fazie 4 `Sequence Diagram`. |

---

## 5. Link do Product Backlogu i oznaczenie pierwszej funkcjonalności

- **Link do Product Backlogu (Jira — z Lab 6):**  
  > `[WKLEJ TUTAJ LINK DO TABLICY / PRODUCT BACKLOGU PROJEKTU MOVIEMATCH W JIRA]`
  *(Przed wysłaniem na Teams wklej bezpośredni adres URL do Product Backlogu zespołu z Lab 6 oraz upewnij się, że poniższa pierwsza historyjka jest oznaczona w backlogu etykietą np. `MVP-FIRST` / umieszczona na samej górze Sprintu 1).*

### Oznaczona pierwsza funkcjonalność do implementacji (Start MVP)

Na podstawie analizy zależności wynikających z opracowanych diagramów UML (ścieżka alternatywna 1 w `Activity Diagram` i blok `alt` w `Sequence Diagram` pokazują, że brak uwierzytelnionego użytkownika blokuje zarówno ocenianie filmów, jak i dołączanie do sesji grupowych), jako **pierwszą funkcjonalność** w Product Backlogu oznaczamy:

1. **Pierwsza funkcjonalność startowa (fundament odblokowujący cały MVP):**  
   **`FR-02` Rejestracja konta + `FR-03` Logowanie użytkownika** (`US-1` i `US-6`, priorytet `Must`, wersja `1.0`, niskie ryzyko).
   - **Uzasadnienie:** Wszystkie pozostałe wymagania MVP (`FR-01`, `FR-04`, `FR-06`, `FR-07`, `FR-08`, `FR-10`, `FR-11`, `FR-13`) wymagają uwierzytelnionego obiektu `Uzytkownik` z zahaszowanym hasłem i przypisanym profilem preferencji.
2. **Kolejny priorytetowy przyrost (rdzeń biznesowy zamodelowany w Lab 7):**  
   Po dostarczeniu uwierzytelniania oraz ankiety/ocen (`FR-01`, `FR-04`), kolejnym kluczowym przyrostem jest zamodelowany scenariusz **`FR-07` Tworzenie grupy → `FR-06` Dołączanie do grupy → `FR-10` Dobieranie grupowo filmu**.
