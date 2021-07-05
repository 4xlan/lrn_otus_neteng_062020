# Настройка удостоверяющего центра.

---

| [<< Пред. (Настройка NTP)](./5_ntp.md) | [\^ К списку](../) | [След. (Настройка DMVPN) >>](./7_dmvpn.md) |

---

## Настройка сервера

Для сервера (mike_rtr_3) был сгенерирован RSA ключ размером 4096 бит, который в дальнейшем был использован для создания trustpoint. И в заключении был активирован http-сервер для УЦ.

        crypto key generate rsa label CA_TP exportable modulus 4096

        crypto pki server CA_TP
        no database archive
        issuer-name CN=ULTORrootCA,OU=ISD,O=ULTOR,C=AF
        hash sha1

        crypto pki trustpoint CA_TP
        enrollment url http://10.255.1.11:80
        subject-name CN=ULTORrootCA,OU=ISD,O=ULTOR,C=AF
        revocation-check crl
        rsakeypair CA_KEY

## Настройка клиентов

Клиентами в данном случае являлись пограничные маршрутизаторы филиалов:

- mike_bgw_1
- mike_bgw_2
- rog_bgw_1
- max_bgw_1
- kam_bgw_1

На каждом из них также был сгенерирован аналогичный ключ, после чего был настроен trustpoint.

> `DEVICE_NAME` - наименование устройства.

        crypto key generate rsa label <DEVICE_NAME> exportable modulus 4096
        
        crypto pki trustpoint ULTOR_TP
        enrollment url http://4.167.34.17:80
        serial-number
        revocation-check crl
        rsakeypair <DEVICE_NAME>
        
> Стоит отметить, что для маршрутизаторов площадки Майк-26 enrollment url был указан внутренний адрес сервера УЦ (`10.255.1.11:80`).

## Отправка запроса на сертификат

С каждого клиента был отправлен запрос на получение сертификата сервера УЦ, а далее - запрос на выпуск сертификата на данном УЦ на базе закрытого ключа, сформированного на устройстве.

        crypto pki authenticate ULTOR_TP
        crypto pki enroll ULTOR_TP

## Подтверждение запросов клиентов

Со стороны сервера УЦ была проведена проверка наличия запросов, сверка отпечатков и последующее одобрение запросов, признанных легитимными.

        show crypto pki server CA_TP requests
        crypto pki server CA_TP grant <NUM>
        
> `NUM` - номер конкретного запроса из результата выполнения вышестоящей команды.

---

| [<< Пред. (Настройка NTP)](./5_ntp.md) | [\^ К списку](../) | [След. (Настройка DMVPN) >>](./7_dmvpn.md) |

---
