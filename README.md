# System Liga - Platforma Zarządzania i Statystyk Sportowych

[![Azure](https://img.shields.io/badge/Deploy-Azure-blue.svg)](https://azure.microsoft.com/)
[![Security](https://img.shields.io/badge/Security-OWASP%20Top%2010-brightgreen.svg)](https://owasp.org/)
[![Status](https://img.shields.io/badge/Status-MVP-orange.svg)]()

System "Liga" to nowoczesna, bezpieczna aplikacja webowa przeznaczona do kompleksowego zarządzania rozgrywkami sportowymi. Zaprojektowano ją w oparciu o chmurę Microsoft Azure, rygorystycznie optymalizując koszty (podejście MVP) oraz implementując najwyższe standardy cyberbezpieczeństwa.

## 📖 Spis Treści
- [Logika Biznesowa](#-logika-biznesowa)
- [Architektura Chmurowa](#-architektura-chmurowa)
- [Cyberbezpieczeństwo (Security First)](#-cyberbezpieczeństwo-security-first)

---

## 🎯 Logika Biznesowa

System realizuje zaawansowane operacje analityczne i statystyczne, dostarczając interesariuszom biznesowym kluczowych informacji o przebiegu rozgrywek. Zaimplementowano następujące główne moduły:

1. **Ranking Drużyn**: Automatyczne wyłanianie najlepszej drużyny na podstawie największej liczby zdobytych punktów w sezonie.
2. **Klasyfikacja Generalna Zawodników**: Wyłanianie najlepszego zawodnika turnieju poprzez agregację największej sumy zdobytych punktów i bramek we wszystkich rozegranych spotkaniach.
3. **Analityka Head-to-Head (H2H)**: Zaawansowany moduł wyłaniający zawodnika "najlepszego na daną drużynę" – system identyfikuje gracza, który grając przeciwko określonemu rywalowi, wykazał się najwyższą skutecznością (najwięcej strzelonych bramek przeciwko tej konkretnej drużynie).

---

## ☁️ Architektura Chmurowa

Projekt został zoptymalizowany pod kątem rygorystycznego limitu kosztowego (100 USD rocznie dla środowiska studenckiego), bez kompromisów w zakresie wydajności i bezpieczeństwa. Wykorzystano model PaaS (Platform as a Service):

* **Azure App Service (Free Tier)**: Odpowiada za hosting aplikacji webowej. Zdejmuje z zespołu obowiązek utrzymania systemu operacyjnego, gwarantując wysoką dostępność.
* **Azure SQL Database (Basic Tier)**: Relacyjna baza danych przechowująca w ustrukturyzowany sposób informacje o zawodnikach, meczach i statystykach. Oferuje przewidywalny koszt i wysoką wydajność dla skomplikowanych zapytań analitycznych.
* **Microsoft Entra ID (Free)**: Realizuje zarządzanie tożsamością oraz autoryzację opartą na rolach (RBAC).

---

## 🛡️ Cyberbezpieczeństwo

Zaprojektowano i zaimplementowano wielowarstwowe mechanizmy ochrony przed najpopularniejszymi wektorami ataków webowych, zgodnie ze standardami **OWASP Top 10** oraz najlepszymi praktykami Microsoft Azure:

* **Ochrona przed SQL Injection**: System w pełni separuje dane wejściowe od logiki zapytań bazodanowych. Zastosowano nowoczesny system mapowania obiektowo-relacyjnego (ORM) oraz ścisłą parametryzację zapytań.
* **Mitygacja XSS i CSRF**: Zaimplementowano automatyczną sanityzację oraz kodowanie danych wejściowych/wyjściowych w warstwie prezentacji (ochrona przed *Cross-Site Scripting*). Operacje mutujące stan aplikacji są chronione kryptograficznie generowanymi tokenami anty-CSRF (*Cross-Site Request Forgery*).
* **Bezpieczne Zarządzanie Sekretami**: Całkowicie wyeliminowano tzw. *hardcoding* danych uwierzytelniających. Wykorzystano **Azure Key Vault** do przechowywania ciągów połączeń i kluczy API. Aplikacja webowa autoryzuje się do magazynu za pomocą **Managed Identity**.
* **Szyfrowanie Danych (W locie i w spoczynku)**: 
  * Wymuszono komunikację sieciową wyłącznie za pomocą protokołu **HTTPS** z wykorzystaniem TLS 1.2 lub nowszego (ochrona przed atakami *Man-in-the-Middle*).
  * Zastosowano mechanizm **TDE (Transparent Data Encryption)** dla bazy Azure SQL, który w czasie rzeczywistym szyfruje dane, dzienniki transakcji i kopie zapasowe na dysku.
