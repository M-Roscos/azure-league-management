# azure-league-management
System zarządzania ligą sportową – projekt oparty na architekturze Serverless oraz usługach platformowych (PaaS) w Microsoft Azure.

# Plan Kosztowy i Architektura Bezpieczeństwa

Zamiast wdrażania dedykowanych maszyn wirtualnych, architektura systemu "Liga" opiera się na usługach platformowych (PaaS) i bezserwerowych (Serverless) w środowisku Microsoft Azure. Pozwala to na redukcję kosztów do absolutnego minimum, przy jednoczesnym podniesieniu poziomu bezpieczeństwa i ułatwieniu zarządzania.

## 1. Szacunkowy plan kosztowy i dobór usług Azure

Poniższa tabela prezentuje zestawienie usług chmurowych dobranych dla środowiska MVP (Minimum Viable Product).

| Usługa Azure | Wybrana Warstwa (SKU) | Szacowany Koszt (Miesięcznie) | Uzasadnienie i Aspekty Bezpieczeństwa |
| :--- | :--- | :--- | :--- |
| **Azure Static Web Apps** | Free (Darmowa) | **0,00 USD** | **Uzasadnienie:** Usługa zostanie wykorzystana do hostowania frontendu (np. React/Angular) oraz zintegrowanego backendu API (uruchamianego jako Serverless Azure Functions). Zapewniona jest natywna integracja z GitHub Actions dla procesów CI/CD.<br><br>**Bezpieczeństwo:** Rozwiązanie wymusza ruch sieciowy po protokole HTTPS, automatycznie odnawia certyfikaty SSL/TLS oraz pozwala na bezpieczną integrację logowania z Microsoft Entra ID. |
| **Azure SQL Database** | Basic Tier (5 DTU) | **~5,00 USD** | **Uzasadnienie:** Realizacja wymagań funkcjonalnych (m.in. wyłanianie najlepszego strzelca przeciwko danej drużynie) opiera się na relacjach i złączeniach (JOIN) tabel (*Mecze, Zawodnicy, Drużyny, Gole*). Wybór bazy relacyjnej jest optymalny dla tego scenariusza. Warstwa Basic (2 GB miejsca) jest w zupełności wystarczająca dla skali aplikacji w fazie MVP i pozwala na optymalizację kosztów budżetu studenckiego.<br><br>**Bezpieczeństwo:** Zastosowano szyfrowanie w spoczynku (TDE - Transparent Data Encryption). Baza danych zostanie zabezpieczona zaporą sieciową (Azure SQL Firewall) i udostępniona wyłącznie dla wewnętrznego API. |
| **Azure Key Vault** | Standard | **~0,05 USD** | **Uzasadnienie:** Magazyn zostanie użyty do bezpiecznego przechowywania kluczy kryptograficznych oraz poświadczeń. Zamiast przechowywać parametry logowania w repozytorium kodu, API będzie pobierać je dynamicznie w czasie działania.<br><br>**Bezpieczeństwo:** Zostanie zaimplementowany mechanizm tożsamości zarządzanej (Managed Identity), co umożliwi uwierzytelnienie API do bazy danych i magazynu kluczy bez konieczności używania tradycyjnych haseł w kodzie aplikacji. |
| **Microsoft Entra ID** | Free | **0,00 USD** | **Uzasadnienie:** Zapewni zarządzanie tożsamością w aplikacji (np. uwierzytelnianie administratorów ligi wprowadzających wyniki meczów).<br><br>**Bezpieczeństwo:** Wdrożony zostanie nowoczesny standard uwierzytelniania oparty na protokołach OAuth 2.0 / OpenID Connect, wraz z możliwością wymuszenia uwierzytelniania wieloskładnikowego (MFA). |
| **Application Insights / Azure Monitor** | Pay-As-You-Go | **0,00 USD** | **Uzasadnienie:** Narzędzie posłuży do monitorowania wydajności i stabilności działania aplikacji oraz API.<br><br>**Bezpieczeństwo:** Umożliwia dokładne logowanie (audyt) wszystkich zdarzeń, wyjątków i prób nieautoryzowanego dostępu. Przestrzeń do 5 GB logów miesięcznie jest dostępna bezpłatnie. |
| **GitHub** | Free | **0,00 USD** | **Uzasadnienie:** Platforma posłuży jako repozytorium kodu oraz środowisko uruchomieniowe dla zautomatyzowanych potoków wdrożeniowych (GitHub Actions).<br><br>**Bezpieczeństwo:** Zostanie skonfigurowane narzędzie *Dependabot* w celu automatycznego skanowania podatności w bibliotekach (CVE) oraz funkcja *Secret Scanning*, zapobiegająca przypadkowemu upublicznieniu kluczy i certyfikatów w kodzie źródłowym. |

---

## 2. Podsumowanie budżetu

* **Miesięczny koszt środowiska:** ok. **5,05 USD**
* **Roczny koszt środowiska:** ok. **60,60 USD**

Szacunki potwierdzają, że zaprojektowana infrastruktura pozwala na pełną realizację projektu w ramach przyznanego limitu studenckiego (100 USD), pozostawiając margines budżetowy na wypadek zwiększonego użycia, testów obciążeniowych lub skalowania bazy danych.

---

## 3. Kluczowe założenia architektury bezpieczeństwa (Security-First)

W projekcie wdrożono rygorystyczne standardy bezpieczeństwa chmurowego, ze szczególnym uwzględnieniem następujących koncepcji:

1. **Podejście Zero Trust i Tożsamości Zarządzane (Managed Identities):** Aplikacja (API) uwierzytelnia się do bazy Azure SQL oraz usługi Key Vault wyłącznie za pomocą swojej wewnętrznej, przypisanej w platformie Azure tożsamości. Całkowicie eliminuje to zjawisko przechowywania ciągów połączeń (*Connection Strings*) w kodzie lub plikach konfiguracyjnych (wdrożenie tzw. *secretless architecture*).
2. **Pełne szyfrowanie danych (Encryption):** Zapewniono szyfrowanie danych w tranzycie (wymuszony standard TLS 1.2/1.3 w ramach Azure Static Web Apps) oraz szyfrowanie danych w spoczynku (aktywny mechanizm TDE dla bazy Azure SQL).
3. **Zasada najmniejszego uprzywilejowania (PoLP - Principle of Least Privilege):** Interfejs użytkownika (Frontend) posiada uprawnienia komunikacyjne wyłącznie z API. API dysponuje dostępem jedynie do wybranych zasobów bazy danych (dzięki precyzyjnie przypisanym rolom w modelu RBAC - Role-Based Access Control). Sama baza danych posiada blokadę dostępu z poziomu publicznej sieci Internet.
