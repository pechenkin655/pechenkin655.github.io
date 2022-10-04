---
date: 2022-10-02
description: "Несколько заклинаний для работы с дампами баз данных"
featured_image: "/hero/mysql.png"
images: ["/hero/mysql.png"]
tags: ["linux", "database administration", "mysql"]
categories: "it"
title: "MySQL: работаем с дампами"
comment: false
---

## Базовые операции

Начнем с простого. Снятие дампа:

```bash
mysqldump --user username --password database
```

Команда отработает, вывалив содержимое базы данных прямо в консоль. Упс.  
Чтобы получить файл дампа поток из `stdout` нужно перенаправить собственно в файл:

```bash
mysqldump --user username --password database > database.sql
```

И, файл создался! Одна проблема, база была не маленькой и теперь в нашем распоряжении файл размером в несколько гигабайт. В следующий раз было бы неплохо его сжать. Желательно на ходу, чтобы не занимать лишнего места. Как то так:

```bash
mysqldump --user username --password database | gzip -9 > database.sql
```

Это сделает дамп меньше. Гораздо меньше.

Чтобы залить дамп обратно делаем все то же, только в обратном порядке:

```bash
# sql
mysql --user username --password database < file.sql
cat file.sql | mysql --user username --password database

# sql.gz при помощи gunzip
gunzip < database.sql.gz | mysql --user username --password database
cat database.sql.gz | gunzip | mysql --user username --password database

# sql.gz при помощи zcat
zcat database.sql.gz | mysql --user username --password database
```

Даже по нескольку вариантов. Я предпочитаю вариант с пайпами, потому как он просто читается с лева на право.

## Удаленный сервер

Если сервер находиться на **удаленной** машине, к консоли которой можно получить доступ то для снятия дампа сложных "заклинаний" не потребуется. Просто подключаемся к нему по SSH и даем команду:

```bash
mysqldump --user username --password database | gzip -9 > database.sql.gz
```

Да, такую же как для локального дампа. Если сервер не на этой машине, но доступен с нее, можно добавить `--hostname db.server.address`, `--port=3306`. И `--compress`, чтобы данные, идущие от сервера базы к нам также был сжаты. После всех манипуляций файл дампа ляжет в текущей директории. Забрать его можно будет используя `sftp`, `scp` или `rsync`.  

А можно ли так, чтобы дамп сразу себе? Рад что вы спросили!

```bash
ssh $SERVER_USERNAME@SERVER_ADDRESS "mysqldump --user username --password database | gzip -9" > database.sql.gz
```

`mysqldump` будет выполнен на удаленном сервере, затем его вывод будет сжат и отправлен в `stdout` который уже на нашей стороне будет перенаправлен в файл. Трюк с `--hostname db.server.address`, `--port=3306` и `--compress` можно применить и здесь.

Чтобы восстановить дамп в данной манере прийдеться исхитриться и затолкать поток обратно:

```bash
# если на хосте нет zcat - медленнее
zcat database.sql.gz | $SERVER_USERNAME@SERVER_ADDRESS "cat | mysql --user username --password database"

# если zcat есть - быстрее
cat database.sql.gz | $SERVER_USERNAME@SERVER_ADDRESS "zcat | mysql --user username --password database"
```

## Удаленный сервер, недоступный извне

Что делать, если нужные утилиты есть только у нас? Или база находиться в части сети, изолированной от интернета, но есть хост, доступный извне, которому разрешен доступ к базе? Нам поможет тоннель!  
Сам по себе `mysqldump` не умеет создавать SSH тоннели. Об этом нам прийдеться позаботиться отдельно.

Чтобы создать SSH тоннель необходимо выполнить такую команду:

```bash
# команда
ssh -L $LOCAL_PORT:$DB_SERVER_ADDESS:$REMOTE_PORT -N $JUMP_SERVER_USERNAME@JUMP_SERVER_ADDRESS
# пример
ssh -L 3307:db.server.address:3306 -N username@host.server.address
```

После создания тоннеля все обращения к порту 3307 на **локальной** машине будут переданы через него на **удаленный** сервер:

```bash
# снять дамп
mysqldump --port 3307 --host 127.0.0.1 --user username --password database | gzip -9 > database.sql.gz
# восстановить дамп
zcat database.sql.gz | mysql --port 3307 --host 127.0.0.1 --user username --password database
```

Для работы через тоннель необходимо использовать параметры `--port` и `--host` для того чтобы указать, что нужно подключаться используя определенный порт, а не локальный сокет.
