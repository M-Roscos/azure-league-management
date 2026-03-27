# Plan Kosztowy i Wybór Usług (Architektura Webowa i Bezpieczeństwo)

Zgodnie z wymaganiami biznesowymi, ograniczeniami budżetowymi (limit 100 USD w ramach konta studenckiego), zaprojektowano architekturę opartą na usługach typu PaaS (Platform as a Service) w chmurze Microsoft Azure. Wybrane rozwiązanie minimalizuje koszty operacyjne, jednocześnie dostarczając wbudowane mechanizmy ochrony na poziomie sieci, aplikacji oraz danych (zgodnie z wytycznymi OWASP).

Poniższe zestawienie prezentuje szacowane miesięczne koszty utrzymania infrastruktury, pozwalające na bezpieczne działanie aplikacji przez cały rok w ramach przydzielonych środków.

## 1. Tabela Kosztów (Szacunek Miesięczny)

| Kategoria | Usługa Azure | Wybrana Warstwa (Tier) | Szacowany Koszt Miesięczny (USD) | Główne Zadanie w Systemie i Aspekty Bezpieczeństwa |
| :--- | :--- | :--- | :--- | :--- |
| **Hosting Webowy (Compute)** | Azure App Service | F1 (Free) | $0.00 | Hosting aplikacji webowej. Zapewnia izolację środowiska, automatyczne łatki OS oraz wymuszenie szyfrowania w locie (HTTPS/TLS 1.2+). |
| **Baza Danych** | Azure SQL Database | Basic (5 DTU, 2GB) | ~$4.90 | Przechowywanie danych ligowych i statystyk. Wymuszone szyfrowanie w spoczynku (TDE) oraz ochrona zaporą sieciową (Firewall). |
| **Zarządzanie Sekretami** | Azure Key Vault | Standard | ~$0.10 | Bezpieczny magazyn kluczy API, certyfikatów i ciągów połączeń (connection strings), chroniący przed ich wyciekiem w kodzie źródłowym. |
| **Zarządzanie Tożsamością** | Microsoft Entra ID | Free | $0.00 | Realizacja uwierzytelniania i autoryzacji (RBAC) dla paneli administracyjnych, zabezpieczająca przed nieuprawnionym dostępem. |
| **Monitorowanie i Audyt** | Azure Application Insights | Pay-As-You-Go | $0.00* | Agregacja logów bezpieczeństwa, wykrywanie anomalii w ruchu webowym oraz monitorowanie wydajności aplikacji. |
| **Całkowity koszt** | | | **~$5.00 / miesiąc** | **Zapas budżetu (rocznie): ~$40.00 (z limitu 100 USD)** |

*\* Koszt wynosi 0 USD pod warunkiem nieprzekroczenia darmowego limitu logów (5 GB/miesiąc), co w fazie MVP jest gwarantowane.*

---

## 2. Uzasadnienie Architektoniczne i Cyberbezpieczeństwo

### Azure App Service (Warstwa F1 - Free)
Wykorzystano usługę App Service do hostowania aplikacji webowej, co zdejmuje z zespołu obowiązek zarządzania i zabezpieczania systemu operacyjnego (PaaS). Środowisko to zostało domyślnie skonfigurowane tak, aby wymuszać komunikację wyłącznie po protokole HTTPS (TLS 1.2 lub nowszym), co chroni dane w locie przed atakami typu Man-in-the-Middle (MitM). Dodatkowo, framework wykorzystany w aplikacji webowej został skonfigurowany do obrony przed atakami **XSS (Cross-Site Scripting)** poprzez automatyczną sanityzację danych wejściowych i wyjściowych, a także do implementacji tokenów anty-**CSRF (Cross-Site Request Forgery)** dla wszystkich formularzy zmieniających stan aplikacji.

### Azure SQL Database (Warstwa Basic)
Relacyjna baza danych jest optymalnym wyborem dla logiki systemu "Liga", wymagającej wykonywania złożonych zapytań agregujących (np. wyliczanie zawodnika, który strzelił najwięcej bramek przeciwko konkretnej drużynie). Warstwa Basic oferuje przewidywalny, stały koszt. W kontekście bezpieczeństwa wdrożono mechanizm **TDE (Transparent Data Encryption)**, który automatycznie szyfruje bazę danych, kopie zapasowe oraz logi transakcyjne w spoczynku. Co najważniejsze, zaimplementowano warstwę dostępu do danych opartą na nowoczesnym rozwiązaniu ORM (Object-Relational Mapping), co zapewnia pełną parametryzację zapytań i całkowicie eliminuje ryzyko ataków **SQL Injection**.

### Azure Key Vault (Warstwa Standard) i Konfiguracja
Zgodnie z najlepszymi praktykami bezpieczeństwa, w kodzie aplikacji nie umieszczono żadnych jawnych danych uwierzytelniających. Zastosowano usługę Azure Key Vault jako scentralizowany magazyn sekretów. Aplikacja webowa zyskała dostęp do tych sekretów za pomocą mechanizmu **Managed Identity** (Zarządzanej Tożsamości) w chmurze Microsoft Entra ID. Zapewnia to bezpieczny proces pobierania konfiguracji w czasie uruchamiania aplikacji, bez konieczności zarządzania dodatkowymi hasłami dostępowymi.

### Monitorowanie i Identyfikacja Zagrożeń (Application Insights)
Zintegrowano usługę telemetryczną w celu monitorowania nie tylko wydajności, ale również bezpieczeństwa. System rejestruje wszystkie próby nieautoryzowanego dostępu, błędy aplikacji (które mogłyby ujawnić wrażliwe dane o strukturze bazy) oraz podejrzane wzorce ruchu webowego.

Podsumowując, zaproponowany stos technologiczny gwarantuje utrzymanie kosztów na poziomie zaledwie ~60 USD rocznie, spełniając rygorystyczne wymagania budżetowe, a jednocześnie wdraża wielowarstwową ochronę zgodną z wytycznymi OWASP Top 10 i standardami chmurowymi Microsoft Azure.
