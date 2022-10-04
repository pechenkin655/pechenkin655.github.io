---
date: 2017-04-09T10:58:08-04:00
description: "Cоздать пользователя в системе и разрешить ему использовать sudo"
featured_image: "/hero/system.png"
images: ["/hero/system.png"]
tags: ["linux", "system administration"]
categories: "it"
title: "Работать под root'ом - плохо"
comment: false
---

## Работать под root'ом - плохо!

TODO: описать почему плохо
<!-- 
Сейчас я работаю под пользователем root. Так быть не должно. Случайно можно снести пол системы и не заметить этого. Чтобы обезопасить себя от ошибок (и сиcтему от моих необдуманных действий) сделаю себе отдельного пользователя, который может использовать `sudo`, а пользователя `root` закрою для входа. Так будет безопаснее. -->

## Может пригодиться

[Генерируем стойкий пароль]( {{< relref "generate-strong-password" >}} )

## Создаем пользователя

```bash
adduser ubuntu
usermod -aG sudo ubuntu
passwd ubuntu
```

Чтобы разрешить испольщовать судо в убунте нужно д=обавить пользоавтел я вгруппу судо

Так же, добавлю к пользователю свой ключ, чтобы заходить, не используя пароль:

```bash
su ubuntu
cd ~/
mkdir -p ~/.ssh/
touch ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
# chown ubuntu:ubuntu /home/ubuntu/.ssh/authorized_keys
vim ~/.ssh/authorized_keys
# INSERT, SHIFT+INSERT, ESC, :, q, ENTER
echo "your-public-key-here" | tee /home/ubuntu/.ssh/authorized_keys
```

Вот и все. Пора проверить, есть ли возможность подключиться ново-созданным пользователем.
Обычно я открываю еще одну консоль, пытаюсь подключиться и выполнить что либо от используя `sudo`:

```bash
ssh ubuntu@server.address
sudo echo "111"
```

Если все прошло хорошо и в консоли отобразились заветные единички то от сессии пользователя root можно отключаться.

Теперь осталось только запретить вход для пользователя root. Для этого нужно создать файл в директории _/etc/ssh/sshd_config.d/_ с любым именем и расширением "conf". Пусть будет _1-disable-root-ssh.conf_ раз уж так. Содержимое будет следующим:

```conf
PermitRootLogin no
```

Далее - проверка конфига и перезагрузка sshd:

```bash
sshd -t
ssytemctl reload sshd
```

Все, при следующей попытке войти как пользователь root, система ответит "ты не пройдешь". Ну то есть "Access denied".

> **NOTE**  
> Если `sshd -t` не выдал в ответ ничего - значит все в порядке.
