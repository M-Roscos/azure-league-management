# ADR 001: Wybór architektury chmurowej i modelu bazy danych  

## 1. Kontekst i wymagania

W ramach projektu systemu informatycznego "Liga" konieczne jest zaprojektowanie architektury zdolnej do obsługi wymagań funkcjonalnych obejmujących wyliczanie skomplikowanych statystyk sportowych, w tym:
* Identyfikację najlepszej drużyny (największa liczba punktów).
* Wyłonienie najlepszego strzelca w lidze.
* Wskazanie zawodnika, który zdobył najwięcej bramek grając przeciwko określonej drużynie przeciwnej.

Dodatkowo projekt posiada ścisłe ograniczenia budżetowe (limit środowiska studenckiego w wysokości 100 USD) oraz wysokie wymagania dotyczące bezpieczeństwa (ochrona przed wyciekiem poświadczeń, logowanie zdarzeń).

## 2. Decyzja

1. **Wybór modelu chmurowego:** Zdecydowano się na wykorzystanie architektury opartej o usługi platformowe (PaaS) oraz bezserwerowe (Serverless) w środowisku Microsoft Azure (Azure Functions, Azure Static Web Apps), zamiast klasycznych maszyn wirtualnych (IaaS).
2. **Wybór silnika bazy danych:** Jako główne źródło danych wybrana została relacyjna baza danych **Azure SQL Database** w warstwie Basic.
3. **Zarządzanie poświadczeniami:** Zdecydowano o całkowitym wyeliminowaniu tradycyjnych ciągów połączeń (Connection Strings) z kodu źródłowego na rzecz użycia tożsamości zarządzanych (Managed Identities) oraz usługi Azure Key Vault.

## 3. Uzasadnienie

* **Dlaczego baza relacyjna (SQL)?** Wymagania funkcjonalne (np. statystyki zawodnika przeciwko konkretnej drużynie) opierają się na silnie ustrukturyzowanych danych i wymagają wielokrotnych złączeń (JOIN) pomiędzy tabelami: `Zawodnicy`, `Druzyny`, `Mecze` oraz `Gole`. Zastosowanie nierelacyjnej bazy danych (NoSQL) w tym scenariuszu prowadziłoby do redundancji danych i znacznego skomplikowania logiki aplikacji. Język SQL jest naturalnym i najbardziej zoptymalizowanym narzędziem do wyliczania tego typu statystyk.
* **Dlaczego Serverless / PaaS?** Minimalizuje to koszty utrzymania środowiska w fazie MVP, pozwalając na bezpieczne zmieszczenie się w przydzielonym budżecie studenckim (szacunkowy koszt bazy to ~5 USD miesięcznie). Model ten zdejmuje również z zespołu obowiązek aktualizacji systemu operacyjnego (patching).
* **Dlaczego Managed Identity?** Stanowi to realizację paradygmatu "Zero Trust". Aplikacja uwierzytelnia się do bazy danych za pomocą tokenów z Microsoft Entra ID przypisanych do zasobu chmurowego, co zapobiega wyciekom poświadczeń poprzez repozytorium GitHub i spełnia najwyższe standardy bezpieczeństwa.

## 4. Konsekwencje

* **Pozytywne:** Środowisko jest wysoce bezpieczne, tanie w utrzymaniu, a baza danych gwarantuje spójność (ACID) transakcji wprowadzania wyników meczów.
* **Negatywne (Wyzwania):** Wymaga to od zespołu deweloperskiego znajomości technologii ORM (np. Entity Framework Core) w celu mapowania obiektowo-relacyjnego oraz poprawnej konfiguracji ról RBAC (Role-Based Access Control) w środowisku Azure.
