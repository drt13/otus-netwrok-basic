
# Лабораторная работа - Настройка протоколов CDP, LLDP и NTP

## Топология

![топология](img/image.png)

## Таблица адресации

|Устройство | Интерфейс | IP-адрес          | Маска подсети   | Шлюз по умолчанию   |
|-----------|-----------|-------------------|-----------------|---------------------|
|R1         |Lo1        |172.16.1.1         | 255.255.255.0   | - |
|           |G0/0/1     |10.22.0.1          | 255.255.255.0   | - |
|S1         |VLAN 1     |10.22.0.2          | 255.255.255.0   | 10.22.0.1 |
|S2         |VLAN 1     |10.22.0.3          | 255.255.255.0   | 10.22.0.1 |

## Задачи

Часть 1. Создание сети и настройка основных параметров устройства
Часть 2. Обнаружение сетевых ресурсов с помощью протокола CDP
Часть 3. Обнаружение сетевых ресурсов с помощью протокола LLDP
Часть 4. Настройка и проверка NTP

### Решение

#### Часть 1. Создание сети и настройка основных параметров устройств

##### Шаг 1. Создание сети

a. Создал сеть согласно топологии.

##### Шаг 2. Настройка и проверка основных параметров маршрутизатора

     a. Назначил маршрутизатору имя устройства.
     b. Отключил поиск DNS, чтобы предотвратить попытки маршрутизатора неверно преобразовывать введенные команды таким образом, как будто они являются именами узлов.
     c. Назначил class в качестве зашифрованного пароля привилегированного режима EXEC.
     d. Назначил cisco в качестве пароля консоли и включите вход в систему по паролю.
     e. Назначил cisco в качестве пароля VTY и включите вход в систему по паролю.
     f. Включил шифрование открытых паролей.
     g. Настроил IP-адресации интерфейсов, как указано в таблице выше.
     h. Сохранил текущую конфигурацию в файл загрузочной конфигурации.

##### Шаг 3. Настройка и проверка основных параметров коммутаторов

     a. Назначил коммутаторам имя устройства.
     b. Отключил поиск DNS, чтобы предотвратить попытки маршрутизатора неверно преобразовывать введенные команды таким образом, как будто они являются именами узлов.
     c. Назначил class в качестве зашифрованного пароля привилегированного режима EXEC.
     d. Назначил cisco в качестве пароля консоли и включите вход в систему по паролю.
     e. Назначил cisco в качестве пароля VTY и включите вход в систему по паролю.
     f. Включил шифрование открытых паролей.
     g. Выключил все интерфейсы, которые не будут использоваться
     h. Сохранил текущую конфигурацию в файл загрузочной конфигурации.

#### Часть 2. Обнаружение сетевых ресурсов с помощью протокола CDP

a. На R1 использовал команду **show cdp interface**, чтобы определить, сколько интерфейсов включено CDP, сколько из них включено и сколько отключено.

     R1#show cdp interface
     Vlan1 is administratively down, line protocol is down
          Sending CDP packets every 60 seconds
          Holdtime is 180 seconds
     GigabitEthernet0/0/0 is administratively down, line protocol is down
          Sending CDP packets every 60 seconds
          Holdtime is 180 seconds
     GigabitEthernet0/0/1 is up, line protocol is up
          Sending CDP packets every 60 seconds
          Holdtime is 180 seconds
     GigabitEthernet0/0/2 is administratively down, line protocol is down
          Sending CDP packets every 60 seconds
          Holdtime is 180 seconds

**Сколько интерфейсов участвует в объявлениях CDP?** 4 интерфейса: VLAN1, G0/0/0, G0/0/1, G0/0/2

**Какие из них активны?** GigabitEthernet0/0/1

b. На R1 использовал команду **show cdp entry S1** , чтобы определить версию IOS, используемую на S1.

     R1#show cdp entry S1

     Device ID: S1
     Entry address(es): 
     Platform: cisco 2960, Capabilities: Switch
     Interface: GigabitEthernet0/0/1, Port ID (outgoing port): FastEthernet0/5
     Holdtime: 163

     Version :
     Cisco IOS Software, C2960 Software (C2960-LANBASEK9-M), Version 15.0(2)SE4, RELEASE SOFTWARE (fc1)
     Technical Support: http://www.cisco.com/techsupport
     Copyright (c) 1986-2013 by Cisco Systems, Inc.
     Compiled Wed 26-Jun-13 02:49 by mnguyen

     advertisement version: 2
     Duplex: full

**Какая версия IOS используется на S1?** Version 15.0(2)SE4, RELEASE SOFTWARE (fc1)

c. На S1 хотел использовать команду **show cdp traffic**, чтобы определить, сколько пакетов CDP было передано, но CiscoPacket Tracer не разделил моего желания и выдал мне **% Invalid input detected at '^' marker**.

     S1#show cdp traffic
               ^
     % Invalid input detected at '^' marker.

**Сколько пакетов имеет выход CDP с момента последнего сброса счетчика?** Так и останется загадкой. По умлочанию **Sending CDP packets every 60 seconds** так что можно примерно посчитать кол-во пакетов.

d. Настроил SVI для VLAN 1 на S1 и S2, используя IP-адреса, указанные в таблице адресации выше. Настроил шлюз по умолчанию для каждого коммутатора на основе таблицы адресов.

e. На R1 выполните команду show cdp entry S1.
     R1#show cdp entry S1

     Device ID: S1
     Entry address(es): 
          IP address : 10.22.0.2

**Какие дополнительные сведения доступны теперь?** Теперь мы видим IP адрес коммутатора S1

f. Отключил CDP глобально на всех устройствах командой **no cdp run** в режиме глобальной конфигурации.

#### Часть 3. Обнаружение сетевых ресурсов с помощью протокола LLDP

a. Ввел команду **lldp run**, чтобы включить LLDP на всех устройствах в топологии.

`R1(config)#lldp run`

b. Чтобы предоставить подробную информацию о S2 на S1 необходимо выполнить команду **show lldp entry S2**, но такой команды в CPT нет, поэтому я использовал команду **show lldp neighbors detail**

     S1#show lldp neighbors detail 
     ------------------------------------------------
     Chassis id: 0006.2A81.4601
     Port id: Fa0/1
     Port Description: FastEthernet0/1
     System Name: S2
     System Description:
     Cisco IOS Software, C2960 Software (C2960-LANBASEK9-M), Version 15.0(2)SE4, RELEASE SOFTWARE (fc1)
     Technical Support: http://www.cisco.com/techsupport
     Copyright (c) 1986-2013 by Cisco Systems, Inc.
     Compiled Wed 26-Jun-13 02:49 by mnguyen
     Time remaining: 90 seconds
     System Capabilities: B
     Enabled Capabilities: B
     Management Addresses - not advertised
     Auto Negotiation - supported, enabled
     Physical media capabilities:
     100baseT(FD)
     100baseT(HD)
     1000baseT(HD)
     Media Attachment Unit type: 10
     Vlan ID: 1
     ------------------------------------------------

**Что такое chassis ID  для коммутатора S2?** Это MAC-адрес коммутатора S2, точнее его интерфейса FastEthernet0/1

c. На всех устройствах использовал команду LLDP, необходимую для отображения топологии физической сети только из выходных данных команды.

     R1#sh lldp neighbors 
     Capability codes:
     (R) Router, (B) Bridge, (T) Telephone, (C) DOCSIS Cable Device
     (W) WLAN Access Point, (P) Repeater, (S) Station, (O) Other
     Device ID           Local Intf     Hold-time  Capability      Port ID
     S1                  Gig0/0/1       120        B               Fa0/5

     Total entries displayed: 1

     S1#show lldp neighbors 
     Capability codes:
     (R) Router, (B) Bridge, (T) Telephone, (C) DOCSIS Cable Device
     (W) WLAN Access Point, (P) Repeater, (S) Station, (O) Other
     Device ID           Local Intf     Hold-time  Capability      Port ID
     S2                  Fa0/1          120        B               Fa0/1
     R1                  Fa0/5          120        R               Gig0/0/1

     Total entries displayed: 2

     S2#sh lldp neighbors 
     Capability codes:
     (R) Router, (B) Bridge, (T) Telephone, (C) DOCSIS Cable Device
     (W) WLAN Access Point, (P) Repeater, (S) Station, (O) Other
     Device ID           Local Intf     Hold-time  Capability      Port ID
     S1                  Fa0/1          120        B               Fa0/1

     Total entries displayed: 1

#### Часть 4. Настройка NTP

##### Шаг 1. Выведите на экран текущее время

Ввел команду **show clock detail** для отображения текущего времени на R1. Отображаемые сведения о текущем времени в таблице:

| Дата     |    Время     |    Часовой пояс   |    Источник времени             |
|----------|--------------|-------------------|---------------------------------|
|Mar 1 1993|*0:15:3.307   |      UTC          |Time source is hardware calendar |

##### Шаг 2. Установка времени

С помощью команды **clock set** установил время на маршрутизаторе R1.

     R1#clock set 13:56:00 29 May 2024 
     R1#sh clock 
     13:56:55.380 UTC Wed May 29 2024

##### Шаг 3. Настройка главного сервера NTP

Настроил R1 в качестве хозяина NTP с уровнем слоя 4.

     R1(config)#ntp master 4

##### Шаг 4. Настройте клиент NTP

a. Выполнил команду **show clock detail** на S1 и S2, чтобы просмотреть настроенное время.Текущее время, в таблице.

|Устройство | Дата     |    Время     |    Часовой пояс   |
|-----------|----------|--------------|-------------------|
| S1        |Mar 1 1993| 0:24:30.759  |      UTC          |
| S2        |Mar 1 1993| 0:25:18.984  |      UTC          |

b. Настроил S1 и S2 в качестве клиентов NTP указав в качестве сервера времени маршрутизатор R1.

     S1(config)#ntp server 10.22.0.1
     S2(config)#ntp server 10.22.0.1

##### Шаг 5. Проверьте настройку NTP

a. Используя команду **show ntp associations** и **show ntp status** , убедился, что S1 и S2 синхронизированы с R1.

     S2#show ntp associations 

     address         ref clock       st   when     poll    reach  delay          offset            disp
     *~10.22.0.1     127.127.1.1     4    13       16      217    0.00           0.00              0.12
     * sys.peer, # selected, + candidate, - outlyer, x falseticker, ~ configured
     
     S2#show ntp status 
     Clock is synchronized, stratum 16, reference is 10.22.0.1
     nominal freq is 250.0000 Hz, actual freq is 249.9990 Hz, precision is 2**24
     reference time is 2492D6E7.000002EC (3:45:43.748 UTC Sat Aug 28 2055)
     clock offset is 0.00 msec, root delay is 0.00  msec
     root dispersion is 10.68 msec, peer dispersion is 0.12 msec.
     loopfilter state is 'CTRL' (Normal Controlled Loop), drift is - 0.000001193 s/s system poll interval is 4, last update was 6 sec ago.

b. Выполнил команду **show clock** на S1 и S2, чтобы просмотреть настроенное время и сравнить ранее записанное время.

     S2#show clock 
     14:17:29.289 UTC Wed May 29 2024

Видим, что время синхронизовалось.

**Вопрос для повторения:**
**Для каких интерфейсов в пределах сети не следует использовать протоколы обнаружения сетевых ресурсов?** Данные протоколы не стоит использовать на пограничных интерфейсах, так как благодаря информации из них злоумышленник может получить сведения для дальнейшего воздействия на наше сетевое оборудование.
