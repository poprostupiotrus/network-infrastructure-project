## Wprowadzenie i Opis Projektu
Projekt obejmuje wykonanie projektu logicznego oraz konfiguracji infrastruktury sieciowej dla Centrali (HQ) oraz dwóch oddziałów zamiejscowych (Branch 1 i Branch 2). Środowisko zostało zaprojektowane 
pod kątem zapewnienia bezpiecznej komunikacji między lokalizacjami oraz pełnego monitoringu urządzeń z wykorzystaniem urządzeń Cisco oraz usług serwerowych: centralnego serwera DHCP (isc-dhcp-server) wdrożonego na systemie Debian 12 
oraz systemu monitorowania Zabbix wraz z demonem SNMP Traps (snmptrapd) działającego na systemie Ubuntu Server. Całość projektu została zrealizowana i przetestowana w środowisku symulacyjnym GNS3.
## Topologia i Plan Adresacji IP
### 1. Schemat Topologii Sieci
Struktura logiczna oraz fizyczne połączenia pomiędzy urządzeniami zrealizowane w środowisku GNS3:
![Topologia sieci w GNS3](topology/network_topology.png)
---
### 2. Plan Adresacji IP — Centrala (HQ)
#### Konwencja Adresowania Interfejsów L3 (SVI):
Dla każdej podsieci użytkowej i serwerowej w Centrali (VLAN 10 do 110) obowiązuje maska `255.255.255.0 (/24)` oraz stały schemat końcówek IP:
* **.1** -> HSRP Virtual IP (Brama domyślna dla stacji roboczych/urządzeń)
* **.2** -> CORE-SW1 (Fizyczny adres IP interfejsu SVI)
* **.3** -> CORE-SW2 (Fizyczny adres IP interfejsu SVI)
#### Tabela podsieci, rozdziału adresów oraz ról HSRP w Centrali:

| VLAN | Nazwa sieci | Podsieć IP | CORE-SW1 IP | CORE-SW2 IP | HSRP Virtual IP | Aktywny Core |
| :---: | :--- | :--- | :--- | :--- | :--- | :--- |
| **10** | MGMT | `10.0.10.0/24` | `10.0.10.2` | `10.0.10.3` | `10.0.10.1` | CORE-SW1 |
| **15** | IT_DEPT | `10.0.15.0/24` | `10.0.15.2` | `10.0.15.3` | `10.0.15.1` | CORE-SW1 |
| **20** | SW1_USERS | `10.0.20.0/24` | `10.0.20.2` | `10.0.20.3` | `10.0.20.1` | CORE-SW1 |
| **21** | SW1_VOIP | `10.0.21.0/24` | `10.0.21.2` | `10.0.21.3` | `10.0.21.1` | CORE-SW1 |
| **30** | SW2_USERS | `10.0.30.0/24` | `10.0.30.2` | `10.0.30.3` | `10.0.30.1` | CORE-SW2 |
| **31** | SW2_VOIP | `10.0.31.0/24` | `10.0.31.2` | `10.0.31.3` | `10.0.31.1` | CORE-SW2 |
| **40** | SW3_USERS | `10.0.40.0/24` | `10.0.40.2` | `10.0.40.3` | `10.0.40.1` | CORE-SW1 |
| **41** | SW3_VOIP | `10.0.41.0/24` | `10.0.41.2` | `10.0.41.3` | `10.0.41.1` | CORE-SW1 |
| **50** | SW4_USERS | `10.0.50.0/24` | `10.0.50.2` | `10.0.50.3` | `10.0.50.1` | CORE-SW2 |
| **51** | SW4_VOIP | `10.0.51.0/24` | `10.0.51.2` | `10.0.51.3` | `10.0.51.1` | CORE-SW2 |
| **60** | SW5_USERS | `10.0.60.0/24` | `10.0.60.2` | `10.0.60.3` | `10.0.60.1` | CORE-SW1 |
| **61** | SW5_VOIP | `10.0.61.0/24` | `10.0.61.2` | `10.0.61.3` | `10.0.61.1` | CORE-SW1 |
| **100** | SERVERS | `10.0.100.0/24` | `10.0.100.2` | `10.0.100.3` | `10.0.100.1` | CORE-SW1 |
| **110** | PRINTERS | `10.0.110.0/24` | `10.0.110.2` | `10.0.110.3` | `10.0.110.1` | CORE-SW2 |
| **90** | GUEST | `10.0.90.0/24` | `10.0.90.2` | `10.0.90.3` | `10.0.90.1` | CORE-SW2 |
| **999** | NATIVE | `10.0.99.0/24` | --- | --- | --- | L2 Only |

#### Adresacja interfejsów zarządzania (SVI) dla switchy dostępowych (VLAN 10):

| Urządzenie | Adres IP | Maska sieci | Brama domyślna (Default Gateway) |
| :--- | :--- | :--- | :--- |
| **ACCESS_SW1** | `10.0.10.11` | `255.255.255.0` | `10.0.10.1` |
| **ACCESS_SW2** | `10.0.10.12` | `255.255.255.0` | `10.0.10.1` |
| **ACCESS_SW3** | `10.0.10.13` | `255.255.255.0` | `10.0.10.1` |
| **ACCESS_SW4** | `10.0.10.14` | `255.255.255.0` | `10.0.10.1` |
| **ACCESS_SW5** | `10.0.10.15` | `255.255.255.0` | `10.0.10.1` |

---

### 3. Plan Adresacji IP — Oddziały (Branches)

#### ODDZIAŁ 1

| Nazwa VLAN | Numer VLAN | Adres sieci | Maska sieci | Brama domyślna |
| :--- | :---: | :--- | :--- | :--- |
| **MGMT** | 10 | `10.1.10.0` | `255.255.255.0` | `10.1.10.1` |
| **USERS** | 20 | `10.1.20.0` | `255.255.255.0` | `10.1.20.1` |
| **LOCAL** | 30 | `10.1.30.0` | `255.255.255.0` | `10.1.30.1` |
| **GUEST** | 40 | `10.1.40.0` | `255.255.255.0` | `10.1.40.1` |
| **IT** | 50 | `10.1.50.0` | `255.255.255.0` | `10.1.50.1` |
| **VOIP** | 60 | `10.1.60.0` | `255.255.255.0` | `10.1.60.1` |

* **Adres IP interfejsu zarządzania (VLAN 10) dla BR1-SW1:** `10.1.10.10`

#### ODDZIAŁ 2

| Nazwa VLAN | Numer VLAN | Adres sieci | Maska sieci | Brama domyślna |
| :--- | :---: | :--- | :--- | :--- |
| **MGMT** | 10 | `10.2.10.0` | `255.255.255.0` | `10.2.10.1` |
| **USERS** | 20 | `10.2.20.0` | `255.255.255.0` | `10.2.20.1` |
| **LOCAL** | 30 | `10.2.30.0` | `255.255.255.0` | `10.2.30.1` |
| **GUEST** | 40 | `10.2.40.0` | `255.255.255.0` | `10.2.40.1` |
| **IT** | 50 | `10.2.50.0` | `255.255.255.0` | `10.2.50.1` |
| **VOIP** | 60 | `10.2.60.0` | `255.255.255.0` | `10.2.60.1` |

* **Adres IP interfejsu zarządzania (VLAN 10) dla BR2-SW1:** `10.2.10.10`

---

### 4. Adresacja Połączeń Infrastrukturalnych i WAN

#### Adresacja pomiędzy warstwą EDGE - CORE (Centrala):

| ROUTER | SWITCH | Adres sieci | ADRES ROUTERA | ADRES SWITCHA |
| :--- | :--- | :--- | :--- | :--- |
| **EDGE-R1** | CORE-SW1 | `10.0.250.0/30` | `10.0.250.1` | `10.0.250.2` |
| **EDGE-R1** | CORE-SW2 | `10.0.250.4/30` | `10.0.250.5` | `10.0.250.6` |
| **EDGE-R2** | CORE-SW1 | `10.0.250.8/30` | `10.0.250.9` | `10.0.250.10` |
| **EDGE-R2** | CORE-SW2 | `10.0.250.12/30` | `10.0.250.13` | `10.0.250.14` |

#### Adresacja WAN:

| ROUTER | WAN ADDRESS |
| :--- | :--- |
| **EDGE-R1** | `192.51.100.1/30` |
| **EDGE-R2** | `192.51.100.5/30` |
| **EDGE-BR1** | `192.51.100.9/30` |
| **EDGE-BR2** | `192.51.100.13/30` |

#### Adresacja interfejsów Loopback na routerach:

| ROUTER | LOOPBACK ADDRESS |
| :--- | :--- |
| **EDGE-R1** | `192.168.254.1` |
| **EDGE-R2** | `192.168.254.2` |
| **EDGE-BR1** | `192.168.254.3` |
| **EDGE-BR2** | `192.168.254.4` |

#### Adresacja interfejsów Tunnel (VPN):
*Pierwszy adres w sieci jest przydzielony do routera w HQ, a drugi adres dla routera Branch.*

| ROUTER (HQ) | ROUTER (BRANCH) | Sieć | IP HQ | IP BRANCH |
| :--- | :--- | :--- | :--- | :--- |
| **EDGE-R1** | EDGE-BR1 | `10.255.0.0/30` | `10.255.0.1` | `10.255.0.2` |
| **EDGE-R1** | EDGE-BR2 | `10.255.0.4/30` | `10.255.0.5` | `10.255.0.6` |
| **EDGE-R2** | EDGE-BR1 | `10.255.0.8/30` | `10.255.0.9` | `10.255.0.10` |
| **EDGE-R2** | EDGE-BR2 | `10.255.0.12/30` | `10.255.0.13` | `10.255.0.14` |

## Konfiguracja Warstwy Dostępowej (Access Layer) w Centrali

Wszystkie przełączniki dostępowe (`ACCESS_SW1` do `ACCESS_SW5` w Centrali) działają w warstwie 2 (Layer 2). Odpowiadają za fizyczne podłączenie urządzeń końcowych, komputerów, telefonów oraz serwerów, realizując przy tym mechanizmy izolacji i bezpieczeństwa na poziomie portów.

### 1. Struktura i Przypisanie VLAN-ów do Przełączników (HQ)

Konfiguracja sieci VLAN na poszczególnych przełącznikach dostępowych została podzielona w następujący sposób:

*   **ACCESS_SW1 (Główny Przełącznik Dostępów / Serwerownia & Sekcja A):**
    *   **VLAN 10 (MGMT):** Sieć zarządcza dla infrastruktury.
    *   **VLAN 15 (IT_DEPT):** Podsieć dla stacji roboczych administratorów (w tym `ADMIN-PC`).
    *   **VLAN 20 (SW1_USERS):** Ruch komputerowy pierwszej grupy pracowników.
    *   **VLAN 21 (SW1_VOIP):** Ruch dla aparatów VoIP pierwszej grupy pracowników.
    *   **VLAN 100 (SERVERS):** Krytyczna strefa serwerowa (`DHCP-SERVER`, `WEB-SERVER`, `ZABBIX-SERVER`, `VOIP-GATEWAY`).
    *   **VLAN 110 (PRINTERS):** Centralna strefa sieciowych urządzeń drukujących.

*   **ACCESS_SW2 do ACCESS_SW5 (Przełączniki dla Sekcji Użytkowników, Gości i Drukarek):**
    Każdy z pozostałych przełączników dostępowych posiada analogiczny zestaw VLAN-ów, umożliwiający obsługę lokalnych pracowników, telefonii, ruchu gościnnego oraz dostęp do urządzeń drukujących i sieci zarządzania:
    *   **VLAN 10 (MGMT):** Dostęp do wirtualnego interfejsu zarządzania danego switcha.
    *   **VLAN 110 (PRINTERS):** Lokalny dostęp do sieciowych drukarek korporacyjnych.
    *   **VLAN 90 (GUEST):** Ruch odizolowanej sieci bezprzewodowej/przewodowej dla gości.
    *   **VLAN-y Użytkowe i VoIP (Dedykowane per Switch):**
        *   Na **ACCESS_SW2**: `VLAN 30` (SW2_USERS) oraz `VLAN 31` (SW2_VOIP)
        *   Na **ACCESS_SW3**: `VLAN 40` (SW3_USERS) oraz `VLAN 41` (SW3_VOIP)
        *   Na **ACCESS_SW4**: `VLAN 50` (SW4_USERS) oraz `VLAN 51` (SW4_VOIP)
        *   Na **ACCESS_SW5**: `VLAN 60` (SW5_USERS) oraz `VLAN 61` (SW5_VOIP)

---

### 2. Bezpieczeństwo DHCP (DHCP Snooping)
W celu ochrony sieci LAN przed nieautoryzowanymi lub złośliwymi serwerami DHCP (ataki typu *Rogue DHCP Server*), na wszystkich przełącznikach dostępowych aktywowano funkcję **DHCP Snooping**:
*   **Porty Zaufane (Trusted Ports):** Fizyczne porty uplink prowadzące w stronę przełączników rdzeniowych `CORE-SW1` i `CORE-SW2` (oraz interfejs podłączenia serwera DHCP na `ACCESS_SW1`) zostały skonfigurowane jako zaufane (`ip dhcp snooping trust`). Tylko przez te porty dopuszczalne jest przesyłanie pakietów typu *DHCP Offer* oraz *DHCP Ack*.
*   **Porty Niezaufane (Untrusted Ports):** Wszystkie porty abonenckie dla użytkowników końcowych, telefonów oraz gości są domyślnie niezaufane. Wpięcie fałszywego serwera DHCP w jakiekolwiek gniazdo biurkowe skutkuje natychmiastowym zablokowaniem nieautoryzowanych pakietów konfiguracyjnych, chroniąc sieć przed paraliżem adresacji.
*   **Włączenie globalne:** Mechanizm został uruchomiony globalnie (`ip dhcp snooping`) i powiązany ze wszystkimi obsługiwanymi VLAN-ami.

---

### 3. Mechanizmy Warstwy Dostępowej (L2 Features)

#### Porty Dostępowe (Access Mode) i Dual-VLAN dla VoIP
*   Wszystkie porty abonenckie zostały na sztywno przełączone w tryb dostępowy (`switchport mode access`).
*   W miejscach pracy wymagających jednoczesnego podłączenia PC oraz aparatu telefonicznego wdrożono technologię **Dual-VLAN**. Przełącznik przesyła pakiety komputera jako nietagowane w dedykowanym VLAN-ie użytkowników (np. `VLAN 30`), a ruch telefoniczny automatycznie taguje i odseparowuje w skorelowanym VLAN-ie VoIP (np. `VLAN 31`) za pomocą komendy `switchport voice vlan [ID]`.

#### Trunking i Bezpieczeństwo Native VLAN (802.1Q)
*   Uplinki do przełączników rdzeniowych działają jako magistrale trunk (`switchport mode trunk`) w standardzie tagowania IEEE 802.1Q.
*   W celu zabezpieczenia sieci przed atakami klasy *VLAN Hopping*, domyślny `VLAN 1` został całkowicie wyłączony z obsługi przesyłu danych. Cały ruch nietagowany (np. ramki kontrolne protokołów L2) został przeniesiony do odizolowanego, nieużywanego produkcyjnie VLAN-u natywnego za pomocą komendy `switchport trunk native vlan 999`.

#### Optymalizacja i Ochrona Topologii STP (Spanning Tree)
*   **PortFast:** Uruchomiony na portach końcowych (Edge Ports). Pomija stany listening/learning, natychmiast podnosząc interfejs w stan `Forwarding` po podłączeniu kabla, co eliminuje problemy z czasem oczekiwania (timeout) stacji na adres IP z DHCP.
*   **BPDU Guard:** Funkcja ochronna powiązana z PortFast. Podłączenie do gniazda ściennego nieautoryzowanego switcha generującego ramki BPDU skutkuje natychmiastowym wprowadzeniem portu w stan awaryjny `err-disabled`, zapobiegając pętlom i destabilizacji drzewa STP.
*   **BPDU Filter:** Funkcja która pozwala wyłączyć przesyłanie pakietów BPDU, została włączona na portach do której wpięte są urządzenia końcowe.

#### Dostęp Zdalny i Zarządzanie (Management Plane)
*   **Wirtualny interfejs SVI (VLAN 10):** Każdy przełącznik posiada skonfigurowany adres IP w dedykowanym `VLAN 10 MGMT` do celów administracyjnych. Do poprawnego routingu ruchu zarządzającego poza sieć lokalną skonfigurowano bramę domyślną (`ip default-gateway 10.0.10.1`) kierującą na wirtualny adres bramy HSRP.
*   **Protokół Telnet zamiast SSH:** Dostęp zdalny do CLI przełączników został skonfigurowany z wykorzystaniem protokołu **Telnet**. Decyzja ta wynikała bezpośrednio z ograniczeń technicznych środowiska symulacyjnego GNS3 — zastosowane obrazy IOU (IOS on Unix) dla przełączników L2 nie posiadały wsparcia dla kryptografii i nie obsługiwały protokołu SSH. W środowisku produkcyjnym bezwzględnie zalecane jest stosowanie protokołu SSH, ponieważ Telnet przesyła dane (w tym hasła) otwartym tekstem, podczas gdy SSH zapewnia w pełni szyfrowaną i bezpieczną komunikację.
*   **Zabezpieczenie linii VTY za pomocą ACL:** Pomimo ograniczeń protokołu Telnet, płaszczyzna zarządzania została maksymalnie zabezpieczona na poziomie kontroli dostępu. Do linii wirtualnych (`line vty 0 4`) przypisano listę kontrolną za pomocą komendy `access-class TELNET-ACCESS in`. Ta lista ACL restrykcyjnie ogranicza możliwość nawiązania sesji zdalnej, zezwalając na dostęp do CLI switcha wyłącznie zautoryzowanym podsieciom działu IT:
    *   `10.0.15.0/24` (Sieć IT w Centrali)
    *   `10.1.50.0/24` (Sieć IT w Oddziale 1)
    *   `10.2.50.0/24` (Sieć IT w Oddziale 2)
