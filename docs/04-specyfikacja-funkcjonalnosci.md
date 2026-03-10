# Specyfikacja Funkcjonalności – TO-BE

> Każda funkcja opisana jest przez: User Story, Case Biznesowy, Przegląd Techniczny i Ryzyka.

---

## F1: Analytics & Performance Feedback Loop

### User Story

```
Jako właściciel firmy korzystający z Floowe,
Chcę widzieć, jak moje opublikowane artykuły performują w Google i social mediach
 (pozycje, ruch organiczny, kliki, zaangażowanie),
Aby wiedzieć, które tematy i typy treści są skuteczne,
 i podejmować lepsze decyzje o kolejnych artykułach.

Kryteria Akceptacji:
- [] Dashboard analityczny wyświetla dane z Google Search Console (pozycje, impresje, CTR)
- [] Dashboard wyświetla dane z Google Analytics (sesje, bounce rate, czas na stronie)
- [] Widoczne metryki social media (zasięg, reakcje, kliknięcia) dla postów powiązanych z artykułem
- [] Dane są aktualizowane co 24 godziny (nie real-time, wystarczy codziennie)
- [] Dla każdego artykułu istnieje widok "szczegółów performance'u"
- [] System AI automatycznie proponuje tematy follow-up na podstawie artykułów z najwyższym ruchem
- [] Użytkownik może ręcznie "odświeżyć" dane jednym kliknięciem
- [] Dashboard działa na mobile (responsive)
```

### Case Biznesowy

Obecny Floowe ma fundamentalny problem pętli zwrotnej (feedback loop): użytkownik publikuje artykuły "w ciemno" i nie wie, czy przynoszą efekty. Bez tej wiedzy nie ma powodu do kontynuowania subskrypcji – jeśli nie widzę wartości, rezygnuję. Analytics dashboard zamknie tę pętlę: użytkownik widzi wzrost pozycji w Google → jest zmotywowany do tworzenia kolejnych treści → pozostaje klientem dłużej. To bezpośrednio przekłada się na **obniżenie churn rate** i **wzrost LTV klienta**. Funkcja może być również elementem feature-gatingu (np. zaawansowane metryki dostępne od planu Standard wzwyż), co podniesie wartość wyższych planów subskrypcji.

### Przegląd Techniczny

**Komponenty do dodania / zmiany:**

- **Backend:**
 - Nowy moduł integracji z Google Search Console API (OAuth 2.0 + Google API Client)
 - Nowy moduł integracji z Google Analytics 4 API (GA4 Data API)
 - Nowy moduł integracji z Social Media APIs (pobieranie metryk postów: FB Insights, LinkedIn Analytics, X Analytics)
 - Serwis agregujący dane (cron job / scheduled task – np. co 24h)
 - Nowa tabela w bazie danych: `article_metrics` (article_id, source, metric_name, metric_value, date)
 - Logika rekomendacji AI: ranking artykułów po ruchu → prompt do LLM o propozycje tematów pokrewnych

- **Frontend:**
 - Nowy widok: `/dashboard/analytics` – główny dashboard z wykresami
 - Nowy widok: `/article/{id}/analytics` – szczegółowy performance artykułu
 - Komponenty wykresów (np. Chart.js / ApexCharts) – ruch w czasie, pozycje, engagement
 - Badge "Top Performer" przy artykułach z najwyższym ruchem

**Kluczowe integracje:**
- Google Search Console API (wymaga weryfikacji domeny przez użytkownika)
- Google Analytics 4 API (wymaga połączenia konta GA użytkownika)
- Facebook Graph API – Insights (pobieranie zasięgu i zaangażowania postów)

**Stos technologiczny (na podstawie zidentyfikowanego stacku .NET/Blazor):**
- Backend: ASP.NET Core – nowe serwisy jako osobne klasy z DI, wywołania do Google APIs przez HttpClient
- Baza danych: SQL Server lub PostgreSQL przez Entity Framework Core – nowa tabela `article_metrics`
- Harmonogram: Hangfire lub Quartz.NET – natywne biblioteki .NET do background jobs
- Frontend: Blazor – komponenty wykresów przez JS interop z Chart.js, lub biblioteka Blazor-native (np. MudBlazor Charts)

### Ryzyka i Wyzwania

1. **Złożoność autoryzacji po stronie użytkownika** – Integracja z GSC i GA4 wymaga, aby użytkownik połączył swoje konto Google i zweryfikował własność domeny. Dla mniej technicznych użytkowników to bariera. Ryzyko: niska adopcja funkcji. Mitygacja: szczegółowe step-by-step guide, wideo instruktażowe, opcja "pomiń" przy konfiguracji.

2. **Ograniczenia API i rate limiting** – Google APIs mają dzienne limity zapytań. Przy dużej liczbie użytkowników synchronizacja danych dla wszystkich kont jednocześnie może natrafiać na te limity. Ryzyko: opóźnienia lub brak danych dla części użytkowników. Mitygacja: kolejkowanie zadań synchronizacji (staggered scheduling), cachowanie wyników, request batching.

3. **Atrybucja ruchu** – Artykuł opublikowany na WordPress i udostępniony na Facebook może generować ruch z wielu źródeł jednocześnie. Przypisanie go do konkretnego artykułu w Floowe wymaga UTM parametrów i spójnej logiki atrybucji. Ryzyko: błędne dane dezinformują użytkownika. Mitygacja: automatyczne dodawanie UTM parametrów do linków social media przy publikacji.

---

## F2: AI Brand Voice Training

### User Story

```
Jako marketer zarządzający treściami dla firmy,
Chcę wgrać próbki dotychczasowych treści mojej marki (artykuły, opisy produktów, posty),
Aby system AI nauczył się stylu, tonu i charakteru mojej komunikacji
 i generował nowe artykuły brzmiące autentycznie, jak moja firma – a nie jak "generyczny AI".

Kryteria Akceptacji:
- [] Użytkownik może wgrać minimum 3, maksimum 20 przykładowych tekstów (plik .txt, .docx, .pdf lub URL)
- [] System analizuje wgrane próbki i wyciąga charakterystyki stylu: długość zdań, słownictwo, styl nagłówków, ton (formalny/nieformalny), używanie pierwszej/trzeciej osoby
- [] Użytkownik widzi "profil głosu marki" – podsumowanie w 3-5 punktach (np. "Piszesz krótko, używasz list, jesteś bezpośredni, unikasz żargonu")
- [] Możliwy podgląd: system generuje akapit przykładowy w Brand Voice przed aktywacją
- [] Checkbox "Użyj Brand Voice" przy każdym generowaniu artykułu (domyślnie: włączony gdy model istnieje)
- [] Użytkownik może wytrenować nowy model (nadpisując stary) lub usunąć Brand Voice
- [] Brand Voice stosowany jest zarówno do artykułów jak i do postów social media
```

### Case Biznesowy

Największą słabością narzędzi AI do content generation jest "generyczność" wyjścia – artykuły brzmią jak napisane przez tego samego, bezstylowego autora. To powoduje, że użytkownicy i tak muszą poświęcać czas na edycję, co niweluje wartość automatyzacji. Brand Voice Training eliminuje ten problem: treści brzmią jak marka, nie jak AI. Funkcja silnie **różnicuje Floowe na tle konkurencji** (Jasper, Copy.ai, Writesonic) i jest argumentem sprzedażowym ("Floowe uczy się Twojego stylu"). Dla użytkowników, którzy przeszkolą model, wyjście natychmiastowo staje się bardziej wartościowe, co **redukuje czas edycji** i **zwiększa satysfakcję**. To bezpośrednio przekłada się na retencję.

### Przegląd Techniczny

**Komponenty do dodania / zmiany:**

- **Backend:**
 - Nowy endpoint: `POST /api/brand-voice/upload` – przyjmuje pliki lub URL-e
 - Parser dokumentów: obsługa .txt, .docx (DocumentFormat.OpenXml), .pdf (PdfPig lub iTextSharp), URL scraping (HtmlAgilityPack) – wszystkie biblioteki natywne dla .NET
 - Serwis analizy stylu: wywołanie LLM z zestawem próbek + prompt do ekstrakcji charakterystyk stylu → zapis jako JSON (`brand_voice_profile`)
 - Storage: `brand_voice_profiles` tabela (user_id, profile_json, model_prompt_addendum, created_at)
 - Modyfikacja serwisu generowania: przy `use_brand_voice=true` → dołączenie `model_prompt_addendum` do system promptu LLM
 - Endpoint: `POST /api/brand-voice/preview` → generuje próbny akapit

- **Frontend:**
 - Nowy widok: `/settings/brand-voice` – onboarding trenowania modelu
 - Drag & drop upload komponent dla plików
 - Karta "Twój profil głosu marki" z podsumowaniem stylu
 - Podgląd wygenerowanego akapitu w Brand Voice
 - Toggle "Użyj Brand Voice" w panelu generowania artykułu

**Kluczowe integracje:**
- LLM (Anthropic Claude / OpenAI GPT-4) – analiza stylu i adaptacja generowania
- Ewentualnie: fine-tuning modelu (bardziej zaawansowane, drogie, ale wyższy jakościowo efekt)

**Stos technologiczny (na podstawie zidentyfikowanego stacku .NET/Blazor):**
- Parsowanie dokumentów: DocumentFormat.OpenXml (.docx), PdfPig (.pdf), HtmlAgilityPack (URL scraping) – biblioteki .NET
- Wektoryzacja stylu: embedding wgranych tekstów przez API (np. OpenAI Embeddings), przechowywanie w pgvector (Postgres) lub osobna usługa (Qdrant)
- LLM: prompt engineering z few-shot examples – wyekstrahowany profil stylu jako część system promptu

### Ryzyka i Wyzwania

1. **Jakość wejścia = jakość wyjścia** – Jeśli użytkownik wgra słabo napisane lub niespójne próbki, model "nauczy się" złego stylu. Ryzyko: użytkownik będzie niezadowolony z wyników i wini narzędzie. Mitygacja: minimalna liczba próbek (min. 3), walidacja długości wgranych tekstów, wyraźna komunikacja "garbage in, garbage out" w UI.

2. **Pełzanie stylu przy wielu aktualizacjach** – Użytkownik trenuje model wielokrotnie, wgrywając różne próbki. Profil głosu marki może stać się niespójny. Ryzyko: pogorszenie jakości generowania zamiast poprawy. Mitygacja: każde trenowanie nadpisuje poprzedni model (nie kumuluje), użytkownik zawsze ma wgląd w aktualny profil.

3. **Koszty LLM przy analizie wielu dokumentów** – Parsowanie i analiza 20 próbek tekstów to znaczne zapytania do LLM. Przy dużej liczbie użytkowników koszty API mogą być wysokie. Ryzyko: erozja marży produktu. Mitygacja: analiza jest jednorazowa (nie powtarzana przy każdym artykule), wynik zapisywany jako statyczny `profile_json`; prompt_addendum dodawany do generowania jest krótki (max ~500 tokenów).

---

## F3: Content Calendar & Smart Scheduling

### User Story

```
Jako właściciel małej firmy,
Chcę planować z wyprzedzeniem harmonogram publikacji treści (np. na cały miesiąc),
Aby artykuły były regularnie publikowane nawet gdy jestem zajęty lub na urlopie,
 bez konieczności logowania się do Floowe w dniu publikacji.

Kryteria Akceptacji:
- [] Widok kalendarza miesięcznego z zaplanowanymi artykułami
- [] Możliwość zaplanowania artykułu na konkretny dzień i godzinę
- [] Opcja: zaplanuj artykuł z podanym tematem TERAZ + ustaw datę i godzinę auto-generowania i auto-publikacji
- [] Artykuł wygenerowany wg harmonogramu trafia do "Oczekujących na zatwierdzenie" LUB jest od razu publikowany (wybór użytkownika w ustawieniach)
- [] Powiadomienie email wysyłane: (a) przed generowaniem [24h wcześniej], (b) po publikacji [natychmiast]
- [] Widok kalendarza pokazuje status każdego zaplanowanego artykułu: Planned / Generating / Ready for Review / Published / Failed
- [] Możliwość przeciągnięcia zaplanowanego artykułu na inny dzień (drag & drop)
- [] AI sugeruje optymalne dni i godziny publikacji na podstawie historycznych danych zaangażowania (jeśli dostępne z F1)
```

### Case Biznesowy

Regularność publikacji jest kluczowa dla SEO (Google nagradza systematyczne serwisy) i dla budowania widowni (czytelnicy wracają, gdy wiedzą, kiedy się czegoś spodziewać). Tymczasem użytkownicy Floowe muszą pamiętać, żeby regularnie się logować i inicjować generowanie. To bariera, która powoduje, że treści pojawiają się nieregularnie – co niweluje efekty SEO. Content Calendar z planowaniem rozwiązuje ten problem: użytkownik planuje raz na miesiąc, Floowe działa automatycznie. To **przesuwa propozycję wartości Floowe z narzędzia do platformy zarządzania contentem** i otwiera drogę do sprzedaży planów agencyjnych (multi-klient, multi-kalendarz). Funkcja calendar jest standardowym wyróżnikiem w narzędziach premium social media (Buffer, Hootsuite) – jej brak jest widoczną luką.

### Przegląd Techniczny

**Komponenty do dodania / zmiany:**

- **Backend:**
 - Nowa tabela: `scheduled_content` (id, user_id, topic, scheduled_datetime, status, article_id FK, channels_mask, notify_before, notify_after)
 - Serwis Schedulera: Hangfire (persistent background jobs dla .NET) – odpytuje `scheduled_content` gdzie `scheduled_datetime <= NOW()` i `status = 'planned'`
 - Przy wyzwoleniu schedulera: wywołanie pipeline'u generowania → (opcjonalnie) pipeline publikacji
 - Nowy endpoint: `POST /api/schedule` – tworzenie zadania
 - Nowy endpoint: `PATCH /api/schedule/{id}` – edycja / przesunięcie
 - Nowy endpoint: `DELETE /api/schedule/{id}` – anulowanie
 - Serwis notyfikacji email (SendGrid lub MailKit) – trigger 24h przed + po publikacji
 - (Opcjonalnie) Push notifications przez Web Push API / Azure Notification Hubs

- **Frontend:**
 - Nowy widok: `/calendar` – widok miesięczny / tygodniowy jako Blazor component z JS interop do FullCalendar.js
 - Drag & drop dla zaplanowanych artykułów przez JS interop
 - Modal "Zaplanuj artykuł": wybór tematu, daty/godziny, kanałów, trybu publikacji (auto / do zatwierdzenia)
 - Status badges na kafelkach kalendarza
 - Panel "Oczekujące na zatwierdzenie" – artykuły wygenerowane czekające na akceptację

**Kluczowe integracje:**
- Hangfire – persistent background jobs natywne dla .NET, zadania przeżywają restart serwera
- Email service (SendGrid lub MailKit + SMTP) – oba mają oficjalne SDK dla .NET
- Opcjonalnie: integracja z F1 (Analytics) – dane o najlepszych godzinach publikacji

**Stos technologiczny (na podstawie zidentyfikowanego stacku .NET/Blazor):**
- Backend: ASP.NET Core + Hangfire jako job scheduler (distributed locks wbudowane)
- Frontend: Blazor z JS interop do bibliotek kalendarzowych (FullCalendar.js)
- Baza danych: SQL Server lub PostgreSQL przez EF Core, kolumna `scheduled_datetime` z indeksem

### Ryzyka i Wyzwania

1. **Niezawodność schedulera przy awariach infrastruktury** – Jeśli serwer lub task queue padnie w momencie, gdy artykuł miał być opublikowany, publikacja nie następuje. Użytkownik nie dostaje artykułu w planowanym terminie. Ryzyko: utrata zaufania do narzędzia. Mitygacja: persistent task queue (zadania nie giną po restarcie), mechanizm retry z exponential backoff, status "Failed" widoczny w kalendarzu + powiadomienie email o błędzie.

2. **Artykuł wygenerowany nie spełnia oczekiwań użytkownika** – Gdy artykuł generuje się automatycznie bez nadzoru, a wynik jest słabej jakości (np. LLM "halucynuje" fakty), użytkownik dowiaduje się po fakcie – artykuł jest już opublikowany. Ryzyko: reputacyjna szkoda dla firmy użytkownika. Mitygacja: domyślny tryb "do zatwierdzenia" (artykuł trafia do review, nie jest od razu publikowany) + użytkownik musi świadomie wybrać tryb "auto-publish".

3. **Timezone i DST** – Planowanie na konkretną godzinę w różnych strefach czasowych, z uwzględnieniem czasu letniego/zimowego (DST), jest technicznie nietrywialnym problemem. Ryzyko: artykuły publikowane o złej godzinie. Mitygacja: przechowywanie wszystkich czasów w UTC w bazie danych, UI konwertujący do strefy czasowej użytkownika (wykrytej z przeglądarki lub ustawień konta), wyraźne oznaczenie strefy czasowej w UI.
