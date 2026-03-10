# Event Storming – Floowe

> **Metodologia:** Event Storming (Alberto Brandolini) 
> Technika eksploracji złożonych domen biznesowych poprzez identyfikację zdarzeń domenowych i ich powiązań z komendami, aktorami oraz systemami zewnętrznymi.

---

## Legenda

| Symbol / Kolor | Znaczenie | Opis |
|---|---|---|
| 🟠 **Zdarzenie domenowe** | Domain Event | Coś, co się wydarzyło w przeszłości (past tense). Fakt w domenie. |
| **Komenda** | Command | Intencja – akcja wywołana przez aktora lub system. Wywołuje zdarzenie. |
| 🟡 **Aktor** | Actor | Użytkownik lub zewnętrzny system, który wydaje komendę |
| 🩷 **System zewnętrzny** | External System | Integracja poza domeną Floowe |
| 🟣 **Polityka / Reakcja** | Policy | Reguła biznesowa: "Gdy zdarzenie X → wykonaj komendę Y" |
| 🟢 **Read Model** | Read Model / View | Dane odczytywane przez aktora do podjęcia decyzji |

---

## AS-IS (Stan Bieżący)

```mermaid
flowchart LR
 %% Actors
 USER["🟡 Użytkownik"]
 AI_SYS["🟡 System AI\n(LLM)"]
 EXT_WP["🩷 WordPress\n(plugin)"]
 EXT_FB["🩷 Facebook\nGraph API"]
 EXT_LI["🩷 LinkedIn\nMarketing API"]
 EXT_X["🩷 X / Twitter\nAPI"]

 %% === REJESTRACJA ===
 subgraph REG["REJESTRACJA & KONFIGURACJA"]
 direction TB
 CMD_REG[" Zarejestruj konto"]
 EVT_REG["🟠 Konto utworzone"]
 CMD_PROFILE[" Skonfiguruj profil\nbiznesowy"]
 EVT_PROFILE["🟠 Profil\nskonfigurowany"]
 CMD_CONNECT[" Połącz kanały\nsocial media"]
 EVT_CONNECTED["🟠 Kanały\npodłączone"]
 end

 %% === GENEROWANIE ===
 subgraph GEN["GENEROWANIE TREŚCI"]
 direction TB
 CMD_IDEAS[" Zleć generowanie\npomysłów"]
 EVT_IDEAS["🟠 Pomysły na artykuły\nwygenerowane"]
 RM_IDEAS["🟢 Lista tematów\ndo wyboru"]
 CMD_SELECT[" Wybierz temat\nartykułu"]
 EVT_TRIGGER["🟠 Generowanie artykułu\nuruchomione"]
 CMD_GENERATE[" Generuj artykuł\n(AI + SEO)"]
 EVT_GENERATED["🟠 Artykuł\nwygenerowany"]
 end

 %% === EDYCJA ===
 subgraph EDIT["EDYCJA & ZATWIERDZANIE"]
 direction TB
 RM_DRAFT["🟢 Draft artykułu\nw edytorze"]
 CMD_EDIT[" Edytuj artykuł"]
 EVT_EDITED["🟠 Artykuł\nzmodyfikowany"]
 CMD_IMG[" Wybierz grafikę"]
 EVT_IMG["🟠 Grafika\nprzypisana"]
 end

 %% === PUBLIKACJA ===
 subgraph PUB["PUBLIKACJA & DYSTRYBUCJA"]
 direction TB
 CMD_PUBLISH[" Opublikuj treść"]
 EVT_WP["🟠 Artykuł opublikowany\nna stronie WWW"]
 EVT_SOC["🟠 Posty social media\nowygenerowane i opublikowane"]
 POL_SOC["🟣 Polityka: Przy publikacji\nartykułu → auto-generuj\ni opublikuj posty SM"]
 end

 %% Flow
 USER --> CMD_REG --> EVT_REG
 EVT_REG --> CMD_PROFILE --> EVT_PROFILE
 EVT_PROFILE --> CMD_CONNECT --> EVT_CONNECTED

 EVT_CONNECTED --> CMD_IDEAS
 AI_SYS --> EVT_IDEAS
 CMD_IDEAS --> EVT_IDEAS --> RM_IDEAS
 RM_IDEAS --> CMD_SELECT --> EVT_TRIGGER
 EVT_TRIGGER --> CMD_GENERATE --> EVT_GENERATED

 EVT_GENERATED --> RM_DRAFT
 RM_DRAFT --> CMD_EDIT --> EVT_EDITED
 EVT_EDITED --> CMD_IMG --> EVT_IMG

 EVT_IMG --> CMD_PUBLISH
 CMD_PUBLISH --> EVT_WP --> EXT_WP
 CMD_PUBLISH --> POL_SOC --> EVT_SOC
 EVT_SOC --> EXT_FB
 EVT_SOC --> EXT_LI
 EVT_SOC --> EXT_X
```

### Kluczowe Zdarzenia AS-IS

| # | Zdarzenie Domenowe | Wyzwalacz | Rezultat |
|---|---|---|---|
| 1 | **Konto utworzone** | Rejestracja użytkownika | Aktywacja konta, start okresu próbnego |
| 2 | **Profil skonfigurowany** | Użytkownik podaje dane firmy i branżę | AI ma kontekst do generowania treści |
| 3 | **Kanały podłączone** | OAuth z Facebook/LinkedIn/X | Możliwość auto-publikacji |
| 4 | **Artykuł wygenerowany** | Wywołanie LLM z kontekstem SEO + profilu | Draft artykułu dostępny w edytorze |
| 5 | **Treść opublikowana** | Komenda publikacji | Artykuł na WWW + posty SM |

---

## TO-BE (Stan Przyszły z Nowymi Funkcjonalnościami)

Proponowane funkcje:
- **F1:** Analytics & Performance Feedback Loop
- **F2:** AI Brand Voice Training 
- **F3:** Content Calendar & Smart Scheduling

```mermaid
flowchart LR
 %% Actors
 USER["🟡 Użytkownik"]
 AI_SYS["🟡 System AI"]
 EXT_GSC["🩷 Google Search\nConsole API"]
 EXT_GA["🩷 Google Analytics\nAPI"]
 EXT_WP["🩷 WordPress"]
 EXT_SM["🩷 Social Media\nAPIs"]

 %% === F1: ANALYTICS ===
 subgraph F1["F1: ANALYTICS & FEEDBACK LOOP"]
 direction TB
 CMD_TRACK[" Zbierz dane\nperformance artykułu"]
 EVT_DATA["🟠 Dane analityczne\nzaktualizowane"]
 RM_DASH["🟢 Dashboard\nAnalityczny"]
 POL_AI["🟣 Polityka: Gdy artykuł\nprzekroczy próg ruchu\n→ zaproponuj follow-up"]
 EVT_SUGGEST["🟠 Follow-up\nzaproponowany"]
 CMD_AI_RECO[" Generuj rekomendacje\ntematów na podstawie\ndanych analitycznych"]
 EVT_RECO["🟠 Tematy rekomendowane\nna podstawie performance'u"]
 end

 %% === F2: BRAND VOICE ===
 subgraph F2["F2: AI BRAND VOICE TRAINING"]
 direction TB
 CMD_UPLOAD[" Wgraj próbki\ntreści marki"]
 EVT_UPLOADED["🟠 Próbki treści\nzaładowane"]
 CMD_TRAIN[" Trenuj model\nBrand Voice"]
 EVT_TRAINED["🟠 Brand Voice\nModel wytrenowany"]
 RM_VOICE["🟢 Podgląd stylu\n(przykładowy akapit)"]
 POL_VOICE["🟣 Polityka: Przy każdym\ngenerowaniu → zastosuj\nBrand Voice Model"]
 EVT_BRANDED["🟠 Artykuł wygenerowany\nz głosem marki"]
 end

 %% === F3: CALENDAR ===
 subgraph F3["F3: CONTENT CALENDAR & SCHEDULING"]
 direction TB
 CMD_PLAN[" Zaplanuj artykuł\nna określony termin"]
 EVT_PLANNED["🟠 Artykuł\nzaplanowany"]
 RM_CAL["🟢 Widok Kalendarza\nTreści"]
 POL_SCHED["🟣 Polityka: W zaplanowanym\nterminie → auto-generuj\ni opublikuj artykuł"]
 EVT_AUTO_GEN["🟠 Artykuł automatycznie\nwygenerowany wg harmonogramu"]
 EVT_AUTO_PUB["🟠 Artykuł automatycznie\nopublikowany wg harmonogramu"]
 CMD_NOTIFY[" Wyślij powiadomienie\no publikacji"]
 EVT_NOTIFIED["🟠 Użytkownik powiadomiony\n(email / push)"]
 end

 %% F1 Flow
 EXT_GSC --> CMD_TRACK
 EXT_GA --> CMD_TRACK
 CMD_TRACK --> EVT_DATA --> RM_DASH
 RM_DASH --> USER
 EVT_DATA --> POL_AI --> EVT_SUGGEST --> CMD_AI_RECO --> EVT_RECO

 %% F2 Flow
 USER --> CMD_UPLOAD --> EVT_UPLOADED
 EVT_UPLOADED --> CMD_TRAIN --> EVT_TRAINED --> RM_VOICE
 RM_VOICE --> USER
 AI_SYS --> POL_VOICE --> EVT_BRANDED

 %% F3 Flow
 USER --> CMD_PLAN --> EVT_PLANNED --> RM_CAL
 RM_CAL --> USER
 EVT_PLANNED --> POL_SCHED
 POL_SCHED --> EVT_AUTO_GEN --> EVT_AUTO_PUB
 EVT_AUTO_PUB --> EXT_WP
 EVT_AUTO_PUB --> EXT_SM
 EVT_AUTO_PUB --> CMD_NOTIFY --> EVT_NOTIFIED --> USER
```

### Nowe Zdarzenia TO-BE

| # | Zdarzenie Domenowe | Funkcja | Co zmienia w systemie |
|---|---|---|---|
| 6 | **Dane analityczne zaktualizowane** | F1 – Analytics | System wie, które treści generują ruch i konwersje |
| 7 | **Follow-up zaproponowany** | F1 – Analytics | AI zamyka feedback loop – generuje kolejne treści na bazie sukcesu |
| 8 | **Brand Voice Model wytrenowany** | F2 – Brand Voice | Każdy przyszły artykuł będzie brzmiał jak marka użytkownika |
| 9 | **Artykuł zaplanowany** | F3 – Calendar | System przejmuje odpowiedzialność za timing publikacji |
| 10 | **Artykuł automatycznie opublikowany wg harmonogramu** | F3 – Calendar | Pełna automatyzacja bez udziału użytkownika |

---

## Kluczowi Aktorzy i Systemy

| Aktor / System | Rola | AS-IS | TO-BE |
|---|---|---|---|
| **Użytkownik** | Inicjuje akcje, edytuje treści | | |
| **System AI (LLM)** | Generuje treści i pomysły | | + Brand Voice |
| **WordPress Plugin** | Publikacja na stronie WWW | | + scheduled posts |
| **Facebook/LinkedIn/X API** | Dystrybucja do social media | | + scheduled |
| **Google Search Console** | Dane o pozycjach w Google | | (F1) |
| **Google Analytics** | Dane o ruchu i zachowaniu użytkowników | | (F1) |
| **Scheduler (Cron/Queue)** | Planowanie i automatyczne wyzwalanie zadań | | (F3) |
| **Notification Service** | Powiadomienia email/push | | (F3) |
