# Предварительная общая конфигурация устройств стенда

---

| [<< Пред. (Адресация)](./1_addr.md) | [\^ К списку](../) | [След. (Внутренняя маршрутизация) >>](./3_int_routing.md) |

---

На данном этапе была проведена классическая базовая конфигурация каждого маршртутизатора.

1. Включено шифрование паролей

        service password-encryption

2. Установлен пароль на привилегированный режим

        enable secret <ENABLE_PASSWORD>

3. Указано имя устройства

        hostname <HOSTNAME>

4. Произведена настройка консоли и всех vty (логин через vty оставлен отключенным)

        line console 0
        logging synchronous
        password <CONSOLE_PASSWORD>
        login

        line vty 0 4
        password <VTY_PASSWORD>

5. Настроено MOTD сообщение
    
        banner motd $Unauthorized access is prohibited.$
        
---

| [<< Пред. (Адресация)](./1_addr.md) | [\^ К списку](../) | [След. (Внутренняя маршрутизация) >>](./3_int_routing.md) |

---
