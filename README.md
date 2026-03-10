# Floowe – Analiza Produktu & Propozycje Funkcjonalności

> Zadanie rekrutacyjne: Techniczny PM/BA dla AI i Automatyzacji 
> Aplikacja: **[Floowe](https://floowe.com)** – AI-powered content creation & publishing SaaS 

---

## Struktura Repozytorium

| Dokument | Opis |
|---|---|
| [Analiza Aplikacji](docs/01-analiza-aplikacji.md) | Przegląd produktu, model biznesowy, UX/UI, punkty integracji, **odkryty błąd rejestracji + korekta stack'u** |
| [Kluczowe Przepływy Użytkownika](docs/02-przeplyw-uzytkownika.md) | Rejestracja, core use cases, płatności, integracje |
| [Event Storming](docs/03-event-storming.md) | Diagram AS-IS i TO-BE z aktorami, komendami i zdarzeniami domenowymi |
| [Specyfikacja Funkcjonalności](docs/04-specyfikacja-funkcjonalnosci.md) | 3 nowe funkcje: User Story, Case Biznesowy, Przegląd Techniczny, Ryzyka |
| [Źródła i Metodologia](SOURCES.md) | Wszystkie źródła, metody weryfikacji, ograniczenia analizy |

---

## TL;DR – Floowe w jednym zdaniu

**Floowe** to polskie SaaS (produkt firmy Leaware) umożliwiające małym i średnim firmom automatyczne generowanie artykułów SEO z pomocą AI oraz ich natychmiastową publikację na stronie WWW i w social mediach (Facebook, LinkedIn, X) – bez angażowania copywritera.

---

## Trzy proponowane funkcje (TO-BE)

| # | Funkcja | Problem, który rozwiązuje |
|---|---|---|
| 1 | **Analytics & Feedback Loop** | Brak informacji zwrotnej o skuteczności treści – użytkownik nie wie, co działa |
| 2 | **AI Brand Voice Training** | Generowane treści brzmią "generycznie" – nie pasują do tonu marki |
| 3 | **Content Calendar + Smart Scheduling** | Brak planowania i zarządzania harmonogramem – użytkownik nie ma kontroli nad rytmem publikacji |

---

## Użyte Narzędzia i Metodologia

- Analiza produktu: desk research (floowe.com, leaware.com, Microsoft Marketplace) + weryfikacja live (HTTP fetch)
- Badanie empiryczne: próba rejestracji z inspekcją DevTools → odkryto błąd EF Core + ustalono rzeczywisty stack (.NET/Blazor)
- Diagramy: Mermaid (renderowane natywnie na GitHub)
- Metodologia Event Storming: Alberto Brandolini
- Pełne źródła: [SOURCES.md](SOURCES.md)
