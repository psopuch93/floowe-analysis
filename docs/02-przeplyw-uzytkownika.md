# Kluczowe Przepływy Użytkownika

---

## 1. Rejestracja i Onboarding

```
[Strona Główna]
 
 
[Klik "Test for free!" / "Register"]
 
 
[Formularz rejestracji] email, hasło, dane firmy
 
 
[Weryfikacja e-mail]
 
 
[Konfiguracja profilu biznesowego]
 Branża / typ działalności
 Słowa kluczowe / tematy
 Język treści
 
 
[Połączenie kanałów] (opcjonalne przy onboardingu)
 Facebook OAuth
 LinkedIn OAuth
 X OAuth
 WordPress Plugin (instalacja + klucz API)
 
 
[Dashboard – gotowy do generowania]
```

**Obserwacje UX:**
- Onboarding jest liniowy i prosty – dobrze
- Brak progress bar / checklisty onboardingowej (użytkownik nie wie, ile kroków zostało)
- Brak "quick win" – pierwszego artykułu generowanego automatycznie jako demo przy rejestracji
- OAuth dla social media może być barierą dla mniej technicznych użytkowników

---

## 2. Główne Przypadki Użycia (Core Use Cases)

### UC-1: Generowanie Artykułu Blogowego

**Aktor:** Właściciel firmy / marketer 
**Trigger:** Użytkownik chce opublikować nowy artykuł na blogu firmowym 

```
[Dashboard]
 
 
[Klik "Nowy artykuł" / "Generuj"]
 
 
[Konfiguracja generowania]
 Temat / słowa kluczowe
 Długość artykułu
 Ton komunikacji
 Język
 
 
[AI generuje propozycje tematów]
 
 
[Użytkownik wybiera temat]
 
 
[AI generuje pełny artykuł] ← wywołanie LLM z kontekstem SEO
 
 
[Edytor artykułu]
 Edycja tekstu
 Dodanie / zmiana grafiki
 Podgląd
 
 
[Wybór kanałów publikacji]
 Strona WWW (WordPress)
 Facebook
 LinkedIn
 X (Twitter)
 
 
[Publikacja] → artykuł na WWW + posty social media (auto-generowane)
```

---

### UC-2: Przeglądanie i Edycja Wygenerowanych Treści

**Aktor:** Użytkownik (z uprawnieniami edytora) 
**Trigger:** Użytkownik chce sprawdzić / poprawić wygenerowany artykuł przed publikacją 

```
[Dashboard → Lista artykułów]
 
 
[Wybór artykułu ze statusem "Draft"]
 
 
[Edytor WYSIWYG]
 Edycja nagłówków, akapitów
 Zmiana grafiki (stock / własna)
 Podgląd SEO meta (title, description)
 Zapis zmian
 
 
[Artykuł gotowy do publikacji]
```

---

### UC-3: Zarządzanie Połączonymi Kanałami

**Aktor:** Administrator konta 
**Trigger:** Użytkownik chce dodać nowy kanał lub odświeżyć token dostępu 

```
[Ustawienia → Integracje]
 
 
[Lista kanałów]
 Status połączenia (connected / expired / error)
 Dodaj nowy kanał → OAuth flow
 Odłącz kanał
 
 
[Test połączenia] → wysłanie testowego posta
```

---

### UC-4: Podgląd Historii Publikacji

**Aktor:** Właściciel konta 
**Trigger:** Użytkownik chce sprawdzić, co zostało opublikowane 

```
[Dashboard → Historia]
 
 
[Lista opublikowanych artykułów]
 Data publikacji
 Kanały, na których opublikowano
 Tytuł artykułu
 Status (published / failed / scheduled*)
 
 
[Szczegóły artykułu] → link do posta, podgląd treści
```

> *Scheduled – funkcjonalność nieobecna w AS-IS, proponowana w TO-BE

---

### UC-5: Zarządzanie Subskrypcją

**Aktor:** Właściciel konta 
**Trigger:** Użytkownik chce zmienić plan lub sprawdzić limity 

```
[Ustawienia → Subskrypcja]
 
 
[Aktualny plan + licznik artykułów]
 X z 8 artykułów wykorzystanych w tym miesiącu
 Następne odnowienie: [data]
 
 
[Upgrade / Downgrade planu]
 
 
[Bramka płatności] → karta kredytowa / Stripe
 
 
[Potwierdzenie zmiany planu]
```

---

## 3. Przepływ Płatności

Floowe stosuje standardowy model płatności SaaS:

- **Bramka płatności:** prawdopodobnie Stripe (standard dla SaaS w Europie)
- **Model:** miesięczna subskrypcja z automatycznym odnowieniem
- **Brak:** rocznej subskrypcji (potencjalna strata retencji i upfront cash flow)
- **Brak:** rozliczeń proporcjonalnych przy upgrade (prorated billing)
- **Brak:** faktury VAT w panelu (wymagane dla B2B w Polsce)

---

## 4. Integracje Techniczne

### WordPress Plugin Flow
```
[Użytkownik instaluje plugin Floowe w WordPress]
 
 
[Plugin pyta o klucz API z konta Floowe]
 
 
[Połączenie: Floowe ↔ WordPress REST API]
 
 
[Po publikacji w Floowe → POST /wp-json/wp/v2/posts]
 
 
[Artykuł pojawia się na blogu WordPress jako opublikowany]
```

### Social Media OAuth Flow
```
[Użytkownik klika "Połącz Facebook"]
 
 
[Redirect na Facebook OAuth]
 
 
[Użytkownik autoryzuje aplikację Floowe]
 
 
[Facebook zwraca access_token]
 
 
[Floowe zapisuje token (szyfrowany) w bazie danych]
 
 
[Przy publikacji: Graph API POST /me/feed]
```

**Uwaga techniczna:** Tokeny social media mają daty ważności (FB: 60 dni dla long-lived tokens). Floowe powinno implementować automatyczne odświeżanie tokenów i powiadomienia przy ich wygaśnięciu – jest to częste źródło błędów publikacji u użytkowników.
