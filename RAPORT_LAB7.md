# Laboratorium nr 7 — Modelowanie systemu MovieMatch za pomocą diagramów UML

**Przedmiot:** Inżynieria oprogramowania (2026–2027), UWM w Olsztynie  
**Projekt:** MovieMatch — responsywna aplikacja webowa (*mobile-first approach*) do indywidualnego i grupowego dobierania filmów (zakres **MVP v1.0**)  
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
**„Sesja grupowa i grupowe dobieranie filmu” (`FR-07`, `FR-06`, `FR-13`, `FR-10`, User Stories 4 i 5)** — z uwzględnieniem uwierzytelniania (`FR-02`, `FR-03`) oraz wymagań niefunkcjonalnych (responsywna aplikacja webowa *mobile-first*, algorytm wektorowy ML, HTTPS, czas odpowiedzi ≤ 3 s).

### 1.1. Model kontekstowy (Context Diagram) — [`01_context_diagram`](pdf/01_context_diagram.pdf)
- **Wewnątrz granicy systemu (Aplikacja webowa *mobile-first* + Backend, MVP v1.0):** zarządzanie kontami (e-mail, nazwa użytkownika, zahaszowane hasło — `FR-02`, `FR-03`), ankieta początkowa (`FR-01`), ocenianie filmów w skali `1.0–10.0` i ranking (`FR-04`, `FR-08`), silnik rekomendacji ML operujący na wektorach filmów i preferencji (`FR-10`, `FR-11`), obsługa sesji grupowych (tymczasowych i stałych), dołączanie linkiem oraz synchronizacja widoku pokoju (`FR-06`, `FR-07`, `FR-13`, `US-5 AC2`).
- **Aktorzy (poza granicą):** `Gość`, `Użytkownik`, `Uczestnik sesji grupowej` (specjalizacja `Użytkownika`), `Administrator grupy` (specjalizacja `Użytkownika`).
- **Otoczenie i systemy zewnętrzne (bez szczegółów API):**
  1. **Przeglądarka WWW (Smartfon / Tablet / Desktop)** — środowisko klienckie uruchamiające responsywny interfejs webowy (*mobile-first*) przez HTTPS (`NFR Kompatybilność`, `NFR Prywatność`, `NFR Obciążenie`).
  2. **Zewnętrzny komunikator (np. Messenger, SMS, WhatsApp)** — kanał poza aplikacją, przez który użytkownik samodzielnie udostępnia wygenerowany w MovieMatch link URL zaproszenia (`US-4 AC1`).
  3. **TMDB (The Movie Database)** — potwierdzone źródło katalogu filmów (~8 244 tytuły), gatunków, obsady, słów kluczowych, czasu trwania i plakatów zasilających aplikację i wektory ML.
  4. **System kopii zapasowych (Backup Storage)** — zewnętrzny magazyn przyjmujący automatyczny, pełny backup bazy danych co najmniej raz na dobę (`NFR Niezawodność`).

### 1.2. Diagram czynności (Activity Diagram) — [`02_activity_diagram`](pdf/02_activity_diagram.pdf)
- Przedstawia przepływ procesu w 3 pasach odpowiedzialności (`Administrator grupy`, `System MovieMatch`, `Uczestnik sesji grupowej`) od zainicjowania grupy w aplikacji webowej (`FR-07`), przez wygenerowanie linku i jego ręczne przekazanie przez zewnętrzny komunikator (`US-4 AC1`), otwarcie linku w przeglądarce (`FR-06`), po zatwierdzenie tytułu, możliwość dobrania kolejnego filmu (np. maraton filmowy) i automatyczne usunięcie grupy tymczasowej po **6 godzinach nieaktywności** (`FR-07`, `US-4 AC5`).
- Zawiera jawne odwołania do wymagań i kryteriów akceptacji (`FR-02`, `FR-03`, `FR-06`, `FR-07`, `FR-10`, `FR-13`, `US-4 AC1, AC3–AC5`, `US-5 AC1–AC3`, `NFR Algorytm`).
- Modeluje **3 ścieżki alternatywne oraz dwie pętle**:
  1. **Alt. 1 — Gość (użytkownik niezalogowany):** przekierowanie do logowania/rejestracji przed dołączeniem do pokoju (`FR-02`, `FR-03`).
  2. **Alt. 2 — Usunięcie uczestnika przez Administratora:** dezaktywacja członkostwa i pominięcie jego ocen oraz wektora preferencji w dobieraniu (`FR-13`, `US-4 AC3–AC4`).
  3. **Alt. 3 — Brak idealnego dopasowania wektorowego ML:** tryb awaryjny proponujący film o najwyższej średniej ocen wewnątrz grupy (`US-5 AC3`).
  4. **Pętla akceptacji i pętla maratonu:** możliwość odrzucenia zaproponowanego tytułu i wylosowania kolejnego, a po wyborze filmu — opcja ponownego dobierania kolejnego tytułu w tej samej sesji (zarówno dla grup stałych, jak i tymczasowych).

### 1.3. Diagram przebiegu (Sequence Diagram) — [`03_sequence_diagram`](pdf/03_sequence_diagram.pdf)
- Opisuje **ten sam scenariusz** z perspektywy chronologicznej wymiany komunikatów pomiędzy 6 liniami życia zgodnymi z architekturą systemu: `Administrator grupy`, `Uczestnik (Użytkownik)`, `Aplikacja Web (HTML/JS, HTTPS)`, `Backend PHP (Serwis Grup)`, `Mikroserwis ML (Python FastAPI)` oraz `Baza MariaDB`.
- Udostępnienie linku zaproszenia odbywa się bezpośrednio między aktorami poza aplikacją (np. przez Messenger/SMS), dołączenie korzysta z sesji PHP (`HttpOnly`, `SameSite=Lax`), a synchronizacja widoku pokoju grupowego realizowana jest przez **polling JS** (`fetch` co 1–2 s).
- Zawiera 5 faz oraz fragmenty złożone UML (`alt`, `opt`, `loop`), w tym blok `opt` automatycznego zamykania/usuwania grupy **tymczasowej** po upływie **6 godzin nieaktywności** przez zadanie `Cron` (`FR-07`, `US-4 AC5`).

### 1.4. Diagram stanu (State Diagram) — [`04_state_diagram`](pdf/04_state_diagram.pdf)
- Modeluje cykl życia kluczowego obiektu dziedzinowego **`Grupa`** (sesja grupowa) z użyciem stanu złożonego `Aktywna Grupa` oraz 7 spójnych stanów:
  - `UTWORZONA` → `OCZEKUJACA` → `W_TRAKCIE_DOBIERANIA` ↔ `PROPOZYCJA_WYGENEROWANA` → `FILM_WYBRANY`,
  - przejście powrotne `FILM_WYBRANY` → `OCZEKUJACA` dla kolejnych dobierań (np. maraton filmowy) dostępne dla grup typu **`STALA`** oraz **`TYMCZASOWA`** (`FR-07`),
  - stan końcowy `ZAKONCZONA (Wygasła)` osiągalny dla grup typu **`TYMCZASOWA`** poprzez zdarzenie czasowe `after(6h nieaktywności)` (`FR-07`, `US-4 AC5`),
  - stan końcowy `USUNIETA (Anulowana)` osiągalny z dowolnego stanu aktywnego poprzez zdarzenie `usunGrupe() [Administrator]` (`FR-13`).

### 1.5. Diagram klas (Class Diagram) — [`05_class_diagram`](pdf/05_class_diagram.pdf)
- Zawiera dokładnie **6 kluczowych klas dziedzinowych** (klucze główne typu `Integer` spójne ze schematem MariaDB `AUTO_INCREMENT`):
  1. `Uzytkownik` — konto (`email {unique}`, `nazwaUzytkownika {unique}`, `hasloHash`, `aktywny`), stan ankiety początkowej, ulubione gatunki, wektor preferencji ML, znacznik `usunietyO` (24h).
  2. `Film` — `tmdbId {unique}`, tytuł, opis, rok produkcji, czas trwania (`runtime`), reżyser, gatunki, obsada, URL plakatu, statystyki TMDB (`sredniaTmdb`, `liczbaGlosow`) i wektor cech ML (`wektorCech`).
  3. `Ocena` — wartość `Decimal(3,1)` w skali `1.0..10.0` (krok `0.1`), daty dodania i modyfikacji (sortowanie i edycja/usuwanie z rankingu).
  4. `Grupa` — nazwa, typ (`TYMCZASOWA` / `STALA`), kod zaproszenia, status spójny 1:1 ze State Diagramem, `utworzonaO` oraz `ostatniaAktywnoscO` (kontrola 6-godzinnego wygasania grupy tymczasowej).
  5. `CzlonkostwoGrupy` — rola (`ADMINISTRATOR` / `UCZESTNIK`), data dołączenia, flaga `aktywne` (wykluczanie preferencji usuniętych członków).
  6. `PropozycjaFilmu` — tryb (`INDYWIDUALNY` / `GRUPOWY`), wynik dopasowania, metoda (`WEKTOR_ML` / `SREDNIA_OCEN`), status propozycji.

---

## 2. Weryfikacja spójności modeli z wymaganiami (Matryca śledzenia)

| Wymaganie / NFR | Model kontekstowy | Activity | Sequence | State (`Grupa`) | Class Diagram |
|---|:---:|:---:|:---:|:---:|:---:|
| **FR-01** Początkowa ankieta (`Must v1.0`) | Granica systemu, relacja `Użytkownik` | — *(poza wybranym scenariuszem)* | — | — | `Uzytkownik.ankietaWypelniona`, `ulubioneGatunki`, `wypelnijAnkiete()` |
| **FR-02** Rejestracja (e-mail, nazwa użytkownika, hasło) | Aktor `Gość` → System | Alt. 1 (przekierowanie Gościa) | `alt` Gość → logowanie/rejestracja | — | `Uzytkownik.email {unique}`, `nazwaUzytkownika {unique}`, `hasloHash` |
| **FR-03** Logowanie użytkownika | Aktor `Użytkownik` → System | Weryfikacja zalogowania | Weryfikacja sesji PHP, `401` | Guard `[zalogowany]` przy `dolaczDoGrupy()` | `Uzytkownik.email`, `nazwaUzytkownika`, `hasloHash`, `aktywny` |
| **FR-04** Ocenianie filmów (`1.0–10.0`) | Granica systemu, katalog (Gość) i oceny (Użytkownik) | Pobranie ocen do ML | Pobranie ocen z MariaDB | Wykorzystanie w `W_TRAKCIE_DOBIERANIA` | `Ocena.wartosc {1.0..10.0, krok 0.1}`, `Uzytkownik.wystawOcene()` |
| **FR-06** Dołączanie do grupy (1 kliknięcie) | `Użytkownik`, `Zewnętrzny komunikator` | Otwarcie linku w przeglądarce i dodanie do pokoju | Faza 1 i Faza 2 | `OCZEKUJACA`: `dolaczDoGrupy()` | `Grupa.dodajCzlonka()`, `CzlonkostwoGrupy` |
| **FR-07** Tworzenie grupy (tymczasowa 6h / stała) | `Administrator` → System | Start + maraton + automatyczne usunięcie po 6h nieaktywności | Faza 1 i Faza 5 (`opt 6h` Cron) | `UTWORZONA`, maraton, `after(6h nieaktywności)` → `ZAKONCZONA` | `Grupa.typ`, `kodZaproszenia`, `ostatniaAktywnoscO` |
| **FR-08** Ranking filmów (sortowanie, edycja, usuwanie) | Granica systemu, `Użytkownik` | — *(poza wybranym scenariuszem)* | — | — | `Ocena.dataDodania`, `edytujOcene()`, `usunOcene()` |
| **FR-10** Dobieranie grupowo filmu (`US-5 AC1–AC3`) | `Uczestnik` ↔ System | Pętla ML + Alt. 3 (średnia) + synchronizacja | Faza 4 i Faza 5 (polling JS) | `W_TRAKCIE_DOBIERANIA` → `PROPOZYCJA_WYGENEROWANA` → `FILM_WYBRANY` | `PropozycjaFilmu` (`GRUPOWY`, `WEKTOR_ML`/`SREDNIA_OCEN`), `Grupa.dobierzFilmGrupowo()` |
| **FR-11** Dobieranie indywidualnie filmu | `Użytkownik` ↔ System | — *(poza wybranym scenariuszem)* | — | — | `PropozycjaFilmu` (`INDYWIDUALNY`), `Uzytkownik.dobierzFilmIndywidualnie()` |
| **FR-13** Zarządzanie grupą (`US-4 AC3–AC4`) | `Administrator` ↔ System | Alt. 2: wyrzucenie i pominięcie ocen | Faza 3 `opt` + filtr `aktywne=true` | `usunCzlonka()`, `zmienNazwe()`, przejście `usunGrupe()` → `USUNIETA` | `Grupa.usunCzlonka()`, `zmienNazwe()`, `usunGrupe()`, `CzlonkostwoGrupy.aktywne` |
| **NFR** Kompatybilność (*web mobile-first*) i Prywatność (HTTPS, 24h) | `Przeglądarka WWW` (*mobile-first*, HTTPS) | Aplikacja webowa | `Aplikacja Web (HTML/JS, HTTPS)` | — | `Uzytkownik.usunietyO [0..1]` |
| **NFR** Bezpieczeństwo (hashowanie haseł) | Zakres wewnątrz granicy systemu | — | — | — | `Uzytkownik.hasloHash` |
| **NFR** Obciążenie (≤ 3 s, do 10 użytkowników) | Etykieta kanału `Przeglądarka WWW` ↔ System | — | Synchronizacja (≤ 3 s) | — | — |
| **NFR** Niezawodność (backup ≥ 1×/dobę) | `System kopii zapasowych` (Backup Storage) | — | — | — | — |
| **NFR** Algorytm ML (wektory filmów i preferencji) | Silnik ML w granicy + `TMDB` | Obliczenie wspólnego profilu ML | Linia życia `Mikroserwis ML (Python FastAPI)` | Aktywność `do /` w `W_TRAKCIE_DOBIERANIA` | `Film.wektorCech`, `Uzytkownik.wektorPreferencji` |

---

## 3. Sprawdzenie spójności zaktualizowanego dokumentu wymagań (pozostałe uwagi)

W nowej wersji dokumentu poprawnie ujednolicono `FR-01` (dodano do MUST i usunięto z listy pominiętych), doprecyzowano formę aplikacji (*webowa, mobile-first*), dodano `nazwę użytkownika` do `FR-02` oraz określono `6 godzin` nieaktywności grupy tymczasowej (`FR-07`, `US-4 AC5`). Pozostają następujące drobne niespójności w samym dokumencie:

1. **`FR-02` vs `US-1` (str. 1 i 4):** W `FR-02` dopisano rejestrację przy użyciu *adresu e-mail, **nazwy użytkownika** i hasła*, natomiast `US-1` nadal wymienia tylko *e-mail i hasło* (brakuje też kryterium akceptacji dotyczącego walidacji / unikalności nazwy użytkownika).
2. **`FR-06` i `US-4 AC1–AC2` (str. 1 i 4):** `US-4 AC2` nadal mówi o *„powiadomieniu dla zaproszonych znajomych”*, a `FR-06` o *„grupie znajomych”*, mimo że system znajomych (`FR-05`) jest dopiero w `v1.2 (Could)`, a w MVP v1.0 użytkownik udostępnia link zewnętrznie (np. przez Messenger).
3. **NFR „Prywatność” (str. 3):** Okno 24h na cofnięcie usunięcia konta nadal nie ma odpowiadającego wymagania funkcjonalnego (`FR`) ani User Story.
4. **Dziennik zmian pkt 3 (str. 8):** Nadal widnieje literówka `FR-11` zamiast **`FR-07`** (*„Zmiana FR-11, dodanie opisu grup Tymczasowe i stałe”*).
5. **`US-3` (str. 4) oraz NFR „Niezawodność” (str. 3):** W `US-3` nadal jest *„po loginie”* zamiast *„nazwy użytkownika / kodu znajomego”*, a w NFR backupu wymieniono tylko *„oceny użytkowników i listy znajomych”* (bez kont, wektorów preferencji i grup z MVP v1.0).
6. **Sekcja 6, pkt 2 (str. 7):** W opisie zakresu v1.0 (*„Profilowanie i zbieranie preferencji”*) warto obok oceniania filmów dopisać również początkową ankietę (`FR-01`), żeby w pełni odpowiadało to tabeli FR i sekcji MoSCoW.

---

## 4. Pytania i decyzje biznesowe do wyjaśnienia z Product Ownerem

| ID | Obszar / Model | Pytanie / Decyzja | Status / Wpływ na implementację i modele |
|---|---|---|---|
| **Q-01** | Context, Class | **Skąd pochodzi katalog filmów i ich metadane?** | **Rozstrzygnięte:** API **TMDB** — zsynchronizowano ~8 244 filmy z gatunkami, obsadą, czasem trwania (`runtime`), słowami kluczowymi i plakatami do bazy MariaDB. |
| **Q-02** | Activity, Sequence (`FR-10`, `US-5 AC3`) | **Jak definiujemy „idealne dopasowanie” w dobieraniu grupowym?** Jaki próg podobieństwa wektorów decyduje o wyborze ścieżki ML zamiast trybu awaryjnego (najwyższa średnia ocen w grupie)? | Warunek rozgałęzienia `alt` w `Mikroserwisie ML` (algorytm *Average + Least Misery*). |
| **Q-03** | Activity, Sequence (`FR-10` vs `FR-11`) | **Czy w dobieraniu grupowym wykluczamy filmy ocenione już przez członków grupy?** (wszystkich, kogokolwiek, czy nie wykluczamy w trybie głównym ML?) | Logika filtrowania kandydatów w `wyznaczRekomendacjeGrupowa()`. |
| **Q-04** | Activity, Sequence, State (`FR-10`) | **Kto w grupie może uruchomić losowanie, odrzucić propozycję i zaakceptować wybrany film?** Każdy uczestnik, wyłącznie Administrator grupy, czy głosowanie wszystkich? | Kontrola uprawnień w `Backendzie PHP` oraz strażniki przejść w `State Diagram`. |
| **Q-05** | State, Sequence (`FR-07`, `FR-13`) | **(a) Czy grupowe dobieranie wymaga min. 2 aktywnych uczestników? (b) Co dzieje się z grupą stałą po odejściu lub usunięciu konta Administratora?** *(Czas wygasania grupy tymczasowej: 6 godzin).* | Strażnik `[aktywni >= 2]` oraz obsługa przekazania/usunięcia grupy stałej. |
| **Q-06** | Class, `FR-01` (`US-2 AC2`) | **Jaka jest minimalna wymagana liczba gatunków w początkowej ankiecie?** (np. 3 gatunki). | Walidacja formularza ankiety początkowej i inicjalizacja `wektorPreferencji`. |
| **Q-07** | Class, `FR-08` (`US-8 AC3`) | **Czy edycja lub usunięcie oceny w rankingu natychmiast przelicza wektor preferencji użytkownika?** (`US-7 AC3` precyzuje to tylko dla wystawienia oceny). | Wywołanie `zaktualizujWektorPreferencji()` w `Ocena.edytujOcene()` / `usunOcene()`. |
| **Q-08** | NFR Prywatność, Class | **Czy usuwanie konta (z 24-godzinnym oknem na cofnięcie) wchodzi w zakres MVP v1.0?** Jeśli tak — czy dopisujemy brakujące FR i US? | Zakres Sprintu 3 (`users.deleted_at` + `Cron`). |
| **Q-09** | Sequence (`US-5 AC2`), NFR Obciążenie | **Jaki mechanizm synchronizacji pokoju grupowego wybieramy w aplikacji webowej?** | **Rozstrzygnięte:** cykliczny **polling JS (`fetch` co 1–2 s)** do PHP — prosty w utrzymaniu i w zupełności wystarczający dla ≤ 10 użytkowników i czasu odpowiedzi ≤ 3 s. |

---

## 5. Link do Product Backlogu i oznaczenie pierwszej funkcjonalności

- **Link do Product Backlogu (Jira — z Lab 6):**  
  > `[WKLEJ TUTAJ LINK DO TABLICY / PRODUCT BACKLOGU PROJEKTU MOVIEMATCH W JIRA]`
  *(Przed wysłaniem na Teams wklej bezpośredni adres URL do Product Backlogu zespołu z Lab 6 oraz upewnij się, że poniższa pierwsza historyjka jest oznaczona w backlogu etykietą np. `MVP-FIRST` / umieszczona na samej górze Sprintu 1).*

### Oznaczona pierwsza funkcjonalność do implementacji (Start MVP)

Na podstawie analizy zależności wynikających z opracowanych diagramów UML (ścieżka alternatywna 1 w `Activity Diagram` i blok `alt` w `Sequence Diagram` pokazują, że brak uwierzytelnionego użytkownika blokuje zarówno ocenianie filmów, jak i dołączanie do sesji grupowych), jako **pierwszą funkcjonalność** w Product Backlogu oznaczamy:

1. **Pierwsza funkcjonalność startowa (fundament odblokowujący cały MVP):**  
   **`FR-02` Rejestracja konta + `FR-03` Logowanie użytkownika** (`US-1` i `US-6`, priorytet `Must`, wersja `1.0`, niskie ryzyko).
   - **Uzasadnienie:** Wszystkie pozostałe wymagania MVP (`FR-01`, `FR-04`, `FR-06`, `FR-07`, `FR-08`, `FR-10`, `FR-11`, `FR-13`) wymagają uwierzytelnionego obiektu `Uzytkownik` (`email`, `nazwaUzytkownika`, `hasloHash`) oraz aktywnej sesji HTTPS.
2. **Kolejny priorytetowy przyrost (rdzeń biznesowy zamodelowany w Lab 7):**  
   Po dostarczeniu uwierzytelniania oraz ankiety/ocen (`FR-01`, `FR-04`), kolejnym kluczowym przyrostem jest zamodelowany scenariusz **`FR-07` Tworzenie grupy → `FR-06` Dołączanie do grupy → `FR-10` Dobieranie grupowo filmu**.
