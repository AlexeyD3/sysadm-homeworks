# Домашнее задание к занятию "3.8. Компьютерные сети, лекция 3"

### Цель задания

В результате выполнения этого задания вы:

1. На практике познакомитесь с маршрутизацией в сетях, что позволит понять устройство больших корпоративных сетей и интернета.
2. Проверите TCP/UDP соединения на хосте (это обычный этап отладки сетевых проблем).
3. Построите сетевую диаграмму.

### Чеклист готовности к домашнему заданию

1. Убедитесь, что у вас установлен `telnet`.
2. Воспользуйтесь пакетным менеджером apt для установки.


### Инструкция к заданию

1. Создайте .md-файл для ответов на задания в своём репозитории, после выполнения прикрепите ссылку на него в личном кабинете.
2. Любые вопросы по выполнению заданий спрашивайте в чате учебной группы и/или в разделе “Вопросы по заданию” в личном кабинете.


### Инструменты/ дополнительные материалы, которые пригодятся для выполнения задания

1. [Зачем нужны dummy интерфейсы](https://tldp.org/LDP/nag/node72.html)

------

## Задание

1. Подключитесь к публичному маршрутизатору в интернет. Найдите маршрут к вашему публичному IP
```
telnet route-views.routeviews.org
Username: rviews
show ip route x.x.x.x/32
show bgp x.x.x.x/32
```
<details>
  <summary>route-views>show ip route 217.70.18.26</summary>

```bash
route-views>show ip route 217.70.18.26
Routing entry for 217.70.16.0/20, supernet
  Known via "bgp 6447", distance 20, metric 0
  Tag 3267, type external
  Last update from 194.85.40.15 00:04:52 ago
  Routing Descriptor Blocks:
  * 194.85.40.15, from 194.85.40.15, 00:04:52 ago
      Route metric is 0, traffic share count is 1
      AS Hops 3
      Route tag 3267
      MPLS label: none
```
</details>

<details>
  <summary>route-views>show bgp 217.70.18.26</summary>
  

```bash
route-views>show bgp 217.70.18.26
BGP routing table entry for 217.70.16.0/20, version 2704305589
Paths: (21 available, best #1, table default)
  Not advertised to any peer
  Refresh Epoch 1
  3267 8641 29319
    194.85.40.15 from 194.85.40.15 (185.141.126.1)
      Origin IGP, metric 0, localpref 100, valid, external, best
      path 7FE16D9EC370 RPKI State valid
      rx pathid: 0, tx pathid: 0x0
  Refresh Epoch 1
  8283 1299 29319 29319
    94.142.247.3 from 94.142.247.3 (94.142.247.3)
      Origin IGP, metric 0, localpref 100, valid, external
      Community: 1299:30000 8283:1 8283:101 8283:102
      unknown transitive attribute: flag 0xE0 type 0x20 length 0x24
        value 0000 205B 0000 0000 0000 0001 0000 205B
              0000 0005 0000 0001 0000 205B 0000 0005
              0000 0002 
      path 7FE1884804A8 RPKI State valid
      rx pathid: 0, tx pathid: 0
  Refresh Epoch 1
  53767 6939 35598 29319 29319 29319
    162.251.163.2 from 162.251.163.2 (162.251.162.3)
      Origin IGP, localpref 100, valid, external
      Community: 53767:2000
      path 7FE154B338C8 RPKI State valid
      rx pathid: 0, tx pathid: 0
  Refresh Epoch 1
  7018 1299 29319 29319
    12.0.1.63 from 12.0.1.63 (12.0.1.63)
      Origin IGP, localpref 100, valid, external
      Community: 7018:5000 7018:37232
      path 7FE03DD47D88 RPKI State valid
      rx pathid: 0, tx pathid: 0
  Refresh Epoch 1
  3303 20485 8641 29319
    217.192.89.50 from 217.192.89.50 (138.187.128.158)
      Origin IGP, localpref 100, valid, external
```
</details>

2. Создайте dummy0 интерфейс в Ubuntu. Добавьте несколько статических маршрутов. Проверьте таблицу маршрутизации.
```bash
wolin@wolinubuntu:~$ sudo modprobe -v dummy numdummies=2
wolin@wolinubuntu:~$ sudo ip link add dummy0 type dummy
wolin@wolinubuntu:~$ sudo ip addr add 192.168.1.1/24 dev dummy0
wolin@wolinubuntu:~$ sudo ip link set dummy0 up
wolin@wolinubuntu:~$ sudo ip ro add to 192.168.1.140 via 192.168.1.1
wolin@wolinubuntu:~$ sudo ip ro add to 192.168.1.142 via 192.168.1.1
wolin@wolinubuntu:~$ route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.1.1     0.0.0.0         UG    600    0        0 wlp5s0
169.254.0.0     0.0.0.0         255.255.0.0     U     1000   0        0 wlp5s0
192.168.1.0     0.0.0.0         255.255.255.0   U     0      0        0 dummy0
192.168.1.0     0.0.0.0         255.255.255.0   U     600    0        0 wlp5s0
192.168.1.140   192.168.1.1     255.255.255.255 UGH   0      0        0 dummy0
192.168.1.142   192.168.1.1     255.255.255.255 UGH   0      0        0 dummy0
```

3. Проверьте открытые TCP порты в Ubuntu, какие протоколы и приложения используют эти порты? Приведите несколько примеров.
```bash
ss -s #статистика по протоколам
ss -p #приложения и порты
так же с помощью netstat можно посмотреть какой порт и протокол использует демон ssh:
wolin@wolinubuntu:~$ sudo netstat -ntlp | grep sshd
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1012/sshd: /usr/sbi 
tcp6       0      0 :::22                   :::*                    LISTEN      1012/sshd: /usr/sbi 
```

4. Проверьте используемые UDP сокеты в Ubuntu, какие протоколы и приложения используют эти порты?
```bash
wolin@wolinubuntu:~$ sudo netstat -pu | grep udp
udp        0      0 192.168.1.118:bootpc    192.168.1.1:bootps      ESTABLISHED 888/NetworkManager 
```

5. Используя diagrams.net, создайте L3 диаграмму вашей домашней сети или любой другой сети, с которой вы работали. 
https://i.ibb.co/dQ8y9Fv/Screenshot-from-2023-02-22-15-31-59.png

*В качестве решения ответьте на вопросы, опишите, каким образом эти ответы были получены и приложите по неоходимости скриншоты*

 ---
 
## Задание для самостоятельной отработки* (необязательно к выполнению)

6. Установите Nginx, настройте в режиме балансировщика TCP или UDP.

7. Установите bird2, настройте динамический протокол маршрутизации RIP.

8. Установите Netbox, создайте несколько IP префиксов, используя curl проверьте работу API.

----

### Правила приема домашнего задания

В личном кабинете отправлена ссылка на .md файл в вашем репозитории.

-----

### Критерии оценки

Зачет - выполнены все задания, ответы даны в развернутой форме, приложены соответствующие скриншоты и файлы проекта, в выполненных заданиях нет противоречий и нарушения логики.

На доработку - задание выполнено частично или не выполнено, в логике выполнения заданий есть противоречия, существенные недостатки. 
 
Обязательными к выполнению являются задачи без указания звездочки. Их выполнение необходимо для получения зачета и диплома о профессиональной переподготовке.
Задачи со звездочкой (*) являются дополнительными задачами и/или задачами повышенной сложности. Они не являются обязательными к выполнению, но помогут вам глубже понять тему.
