# Домашнее задание к занятию "3.5. Файловые системы"

### Цель задания

В результате выполнения этого задания вы: 

1. Научитесь работать с инструментами разметки жестких дисков, виртуальных разделов - RAID массивами и логическими томами, конфигурациями файловых систем. Основная задача - понять, какие слои абстракций могут нас отделять от файловой системы до железа. Обычно инженер инфраструктуры не сталкивается напрямую с настройкой LVM или RAID, но иметь понимание, как это работает - необходимо.
1. Создадите нештатную ситуацию работы жестких дисков и поймете, как система RAID обеспечивает отказоустойчивую работу.


### Чеклист готовности к домашнему заданию

1. Убедитесь, что у вас на новой виртуальной машине (шаг 3 задания) установлены следующие утилиты - `mdadm`, `fdisk`, `sfdisk`, `mkfs`, `lsblk`, `wget`.  
2. Воспользуйтесь пакетным менеджером apt для установки необходимых инструментов


### Инструменты/ дополнительные материалы, которые пригодятся для выполнения задания

1. Разряженные файлы - [sparse](https://ru.wikipedia.org/wiki/%D0%A0%D0%B0%D0%B7%D1%80%D0%B5%D0%B6%D1%91%D0%BD%D0%BD%D1%8B%D0%B9_%D1%84%D0%B0%D0%B9%D0%BB)
2. [Подробный анализ производительности RAID,3-19 страницы](https://www.baarf.dk/BAARF/0.Millsap1996.08.21-VLDB.pdf).
3. [RAID5 write hole](https://www.intel.com/content/www/us/en/support/articles/000057368/memory-and-storage.html).


------

## Задание

1. Узнайте о [sparse](https://ru.wikipedia.org/wiki/%D0%A0%D0%B0%D0%B7%D1%80%D0%B5%D0%B6%D1%91%D0%BD%D0%BD%D1%8B%D0%B9_%D1%84%D0%B0%D0%B9%D0%BB) (разряженных) файлах.

```bash
sprase - файл, в котором последовательности нулевых байтов заменены на информацию об этих последовательностях (список дыр).

Преимущества:

экономия дискового пространства. Использование разрежённых файлов считается одним из способов сжатия данных на уровне файловой системы;
отсутствие временных затрат на запись нулевых байт;
увеличение срока службы запоминающих устройств.
Недостатки:

накладные расходы на работу со списком дыр;
фрагментация файла при частой записи данных в дыры;
невозможность записи данных в дыры при отсутствии свободного места на диске;
невозможность использования других индикаторов дыр, кроме нулевых байт.
```

1. Могут ли файлы, являющиеся жесткой ссылкой на один объект, иметь разные права доступа и владельца? Почему?
```bash
Не могут, т.к. это ссылки на один и тот же inode - в котором хранятся права доступа и имя владельца.
```

2. Сделайте `vagrant destroy` на имеющийся инстанс Ubuntu. Замените содержимое Vagrantfile следующим:

    ```bash
    Vagrant.configure("2") do |config|
      config.vm.box = "bento/ubuntu-20.04"
      config.vm.provider :virtualbox do |vb|
        lvm_experiments_disk0_path = "/tmp/lvm_experiments_disk0.vmdk"
        lvm_experiments_disk1_path = "/tmp/lvm_experiments_disk1.vmdk"
        vb.customize ['createmedium', '--filename', lvm_experiments_disk0_path, '--size', 2560]
        vb.customize ['createmedium', '--filename', lvm_experiments_disk1_path, '--size', 2560]
        vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk0_path]
        vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 2, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk1_path]
      end
    end
    ```

    Данная конфигурация создаст новую виртуальную машину с двумя дополнительными неразмеченными дисками по 2.5 Гб.

3. Используя `fdisk`, разбейте первый диск на 2 раздела: 2 Гб, оставшееся пространство.

```bash
vagrant@vagrant:~$ sudo fdisk /dev/sdb

Welcome to fdisk (util-linux 2.34).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0xfc0d172e.

Command (m for help): F
Unpartitioned space /dev/sdb: 2.51 GiB, 2683305984 bytes, 5240832 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes

Start     End Sectors  Size
 2048 5242879 5240832  2.5G

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 
First sector (2048-5242879, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-5242879, default 5242879): +2G

Created a new partition 1 of type 'Linux' and of size 2 GiB.

Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (2-4, default 2): 
First sector (4196352-5242879, default 4196352): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (4196352-5242879, default 5242879): 

Created a new partition 2 of type 'Linux' and of size 511 MiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

```

4. Используя `sfdisk`, перенесите данную таблицу разделов на второй диск.

```bash
vagrant@vagrant:~$ sudo sfdisk -d /dev/sdb > sdb.dump
vagrant@vagrant:~$ sudo sfdisk /dev/sdc < sdb.dump
Checking that no-one is using this disk right now ... OK

Disk /dev/sdc: 2.51 GiB, 2684354560 bytes, 5242880 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

>>> Script header accepted.
>>> Script header accepted.
>>> Script header accepted.
>>> Script header accepted.
>>> Created a new DOS disklabel with disk identifier 0xfc0d172e.
/dev/sdc1: Created a new partition 1 of type 'Linux' and of size 2 GiB.
/dev/sdc2: Created a new partition 2 of type 'Linux' and of size 511 MiB.
/dev/sdc3: Done.

New situation:
Disklabel type: dos
Disk identifier: 0xfc0d172e

Device     Boot   Start     End Sectors  Size Id Type
/dev/sdc1          2048 4196351 4194304    2G 83 Linux
/dev/sdc2       4196352 5242879 1046528  511M 83 Linux

The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

```

5. Соберите `mdadm` RAID1 на паре разделов 2 Гб.

```bash
vagrant@vagrant:~$ sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sd[bc]1
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
Continue creating array? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.

```

6. Соберите `mdadm` RAID0 на второй паре маленьких разделов.

```bash
vagrant@vagrant:~$ sudo mdadm --create /dev/md1 --level=0 --raid-devices=2 /dev/sd[bc]2
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md1 started.
```

7. Создайте 2 независимых PV на получившихся md-устройствах.

```bash
vagrant@vagrant:~$ sudo pvcreate /dev/md0
  Physical volume "/dev/md0" successfully created.
vagrant@vagrant:~$ sudo pvcreate /dev/md1
  Physical volume "/dev/md1" successfully created.
```

8. Создайте общую volume-group на этих двух PV.

```bash
vagrant@vagrant:~$ sudo vgcreate devops-netology /dev/md[01]
  Volume group "devops-netology" successfully created
vagrant@vagrant:~$ sudo vgs
  VG              #PV #LV #SN Attr   VSize   VFree 
  devops-netology   2   0   0 wz--n-  <2.99g <2.99g
  ubuntu-vg         1   1   0 wz--n- <62.50g 31.25g
```

9. Создайте LV размером 100 Мб, указав его расположение на PV с RAID0.

```bash
vagrant@vagrant:~$ sudo lvcreate -L 100m -n devops-netology-lv devops-netology /dev/md1
  Logical volume "devops-netology-lv" created.
vagrant@vagrant:~$ sudo lvs -o +devices
  LV                 VG              Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert Devices     
  devops-netology-lv devops-netology -wi-a----- 100.00m                                                     /dev/md1(0) 
  ubuntu-lv          ubuntu-vg       -wi-ao---- <31.25g                                                     /dev/sda3(0)

```

10. Создайте `mkfs.ext4` ФС на получившемся LV.

```bash
vagrant@vagrant:~$ sudo mkfs.ext4 -L netology-ext4 -m 1 /dev/mapper/devops--netology-devops--netology--lv
mke2fs 1.45.5 (07-Jan-2020)
Creating filesystem with 25600 4k blocks and 25600 inodes

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (1024 blocks): done
Writing superblocks and filesystem accounting information: done
```

11. Смонтируйте этот раздел в любую директорию, например, `/tmp/new`.

```bash
vagrant@vagrant:~$ mkdir /tmp/new
vagrant@vagrant:~$ sudo mount  /dev/mapper/devops--netology-devops--netology--lv /tmp/new
vagrant@vagrant:~$ mount | grep netology
/dev/mapper/devops--netology-devops--netology--lv on /tmp/new type ext4 (rw,relatime,stripe=256)
```

12. Поместите туда тестовый файл, например `wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz`.

```bash
vagrant@vagrant:~$ cd /tmp/new
vagrant@vagrant:/tmp/new$ sudo wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz
--2023-02-20 11:58:00--  https://mirror.yandex.ru/ubuntu/ls-lR.gz
Resolving mirror.yandex.ru (mirror.yandex.ru)... 213.180.204.183, 2a02:6b8::183
Connecting to mirror.yandex.ru (mirror.yandex.ru)|213.180.204.183|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 24748909 (24M) [application/octet-stream]
Saving to: ‘/tmp/new/test.gz’

/tmp/new/test.gz    100%[===================>]  23.60M  4.96MB/s    in 4.6s    

2023-02-20 11:58:10 (5.08 MB/s) - ‘/tmp/new/test.gz’ saved [24748909/24748909]

```

13. Прикрепите вывод `lsblk`.

```bash
vagrant@vagrant:/tmp/new$ lsblk
NAME                                 MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
loop0                                  7:0    0 61.9M  1 loop  /snap/core20/1328
loop2                                  7:2    0 67.2M  1 loop  /snap/lxd/21835
loop3                                  7:3    0 63.3M  1 loop  /snap/core20/1822
loop4                                  7:4    0 49.9M  1 loop  /snap/snapd/18357
loop5                                  7:5    0 91.9M  1 loop  /snap/lxd/24061
sda                                    8:0    0   64G  0 disk  
├─sda1                                 8:1    0    1M  0 part  
├─sda2                                 8:2    0  1.5G  0 part  /boot
└─sda3                                 8:3    0 62.5G  0 part  
  └─ubuntu--vg-ubuntu--lv            253:0    0 31.3G  0 lvm   /
sdb                                    8:16   0  2.5G  0 disk  
├─sdb1                                 8:17   0    2G  0 part  
│ └─md0                                9:0    0    2G  0 raid1 
└─sdb2                                 8:18   0  511M  0 part  
  └─md1                                9:1    0 1018M  0 raid0 
    └─devops--netology-devops--netology--lv
                                     253:1    0  100M  0 lvm   /tmp/new
sdc                                    8:32   0  2.5G  0 disk  
├─sdc1                                 8:33   0    2G  0 part  
│ └─md0                                9:0    0    2G  0 raid1 
└─sdc2                                 8:34   0  511M  0 part  
  └─md1                                9:1    0 1018M  0 raid0 
    └─devops--netology-devops--netology--lv
```

14. Протестируйте целостность файла:

    ```bash
    root@vagrant:~# gzip -t /tmp/new/test.gz
    root@vagrant:~# echo $?
    0
    ```
15. Используя pvmove, переместите содержимое PV с RAID0 на RAID1.

```bash
vagrant@vagrant:/tmp/new$ sudo pvmove -n devops-netology-lv /dev/md1 /dev/md0
  /dev/md1: Moved: 12.00%
  /dev/md1: Moved: 100.00%
vagrant@vagrant:/tmp/new$ sudo lvs -o +devices
  LV                 VG              Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert Devices     
  devops-netology-lv devops-netology -wi-ao---- 100.00m                                                     /dev/md0(0) 
  ubuntu-lv          ubuntu-vg       -wi-ao---- <31.25g                                                     /dev/sda3(0)

```

16. Сделайте `--fail` на устройство в вашем RAID1 md.

```bash
vagrant@vagrant:/tmp/new$ sudo mdadm --fail /dev/md0 /dev/sdb1
mdadm: set /dev/sdb1 faulty in /dev/md0
```

17. Подтвердите выводом `dmesg`, что RAID1 работает в деградированном состоянии.

```bash
vagrant@vagrant:/tmp/new$ dmesg | grep md0 | tail -n 2
[ 3234.899235] md/raid1:md0: Disk failure on sdb1, disabling device.
               md/raid1:md0: Operation continuing on 1 devices.
```

18. Протестируйте целостность файла, несмотря на "сбойный" диск он должен продолжать быть доступен:

    ```bash
    root@vagrant:~# gzip -t /tmp/new/test.gz
    root@vagrant:~# echo $?
    0
    ```

19. Погасите тестовый хост, `vagrant destroy`.

```bash
wolin@wolinubuntu:~/netology/vagrant_1$ vagrant destroy
    default: Are you sure you want to destroy the 'default' VM? [y/N] y
==> default: Forcing shutdown of VM...
==> default: Destroying VM and associated drives...
```
 
 ---

 
*В качестве решения ответьте на вопросы и опишите, каким образом эти ответы были получены*

----

### Правила приема домашнего задания

В личном кабинете отправлена ссылка на .md файл в вашем репозитории.


### Критерии оценки

Зачет - выполнены все задания, ответы даны в развернутой форме, приложены соответствующие скриншоты и файлы проекта, в выполненных заданиях нет противоречий и нарушения логики.

На доработку - задание выполнено частично или не выполнено, в логике выполнения заданий есть противоречия, существенные недостатки. 
