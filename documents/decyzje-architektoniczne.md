# Dokumentacja Decyzji Architektonicznych (ADR) - System "Liga"

Poniższy dokument rejestruje kluczowe decyzje architektoniczne podjęte podczas projektowania webowego systemu statystyk sportowych "Liga". Każdy wpis (Record) definiuje kontekst biznesowy, podjętą decyzję technologiczną oraz jej konsekwencje dla bezpieczeństwa i kosztów.

---

## ADR-001: Wybór platformy hostingowej dla aplikacji webowej

**Kontekst:** System wymaga niezawodnego środowiska uruchomieniowego dla aplikacji webowej (WebApp). Z powodu rygorystycznego budżetu (limit 100 USD rocznie na koncie studenckim) wykluczono klastry Kubernetes (AKS) oraz dedykowane maszyny wirtualne (IaaS), które generują wysokie koszty stałe i wymagają zaawansowanego zarządzania poprawkami bezpieczeństwa (patch management).

**Decyzja:** Wybrano usługę **Azure App Service w warstwie darmowej (F1 - Free)**.

**Konsekwencje:**
* **Pozytywne (Biznes):** Zredukowano koszty hostingu do 0 USD. Zapewniono szybki czas wdrożenia dzięki architekturze PaaS.
* **Pozytywne (Security):** System operacyjny jest zarządzany i łatany przez Microsoft. Wymuszono na poziomie konfiguracji usługi komunikację wyłącznie za pomocą protokołu HTTPS (TLS 1.2+), co skutecznie mityguje ryzyko ataków Man-in-the-Middle (MitM) bez konieczności dodatkowej konfiguracji serwera WWW.
* **Negatywne:** Warstwa F1 posiada limity obliczeniowe (60 minut CPU na dzień), co jest wystarczające dla fazy MVP, ale będzie wymagało skalowania (Scale-Up) przy przejściu na produkcję.

---

## ADR-002: Wybór relacyjnej bazy danych i mitygacja SQL Injection

**Kontekst:** Logika biznesowa systemu opiera się na relacjach (drużyna posiada zawodników, zawodnicy zdobywają punkty/bramki w konkretnych meczach przeciwko innym drużynom). Wymagane jest optymalne wyliczanie skomplikowanych statystyk (np. "najlepszy zawodnik przeciwko konkretnej drużynie"). Baza danych musi być również odporna na najczęstsze ataki webowe (OWASP Top 10).

**Decyzja:** Zaprojektowano wykorzystanie **Azure SQL Database w warstwie Basic (5 DTU)** w połączeniu z warstwą dostępu do danych opartą na systemie mapowania obiektowo-relacyjnego (**ORM**).

**Konsekwencje:**
* **Pozytywne (Biznes):** Miesięczny koszt utrzymany na poziomie ok. 4.90 USD. Baza relacyjna natywnie i bardzo wydajnie obsługuje agregacje niezbędne do wyliczania rankingów (GROUP BY, JOIN).
* **Pozytywne (Security):** Wdrożono na poziomie bazy mechanizm **TDE (Transparent Data Encryption)**, szyfrujący dane w spoczynku.
  * Zastosowanie ORM gwarantuje pełną parametryzację zapytań bazodanowych. Bezpośrednio eliminuje to możliwość wstrzyknięcia złośliwego kodu SQL przez użytkownika (ochrona przed **SQL Injection**).
* **Negatywne:** Wymagany jest staranny projekt indeksów, aby zapytania analityczne nie wyczerpały limitu 5 DTU w warstwie Basic.

---

## ADR-003: Zarządzanie sekretami i konfiguracją

**Kontekst:** Aplikacja musi łączyć się z bazą danych oraz zewnętrznymi usługami. Przechowywanie ciągów połączeń (connection strings) i kluczy API w kodzie źródłowym (hardcoding) lub w nieszyfrowanych plikach konfiguracyjnych jest krytycznym błędem bezpieczeństwa, często prowadzącym do całkowitego przejęcia systemu.

**Decyzja:** Zaimplementowano **Azure Key Vault (warstwa Standard)** do przechowywania wszystkich sekretów. Aplikacja webowa otrzymuje do nich dostęp za pomocą **Managed Identity** (Zarządzanej Tożsamości) w Microsoft Entra ID.

**Konsekwencje:**
* **Pozytywne (Biznes):** Bardzo niski koszt usługi (~0.10 USD/miesiąc). Uproszczenie zarządzania dostępem w zespole projektowym.
* **Pozytywne (Security):** Żadne hasła ani klucze nie są widoczne w repozytorium kodu. Nawet w przypadku wycieku kodu źródłowego napastnik nie uzyska dostępu do bazy danych, ponieważ tożsamość aplikacji jest przypisana bezpośrednio do zasobu Azure App Service.
* **Negatywne:** Dodatkowa złożoność w lokalnym środowisku programistycznym (wymaga logowania przez Azure CLI w celu pobierania sekretów z deweloperskiej instancji Key Vault).

---

## ADR-004: Ochrona frontendu przed atakami XSS i CSRF

**Kontekst:** Panel administracyjny pozwalający na wprowadzanie wyników meczów (mutacja stanu aplikacji) jest narażony na ataki polegające na wstrzyknięciu złośliwego kodu JavaScript (XSS) lub wymuszeniu nieautoryzowanych akcji (CSRF).

**Decyzja:** Wybrano nowoczesny framework webowy renderujący widoki po stronie serwera (np. ASP.NET Core MVC / Razor lub odpowiednik dla innego języka), który posiada wbudowane mechanizmy sanityzacji.

**Konsekwencje:**
* **Pozytywne (Security):** Wszystkie dane wprowadzane przez użytkowników (np. nazwy drużyn, nazwiska zawodników) są automatycznie kodowane w HTML przed wyświetleniem, co neutralizuje wektory ataków **XSS (Cross-Site Scripting)**.
  * Do każdego formularza POST wstrzykiwany jest kryptograficzny token weryfikacyjny (*Anti-Forgery Token*). System weryfikuje jego zgodność z ciasteczkiem sesyjnym, co stanowi pełną ochronę przed **CSRF (Cross-Site Request Forgery)**.
