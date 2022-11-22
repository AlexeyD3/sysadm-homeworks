# Домашнее задание к занятию "3.4. Операционные системы, лекция 2"

1. На лекции мы познакомились с [node_exporter](https://github.com/prometheus/node_exporter/releases). В демонстрации его исполняемый файл запускался в background. Этого достаточно для демо, но не для настоящей production-системы, где процессы должны находиться под внешним управлением. Используя знания из лекции по systemd, создайте самостоятельно простой [unit-файл](https://www.freedesktop.org/software/systemd/man/systemd.service.html) для node_exporter:

    * поместите его в автозагрузку,
    * предусмотрите возможность добавления опций к запускаемому процессу через внешний файл (посмотрите, например, на `systemctl cat cron`),
    * удостоверьтесь, что с помощью systemctl процесс корректно стартует, завершается, а после перезагрузки автоматически поднимается.  
  
  
> Создал unit-файл, пользователя и группу для node_exporter, поместил в `usr/local/bin` запустил и командой `sudo systemctl enable node_exporter` добавил в автозагрузку  
  
```bash 
sudo systemctl status node_exporter
● node_exporter.service - Prometheus Node Exporter
     Loaded: loaded (/etc/systemd/system/node_exporter.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2022-11-15 12:58:54 UTC; 46s ago
   Main PID: 619 (node_exporter)
      Tasks: 4 (limit: 1131)
     Memory: 14.5M
     CGroup: /system.slice/node_exporter.service
             └─619 /usr/local/bin/node_exporter

Nov 15 12:58:54 ubuntu-focal node_exporter[619]: ts=2022-11-15T12:58:54.854Z caller=node_exporter.go:115 level=info collector=thermal_zone
Nov 15 12:58:54 ubuntu-focal node_exporter[619]: ts=2022-11-15T12:58:54.854Z caller=node_exporter.go:115 level=info collector=time
Nov 15 12:58:54 ubuntu-focal node_exporter[619]: ts=2022-11-15T12:58:54.854Z caller=node_exporter.go:115 level=info collector=timex
Nov 15 12:58:54 ubuntu-focal node_exporter[619]: ts=2022-11-15T12:58:54.854Z caller=node_exporter.go:115 level=info collector=udp_queues
Nov 15 12:58:54 ubuntu-focal node_exporter[619]: ts=2022-11-15T12:58:54.854Z caller=node_exporter.go:115 level=info collector=uname
Nov 15 12:58:54 ubuntu-focal node_exporter[619]: ts=2022-11-15T12:58:54.854Z caller=node_exporter.go:115 level=info collector=vmstat
Nov 15 12:58:54 ubuntu-focal node_exporter[619]: ts=2022-11-15T12:58:54.854Z caller=node_exporter.go:115 level=info collector=xfs
Nov 15 12:58:54 ubuntu-focal node_exporter[619]: ts=2022-11-15T12:58:54.854Z caller=node_exporter.go:115 level=info collector=zfs
Nov 15 12:58:54 ubuntu-focal node_exporter[619]: ts=2022-11-15T12:58:54.854Z caller=node_exporter.go:199 level=info msg="Listening on" address=:9100
Nov 15 12:58:54 ubuntu-focal node_exporter[619]: ts=2022-11-15T12:58:54.856Z caller=tls_config.go:195 level=info msg="TLS is disabled." http2=false
```
  
> содержимое unit-файла:  
  
```bash
[Unit]
Description=Prometheus Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
EnvironmentFile=-/etc/default/node_exporter
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter $OPTIONS
Restart=always

[Install]
WantedBy=multi-user.target
```  
  
> содержимое `EnvironmentFile=-/etc/default/node_exporter`
```bash
OPTIONS="--collector.disable-defaults --collector.netstat --collector.meminfo --collector.cpu --collector.filesystem"
```  
  
> в `Vagrantfile` добавил строку `config.vm.network "forwarded_port", guest: 9100, host: 9109, host_ip: "127.0.0.1", auto_correct: true`  
> `vagrant reload`, виртуальная машина перезапущена, порты проброшены, работу `node_exporter` (а так же его автозагрузку) и доступность метрик проверил прямо на хостовой машине по адресу `localhost:9109/metrics`  
   

1. Ознакомьтесь с опциями node_exporter и выводом `/metrics` по-умолчанию. Приведите несколько опций, которые вы бы выбрали для базового мониторинга хоста по CPU, памяти, диску и сети.

>    CPU:  
>       `node_cpu_second_total`  
>  
> Memory:  
>       `node_memory_MemTotal_bytes`  
>       `node_memory_MemFree_bytes`  
>       `node_memory_Cached_bytes`  
>       `node_memory_Buffers_bytes`  
>  
>   Disk:  
>       `node_filesystem_size_bytes`  
>       `node_filesystem_avail_bytes`  
>       `node_filesystem_free_bytes`  
>       `node_disk_read_bytes_total`  
>       `node_disk_written_bytes_total`  
>  
>Network:  
>       `node_network_receive_bytes_total`  
>       `node_network_transmit_bytes_total`  
   

1. Установите в свою виртуальную машину [Netdata](https://github.com/netdata/netdata). Воспользуйтесь [готовыми пакетами](https://packagecloud.io/netdata/netdata/install) для установки (`sudo apt install -y netdata`). После успешной установки:
    * в конфигурационном файле `/etc/netdata/netdata.conf` в секции [web] замените значение с localhost на `bind to = 0.0.0.0`,
    * добавьте в Vagrantfile проброс порта Netdata на свой локальный компьютер и сделайте `vagrant reload`:

    ```bash
    config.vm.network "forwarded_port", guest: 19999, host: 19999
    ```

    После успешной перезагрузки в браузере *на своем ПК* (не в виртуальной машине) вы должны суметь зайти на `localhost:19999`. Ознакомьтесь с метриками, которые по умолчанию собираются Netdata и с комментариями, которые даны к этим метрикам.  
  
> все получилось: [screenshot веб интерфейса netdata](https://i.ibb.co/qyMvwTJ/Screenshot-from-2022-11-15-17-49-12.png)
> стоит отметить, что `vagrant` пробросил порт на `2200` (о чём сообщил при запуске виртуальной машины) из-за того, что порт `19999` был занят под локальный `netdata` 
1. Можно ли по выводу `dmesg` понять, осознает ли ОС, что загружена не на настоящем оборудовании, а на системе виртуализации?  
  
> Да, можно, в том числе можно узнать провайдера виртуальной машины
```bash
vagrant@ubuntu-focal:~/netology$ dmesg | grep -i Virtual
[    0.000000] DMI: innotek GmbH VirtualBox/VirtualBox, BIOS VirtualBox 12/01/2006
[    0.002460] CPU MTRRs all blank - virtualized system.
[    0.071722] Booting paravirtualized kernel on KVM
[    3.424336] systemd[1]: Detected virtualization oracle.

```  
  
1. Как настроен sysctl `fs.nr_open` на системе по-умолчанию? Узнайте, что означает этот параметр. Какой другой существующий лимит не позволит достичь такого числа (`ulimit --help`)?  
```bash
sudo sysctl -a | grep fs.nr_open
fs.nr_open = 1048576
```
> этот параметр (по умолчанию 1048576) ограничивает лимит открытых файлов для каждого процесса (изменить с сохранением даже при перезагрузке мы можем в `/etc/sysctl.conf fs.file-max=<число>`)
> `ulimit -Sn` - "мягкий" лимит, будет выведено предупреждение на превышение лимита, но файлы продолжат открываться (если "мягкий" лимит не поддерживается процессом лимит будет действовать точно так же, как "жесткий")  
> `ulimit -Hn` - "жесткий" лимит, ограничит открытие файлов
> настройки `ulimit` не могут позволить процессам превышать лимит установленный в `fs.nr_open`
> Так же лимиты накладываются через:
> SystemD – `system.conf` параметры: `LimitNOFILE=<число>`	`DefaultLimitNOFILE=<число>`
> PAM (Pluggable Authentication Modules) - `limits.conf` параметр: `nofile`

1. Запустите любой долгоживущий процесс (не `ls`, который отработает мгновенно, а, например, `sleep 1h`) в отдельном неймспейсе процессов; покажите, что ваш процесс работает под PID 1 через `nsenter`. Для простоты работайте в данном задании под root (`sudo -i`). Под обычным пользователем требуются дополнительные опции (`--map-root-user`) и т.д.  
  
```bash
root@ubuntu-focal:~# unshare -f --pid --mount-proc ping ya.ru >/dev/null
^Z
[1]+  Stopped                 unshare -f --pid --mount-proc ping ya.ru > /dev/null
root@ubuntu-focal:~# ps | grep unshare
  20525 pts/3    00:00:00 unshare
root@ubuntu-focal:~# nsenter --target 20525 --pid --mount
root@ubuntu-focal:/# pstree -p
ping(1)
 
```  
  
1. Найдите информацию о том, что такое `:(){ :|:& };:`. Запустите эту команду в своей виртуальной машине Vagrant с Ubuntu 20.04 (**это важно, поведение в других ОС не проверялось**). Некоторое время все будет "плохо", после чего (минуты) – ОС должна стабилизироваться. Вызов `dmesg` расскажет, какой механизм помог автоматической стабилизации. Как настроен этот механизм по-умолчанию, и как изменить число процессов, которое можно создать в сессии?

> это так называемая fork-бомба, задаём фнкцию с именем `:`, затем передаём в неё саму себя на выполнение в фон (с помощью `&`) и снова вызываем с помощью последнего `:`, который ранее объявили функцией. Процессы множатся забирая все доступные `PID`.  
> вывод `dmesg:`  
> `cgroup: fork rejected by pids controller in /user.slice/user-1000.slice/session-12.scope`  
> механизм `cgroup` помог выйти из цикла бесконечного создания процессов при превышении лимита количества запущенных процессов для одного пользователя.  
> `ulimit -u` ограничивает количество процессов для каждой пользовательской сессии (в данном случае лимит 3770).  
> мы можем изменить лимит командой `ulimit -u <число>`, но только для текущей сессии, установленный лимит вернется на исходное значение.  
> для выставления постоянного лимита нужно обратиться к одному из глобальных файлов конфигурации `PAM` (упомнутой мной ранее для лимита открытых файлов)   
> `/etc/security/limits.conf` и задать парметры с помощью шаблона:  
> `<группа пользователя> <тип ограничения мягкое\жесткое> <предмет ограничения> <значение>`  
> например: `<@prod> <hard> <nproc> <4770>`
```bash
vagrant@ubuntu-focal:/$ ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 3770
max locked memory       (kbytes, -l) 65536
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 3770
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```
 ---

## Как сдавать задания

Обязательными к выполнению являются задачи без указания звездочки. Их выполнение необходимо для получения зачета и диплома о профессиональной переподготовке.

Задачи со звездочкой (*) являются дополнительными задачами и/или задачами повышенной сложности. Они не являются обязательными к выполнению, но помогут вам глубже понять тему.

Домашнее задание выполните в файле readme.md в github репозитории. В личном кабинете отправьте на проверку ссылку на .md-файл в вашем репозитории.

Также вы можете выполнить задание в [Google Docs](https://docs.google.com/document/u/0/?tgif=d) и отправить в личном кабинете на проверку ссылку на ваш документ.
Название файла Google Docs должно содержать номер лекции и фамилию студента. Пример названия: "1.1. Введение в DevOps — Сусанна Алиева".

Если необходимо прикрепить дополнительные ссылки, просто добавьте их в свой Google Docs.

Перед тем как выслать ссылку, убедитесь, что ее содержимое не является приватным (открыто на комментирование всем, у кого есть ссылка), иначе преподаватель не сможет проверить работу. Чтобы это проверить, откройте ссылку в браузере в режиме инкогнито.

[Как предоставить доступ к файлам и папкам на Google Диске](https://support.google.com/docs/answer/2494822?hl=ru&co=GENIE.Platform%3DDesktop)

[Как запустить chrome в режиме инкогнито ](https://support.google.com/chrome/answer/95464?co=GENIE.Platform%3DDesktop&hl=ru)

[Как запустить  Safari в режиме инкогнито ](https://support.apple.com/ru-ru/guide/safari/ibrw1069/mac)

Любые вопросы по решению задач задавайте в чате Slack.

---
