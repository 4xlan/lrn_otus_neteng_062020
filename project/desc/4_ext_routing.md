# Настройка внешней маршрутизации.

---

| [<< Пред. (Внутренняя маршрутизация)](./3_int_routing.md) | [\^ К списку](../) | [След. (Настройка NTP) >>](./5_ntp.md) |

---

Внешняя маршрутизация, а т.е. маршрутизация между площадками, организована посредством BGP. Между провайдерами есть договорённость о том, что они фильтруют подсети, относящиеся к приватным диапазонам, как на входящих, так и на исходящих соединениях. Клиентам выдается по умолчанию только дефолтный маршрут.
Настройка выполняется с применением шаблонов - в GENERAL используем фильтрацию частных подсетей, далее в политике CLIENT дополнительно к шаблону GENERAL добавляются специфичные для клиента опции.

На клиентских устройствах в рамках конфигурации BGP также добавляем фильтрацию на входящие и исходящие соединения. На входящих принимаем только стандартный маршрут, на исходящих отдаем только адреса площадки. На Майк-26 настраиваем приоритет устройств так, чтобы трафик приоритетно отправлялся с mike_bgw_1. Между mike_bgw_1 и mike_bgw_2 также устанавливаем iBGP соединение. <!-- Зачем? -->

Дополнительно, на клиентских устройствах на приходящий стандартный маршрут от провайдера вешаем заведомо увеличенную метрику (для того, чтобы впоследствии настроенные vpn соединения были приоритетней). Настройка выполняется также на базе шаблонов.

---

**Табл. 1 - Список AS**

| ASN | Локация |
| :--: | -- |
| 451 | Cir.Ru |
| 1984 | Tritel |
| 2541 | ShanNet |
| 3120 | IenCom |
| 27900 | Рогейн |
| 28910 | Майк-26 |

---


## Майк-26

1. mike_bgw_1

        router bgp 28910
        template peer-policy GENERAL
        filter-list 1 out
        prefix-list IN_FILTER in
        exit-peer-policy
        !
        template peer-policy PROVIDER
        route-map TRITEL_PRIOR in
        inherit peer-policy GENERAL 1
        exit-peer-policy
        !
        bgp log-neighbor-changes
        neighbor 4.167.0.1 remote-as 1984
        neighbor 4.167.0.1 inherit peer-policy PROVIDER
        neighbor 10.255.1.1 remote-as 28910
        
        ip as-path access-list 1 permit $^
        ip as-path access-list 1 deny .*
        
        ip prefix-list IN_FILTER seq 5 permit 0.0.0.0/0
        ip prefix-list IN_FILTER seq 10 deny 0.0.0.0/0 le 32
            
2. mike_bgw_2

        router bgp 28910
        template peer-policy GENERAL
        filter-list 1 out
        prefix-list IN_FILTER in
        exit-peer-policy
        !
        template peer-policy PROVIDER
        route-map SHANNET_PRIOR in
        inherit peer-policy GENERAL 1
        exit-peer-policy
        !
        bgp log-neighbor-changes
        neighbor 1.19.0.1 remote-as 2541
        neighbor 1.19.0.1 inherit peer-policy PROVIDER
        neighbor 10.255.1.0 remote-as 28910
        
        ip as-path access-list 1 permit $^
        ip as-path access-list 1 deny .*
        
        ip prefix-list IN_FILTER seq 5 permit 0.0.0.0/0
        ip prefix-list IN_FILTER seq 10 deny 0.0.0.0/0 le 32


## Рогейн

1. rog_bgw_1

        router bgp 27900
        template peer-policy GENERAL
        filter-list 1 out
        prefix-list IN_FILTER in
        prefix-list OUT_FILTER out
        exit-peer-policy
        !
        template peer-policy PROVIDER
        route-map INCOMING_ROUTE_FIX in
        inherit peer-policy GENERAL 1
        exit-peer-policy
        !
        bgp log-neighbor-changes
        neighbor 2.192.0.1 remote-as 451
        neighbor 2.192.0.1 inherit peer-policy PROVIDER
        neighbor 3.141.0.1 remote-as 3120
        neighbor 3.141.0.1 inherit peer-policy PROVIDER
        
        ip as-path access-list 1 permit $^
        ip as-path access-list 1 deny .*

        ip prefix-list IN_FILTER seq 5 permit 0.0.0.0/0
        ip prefix-list IN_FILTER seq 10 deny 0.0.0.0/0 le 32
        !
        ip prefix-list OUT_FILTER seq 5 permit 2.192.0.0/16
        ip prefix-list OUT_FILTER seq 7 permit 3.141.0.0/16
        ip prefix-list OUT_FILTER seq 10 deny 0.0.0.0/0 le 32

## ShanNet

1. ip1_rtr_0

        router bgp 2541
        template peer-policy GENERAL
        distribute-list LOCNET_BAN in
        distribute-list LOCNET_BAN out
        exit-peer-policy

        template peer-policy CLIENT
        filter-list 403 out
        default-originate
        exit-peer-policy

        bgp log-neighbor-changes
        redistribute connected
        neighbor 1.19.194.112 remote-as 28910
        neighbor 1.19.194.112 inherit peer-policy CLIENT
        neighbor 1.255.255.1 remote-as 451
        neighbor 1.255.255.1 inherit peer-policy GENERAL
        neighbor 4.255.255.0 remote-as 1984
        neighbor 4.255.255.0 inherit peer-policy GENERAL
        
        ip as-path access-list 403 deny .*
        
        ip access-list extended LOCNET_BAN
        deny   ip 10.0.0.0 0.0.0.255 any
        deny   ip 172.16.0.0 0.15.255.255 any
        deny   ip 192.168.0.0 0.0.255.255 any
        permit ip any any

## Cir.Ru

1. ip2_rtr_0

        router bgp 451
        template peer-policy GENERAL
        distribute-list LOCNET_BAN in
        distribute-list LOCNET_BAN out
        exit-peer-policy

        template peer-policy CLIENT
        distribute-list LOCNET_BAN in
        filter-list 403 out
        default-originate
        exit-peer-policy

        bgp log-neighbor-changes
        redistribute connected
        neighbor 1.255.255.0 remote-as 2541
        neighbor 1.255.255.0 inherit peer-policy GENERAL
        neighbor 2.192.198.33 remote-as 27900
        neighbor 2.192.198.33 inherit peer-policy CLIENT
        neighbor 2.255.255.1 remote-as 3120
        neighbor 2.255.255.1 inherit peer-policy GENERAL
        
        ip as-path access-list 403 deny .*
        
        ip access-list extended LOCNET_BAN
        deny   ip 10.0.0.0 0.0.0.255 any
        deny   ip 172.16.0.0 0.15.255.255 any
        deny   ip 192.168.0.0 0.0.255.255 any
        permit ip any any

## IenCom

1. ip3_rtr_0

        router bgp 3120
        template peer-policy GENERAL
        distribute-list LOCNET_BAN in
        distribute-list LOCNET_BAN out
        exit-peer-policy

        template peer-policy CLIENT
        distribute-list LOCNET_BAN in
        filter-list 403 out
        default-originate
        exit-peer-policy

        bgp log-neighbor-changes
        redistribute connected
        neighbor 2.255.255.0 remote-as 451
        neighbor 2.255.255.0 inherit peer-policy GENERAL
        neighbor 3.141.245.65 remote-as 27900
        neighbor 3.141.245.65 inherit peer-policy CLIENT
        neighbor 3.255.255.1 remote-as 1984
        neighbor 3.255.255.1 inherit peer-policy GENERAL

        ip as-path access-list 403 deny .*
        
        ip access-list extended LOCNET_BAN
        deny   ip 10.0.0.0 0.0.0.255 any
        deny   ip 172.16.0.0 0.15.255.255 any
        deny   ip 192.168.0.0 0.0.255.255 any
        permit ip any any
 
## Tritel

1. ip4_rtr_0

        router bgp 1984
        template peer-policy GENERAL
        distribute-list LOCNET_BAN in
        distribute-list LOCNET_BAN out
        exit-peer-policy
        !
        template peer-policy CLIENT
        distribute-list LOCNET_BAN in
        filter-list 403 out
        default-originate
        exit-peer-policy
        !
        bgp log-neighbor-changes
        redistribute connected
        neighbor 3.255.255.0 remote-as 3120
        neighbor 3.255.255.0 inherit peer-policy GENERAL
        neighbor 4.167.34.17 remote-as 28910
        neighbor 4.167.34.17 inherit peer-policy CLIENT
        neighbor 4.255.255.1 remote-as 2541
        neighbor 4.255.255.1 inherit peer-policy GENERAL

        ip as-path access-list 403 deny .*
        
        ip access-list extended LOCNET_BAN
        deny   ip 10.0.0.0 0.0.0.255 any
        deny   ip 172.16.0.0 0.15.255.255 any
        deny   ip 192.168.0.0 0.0.255.255 any
        permit ip any any
 
---

| [<< Пред. (Внутренняя маршрутизация)](./3_int_routing.md) | [\^ К списку](../) | [След. (Настройка NTP) >>](./5_ntp.md) |

---
