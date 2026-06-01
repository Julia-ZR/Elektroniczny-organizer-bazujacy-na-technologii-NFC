# NFC Warehouse Organizer

## Opis projektu

Projekt realizowany w ramach pracy dyplomowej zakłada stworzenie organizera logistyczno-ewidencyjnego opartego na technologii NFC. System umożliwia szybkie lokalizowanie zasobów i produktów w magazynach, przedsiębiorstwach oraz centrach dystrybucji, usprawniając procesy inwentaryzacyjne i zarządzanie zasobami.

Urządzenie wykorzystuje tagi NFC zawierające unikalne identyfikatory przypisane do konkretnych lokalizacji magazynowych. Po zeskanowaniu tagu przez czytnik system odczytuje identyfikator i pobiera szczegółowe informacje zapisane lokalnie na karcie microSD. Dane przechowywane są w plikach tekstowych `.txt`, co pozwala na łatwą organizację, edycję oraz tworzenie kopii zapasowych.

Zastosowane rozwiązanie zwiększa bezpieczeństwo danych — odczyt tagu NFC za pomocą zewnętrznego urządzenia ujawnia jedynie identyfikator, bez dostępu do szczegółowych informacji o zasobach.

Tagi NFC mogą być montowane na:

* szafkach,
* szufladach,
* pojemnikach,
* regałach magazynowych.

Po zeskanowaniu tagu użytkownik otrzymuje informacje o lokalizacji produktu w formacie:

```txt
<nr_szafki>_<nr_szuflady>_<nr_pojemnika>
```

Przykład:

```txt
520_030_210
```

Możliwe jest również rozszerzenie struktury lokalizacji, np. o numer alejki magazynowej.

---

# Architektura systemu

System został zaprojektowany w oparciu o dwie współpracujące płytki STM32:

* główną jednostkę wykonawczą,
* moduł peryferyjny odpowiedzialny za komunikację NFC i obsługę pamięci.

## Wykorzystane podzespoły

<img width="739" height="258" alt="image" src="https://github.com/user-attachments/assets/26991d6f-f392-4b6b-bc62-5ad83e38935c" />

* STM32F769I-DISCO (główna jednostka wykonawcza)
* NUCLEO-L476RG
* X-NUCLEO-NFC08A1
* czytnik kart microSD Waveshare 3947

---

## Główna jednostka wykonawcza

Centralnym elementem systemu jest STM32F769I-DISCO. Płytka została wybrana ze względu na wysoką wydajność, wbudowany wyświetlacz graficzny LCD oraz natywne wsparcie dla frameworka TouchGFX.

STM32F769I-DISCO odpowiada za:

* obsługę interfejsu graficznego (GUI),
* wyświetlanie danych na ekranie dotykowym,
* zarządzanie logiką systemu,
* komunikację z modułem peryferyjnym.

Interfejs użytkownika został zaprojektowany z wykorzystaniem TouchGFX, co umożliwia wygodne zarządzanie zasobami oraz intuicyjną obsługę urządzenia.

---

## Moduł peryferyjny

Funkcję modułu komunikacyjnego i magazynowego pełni NUCLEO-L476RG.

Moduł odpowiada za:

* obsługę tagów NFC,
* komunikację z kartą microSD,
* odczyt i zapis danych,
* przesyłanie informacji do głównej jednostki wykonawczej.

Ze względu na ograniczoną liczbę dostępnych pinów GPIO w STM32F769I-DISCO oraz duże wykorzystanie zasobów przez wyświetlacz LCD i TouchGFX, zdecydowano się na rozdzielenie funkcjonalności systemu pomiędzy dwa mikrokontrolery.

Takie podejście pozwoliło:

* zwiększyć modularność projektu,
* uprościć integrację komponentów,
* poprawić stabilność działania,
* umożliwić dalszą rozbudowę systemu.

---

# Komunikacja między modułami

Komunikacja pomiędzy płytkami realizowana jest za pomocą interfejsu UART.

Przesyłane dane obejmują:

* zawartość tagów NFC,
* dane zapisane na karcie microSD,
* status gotowości modułów,
* komunikaty sterujące systemem.

Główna jednostka wykonawcza wysyła również polecenia sterujące do modułu peryferyjnego.

---

# Schemat blokowy

<img width="709" height="618" alt="image" src="https://github.com/user-attachments/assets/c52156f0-e6ae-4cec-85b6-9fc4d17ea8d8" />

---

# Obsługa NFC

Płytka NUCLEO-L476RG została rozszerzona o nakładkę X-NUCLEO-NFC08A1 wykorzystującą interfejs SPI oraz układ ST25R3916 obsługujący standard NFC.

Komunikacja z modułem NFC realizowana jest przy pomocy biblioteki X-CUBE-NFC6.

System umożliwia:

* odczyt tagów NFC,
* zapis danych na tagach,
* obsługę wiadomości NDEF,
* identyfikację tagów na podstawie UID.

Urządzenie obsługuje tagi NFC o maksymalnej pojemności 256 B.

W pamięci tagów przechowywane są:

* UID tagu,
* wiadomość NDEF zawierająca:

  * numer szafki,
  * listę produktów,
  * liczbę dostępnych elementów.

---

# Obsługa pamięci microSD

Do modułu peryferyjnego podłączono czytnik kart microSD Waveshare 3947 wykorzystujący interfejs SDMMC.

Karta microSD pełni funkcję lokalnej bazy danych systemu i przechowuje:

* informacje o stanach magazynowych,
* przypisane lokalizacje,
* listy produktów,
* konfigurację systemu.

Przechowywanie danych na wymiennej pamięci umożliwia:

* łatwe tworzenie kopii zapasowych,
* eksport danych do arkuszy kalkulacyjnych,
* przenoszenie danych pomiędzy urządzeniami,
* zabezpieczenie informacji przed utratą w przypadku awarii urządzenia.

---

# Oprogramowanie

Oprogramowanie układowe dla obu mikrokontrolerów zostało przygotowane w środowisku STM32CubeIDE. Wstępna konfiguracja peryferiów, interfejsów komunikacyjnych oraz bibliotek została wykonana przy pomocy STM32CubeMX.

W projekcie wykorzystano kilka kluczowych bibliotek i frameworków:

* **TouchGFX** – odpowiada za obsługę nowoczesnego interfejsu graficznego oraz wyświetlanie danych na ekranie dotykowym LCD.
* **FreeRTOS** – system operacyjny czasu rzeczywistego odpowiedzialny za rozdzielenie zadań GUI i komunikacji UART, dzięki czemu interfejs pozostaje responsywny podczas obsługi NFC i pamięci.
* **FatFs** – biblioteka obsługująca system plików FAT32 na kartach microSD. Wykorzystywana do odczytu i zapisu plików magazynowych `.txt`.
* **X-CUBE-NFC6** – pakiet bibliotek ST do obsługi czytnika NFC ST25R3916 oraz komunikacji z tagami NFC.
* **HAL (Hardware Abstraction Layer)** – warstwa abstrakcji sprzętowej upraszczająca obsługę interfejsów UART, SPI, SDMMC i GPIO.

---

# Funkcjonalności systemu

System umożliwia:

* szybkie lokalizowanie produktów,
* dodawanie nowych elementów do ewidencji,
* usuwanie istniejących wpisów,
* aktualizację stanów magazynowych,
* automatyzację procesów inwentaryzacyjnych,
* integrację z systemami ERP,
* przeglądanie danych na ekranie dotykowym.

Automatyzacja procesów magazynowych pozwala ograniczyć błędy logistyczne i ludzkie oraz zwiększyć kontrolę nad zasobami przedsiębiorstwa.

---

# Główne założenia projektowe

* wykorzystanie powszechnie dostępnych systemów mikroprocesorowych,
* implementacja oprogramowania w języku C/C++,
* możliwość dalszej rozbudowy firmware’u,
* zastosowanie bezpłatnych narzędzi programistycznych,
* prezentacja danych na czytelnym wyświetlaczu LCD,
* przygotowanie GUI przy pomocy TouchGFX,
* obsługa tagów NFC typu 4 i 5,
* obsługa kart microSD,
* możliwość ewidencjonowania do:

  * 40 elementów,
  * 40 szafek,
  * 40 szuflad.

---
# Kompletny system
<img width="618" height="482" alt="image" src="https://github.com/user-attachments/assets/3eaee02a-b487-4a9c-8c05-f39c86526a8a" />

Urządzenie z obudową

<img width="484" height="482" alt="image" src="https://github.com/user-attachments/assets/4fc9a6af-0bdd-455f-9e3f-c88032e6d225" />

Urządzenie bez obudowy

---
# Obudowa
Obudowa została wykonana w programie Fusion i składa się z dwóch części – pokrywy i obudowy. Do realizacji projektu wykorzystano następujące narzędzia 
* wyciągania 
* wycinania 
* robienia otworu
* zaokrąglenia narożników
<img width="665" height="459" alt="image" src="https://github.com/user-attachments/assets/a3b8042a-1670-4799-a1c1-c956de48019d" />

Wnętrze obudowy

---
# Algorytm pracy programu
<img width="1387" height="424" alt="image" src="https://github.com/user-attachments/assets/90c96d44-ef8a-409d-ac5c-44c014a36bb1" />


---
# Badania testowe
Wykonano następujące testy jednostkowe:
* komunikacji z komputerem PC
* komunikacji UART między dwoma płytkami
* odczytu tagu NFC
* odczytu z karty μSD
* wyświetlacza (biblioteka TouchGFX)

Testy dla każdego komponentu przebiegły pomyślnie.

W ramach badań funkcjonalnych wykonano test działania systemu, który potwierdził poprawne działanie docelowego systemu.
<img width="638" height="478" alt="image" src="https://github.com/user-attachments/assets/377fcf1b-f6fe-482f-8d92-ca0caf2b2cbd" />

Działający docelowy system

---
# Badanie poboru prądu

Badanie poboru prądu przeprowadzono za pomocą urządzenia Keweisi KWS-MX18L, które jest testerem USB z wyświetlaczem IPS. Na podstawie pomiarów sporządzono wykres poboru prądu w funkcji czasu.

<img width="366" height="300" alt="image" src="https://github.com/user-attachments/assets/7289b7e2-55c1-486e-9231-ad814ae27df2" />

Keweisi KWS-MX18L

<img width="740" height="433" alt="image" src="https://github.com/user-attachments/assets/eed83ae5-3430-430f-b0bf-6a8b1ded2b86" />

Wykres poboru prądu w funkcji czasu

Ponadto oszacowano stopień teoretycznego wykorzystania akumulatora, który wynosił 49,44 % dla powerbanka o pojemności 20 000 mAh.

---
# Potencjalne zastosowania
* magazyny,
* centra logistyczne,
* przedsiębiorstwa produkcyjne,
* archiwizacja zasobów,
* automatyzacja inwentaryzacji,
* zarządzanie wyposażeniem i częściami.
---
# Podsumowanie
Zaprojektowano i praktycznie wykonano elektroniczny organizer, w którym występuje obsługa:
* NFC
* TouchGFX
* karty pamięci

Zaprojektowano dedykowaną obudowę urządzenia oraz przetestowano urządzenie pod kątem testów jednostkowych, testów działania systemu i poboru prądu.

Dalsze prace:
* opracowanie projektu płytki PCB
* automatyczne generowanie raportów
* wykresy deficytów produktów

