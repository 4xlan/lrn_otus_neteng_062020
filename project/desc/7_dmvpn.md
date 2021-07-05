# Настройка DMVPN

---

[<< Пред. (Настройка УЦ)](./6_ca.md) | [\^ К списку](../) | [След. (Настройка IPSec) >>](./8_ipsec.md) |

---

Сервер DMVPN располагается в головном офисе (Майк-26). Ввиду наличия двух выходных маршрутизаторов мы можем настроить схему DualCloud с Phase 3, тем самым упрощая жизнь при потенциальном появлении новых площадок, а также обеспечивая некоторую избыточность. Таким образом получаем следующую схему:

- Два hub: mike_bgw_1 и mike_bgw_2.
- Каждый spoke имеет два туннеля.

## Настройка hub

Создаем туннельный интерфейс (номер интерфейса равен номеру оборудования), указываем основные параметры, такие как:

- Адрес интерфейса
- Значение MTU
- Значение MSS
- Ширину канала
- Конфигурацию NHRP (аутентификация, идентификатор, отправка redirect сообщений, необходимых для Phase 3)
- Tunnel key (т.к. имеем два туннеля в конфигурации, данный параметр позволит нам определить трафик к конкретному контуру)
- Исходящий физический интерфейс

На примере max_bgw_1:

        interface Tunnel1
        no shutdown
        bandwidth 100000
        ip address 192.168.10.1 255.255.255.0
        no ip redirects
        ip mtu 1400
        ip nhrp authentication <AUTHKEY1>
        ip nhrp network-id 1
        ip nhrp redirect
        ip tcp adjust-mss 1360
        tunnel source Ethernet0/3
        tunnel mode gre multipoint
        tunnel key 1

## Настройка spoke

Создаем туннельные интерфейсы (нумерацию делаем аналогично номеру туннеля hub конкретного контура). В целом конфигурация схожа с настройкой hub, за исключением отсутствия части специфичных для hub атрибутов, а также указания таких дополнительных параметров как:

- Реального адреса hub для получения multicast пакетов
- Маппинга соответствия реального адреса hub с "внутренним"
- Указания внутреннего адреса hub, как next-hop сервера
- Включением redirect сообщений командой `ip nhrp shortcut`

На примере rog_bgw_1:

        interface Tunnel1
        no shutdown
        bandwidth 100000
        ip address 192.168.10.10 255.255.255.0
        no ip redirects
        ip mtu 1400
        ip nhrp authentication <AUTHKEY1>
        ip nhrp map multicast 4.167.34.17
        ip nhrp map 192.168.10.1 4.167.34.17
        ip nhrp network-id 1
        ip nhrp nhs 192.168.10.1
        ip tcp adjust-mss 1360
        tunnel source Ethernet0/3
        tunnel mode gre multipoint
        tunnel key 1

        interface Tunnel2
        no shutdown
        bandwidth 100000
        ip address 192.168.20.10 255.255.255.0
        no ip redirects
        ip mtu 1400
        ip nhrp authentication <AUTHKEY2>
        ip nhrp map multicast 1.19.194.112
        ip nhrp map 192.168.20.1 1.19.194.112
        ip nhrp network-id 2
        ip nhrp nhs 192.168.20.1
        ip tcp adjust-mss 1360
        tunnel source Ethernet0/2
        tunnel mode gre multipoint
        tunnel key 2

---

[<< Пред. (Настройка УЦ)](./6_ca.md) | [\^ К списку](../) | [След. (Настройка IPSec) >>](./8_ipsec.md) |

---
