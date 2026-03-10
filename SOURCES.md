# Źródła i Metodologia

> Pełna transparentność: co zostało sprawdzone, kiedy i w jaki sposób. 
> Data analizy: **marzec 2026**

---

## Metodologia

Analiza oparta wyłącznie na publicznie dostępnych źródłach (brak dostępu do kodu źródłowego ani panelu administracyjnego). Zastosowano trzy poziomy weryfikacji:

1. **Desk research** – strona produktowa, materiały marketingowe, wpisy blogowe producenta
2. **Weryfikacja techniczna** – bezpośrednie fetche HTTP, analiza odpowiedzi serwera
3. **Badanie własne** – próba rejestracji w aplikacji z inspekcją konsoli przeglądarki (DevTools)

---

## Źródła Pierwotne

### Strona Produktowa
| URL | Co zweryfikowano | Metoda |
|---|---|---|
| `https://floowe.com/en/` | Funkcje, cennik, FAQ, model biznesowy | HTTP fetch (live, marzec 2026) |
| `https://floowe.leaware.com/register` | Formularz rejestracji, stack technologiczny, błędy runtime | HTTP fetch + DevTools (live) |

### Materiały Producenta (Leaware)
| URL | Zawartość | Data publikacji |
|---|---|---|
| `https://leaware.com/effortless-buzz-mastering-social-automation-with-floowe/` | Opis funkcji: analytics, auto-scheduling, social listening | Sierpień 2024 |
| `https://events.pblink.co.uk/en-gb/floowe-the-innovative-ai-tool-for-easy-content-creation_by_leaware-25.03.25` | Webinar – opis możliwości produktu | Luty 2025 |
| `https://torun.pl/pl/aktualnosc/tak-sie-robi-biznes-w-toruniu-7-leaware` | Floowe jako flagship produkt Leaware | 2025 |

### Marketplace (Microsoft)
| URL | Zawartość |
|---|---|
| `https://azuremarketplace.microsoft.com/en-us/marketplace/apps/leaspzoo1677657764309.floowe` | Oficjalny opis aplikacji, target audience |
| `https://marketplace.microsoft.com/en-us/product/web-apps/leawaresa.floowe_leaware` | flooWe – inna gałąź produktu (process management, nie content) |

---

## Ograniczenia Analizy

| Ograniczenie | Wpływ na analizę | Jak zarządzono |
|---|---|---|
| **Brak dostępu do panelu po zalogowaniu** – robots.txt blokuje crawlery, a rejestracja jest aktualnie zepsuta | Nie można zweryfikować UX wewnętrznych ekranów z pierwszej ręki | Oparto się na materiałach webinarowych i zrzutach ekranu z bloga Leaware |
| **Brak dostępu do kodu źródłowego** | Nie można ocenić jakości architektury wewnętrznej | Stack technologiczny ustalono na podstawie błędów konsolowych (Blazor/.NET/EF Core) |
| **Brak danych o liczbie użytkowników** | Nie można ocenić trakcji i PMF | Pośrednie wskaźniki: early-stage placeholders w sekcji News, błąd rejestracji, brak social proof |
| **Blog Leaware opisuje funkcje (analytics, scheduling) nieobecne na stronie produktowej** | Niejasne, co faktycznie istnieje w produkcie | Potraktowano jako aspirational copy lub funkcje w trakcie budowy – uwzględniono w TO-BE |

---

## Odkrycia Własne (Badanie Empiryczne)

### Próba rejestracji – marzec 2026
- **Obserwacja:** Endpoint `/register` zwraca pustą stronę (HTTP 200, puste body) przy bezpośrednim wywołaniu
- **Obserwacja:** Formularz renderuje się przy pełnym wczytaniu strony w przeglądarce, ale próba rejestracji wywołuje `System.InvalidOperationException` w EF Core
- **Wniosek:** Rejestracja nowych użytkowników jest aktualnie niedziałająca – krytyczna regresja w kluczowym user journey
- **Stack potwierdzony przez stack trace:** Blazor (frontend) + ASP.NET Core + Entity Framework Core (ORM)

---

## Metodologia Event Storming

Event Storming przeprowadzono zgodnie z metodyką Alberto Brandoliniego:
- **Zdarzenia domenowe** (Domain Events) – fakty w czasie przeszłym, nieodwracalne
- **Komendy** (Commands) – intencje aktorów wywołujące zdarzenia
- **Polityki** (Policies) – reguły reaktywne: "gdy X → wywołaj Y"
- **Aktorzy** – użytkownicy i systemy zewnętrzne inicjujące komendy
- **Read Models** – dane odczytywane przez aktorów do podjęcia decyzji

Diagramy wygenerowano w składni Mermaid (renderowane natywnie na GitHub).
