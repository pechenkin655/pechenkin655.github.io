---
date: 2017-04-09T10:58:08-04:00
description: "???"
featured_image: "/hero/mysql.png"
images: ["/hero/mysql.png"]
tags: ["linux", "database administration", "mysql"]
categories: "it"
title: "MySQL: работаем с дампами"
---

## Базовые операции

```bash
mysqldump --user username --password database
```

Команда отработает, вывалив содержимое базы данных прямо в консоль. Упс.  
Чтобы получить файл дампа нужно перенаправить `stdout` команды в файл:

```bash
mysqldump --user username --password database > database.sql
```

И, файл создался! Одна проблема, база была не маленькой и теперь в нашем распоряжении файл размером в несколько гигабайт. Было бы неплохо его сжать. Желательно на ходу, чтобы не занимать лишнего места:

```bash
mysqldump --user username --password database | gzip -9 > database.sql
```

Это сделает дамп меньше. Гораздо меньше.
Ради интереса можно убрать перенаправление и посмотреть что там на выходе из `gzip`. Там будут попытки терминала интерпретировать поток данных, который отдает `gzip`. Это свойство еще пригодиться немного позже.

Чтобы залить дамп делаем:

TODO: TEST!!!!
```bash
# sql
mysql --user username --password database < file.sql

# sql.gz при помощи gunzip
gunzip < database.sql.gz | mysql --user username --password database

# sql.gz при помощи zcat
zcat database.sql.gz | mysql --user username --password database
```

## Удаленный сервер

Если сервер находиться на **удаленной** машине, к консоли которой можно получить доступ то для снятия дампа сложных "заклинаний" не потребуется. Просто подключаемся к нему по SSH и даем команду:

```bash
mysqldump --user username --password database | gzip -9 > database.sql.gz
```

Да, такую же как для локального дампа. Если сервер не на этой машине, но доступен с нее, можно добавить `--hostname db.server.address`, `--port=3306`. И `--compress`, чтобы поток от сервера базы к нам также был сжат. После всех манипуляций файл дампа ляжет в текущей директории. Забрать его можно будет используя `sftp`, `scp` или `rsync`.  
А можно ли так чтобы дамп сразу себе? Рад что вы спросили:

```bash
ssh $SERVER_USERNAME@SERVER_ADDRESS "mysqldump --user username --password database | gzip -9" > database.sql.gz
```

`mtsqldump` будет выполнен на удаленном сервере, затем его вывод будет сжат и отправлен в `stdout` который уже на нашей стороне будет перенаправлен в файл. Трюк с `--hostname db.server.address`, `--port=3306` и `--compress` можно применить и здесь.

Чтобы восстановить дамп в такой манере прийдеться исхитриться.

```bash
zcat database.sql.gz | $SERVER_USERNAME@SERVER_ADDRESS "cat | mysql --user username --password database"
```

TODO: TEST!!! ^^^
> **WARNING**  
> Заклинание из раздела теоретической магии! Если вы видите это предупреждение, я забыл его протестить.

## Удаленный сервер, недоступный извне

Что делать, если нужные утилиты есть только у нас? Например, база находиться в части сети, изолированной от интернета, но есть хост, доступный извне, которому разрешен доступ к базе. Нам поможет тоннель! Сам по себе `mysqldump` не умеет устанавливать SSH тоннели для подключения. Об этом прийдеться позаботиться отдельно.  
Чтобы поднять SSH тоннель необходимо выполнить такую команду:

```bash
# команда
ssh -L $LOCAL_PORT:$DB_SERVER_ADDESS:$REMOTE_PORT -N $JUMP_SERVER_USERNAME@JUMP_SERVER_ADDRESS
# пример
ssh -L 3307:db.infra.lan:3306 -N username@host.infra.lan
```

После подъема тоннеля все обращения к порту 3307 на **локальной** машине будут переданы через него на **удаленный** сервер:

```bash
# снять дамп
mysqldump --port 3307 --host 127.0.0.1 --user username --password database | gzip -9 > database.sql.gz
# восстановить дамп
zcat database.sql.gz | mysql --user username --password database
```

> **NOTE**  
> Для работы через тоннель необходимо использовать параметры `--port` и `--host` для того чтобы указать, что нужно подключаться используя определенный порт, а не локальный сокет.

