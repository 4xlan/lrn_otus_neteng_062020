# Лабораторная работа 12.

## Цели работы

1. Настроить DHCP в офисе Москва.
2. Настроить синхронизацию времени в офисе Москва.
3. Настроить NAT в офисе Москва, C.-Петербург и Чокурдах.

## Задачи

1. Настроить NAT(PAT) на R14 и R15. Трансляция должна осуществляться в адрес автономной системы AS1001.
2. Настроить NAT(PAT) на R18. Трансляция должна осуществляться в пул из 5 адресов автономной системы AS2042.
3. Настроить статический NAT для R20.
4. Настроить NAT так, чтобы R19 был доступен с любого узла для удаленного управления.
5. Настроить статический NAT(PAT) для офиса Чокурдах.
6. Настроить DHCP сервер в офисе Москва на маршрутизаторах R12 и R13. VPC1 и VPC7 должны получать сетевые настройки по DHCP
7. Настроить NTP сервер на R12 и R13. Все устройства в офисе Москва должны синхронизировать время с R12 и R13.

## План работ

1. Настройка DHCP. Москва.
2. Настройка NAT. Москва.
3. Настройка NTP. Москва.
4. Настройка NAT. С.-Петербург.
5. Настройка NAT. Чокурдах.

--- 
 
**Рис. 1. - Общая схема сети.**

![Схема сети](l12.png)

---

Все конфигурационные файлы расположены в каталоге [cfg](./cfg/).

---

#### Изменения в топологии в последующих работах.

Не зафиксировано.

---

#### Изменения в топологии по отношению к предыдущим работам.

1. Исправлена работа NAT на площадке Москва (из [Л.Р. 9](../lab_9)).

---

## Выполнение

### Настройка DHCP. Москва.

Согласно работе [3-1](../lab_3_p1) и [3-2](../lab_3_p2), DHCP уже был настроен на площадке Москва.

### Настройка NAT. Москва.

Соглаcно работе [9](../lab_9), NAT уже был настроен на площадке Москва. В данной работе убираем предыдущую конфигурацию и настраиваем заново, согласно ТЗ.

#### Настройка статического NAT для R20.

Допустим, для R20 выделен адрес 200.200.200.20/24. Тогда, на R15 и R14 выполняем следующее:

1. Устанавливаем Ethernet0/2 как внешний интерфейс.

        interface Ethernet0/2
          ip nat outside

2. Указываем остальные активные интерфейсы как внутренние (т.к. в данной топологии сети трафик до любого узла может быть направлен через любой интерфейс (за исключением R19 и R20)).

        interface range Ethernet0/0-1, Ethernet0/3, Ethernet1/0
          ip nat inside


3. Пробрасываем внешний адрес до внутреннего адреса R20.

        ip nat inside source static 10.255.0.11 200.200.200.20
        
4. Так как 200.200.200.0/24 уже была анонсирована в предыдущих работах, доп.действий не требуется. Проверяем `show ip nat translations` на R14 и R15.

        Pro Inside global      Inside local       Outside local      Outside global
        --- 200.200.200.20     10.255.0.11        ---                ---

5. И финально проверяем доступность с R27, видим что узел доступен.

        R27>trace 200.200.200.20
        Type escape sequence to abort.
        Tracing the route to 200.200.200.20
        VRF info: (vrf in name/id, vrf out name/id)
        1 10.255.0.38 1 msec 0 msec 1 msec
        2 10.255.0.32 0 msec 1 msec 0 msec
        3 10.255.0.28 1 msec 1 msec 1 msec
        4 200.200.200.20 30 msec 1 msec 2 msec
        5 10.255.0.51 1 msec 2 msec 2 msec
        6 200.200.200.20 2 msec *  3 msec
        
#### Настроить NAT так, чтобы R19 был доступен с любого узла для удаленного управления.

В данном примере пробросим telnet порт R19 через R14 и R15.

> Но использовать, конечно, лучше VPN и SSH.

1. R14: `ip nat inside source static tcp 10.255.0.9 23 10.255.0.46 27960`

2. R15: `ip nat inside source static tcp 10.255.0.9 23 10.255.0.12 27015` 

3. Проверяем трансляцию адресов.

        R14#show ip nat translations 
        Pro Inside global      Inside local       Outside local      Outside global
        tcp 10.255.0.46:27960  10.255.0.9:23      ---                ---
        --- 200.200.200.20     10.255.0.11        ---                ---


        R15#show ip nat translations
        Pro Inside global      Inside local       Outside local      Outside global
        tcp 10.255.0.12:27015  10.255.0.9:23      ---                ---
        --- 200.200.200.20     10.255.0.11        ---                ---

4. Проверяем подключение с R27 через адрес R15 (т.к. в предыдущих работах весь трафик первоочередно направляется на R15 и попытка подключения через R14 при живом канале R15 может не увенчаться успехом).

        R27>telnet 10.255.0.12 27015
        Trying 10.255.0.12, 27015 ... Open
        Unauthorized access is prohibited.

        User Access Verification

        Password: 
        R19>

5. Еще раз смотрим в таблицу трансляции, видим соединение:

        R15#sh ip nat translations 
        Pro Inside global      Inside local       Outside local      Outside global
        tcp 10.255.0.12:27015  10.255.0.9:23      10.255.0.39:50064  10.255.0.39:50064
        tcp 10.255.0.12:27015  10.255.0.9:23      ---                ---
        --- 200.200.200.20     10.255.0.11        ---                ---

### Настройка NTP. Москва.

1. На R12 и R13 указываем уровень сервера:

    `ntp master 2`
    
2. Также, на обоих роутерах заходим в конфигурацию DHCP-сервера, настраиваем распространение адресов NTP серверов:

        option 42 ip <IP1> <IP2>
        
    Параметры `IP1` и `IP2` перечислены в табл. 1.
    
    **Табл. 1 - Сопоставление переменных с адресами устройств.**
    
    | Маршрутизатор | IP1 | DEVNAME1 | IP2 | DEVNAME2 |
    | -- | -- | -- | -- | -- |
    | R12 | 10.1.10.1 | R12 | 10.1.10.2 | R13 |
    | R13 | 10.1.11.1 | R13 | 10.1.11.2 | R12 |

3. На устройствах R14-15, R19-20 включаем клиент ntp командами:

        ntp server <IP1> prefer
        ntp server <IP2>
        ntp update-calendar

    Где `IP1`:
    
    * R14, R19: 10.255.0.0
    * R15, R20: 10.255.0.6
    
    `IP2`:
    
    * R14, R19: 10.255.0.6
    * R15, R20: 10.255.0.0
        
> В прод окружениях имеет смысл использовать также аутентификацию и ограничение доступа для клиентов NTP.

### Настройка NAT. С.-Петербург.

В предыдущих работах с данной площадки анонсировалась сеть 100.100.100.0/24. Предположим, что адреса 100.100.100.[1-5] используются для выхода в сеть. Тогда настраиваем R18 (предварительно вся предыдущая конфигурация NAT удалена):

1. Создаем пул адресов для NAT (`POOL_NAME`- наименование пула): 
            
        ip nat pool <POOL_NAME> 100.100.100.1 100.100.100.5 netmask 255.255.255.0 

2. Создаем access-list, адреса из которого смогут воспользоваться NAT (`NUM` - номер листа):

        access-list <NUM> permit 10.2.0.0 0.0.255.255
        access-list <NUM> permit 10.255.0.18
        access-list <NUM> permit 10.255.0.14

3. Включаем NAT:

        ip nat inside source list <NUM> pool <POOL_NAME>
        
4. Интерфейсы R18:Ethernet0/2-3 отмечаем как outside, а R18:Ethernet0/0-1 - как inside.

5. Проверяем соединение с VPC до R21 (Ламас) и смотрим таблицу трансляций:

        R18> show ip nat translations    
        Pro Inside global      Inside local       Outside local      Outside global
        udp 100.100.100.1:225  10.2.21.11:225     10.255.0.13:226    10.255.0.13:226
        --- 100.100.100.1      10.2.21.11         ---                ---

6. С VPC8 выполняем аналогичную трассировку, проверяем изменения в таблице:

        R18> show ip nat translations 
        Pro Inside global      Inside local       Outside local      Outside global
        udp 100.100.100.2:21952 10.2.20.11:21952  10.255.0.13:21953  10.255.0.13:21953
        --- 100.100.100.2      10.2.20.11         ---                ---
        udp 100.100.100.1:225  10.2.21.11:225     10.255.0.13:226    10.255.0.13:226
        --- 100.100.100.1      10.2.21.11         ---                ---

### Настройка NAT. Чокурдах.

1. Интерфейсы R28:Ethernet0/0-1 указываем как outside, R28:Ethernet0/2.{30,31} указываем как inside.
2. Создаем access-list, куда включаем обе клиентские подсети.

        ip access-list extended ADDR_TO_NAT
          10 permit ip 10.3.1.72 0.0.0.7 any
          20 permit ip 10.3.1.80 0.0.0.7 any
          50 deny ip any any
          
3. Создаем route-map на каждый выходной интерфейс (R25 и R26):

        route-map NAT_TO_R25
          match ip address ADDR_TO_NAT
          match interface Ethernet0/1
          exit
          
        route-map NAT_TO_R26
          match ip address ADDR_TO_NAT
          match interface Ethernet0/0
          exit
          
4. Включаем PAT на интерфейсах:

        ip nat inside source route-map NAT_TO_R25 interface Ethernet0/1 overload
        ip nat inside source route-map NAT_TO_R26 interface Ethernet0/0 overload

5. Проверяем работу NAT посредством пинга R22 (Киторн) с VPC31:

        VPCS> trace 10.255.0.47
        trace to 10.255.0.47, 8 hops max, press Ctrl+C to stop
        1   10.3.1.81   1.799 ms  0.579 ms  0.610 ms
        2   10.255.0.40   1.260 ms  0.714 ms  0.727 ms
        3   10.255.0.32   1.190 ms  0.751 ms  0.753 ms
        4   *10.255.0.28   1.423 ms (ICMP type:3, code:3, Destination port unreachable)  *