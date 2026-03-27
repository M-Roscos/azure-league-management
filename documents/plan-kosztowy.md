# Plan Kosztowy i Wybór Usług (Architektura Azure)

Zgodnie z przyjętymi założeniami biznesowymi oraz architektonicznymi (MVP, rygorystyczny limit 100 USD dla konta studenckiego, podejście "Security First"), zaprojektowano architekturę opartą na usługach typu PaaS (Platform as a Service) oraz Serverless. Pozwala to na maksymalną redukcję kosztów operacyjnych przy jednoczesnym zachowaniu wysokich standardów bezpieczeństwa i skalowalności w przyszłości.

Aby zmieścić się w budżecie 100 USD (który w przypadku kont studenckich zazwyczaj musi wystarczyć na cały rok), architekturę oparto w głównej mierze na bezpłatnych wariantach usług (Free Tier) oraz zasobach o przewidywalnym, minimalnym koszcie miesięcznym.

## 1. Tabela Kosztów (Szacunek Miesięczny)

| Kategoria | Usługa Azure | Wybrana Warstwa (Tier) | Szacowany Koszt Miesięczny (USD) | Główne Zadanie w Systemie |
| :--- | :--- | :--- | :--- | :--- |
| **Aplikacja (Compute)** | Azure App Service | F1 (Free) | $0.00 | Hosting aplikacji backendowej (API) oraz ewentualnego interfejsu webowego (SPA). |
| **Baza Danych** | Azure SQL Database | Basic (5 DTU, 2GB) | ~$4.90 | Relacyjna baza danych przechowująca informacje o drużynach, zawodnikach i statystykach. |
| **Bezpieczeństwo** | Azure Key Vault | Standard | ~$0.10 | Bezpieczne przechowywanie sekretów, ciągów połączeń (connection strings) oraz kluczy API. |
| **Tożsamość** | Microsoft Entra ID | Free | $0.00 | Zarządzanie tożsamością, uwierzytelnianie i autoryzacja użytkowników/administratorów. |
| **Monitorowanie** | Azure Application Insights | Pay-As-You-Go | $0.00* | Zbieranie logów, monitorowanie wydajności aplikacji i wykrywanie anomalii. |
| **Całkowity koszt** | | | **~$5.00 / miesiąc** | **Zapas budżetu do końca roku: ~$40.00** |

*\* Koszt wynosi 0 USD pod warunkiem nieprzekroczenia darmowego limitu logów (5 GB/miesiąc), co w fazie MVP jest wysoce prawdopodobne.*

---

## 2. Uzasadnienie Wyboru Usług Chmurowych

### Azure App Service (Warstwa F1)
Wybrano usługę App Service ze względu na pełne zarządzanie środowiskiem uruchomieniowym (PaaS). Odciąża to zespół z zadań administracyjnych związanych z systemem operacyjnym. Warstwa darmowa (F1) udostępnia wystarczające zasoby (m.in. 1 GB RAM, 60 minut współdzielonego czasu obliczeniowego dziennie) do obsługi wczesnej fazy MVP, testów i demonstracji działania logiki biznesowej dla systemu "Liga".

### Azure SQL Database (Warstwa Basic / 5 DTU)
Logika biznesowa systemu – opierająca się na relacjach pomiędzy zawodnikami, drużynami i rozegranymi meczami (wyliczanie sumy punktów, bramek, analiza skuteczności przeciwko konkretnym przeciwnikom) – naturalnie predysponuje system do użycia relacyjnej bazy danych. Azure SQL Database zapewnia wbudowane mechanizmy bezpieczeństwa (szyfrowanie w spoczynku - TDE, firewall na poziomie serwera). Warstwa Basic oferuje stały, przewidywalny koszt poniżej 5 USD miesięcznie, gwarantując jednocześnie wystarczającą wydajność dla celów projektowych.

### Azure Key Vault (Warstwa Standard)
Zgodnie z wymaganiem "Security First", w kodzie źródłowym nie mogą znajdować się żadne poświadczenia. Zaprojektowano wykorzystanie Azure Key Vault jako centralnego magazynu certyfikatów i haseł (np. *SQL Connection String*). Aplikacja w Azure App Service uwierzytelnia się w Key Vault za pomocą Managed Identity (Zarządzanej Tożsamości), co całkowicie eliminuje potrzebę ręcznego zarządzania poświadczeniami. Koszt usługi to zaledwie ułamki centów za 10 000 operacji kryptograficznych.

### Microsoft Entra ID (dawniej Azure AD)
Wykorzystano darmową warstwę usługi zarządzania tożsamością w celu implementacji autoryzacji opartej na rolach (RBAC). Pozwala to na bezpieczne odseparowanie uprawnień administratorów (dodawanie wyników meczów) od użytkowników standardowych (przeglądanie statystyk najlepszych drużyn i zawodników).

### Azure Application Insights
Zintegrowano usługę monitoringu w celu śledzenia żądań HTTP, powolnych zapytań do bazy danych (co jest kluczowe przy wyliczaniu skomplikowanych statystyk zawodników) oraz ewentualnych wyjątków aplikacyjnych. Bezpieczeństwo jest wspierane poprzez alerty reagujące na nietypowy ruch lub błędy autoryzacji.
