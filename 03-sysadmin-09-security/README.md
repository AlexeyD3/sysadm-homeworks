# Домашнее задание к занятию "3.9. Элементы безопасности информационных систем"


### Цель задания

В результате выполнения этого задания вы: 

1. Настроите парольный менеджер, что позволит не использовать один и тот же пароль на все ресурсы и удобно работать с множеством паролей.
2. Настроите веб-сервер на работу с https. Сегодня https является стандартом в интернете. Понимание сути работы центра сертификации, цепочки сертификатов позволит понять, на чем основывается https протокол.
3. Сконфигурируете ssh клиент на работу с разными серверами по-разному, что дает большую гибкость ssh соединений. Например, к некоторым серверам мы можем обращаться по ssh через приложения, где недоступен ввод пароля.
4. Поработаете со сбором и анализом трафика, которые необходимы для отладки сетевых проблем


### Инструкция к заданию

1. Создайте .md-файл для ответов на задания в своём репозитории, после выполнения прикрепите ссылку на него в личном кабинете.
2. Любые вопросы по выполнению заданий спрашивайте в чате учебной группы и/или в разделе “Вопросы по заданию” в личном кабинете.


### Инструменты/ дополнительные материалы, которые пригодятся для выполнения задания

1. [SSL + Apache2](https://digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-ubuntu-20-04)

------

## Задание

1. Установите Bitwarden плагин для браузера. Зарегестрируйтесь и сохраните несколько паролей.

![Bitwarden](https://i.ibb.co/f8zKhbN/Screenshot-from-2023-02-22-15-44-27.png)


2. Установите Google authenticator на мобильный телефон. Настройте вход в Bitwarden акаунт через Google authenticator OTP.

![Google authenticator](https://i.ibb.co/2Kqjfdp/Screenshot-from-2023-02-22-15-51-03.png)


3. Установите apache2, сгенерируйте самоподписанный сертификат, настройте тестовый сайт для работы по HTTPS.

```bash
vagrant@vagrant:~$ sudo apt install apache2
vagrant@vagrant:~$ sudo a2enmod ssl
vagrant@vagrant:~$ sudo systemctl restart apache2
vagrant@vagrant:~$ sudo ufw allow "Apache"
vagrant@vagrant:~$ sudo openssl req -x509 -nodes -days 3650 -newkey rsa:4096 -keyout /etc/ssl/private/test.key -out /etc/ssl/certs/test.cert -subj "/C=RU/ST=Lybyanka/L=Moscow/O=example/OU=COM/CN=www.example.com"
vagrant@vagrant:~$ sudo nano /etc/apache2/sites-available/www.example.com.conf
<VirtualHost *:443>
ServerName example.com
DocumentRoot /var/www/example.com
SSLEngine on
SSLCertificateFile /etc/ssl/certs/test.cert
SSLCertificateKeyFile /etc/ssl/private/test.key
</VirtualHost>
vagrant@vagrant:~$ sudo mkdir /var/www/example.com
vagrant@vagrant:~$ sudo nano /var/www/example.com/index.html
<h1>Netology test site with SSL by Alexey D</h1>

vagrant@vagrant:~$ sudo apt-get install links
vagrant@vagrant:~$ links https://127.0.0.1
```
![сайт](https://i.ibb.co/YRh76xf/Screenshot-from-2023-02-22-16-23-00.png)

4. Проверьте на TLS уязвимости произвольный сайт в интернете (кроме сайтов МВД, ФСБ, МинОбр, НацБанк, РосКосмос, РосАтом, РосНАНО и любых госкомпаний, объектов КИИ, ВПК ... и тому подобное).

```bash
vagrant@vagrant:~/tls/testssl.sh$ ./testssl.sh -e --fast --parallel https://netology.ru/
```


```bash
###########################################################
    testssl.sh       3.2rc2 from https://testssl.sh/dev/
    (88763f4 2023-02-20 20:29:14)

      This program is free software. Distribution and
             modification under GPLv2 permitted.
      USAGE w/o ANY WARRANTY. USE IT AT YOUR OWN RISK!

       Please file bugs @ https://testssl.sh/bugs/

###########################################################

 Using "OpenSSL 1.0.2-bad (1.0.2k-dev)" [~183 ciphers]
 on vagrant:./bin/openssl.Linux.x86_64
 (built: "Sep  1 14:03:44 2022", platform: "linux-x86_64")


Testing all IPv4 addresses (port 443): 188.114.98.224 188.114.99.224
-----------------------------------------------------
 Start 2023-02-22 13:36:32        -->> 188.114.98.224:443 (netology.ru) <<--

 Further IP addresses:   188.114.99.224 2a06:98c1:3122:e000::
                         2a06:98c1:3123:e000:: 
 rDNS (188.114.98.224):  --
 Service detected:       HTTP



 Testing all 183 locally available ciphers against the server, ordered by encryption strength 


Hexcode  Cipher Suite Name (OpenSSL)       KeyExch.   Encryption  Bits     Cipher Suite Name (IANA/RFC)
-----------------------------------------------------------------------------------------------------------------------------
 xcc14   ECDHE-ECDSA-CHACHA20-POLY1305-OLD ECDH 256   ChaCha20    256      TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256_OLD  
 xcc13   ECDHE-RSA-CHACHA20-POLY1305-OLD   ECDH 256   ChaCha20    256      TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256_OLD    

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

 Done 2023-02-22 13:36:47 [  17s] -->> 188.114.99.224:443 (netology.ru) <<--

-----------------------------------------------------
Done testing now all IP addresses (on port 443): 188.114.98.224 188.114.99.224
```

5. Установите на Ubuntu ssh сервер, сгенерируйте новый приватный ключ. Скопируйте свой публичный ключ на другой сервер. Подключитесь к серверу по SSH-ключу.

```bash
wolin@wolinubuntu:~$ apt install openssh-server

wolin@wolinubuntu:~$ sudo systemctl start sshd.service

wolin@wolinubuntu:~$ sudo systemctl enable sshd.service

wolin@wolinubuntu:~$ ssh-keygen

wolin@wolinubuntu:~$ ssh-copy-id alex@192.168.1.140
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
alex@192.168.1.140's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'alex@192.168.1.140'"
and check to make sure that only the key(s) you wanted were added.

wolin@wolinubuntu:~$ ssh alex@192.168.1.140
Welcome to Ubuntu 20.04.5 LTS (GNU/Linux 5.15.0-46-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

0 updates can be applied immediately.


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
New release '22.04.1 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

Your Hardware Enablement Stack (HWE) is supported until April 2025.
Last login: Wed Feb 22 16:51:44 2023 from 192.168.1.118


```

6. Переименуйте файлы ключей из задания 5. Настройте файл конфигурации SSH клиента, так чтобы вход на удаленный сервер осуществлялся по имени сервера.

```bash
wolin@wolinubuntu:~/.ssh$ mv id_rsa.pub rsa-netology.pub
wolin@wolinubuntu:~/.ssh$ mv id_rsa rsa-netology
wolin@wolinubuntu:~/.ssh$ nano $HOME/.ssh/config                       
Host netology
  User alex
  HostName 192.168.1.140
  IdentityFile ~/.ssh/rsa-netology

wolin@wolinubuntu:~/.ssh$ ssh netology
Welcome to Ubuntu 20.04.5 LTS (GNU/Linux 5.15.0-46-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

0 updates can be applied immediately.


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
New release '22.04.1 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

Your Hardware Enablement Stack (HWE) is supported until April 2025.
Last login: Wed Feb 22 16:57:58 2023 from 192.168.1.118
```

7. Соберите дамп трафика утилитой tcpdump в формате pcap, 100 пакетов. Откройте файл pcap в Wireshark.

```bash
alex@Alex-VirtualBox:~$ sudo tcpdump -c 100 -w netology.pcap                      #собрали дамп на сервере
wolin@wolinubuntu:~/netology$ scp netology:~/netology.pcap .\netology.pcap        #скопировали дамп на клиента
```

![wireshark](https://i.ibb.co/Sc6j0XG/Screenshot-from-2023-02-22-20-58-05.png)


*В качестве решения приложите: скриншоты, выполняемые команды, комментарии (по необходимости).*

 ---
 
## Задание для самостоятельной отработки* (необязательно к выполнению)

8. Просканируйте хост scanme.nmap.org. Какие сервисы запущены?

9. Установите и настройте фаервол ufw на web-сервер из задания 3. Откройте доступ снаружи только к портам 22,80,443

----

### Правила приема домашнего задания

В личном кабинете отправлена ссылка на .md файл в вашем репозитории.

-----

### Критерии оценки

Зачет - выполнены все задания, ответы даны в развернутой форме, приложены соответствующие скриншоты и файлы проекта, в выполненных заданиях нет противоречий и нарушения логики.

На доработку - задание выполнено частично или не выполнено, в логике выполнения заданий есть противоречия, существенные недостатки. 
 
Обязательными к выполнению являются задачи без указания звездочки. Их выполнение необходимо для получения зачета и диплома о профессиональной переподготовке.
Задачи со звездочкой (*) являются дополнительными задачами и/или задачами повышенной сложности. Они не являются обязательными к выполнению, но помогут вам глубже понять тему.
