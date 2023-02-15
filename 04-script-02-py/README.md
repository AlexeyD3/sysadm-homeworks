# Домашнее задание к занятию "4.2. Использование Python для решения типовых DevOps задач"

### Цель задания

В результате выполнения этого задания вы:

1. Познакомитесь с синтаксисом Python.
2. Узнаете, для каких типов задач его можно использовать.
3. Воспользуетесь несколькими модулями для работы с ОС.


### Инструкция к заданию

1. Установите Python 3 любой версии.
2. Скопируйте в свой .md-файл содержимое этого файла; исходники можно посмотреть [здесь](https://raw.githubusercontent.com/netology-code/sysadm-homeworks/devsys10/04-script-01-bash/README.md).
3. Заполните недостающие части документа решением задач (заменяйте `???`, остальное в шаблоне не меняйте, чтобы не сломать форматирование текста, подсветку синтаксиса). Вместо логов можно вставить скриншоты по желанию.
4. Для проверки домашнего задания преподавателем в личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
5. Любые вопросы по выполнению заданий спрашивайте в чате учебной группы и/или в разделе “Вопросы по заданию” в личном кабинете.

------

## Задание 1

Есть скрипт:
```python
#!/usr/bin/env python3
a = 1
b = '2'
c = a + b
```

### Вопросы:

| Вопрос  | Ответ |
| ------------- | ------------- |
| Какое значение будет присвоено переменной `c`?  | `TypeError: unsupported operand type(s) for +: 'int' and 'str'` |
| Как получить для переменной `c` значение 12?  | присвоить `a = '1'` используя кавычки  |
| Как получить для переменной `c` значение 3?  | убрать кавычки у `b = 2`  |

------

## Задание 2

Мы устроились на работу в компанию, где раньше уже был DevOps Engineer. Он написал скрипт, позволяющий узнать, какие файлы модифицированы в репозитории, относительно локальных изменений. Этим скриптом недовольно начальство, потому что в его выводе есть не все изменённые файлы, а также непонятен полный путь к директории, где они находятся. 

Как можно доработать скрипт ниже, чтобы он исполнял требования вашего руководителя?

```python
#!/usr/bin/env python3

import os

bash_command = ["cd ~/netology/sysadm-homeworks", "git status"]
result_os = os.popen(' && '.join(bash_command)).read()
is_change = False
for result in result_os.split('\n'):
    if result.find('modified') != -1:
        prepare_result = result.replace('\tmodified:   ', '')
        print(prepare_result)
        break
```

### Ваш скрипт:
```python
#!/usr/bin/env python3

import os

cdpath = os.getcwd()
bash_command = ["cd "+cdpath, "git status"]
result_os = os.popen(' && '.join(bash_command)).read()

for result in result_os.split('\n'):
    if result.find('modified') != -1:
        prepare_result = result.replace('\tmodified:   ', '/')
        print(cdpath+prepare_result)

```

### Вывод скрипта при запуске при тестировании:
![вывод](https://i.ibb.co/VpF3q9j/Screenshot-from-2022-12-14-20-36-42.png)

------

## Задание 3

Доработать скрипт выше так, чтобы он не только мог проверять локальный репозиторий в текущей директории, но и умел воспринимать путь к репозиторию, который мы передаём как входной параметр. Мы точно знаем, что начальство коварное и будет проверять работу этого скрипта в директориях, которые не являются локальными репозиториями.

### Ваш скрипт:
```python
#!/usr/bin/env python3
import os
import sys
cdpath = os.getcwd()
if len(sys.argv) >= 2:
    cdpath=sys.argv[1]
bash_command = ["exec 2>&1", "cd "+cdpath, "git status"]
result_os = os.popen(' && '.join(bash_command)).read()
for result in result_os.split('\n'):

    if result.find('modified') != -1:
        prepare_result = result.replace('\tmodified:   ', '/')
        print("\033[34m{}".format(cdpath+prepare_result))
        
    if result.find("can't cd to") != -1:
        print("\033[31m{}".format('error: wrong argument, check path'))
        
    if result.find('fatal') != -1:
        print("\033[31m{}".format('error: .git repository is not found in this directory'))
```

### Вывод скрипта при запуске при тестировании:
![вывод](https://i.ibb.co/160CfCj/Screenshot-from-2022-12-14-23-07-47.png)

------

## Задание 4

Наша команда разрабатывает несколько веб-сервисов, доступных по http. Мы точно знаем, что на их стенде нет никакой балансировки, кластеризации, за DNS прячется конкретный IP сервера, где установлен сервис. 

Проблема в том, что отдел, занимающийся нашей инфраструктурой очень часто меняет нам сервера, поэтому IP меняются примерно раз в неделю, при этом сервисы сохраняют за собой DNS имена. Это бы совсем никого не беспокоило, если бы несколько раз сервера не уезжали в такой сегмент сети нашей компании, который недоступен для разработчиков. 

Мы хотим написать скрипт, который: 
- опрашивает веб-сервисы, 
- получает их IP, 
- выводит информацию в стандартный вывод в виде: <URL сервиса> - <его IP>. 

Также, должна быть реализована возможность проверки текущего IP сервиса c его IP из предыдущей проверки. Если проверка будет провалена - оповестить об этом в стандартный вывод сообщением: [ERROR] <URL сервиса> IP mismatch: <старый IP> <Новый IP>. Будем считать, что наша разработка реализовала сервисы: `drive.google.com`, `mail.google.com`, `google.com`.

### Ваш скрипт:
```python
#!/usr/bin/env python3
import time
import socket

#host_dns_list = ['yandex.ru', 'mail.ru', 'mail.yandex.ru'] брал yandex так как гугл не менял IP
host_dns_list = ['drive.google.com', 'mail.google.com', 'google.com']
dict_ip = {}
dict_old_ip = {}
while True: # бесконечный цикл проверки
    for host_dns in host_dns_list:
        dict_ip.update({host_dns: socket.gethostbyname(host_dns)})
        if len(dict_old_ip.get(host_dns, 'none')) > 4:
            if dict_old_ip.get(host_dns) != dict_ip.get(host_dns):
                print("[ERROR] <"+host_dns+"> IP mismatch: <"+dict_old_ip.get(host_dns)+"> <"+dict_ip.get(host_dns)+">")
        dict_old_ip = dict_ip.copy()
        print("<"+host_dns+"> - <"+dict_ip.get(host_dns)+">")
    time.sleep(2) # задержка между циклами проверки
```

### Вывод скрипта при запуске при тестировании:
![вывод](https://i.ibb.co/K9WGyMc/Screenshot-from-2023-01-23-14-48-35.png)

------

## Дополнительное задание (со звездочкой*) - необязательно к выполнению

Так получилось, что мы очень часто вносим правки в конфигурацию своей системы прямо на сервере. Но так как вся наша команда разработки держит файлы конфигурации в github и пользуется gitflow, то нам приходится каждый раз: 
* переносить архив с нашими изменениями с сервера на наш локальный компьютер, 
* формировать новую ветку, 
* коммитить в неё изменения, 
* создавать pull request (PR) 
* и только после выполнения Merge мы наконец можем официально подтвердить, что новая конфигурация применена. 

Мы хотим максимально автоматизировать всю цепочку действий. 
* Для этого нам нужно написать скрипт, который будет в директории с локальным репозиторием обращаться по API к github, создавать PR для вливания текущей выбранной ветки в master с сообщением, которое мы вписываем в первый параметр при обращении к py-файлу (сообщение не может быть пустым).
* При желании, можно добавить к указанному функционалу создание новой ветки, commit и push в неё изменений конфигурации. 
* С директорией локального репозитория можно делать всё, что угодно. 
* Также, принимаем во внимание, что Merge Conflict у нас отсутствуют и их точно не будет при push, как в свою ветку, так и при слиянии в master. 

Важно получить конечный результат с созданным PR, в котором применяются наши изменения. 

### Ваш скрипт:
```python
#!/usr/bin/env python3

from datetime import datetime
import json
import os
import requests
import re
import subprocess
import sys
import time

def git_exec(command):
    print(command)
    if command.find("git commit") >= 0:
        command_splitted = ["git", "commit", "-m"]
        command_splitted.append(command.split('git commit -m ')[1])
    else:
        command_splitted = command.split()
    command_e = subprocess.Popen(command_splitted, stdout=subprocess.PIPE,
                                 stderr=subprocess.STDOUT, cwd=resolved_path, text=True)
    e = command_e.communicate()[0].split('\n')[0]
    if e.find('fatal:') >= 0:
        print(
            f'В папке {resolved_path} не найден git репозиторий.')
        exit()
    return e


token = ""
if token == "":
    print(f"""
\t!!! переменная "token" пуста, задайте парметр !!!

\thttps://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token
""")
    exit()

try:
    if sys.argv[1]:
        message = " ".join(sys.argv[1:])
except IndexError:
    print(
        f"Введите сообщение для реквеста\n")
    exit()

path = "./"
resolved_path = os.path.normpath(os.path.abspath(
    os.path.expanduser(os.path.expandvars(path))))

try:
    origin_push_url = git_exec("git remote get-url --push origin")
except FileNotFoundError:
    print(
        f'Не удалось найти папку {path}'
    )
    exit()

if origin_push_url.find('fatal:') >= 0:
    print(
        f'В папке {resolved_path} не найден git репозиторий.')
    exit()

gh_acc, gh_repo = re.split('git@github.com:|/|.git', origin_push_url)[1:3]

repo_url = f'https://api.github.com/repos/{gh_acc}/{gh_repo}'

headers = {"Authorization": f"token {token}",
           "Accept": "application/vnd.github.v3+json"}

git_status = subprocess.Popen(["git", "status", "--porcelain"], stdout=subprocess.PIPE,
                              stderr=subprocess.STDOUT, cwd=resolved_path, text=True).communicate()[0].split('\n')

cur_time = datetime.now()
branch_name = f"""{datetime.strftime(cur_time, "%Y-%m-%d_%H%M%S")}-config-local-edit"""
date_commit_text = datetime.strftime(cur_time, "%Y-%m-%d %H:%M:%S")

# exit()
if len(git_status) > 1 or git_status[0] != '':
    git_exec(f"git checkout -b {branch_name}")
    git_exec(f"git add .")
    git_exec(f"git commit -m 'config local edit at {date_commit_text}'")
    git_exec(f"git push --set-upstream origin {branch_name}")
    r = requests.get(f"{repo_url}/branches/{branch_name}", headers=headers)
    git_exec(f"git checkout main")
    while r.status_code >= 300:
        r = requests.get(f"{repo_url}/branches/{branch_name}", headers=headers)
        print(f'Ещё не создан репозиторий. GitHub: {r}, {r.content}')
        time.sleep(1)
    payload = {"title": branch_name, "body": message,
               "head": branch_name, "base": "main"}
    r = requests.post(f"{repo_url}/pulls", headers=headers,
                      data=json.dumps(payload))
    if r.status_code >= 300:
        print(
            f"Ошибка! Ответ GitHub API на создание Pull Request: {r}\n\n{r.content}\n")
        exit()
    else:
        print(f'Ответ GitHub на создание Pull Request: {r}')
        git_exec(f"git branch -D {branch_name}")
    pull_req_merge_url = f"{r.json()['url']}/merge"
    payload = {"commit_title": f"MERGED {branch_name} into main"}
    r = requests.put(pull_req_merge_url, headers=headers,
                     data=json.dumps(payload))
    if r.status_code >= 300:
        print(
            f"Ошибка! Ответ GitHub API на слияние Pull Request: {r}\n\n{r.content}\n")
        exit()
    else:
        print(f'Ответ GitHub на слияние Pull Request: {r}')
        git_exec(f"git push origin -d {branch_name}")
        print(f'\nЗагрузка изменений main:\n')
        os.popen(f"cd {resolved_path} && git pull").read()
        print(f'\n')
```

### Вывод скрипта при запуске при тестировании:
```
???
```

----

### Правила приема домашнего задания

В личном кабинете отправлена ссылка на .md файл в вашем репозитории.

-----

### Критерии оценки

Зачет - выполнены все задания, ответы даны в развернутой форме, приложены соответствующие скриншоты и файлы проекта, в выполненных заданиях нет противоречий и нарушения логики.

На доработку - задание выполнено частично или не выполнено, в логике выполнения заданий есть противоречия, существенные недостатки. 
 
Обязательными к выполнению являются задачи без указания звездочки. Их выполнение необходимо для получения зачета и диплома о профессиональной переподготовке.
Задачи со звездочкой (*) являются дополнительными задачами и/или задачами повышенной сложности. Они не являются обязательными к выполнению, но помогут вам глубже понять тему.

