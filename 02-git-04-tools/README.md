# Домашнее задание к занятию «Инструменты Git»

### Цель задания

В результате выполнения этого задания вы научитесь работать с утилитами Git и  потренируетесь решать типовые задачи, возникающие при работе в команде. 

### Инструкция к заданию

1. Склонируйте [репозиторий](https://github.com/hashicorp/terraform) с исходным кодом Terraform.
2. Создайте файл для ответов на задания в своём репозитории, после выполнения, прикрепите ссылку на .md-файл с ответами в личном кабинете.
3. Любые вопросы по решению задач задавайте в чате учебной группы.

------

## Задание

В клонированном репозитории:

1. Найдите полный хеш и комментарий коммита, хеш которого начинается на `aefea`.

```bash
git show aefea  
aefead2207ef7e2aa5dc81a34aedf0cad4c32545
```

2. Какому тегу соответствует коммит `85024d3`?

```bash
git show 85024d3
tag: v0.12.23
```

3. Сколько родителей у коммита `b8d720`? Напишите их хеши.

```bash
git show  --pretty=format:' %P' b8d720  
56cd7859e05c36c06b56d013b55a252d0bb7e158 9ea88f22fc6269854151c571162c5bcf958bee2b
```

4. Перечислите хеши и комментарии всех коммитов которые были сделаны между тегами  v0.12.23 и v0.12.24.

```bash
git log v0.12.23..v0.12.24 --oneline  
33ff1c03bb (tag: v0.12.24) v0.12.24
b14b74c493 [Website] vmc provider links
3f235065b9 Update CHANGELOG.md
6ae64e247b registry: Fix panic when server is unreachable
5c619ca1ba website: Remove links to the getting started guide's old location
06275647e2 Update CHANGELOG.md
d5f9411f51 command: Fix bug when using terraform login on Windows
4b6d06cc5d Update CHANGELOG.md
dd01a35078 Update CHANGELOG.md
225466bc3e Cleanup after v0.12.23 release
```

5. Найдите коммит в котором была создана функция `func providerSource`, ее определение в коде выглядит 
так `func providerSource(...)` (вместо троеточия перечислены аргументы).

```bash
git log -S'func providerSource(' --oneline  
8c928e8358 main: Consult local directories as potential mirrors of providers
```

6. Найдите все коммиты в которых была изменена функция `globalPluginDirs`.

```bash
git grep 'func globalPluginDirs' #нашли файлы содержащие функцию  
git log -L:'globalPluginDirs':plugins.go --oneline 
78b1220558 Remove config.go and update things using its aliases
52dbf94834 keep .terraform.d/plugins for discovery
41ab0aef7a Add missing OS_ARCH dir to global plugin paths
66ebff90cd move some more plugin search path logic to command
8364383c35 Push plugin discovery down into command package
```

7. Кто автор функции `synchronizedWriters`? 

```bash
git log -S'func synchronizedWriters' --first-parent --pretty=format:'%h - %an %ae'

#получаем вывод с хешами двух авторов
dcf0dba6f4 - James Bardin j.bardin@gmail.com
5ac311e2a9 - Martin Atkins mart@degeneration.co.uk

#проверяем изменения и устанавливаем, что автор функции Martin Atkins
git show dcf0dba6f4
git show 5ac311e2a9
```

*В качестве решения ответьте на вопросы и опишите каким образом эти ответы были получены*

---

### Правила приема домашнего задания

В личном кабинете отправлена ссылка на .md файл в вашем репозитории.

### Критерии оценки

Зачет - выполнены все задания, ответы даны в развернутой форме, приложены соответствующие скриншоты и файлы проекта, в выполненных заданиях нет противоречий и нарушения логики.

На доработку - задание выполнено частично или не выполнено, в логике выполнения заданий есть противоречия, существенные недостатки. 
