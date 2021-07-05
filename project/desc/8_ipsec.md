# Настройка IPSec

---

| [ << Пред. (Настройка DMVPN)](./7_dmvpn.md) | [\^ К списку](../) | [След. (Настройка EIGRP) >>](./9_eigrp.md) |

---

Данная конфигурация является общей, вне зависимости от принадлежности устройства к hub/spoke.

1. Создаем политику для ISAKMP и актуализируем параметр keepalive.

        crypto isakmp policy 1
        encr aes 256
        hash sha256
        group 14
        crypto isakmp keepalive 60 5
        
2. Настраиваем transform-set для IPSec. Используем тип transport, т.к. уже используем DMVPN:

        crypto ipsec transform-set vpn-set ah-sha256-hmac esp-aes 
        mode transport
        
3. Настраиваем непосредственно профиль IPSec, указывая созданный transform-set.

        crypto ipsec profile vpn-ipsec
        set transform-set vpn-set 
        
4. На туннелях из предыдущей работы активируем шифрование с использованием профиля.

        tunnel protection ipsec profile vpn-ipsec shared

---

| [ << Пред. (Настройка DMVPN)](./7_dmvpn.md) | [\^ К списку](../) | [След. (Настройка EIGRP) >>](./9_eigrp.md) |

---
