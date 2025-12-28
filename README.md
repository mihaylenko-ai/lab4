# Лабораторная работа №3. Underlay на основе eBGP

## Цель

Настроить протокол eBGP для Underlay сети.

## Постановка задачи

Произвести настройку протокола eBGP для обеспечения связности Underlay сети.

## Описание задачи

В поставленой задаче требуется произвести настройку протокола eBGP для побеспечения IP связности Underlay сети в рамках CLOS архитектуры. 

В качестве исходной, взять сеть из лабораторной работы №1.

# Введение

## Термины

- Underlay-сеть (сеть подложки) — физическая сеть, состоящая из аппаратных устройств, таких как коммутаторы и кабели, которая обеспечивает базовую связность и передачу пакетов
между физическими узлами
- Spine-Leaf (CLOS) архитектура - широко применяемая в рамках высоконагруженных сетей архитекрута сети, состоящая из Leaf устройств, обеспецивающих подключение конечный устройств, и 
Spine устройств, обеспечивающих связность нижестоящих Leaf устройств.
- eBGP - .

## Оборудование

1. Виртуальный коммутатор окружения Eve-NG на базе операционной системы vEOS версии EOS-4.29.2F
2. Виртуальный хост окружения Eve-NG

## Именование и термины

В качестве исходной сети была использована ранее спроектированная лабораторная сеть из Лаборатоной роботы №1. Все сетевые устройства имеют свои уникальные имена, отражающие их
функциональное назначение:

- S1 - Spine коммутатор №1
- S2 - Spine коммутатор №2

- L1 - Leaf коммутатор №1
- L2 - Leaf коммутатор №2
- L3 - Leaf коммутатор №3

- PC11 - Виртуальный хост №1, подкчобенный к Leaf коммутатору №1
- PC21 - Виртуальный хост №1, подкчобенный к Leaf коммутатору №2
- PC31 - Виртуальный хост №1, подкчобенный к Leaf коммутатору №3
- PC32 - Виртуальный хост №2, подкчобенный к Leaf коммутатору №3

### Таблица адресов сетевых устройств Spine

|N|Device|Port|IP Address|Prefix|
|:-:|:-:|:-:|:-:|:-:|
|1|S1|eth1|10.1.1.1|30|
|2|S1|eth2|10.1.2.1|30|
|3|S1|eth3|10.1.3.1|30|
|4|S2|eth1|10.2.1.1|30|
|5|S2|eth2|10.2.2.1|30|
|6|S2|eth3|10.2.3.1|30|

### Таблица адресов сетевых устройств Leaf

|N|Device|Port|IP Address|Prefix|
|:-:|:-:|:-:|:-:|:-:|
|1|L1|eth1|10.1.1.2|30|
|2|L1|eth2|10.2.1.2|30|
|3|L1|eth8|192.168.1.1|24|
|4|L2|eth1|10.1.2.2|30|
|5|L2|eth2|10.2.2.2|30|
|6|L2|eth8|192.168.2.1|24|
|7|L3|eth1|10.1.3.2|30|
|8|L3|eth2|10.2.3.2|30|
|9|L3|eth8|192.168.3.1|24|

### Таблица Loopback адресов сетевых устройств

|N|Device|Port|IP Address|Prefix|
|:-:|:-:|:-:|:-:|:-:|
|1|S1|Lo0|172.16.1.1|32|
|2|S2|Lo0|172.16.2.1|32|
|3|L1|Lo0|172.16.11.1|32|
|4|L2|Lo0|172.16.12.1|32|
|5|L3|Lo0|172.16.13.1|32|

### Таблица адресов конечных устройств

|N|Device|Port|IP Address|Prefix|
|:-:|:-:|:-:|:-:|:-:|
|1|PC11|eth0|192.168.1.10|24|
|2|PC21|eth0|192.168.2.10|24|
|3|PC31|eth0|192.168.3.10|24|
|4|PC32|eth0|192.168.3.11|24|

## Описание стенда

В рамках лабораторной работы на предоставленном учебным центром лабораторном окружении было использовано пять коммутаторов. Данные коммутаторы были соеденины линиями связи по схеме CLOS,
два из которых (S1 и S2) выступают в качестве Spine устройств, и три (L1,L2 и L3) в качестве Leaf устройств. Схема сети в рамках лабораторного окружения представлена на рисунке ниже

![netmap](images/netmap.jpg)

## Настройка устройств

В рамках учебной лабораторной среды на всех сетевых устройствах был настроен протокол eBGP. 

Ниже приведены частичные настройки файлов конфигураций сетевых устройств:

#### Spine устройства

**S1**

```
hostname S1
!
interface Ethernet1
   description <leaf L1>
   mtu 9214
   no switchport
   ip address 10.1.1.1/30
!
interface Ethernet2
   description <leaf L2>
   mtu 9214
   no switchport
   ip address 10.1.2.1/30
!
interface Ethernet3
   description <leaf L3>
   mtu 9214
   no switchport
   ip address 10.1.3.1/30
!
interface Loopback0
   ip address 172.16.1.1/32
!
ip routing
!
peer-filter LEAFS_ASN
   10 match as-range 65501-65503 result accept
!
router bgp 65550
   router-id 1.0.1.1
   no bgp default ipv4-unicast
   timers bgp 1 3
   distance bgp 20 200 200
   bgp listen range 10.0.0.0/8 peer-group LEAF peer-filter LEAFS_ASN
   neighbor LEAF peer group
   neighbor LEAF out-delay 0
   neighbor LEAF bfd
   !
   address-family ipv4
      neighbor LEAF activate
      network 172.16.1.1/32
!
end
```

**S2**

```
hostname S2
!
interface Ethernet1
   description <leaf L1>
   mtu 9214
   no switchport
   ip address 10.2.1.1/30
!
interface Ethernet2
   description <leaf L2>
   mtu 9214
   no switchport
   ip address 10.2.2.1/30
!
interface Ethernet3
   description <leaf L3>
   mtu 9214
   no switchport
   ip address 10.2.3.1/30
!
interface Loopback0
   ip address 172.16.2.1/32
!
ip routing
!
peer-filter LEAFS_ASN
   10 match as-range 65501-65503 result accept
!
router bgp 65550
   router-id 1.0.1.2
   no bgp default ipv4-unicast
   timers bgp 1 3
   distance bgp 20 200 200
   bgp listen range 10.0.0.0/8 peer-group LEAF peer-filter LEAFS_ASN
   neighbor LEAF peer group
   neighbor LEAF out-delay 0
   neighbor LEAF bfd
   !
   address-family ipv4
      neighbor LEAF activate
      network 172.16.2.1/32
!
end
```

#### Leaf устройства

**L1**

```
hostname L1
!
interface Ethernet1
   description <spine S1>
   mtu 9214
   no switchport
   ip address 10.1.1.2/30
!
interface Ethernet2
   description <spine S2>
   mtu 9214
   no switchport
   ip address 10.2.1.2/30
!
interface Ethernet8
   description <PC11>
   mtu 9214
   no switchport
   ip address 192.168.1.1/24
!
interface Loopback0
   ip address 172.16.11.1/32
!
ip routing
!
router bgp 65501
   router-id 1.0.0.1
   no bgp default ipv4-unicast
   timers bgp 1 3
   distance bgp 20 200 200
   maximum-paths 2 ecmp 2
   neighbor SPINE peer group
   neighbor SPINE remote-as 65550
   neighbor SPINE out-delay 0
   neighbor SPINE bfd
   neighbor 10.1.1.1 peer group SPINE
   neighbor 10.2.1.1 peer group SPINE
   !
   address-family ipv4
      neighbor SPINE activate
      network 172.16.11.1/32
      network 192.168.1.0/24
!
end
```

**L2**

```
hostname L2
!
interface Ethernet1
   description <spine S1>
   mtu 9214
   no switchport
   ip address 10.1.2.2/30
!
interface Ethernet2
   description <spine S2>
   mtu 9214
   no switchport
   ip address 10.2.2.2/30
!
interface Ethernet8
   description <PC21>
   mtu 9214
   no switchport
   ip address 192.168.2.1/24
!
interface Loopback0
   ip address 172.16.12.1/32
!
ip routing
!
router bgp 65502
   router-id 1.0.0.2
   no bgp default ipv4-unicast
   timers bgp 1 3
   distance bgp 20 200 200
   maximum-paths 2 ecmp 2
   neighbor SPINE peer group
   neighbor SPINE remote-as 65550
   neighbor SPINE out-delay 0
   neighbor SPINE bfd
   neighbor 10.1.2.1 peer group SPINE
   neighbor 10.2.2.1 peer group SPINE
   !
   address-family ipv4
      neighbor SPINE activate
      network 172.16.12.1/32
      network 192.168.2.0/24
!
end
```

**L3**

```
hostname L3
!
vlan 3
   name users
!
interface Ethernet1
   description <spine S1>
   mtu 9214
   no switchport
   ip address 10.1.3.2/30
!
interface Ethernet2
   description <spine S2>
   mtu 9214
   no switchport
   ip address 10.2.3.2/30
!
interface Ethernet7
   description <PC31>
   mtu 9214
   switchport access vlan 3
!
interface Ethernet8
   description <PC32>
   mtu 9214
   switchport access vlan 3
!
interface Loopback0
   ip address 172.16.13.1/32
!
interface Vlan3
   description <User`s VLAN>
   ip address 192.168.3.1/24
!
ip routing
!
router bgp 65503
   router-id 1.0.0.3
   no bgp default ipv4-unicast
   timers bgp 1 3
   distance bgp 20 200 200
   maximum-paths 2 ecmp 2
   neighbor SPINE peer group
   neighbor SPINE remote-as 65550
   neighbor SPINE out-delay 0
   neighbor SPINE bfd
   neighbor 10.1.3.1 peer group SPINE
   neighbor 10.2.3.1 peer group SPINE
   !
   address-family ipv4
      neighbor SPINE activate
      network 172.16.13.1/32
      network 192.168.3.0/24
!
end
```

## Описание типовых настроек



# Заключение

## Проверка работы сденда и результаты работы

Стенд может считаться рабочим в случае установления IP связности между всеми устройствами в рамках лабораторной сети, как следствие появления на всех устройствах в таблице
маршрутизации информации обо всех IP адресах интерфейсов иных устройствах. Так же каждое устройство должно утановить соответствующие связи со своими соседями. Тесты работы
производился посредством отправики ICMP Echo запроса с конечного устройства PC11. 

**Таблица маршрутизации eBGP на коммутаторе S1**

![S1](images/bgp_S1.jpg)

**Таблица соседства eBGP на коммутаторе S1**

![S1](images/neighbor_S1.jpg)

**Таблица маршрутизации eBGP на коммутаторе S2**

![S1](images/bgp_S2.jpg)

**Таблица соседства eBGP на коммутаторе S2**

![S1](images/neighbor_S2.jpg)

**Таблица маршрутизации eBGP на коммутаторе L1**

![S1](images/bgp_L1.jpg)

![S1](images/bgp_as_L1.jpg)

**Таблица соседства eBGP на коммутаторе L1**

![S1](images/neighbor_L1.jpg)

**Таблица маршрутизации eBGP на коммутаторе L2**

![S1](images/bgp_L2.jpg)

![S1](images/bgp_as_L2.jpg)


**Таблица соседства eBGP на коммутаторе L2**

![S1](images/neighbor_L2.jpg)

**Таблица маршрутизации eBGP на коммутаторе L3**

![S1](images/bgp_L3.jpg)

![S1](images/bgp_as_L3.jpg)


**Таблица соседства eBGP на коммутаторе L3**

![S1](images/neighbor_L3.jpg)


Теперь проверм вязность сети с тестового конечного устройства PC11 на все IP адреса интерфейсов Loopback всех сеетвых устройств, а тек же на все конечные устройства в рамках
лабораторной работы, что подтвердит IP связность всех устройств в сети:

**Получение ECHO ICMP ответа от Loopback адресов сетевых устройств лабораторной сети**

![S1](images/P11toSW.jpg)

**Получение ECHO ICMP ответа от других конечных устройств, подключенных к иным Leaf коммутаторам лабораторной сети**

![S1](images/P11toEND.jpg)


## Вывод

Был настроен протокол eBGP на рабочем стенде, собранного в соответствии с CLOS архитекрурой. Произведены испытания, показавшие наличие IP связности сетевых усторйств посредством
протокола eBGP между уровнями Spine и Leaf, а так же конечными устройствами.