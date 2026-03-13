### Dokument Architektury IT: Platforma Dopasowania Opiekunów

#### 1\. Fundamenty Architektoniczne i Cele Systemu

Nadrzędnym celem technicznym platformy jest implementacja  **warstwy zaufanego dostępu**  ( *trusted care access layer* ). Architektura musi realizować model „managed marketplace”, w którym system nie pełni jedynie roli pasywnej tablicy ogłoszeń, lecz aktywnie zarządza ryzykiem, weryfikacją i ciągłością świadczonych usług.Kluczowe parametry systemu:

* **Niezawodność (Reliability):**  Mechanizmy redundancji w puli opiekunów, zapewniające ciągłość opieki w modelu 24/7.  
* **Automatyzacja weryfikacji:**  Zintegrowany rurociąg walidacji poświadczeń (vetting pipeline) jako warunek  *sine qua non*  dopuszczenia do transakcji.  
* **Skalowalność transgraniczna:**  Architektura przygotowana na korytarz Polska-Niemcy/Austria, uwzględniająca specyfikę lokalnych rynków (np. silnie zinstytucjonalizowany rynek austriacki).  
* **Integralność danych:**  Rygorystyczne śledzenie historii dopasowań i wskaźników jakościowych w celu minimalizacji tzw. „breakdown risk” (ryzyka przerwania opieki).  
* **Compliance by Design:**  Automatyzacja raportowania DAC7 oraz obsługi dokumentacji delegowania od momentu inicjalizacji transakcji.

#### 2\. Architektura Danych i Struktury Bazodanowe

Baza danych (PostgreSQL) wykorzystuje pola typu  **JSONB**  dla encji „Potrzeby Opiekuńcze”, co pozwala na elastyczne definiowanie parametrów zdrowotnych (demencja, mobilność, inkontynencja) bez konieczności kosztownych migracji schematu przy ekspansji na nowe rynki.| Kluczowa Encja | Wymagane Pola Danych | Cel Biznesowy (Compliance/Dopasowanie) || \------ | \------ | \------ || **Profil Opiekuna** | Poziom językowy (progresja), certyfikaty (Dementia/Mental Health), historia re-matchingów, wskaźnik retencji, lokalizacja. | Precyzyjne dopasowanie i unikanie „breakdowns”. Śledzenie progresji językowej pod kątem wyższych stawek. || **Profil Rodziny** | Typ opieki (godzinowa/całodobowa), JSONB: Specyfika schorzeń, preferencje, walidacja budżetu (PL: 50–60 PLN/h; DE: €2,500–€3,500/mc). | Ustalanie „Price-floor” i „Price-ceiling” zgodnie z rynkowymi kotwicami cenowymi. || **Moduł Transakcyjny** | **Financial Account Identifier (IBAN)** ,  **Total Consideration** , Rezydencja podatkowa, NIP/PESEL, status wypłaty. | Pełna zgodność z raportowaniem  **DAC7** ; automatyczne rurociągi danych dla organów skarbowych. |

#### 3\. Wymagania Funkcjonalne – Faza 1: MVP (Poland-First)

Faza MVP koncentruje się na aglomeracjach (Warszawa, Kraków, Wrocław). Architektura musi wspierać „ścieżkę badawczo-rozwojową” (R\&D storyline) wymaganą dla grantów takich jak  **Ścieżka SMART**  czy  **Startup Booster Poland – Smart UP** .

* **Silnik dopasowania (Basic Match):**  Filtrowanie po geolokalizacji i dostępności z uwzględnieniem podstawowych kwalifikacji.  
* *Dlaczego teraz?*  Niezbędne do wygenerowania pierwszej płynności rynkowej i zbierania danych do walidacji hipotez R\&D.  
* **System płatności i DAC7 Ready:**  Integracja z Stripe/lokalnym PSP gromadząca IBAN i dane podatkowe od pierwszej transakcji.  
* *Dlaczego teraz?*  Uniknięcie długu technicznego w raportowaniu unijnym; wymóg transparentności finansowej.  
* **Vetting Pipeline (Poziom 1):**  Cyfrowy workflow przesyłania dokumentów tożsamości i zaświadczeń o niekaralności.  
* *Dlaczego teraz?*  Budowa fundamentu zaufania w modelu „managed marketplace”.  
* **Analityka Wyników (Outcome Measurement):**  Moduł śledzący redukcję incydentów i stabilność opieki.  
* *Dlaczego teraz?*  Kluczowy parametr dla rozliczeń grantów (np. FERS) i udowodnienia innowacyjności algorytmu.

#### 4\. Wymagania Funkcjonalne – Faza 2: Skalowanie i Ekspansja (Germany/EU)

Ekspansja wymaga automatyzacji procesów transgranicznych, ze szczególnym uwzględnieniem korytarza PL → DE/AT.| Funkcjonalność | Bariera wejścia (Niemcy/Austria), którą funkcja niweluje || \------ | \------ || **Automatyzacja formularzy PD A1** | Eliminacja bariery biurokratycznej przy delegowaniu opiekunów z Polski do Niemiec. || **Wielojęzyczność Operacyjna (PL/DE)** | Bariera komunikacyjna i błędy w przekazywaniu instrukcji opiekuńczych. || **Integracja z austriackim systemem 24h** | Brak dostępu do silnie zinstytucjonalizowanego i dotowanego rynku opieki całodobowej w Austrii. || **Dynamiczne pakiety po-szpitalne** | Wysoka cena jednorazowych usług; system subskrypcyjny stabilizuje cash-flow platformy. |

#### 5\. Algorytmy Ciągłości Opieki i Zarządzania Ryzykiem

Sercem platformy jest filozofia  **„breakdown avoidance”**  – priorytetyzacja stabilności relacji nad prostym dopasowaniem cech.**Algorytm Zastępstw (Continuity Guarantee):**

* **Krok 1:**  Identyfikacja opiekunów w „standby pool” na podstawie geolokalizacji i gotowości operacyjnej.  
* **Krok 2:**  Ranking kandydatów według wskaźnika „Complexity Match” (doświadczenie w podobnych przypadkach medycznych).  
* **Krok 3:**  Automatyczne trasowanie powiadomień z gwarantowanym czasem odpowiedzi (SLA) 24-72h.  
* **Krok 4:**  Mechanizm „Case Escalation” do Panelu Koordynatora w przypadku braku automatycznego akceptu w ciągu 4h.**Predictor Przerwania Opieki (Risk Flags):**  
* **Krok 1:**  Analiza sygnałów pasywnych: brak logowań w dzienniku aktywności, spadek satysfakcji rodziny (NPS), opóźnienia w komunikacji.  
* **Krok 2:**  Analiza sygnałów aktywnych: zgłoszenia wypalenia zawodowego lub problemów zdrowotnych opiekuna.  
* **Krok 3:**  Flagowanie przypadków wysokiego ryzyka i prewencyjne uruchomienie procedury „Care Continuity Planning”.

#### 6\. Cyfrowy Portfel Poświadczeń (Digital Credential Wallet)

Moduł ten zapewnia „przenośność” profilu zawodowego opiekuna, co jest kluczowe dla retencji w korytarzu transgranicznym.**Cechy Portfela:**

* **Weryfikowalność:**  Blockchain lub szyfrowane sumy kontrolne dla certyfikatów (np. opieka w demencji, pierwsza pomoc).  
* **Language Progression Tracking:**  Śledzenie postępów językowych (PL → DE), bezpośrednio skorelowane z automatycznym wzrostem stawki godzinowej.  
* **Mental Health Support:**  Rejestr sesji wsparcia psychologicznego i szkoleń z zakresu radzenia sobie ze stresem.**Korzyści:**  
* **Dla Opiekuna:**  Budowa trwałego kapitału zawodowego; wyższe zarobki dzięki udokumentowanym kompetencjom.  
* **Dla Rodziny:**  Redukcja lęku przed nieznanym pracownikiem; pewność co do autentyczności uprawnień medycznych.

#### 7\. Bezpieczeństwo, Compliance i Raportowanie

Platforma musi rygorystycznie przestrzegać wytycznych unijnych, w tym nadchodzącej Dyrektywy o Pracy Platformowej ( *Platform Work Directive* ).**Lista kontrolna (Checklist):**

*   **Anti-Control Measures:**  System  **nie może**  jednostronnie narzucać grafików ani stosować kar algorytmicznych za odrzucenie zlecenia (unikanie domniemania stosunku pracy).  
*   **DAC7 Pipeline:**  Automatyczne pole „Total Consideration” i eksport danych do KAS w formacie XML/JSON.  
*   **Audit Logs:**  Niezmienny zapis każdej modyfikacji w profilu opiekuna i historii transakcji.  
*   **RODO (Health Data):**  Szyfrowanie danych wrażliwych seniora (Art. 9 RODO) z dostępem ograniczonym czasowo dla aktywnego opiekuna.  
*   **Rate Limiting:**  Zabezpieczenie API przed atakami typu brute-force i scrapingiem bazy opiekunów.

#### 8\. Stos Technologiczny i Integracje

Wybrany stos technologiczny ma na celu minimalizację długu technicznego przy szybkim skalowaniu z Polski na rynki DACH.| Moduł / Komponent | Sugerowana Technologia | Uzasadnienie || \------ | \------ | \------ || **Frontend** | **Next.js** | Wysokie SEO (pozyskiwanie rodzin) i wydajność UX na urządzeniach mobilnych. || **Backend** | **Node.js (NestJS)** | Skalowalność mikroserwisowa dla algorytmów dopasowania i zastępstw. || **Baza Danych** | **PostgreSQL** | Obsługa JSONB dla elastycznych potrzeb opiekuńczych i silna spójność dla transakcji. || **Płatności** | **Stripe (Split-payment)** | Obsługa płatności transgranicznych, IBAN i automatyzacja podziału marży. || **Weryfikacja (KYC)** | **Onfido / Jumio** | Automatyczna lustracja dokumentów tożsamości (ID, paszporty) w skali UE. || **Analityka BI** | **Metabase / Tableau** | Monitorowanie marży brutto na klienta (LTV/CAC) i efektywności operacyjnej. |  
