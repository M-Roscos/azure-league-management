# ADR 002: Wybór stosu technologicznego dla warstwy backendowej i frontendowej

## 1. Kontekst i wymagania

Wdrożenie systemu "Liga" w modelu Serverless na platformie Azure wymaga doboru odpowiednich języków programowania oraz frameworków dla warstwy logiki biznesowej (API) oraz warstwy prezentacji (interfejs użytkownika). Zastosowane technologie muszą natywnie wspierać wybrane wcześniej mechanizmy bezpieczeństwa chmurowego (Managed Identities, Azure Key Vault) oraz umożliwiać bezpieczną i wydajną komunikację z relacyjną bazą danych (Azure SQL). Ponadto kod musi być łatwy w utrzymaniu i automatycznym wdrażaniu (CI/CD) poprzez GitHub Actions.

## 2. Decyzja

1. **Warstwa Backendowa (API):** Do implementacji logiki biznesowej uruchamianej w ramach usługi Azure Functions wybrano język **C#** oraz platformę **.NET 8 (LTS)** w modelu izolowanym (Isolated Worker Model).
2. **Warstwa Dostępu do Danych (ORM):** Do komunikacji z bazą Azure SQL wybrano bibliotekę **Entity Framework Core (EF Core)**.
3. **Warstwa Frontendowa (UI):** Do budowy interfejsu użytkownika hostowanego w Azure Static Web Apps wybrano bibliotekę **React** z wykorzystaniem języka **TypeScript**.

## 3. Uzasadnienie

* **Platforma .NET 8 i C#:** Jako natywna technologia firmy Microsoft, środowisko .NET oferuje bezproblemową i najlepiej udokumentowaną integrację z usługami Azure. Wykorzystanie pakietu `Azure.Identity` oraz klasy `DefaultAzureCredential` pozwala na bezobsługowe uwierzytelnianie API w usługach Key Vault i Azure SQL za pomocą tożsamości zarządzanych, spełniając tym samym restrykcyjne wymogi bezpieczeństwa (brak haseł w kodzie).
* **Entity Framework Core:** Narzędzie to znacząco przyspiesza proces mapowania obiektowo-relacyjnego. Złożone wymagania analityczne (np. wyłonienie zawodnika, który strzelił najwięcej bramek przeciwko konkretnej drużynie) można zrealizować za pomocą bezpiecznych zapytań LINQ. EF Core automatycznie parametryzuje zapytania SQL, co stanowi fundamentalne zabezpieczenie przed atakami typu SQL Injection.
* **React + TypeScript:** Zastosowanie TypeScriptu wprowadza silne typowanie po stronie frontendu, co znacząco redukuje liczbę błędów na etapie kompilacji i ułatwia integrację z typowanym API w C#. Biblioteka React stanowi standard rynkowy, a jej architektura oparta na komponentach idealnie wpisuje się w charakterystykę usługi Azure Static Web Apps.

## 4. Konsekwencje

* **Pozytywne:** Pełne bezpieczeństwo na poziomie dostępu do danych (ochrona przed SQL Injection), silne typowanie w całym cyklu przepływu danych (od bazy SQL, przez C#, aż po TypeScript), łatwość konfiguracji potoków CI/CD.
* **Negatywne (Wyzwania):** Konieczność zarządzania polityką CORS (Cross-Origin Resource Sharing) pomiędzy niezależnie hostowanym API (Azure Functions) a frontendem (Static Web Apps). Ponadto lokalne testowanie mechanizmu `DefaultAzureCredential` wymaga poprawnej konfiguracji środowiska deweloperskiego (np. zalogowania się do Azure CLI kontem posiadającym odpowiednie uprawnienia RBAC).
