# Настройка внутренней маршрутизации и обеспечение доступа в сеть Интернет.

---

| [<< Пред. (Общая конфигурация устройств)](./2_preconf.md) | [\^ К списку](../) | [След. (Маршрутизация между площадками) >>](./4_ext_routing.md) |

---

## OSPF

В целях обеспечения маршрутизации внутри локаций Майк-26 и Рогейн настраиваем OSPF. Камино и Рогейн ввиду размеров сети, на данном этапе настройки пропускаются.

### Майк-26

> Также на данной площадке используется iBGP для соединения между пограничными маршрутизаторами, что будет отображено в [Настройка маршрутизации между площадками.](./4_ext_routing.md)

В качестве зоны 0 используем выходные маршрутизаторы (mike_bgw_1, mike_bgw_2), остальные маршрутизаторы добавляем в зону 1, отмеченную как stub. В данную зону будем отдавать маршрут по умолчанию.

1. mike_bgw_1

        router ospf 1
        area 1 stub
        passive-interface default
        no passive-interface Ethernet0/0
        no passive-interface Ethernet0/1
        no passive-interface Ethernet0/2
        default-information originate
        
        interface range Ethernet0/0-2
        ip ospf 1 area 0
        
        interface Loopback0
        ip ospf 1 area 0
        
2. mike_bgw_2

        router ospf 1
        area 1 stub
        passive-interface default
        no passive-interface Ethernet0/0
        no passive-interface Ethernet0/1
        no passive-interface Ethernet0/2
        default-information originate
        
        interface range Ethernet0/0-2
        ip ospf 1 area 0
        
        interface Loopback0
        ip ospf 1 area 0
        
3. mike_rtr_1

        router ospf 1
        passive-interface default
        no passive-interface Ethernet0/0
        no passive-interface Ethernet0/1
        no passive-interface Ethernet0/2

        interface range Ethernet0/0-2
        ip ospf 1 area 1

3. mike_rtr_2

        router ospf 1
        passive-interface default
        no passive-interface Ethernet0/0
        no passive-interface Ethernet0/1
        no passive-interface Ethernet0/2
        
        interface range Ethernet0/0-2
        ip ospf 1 area 1

4. mike_rtr_3

        router ospf 1
        passive-interface default
        no passive-interface Ethernet0/0
        no passive-interface Ethernet0/1

        interface range Ethernet0/0-1
        ip ospf 1 area 1   
        
### Рогейн

Аналогично Майк-26 настраиваем OSPF, за исключением зоны - здесь все маршрутизаторы находятся в backbone.

1. rog_bgw_1

        router ospf 1
        passive-interface default
        no passive-interface Ethernet0/0
        no passive-interface Ethernet0/1
        default-information originate

        interface range Ethernet0/0-1
        ip ospf 1 area 0
        
        interface Loopback0
        ip ospf 1 area 0
        
2. rog_rtr_1

        router ospf 1
        passive-interface default
        no passive-interface Ethernet0/0
        no passive-interface Ethernet0/1
        no passive-interface Ethernet0/2
        
        interface range Ethernet0/0-2, Loopback0
        ip ospf 1 area 0
        
3. rog_rtr_2

        router ospf 1
        passive-interface default
        no passive-interface Ethernet0/0
        no passive-interface Ethernet0/1
        no passive-interface Ethernet0/2
        
        interface range Ethernet0/0-2, Loopback0
        ip ospf 1 area 0
        
4. rog_rtr_3

        router ospf 1
        passive-interface default
        no passive-interface Ethernet0/0
        no passive-interface Ethernet0/1
        
        interface range Ethernet0/0-1, Loopback0
        ip ospf 1 area 0
        
## NAT

### Майк-26

На выходных маршрутизаторах настраиваем PAT, ограничиваем по подсети.

1. mike_bgw_1 / mike_bgw_2

        interface range Ethernet0/0-2
          ip nat inside
          
        interface Ethernet0/3
          ip nat outside
          
        ip access-list extended SUBN_TO_NAT
          10 permit ip 10.255.1.0 0.0.0.255 any
          40 deny ip any any
          exit
        
        route-map NAT_MAP
          match ip address SUBN_TO_NAT
          match interface Ethernet0/3
          exit
          
        ip nat source route-map NAT_MAP interface Ethernet0/3 overload

### Рогейн

Также настраиваем PAT с ограничением по подсети.

    ip nat inside source route-map CIR_NAT interface Ethernet0/2 overload
    ip nat inside source route-map IEN_NAT interface Ethernet0/3 overload
    
    ip access-list extended SUBN_TO_NAT
    permit ip 10.255.2.0 0.0.0.255 any
    deny   ip any any
    
    route-map IEN_NAT permit 10
    match ip address SUBN_TO_NAT
    match interface Ethernet0/3
    
    route-map CIR_NAT permit 10
    match ip address SUBN_TO_NAT
    match interface Ethernet0/2

## Статические маршруты

### Камино

На данной площадке был настроен статический адрес 0.0.0.0/0 в сторону провайдера.

    ip route 0.0.0.0 0.0.0.0 Ethernet0/3 1.18.0.1 name ShanNet 250

### Максвелл

На данной площадке был настроен статический адрес 0.0.0.0/0 в сторону провайдера.

    ip route 0.0.0.0 0.0.0.0 Ethernet0/3 2.193.0.1 name Cir.Ru 250


---

| [<< Пред. (Общая конфигурация устройств)](./2_preconf.md) | [\^ К списку](../) | [След. (Маршрутизация между площадками) >>](./4_ext_routing.md) |

---
