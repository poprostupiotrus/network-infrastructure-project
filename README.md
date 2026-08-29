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
