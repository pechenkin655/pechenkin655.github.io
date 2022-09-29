---
date: 2017-04-09T10:58:08-04:00
description: "Wordpress host"
featured_image: "/images/Pope-Edouard-de-Beaumont-1844.jpg"
images: ["/images/Pope-Edouard-de-Beaumont-1844.jpg"]
tags: ["scene"]
categories: "Story"
title: "Wordpress host"
---

## План

План достаточно прост:
- Каждый проект получает своего пользователя в системе
- Проект располагаеться в домашней директории пользователя
- пользователь имеет доступ к этой директории через фтп\сфтп
- опционалоно - через ссш
- вордпресс в режиме прямого доступа (wp_direct)
- обслуживает все nginx php fpm последних версий, что поддерживается вуордпрессом
TODO: можно ли организовать несоколько пулов php fpm чтобы иметь несколько сокетов
TODO: отметить в конце статьичто все проекты на одном фпм а это потенциально опастно и сложно будет развести нагрузку есдли один из проектов начнет жрать.
TODO: следующая статья - автоматизация создания новых польхователей при помощи ансибла
TODO: следующая статься - то же но каждый это отдельный стек докера.


TODO: пермишенсы по умолчанию
Subsystem sftp internal-sftp -m 0755 -u 022

TODO: скрипт который пройдеться по страничке и заменить названия в броузере чтобы можно было на перед задать названия и логины


условности  
имя польщовалемя, название проекта совпадают
имя баз данных совпадают с именем проета `wp-test`  
пароль везде `password`  
базовый домен будет `wp.local`  
домен сайта будет `wp-test.wp.local`  
имя админского польщователя `admin`

меняйте как хотите

^^^ Заметки ^^^
---

## Проект. Много их

Разместить один сайт на Wordpress. Казалось бы не сложно. Взять самую свежую систему, поставить на нее php, nginx, mysql, закинуть файлы Wordpress в нужное место и все, работает! И да и нет. Особенно, когда оказывается, что сайтов должно быть несколько.


## Система

Как основу я выбрал самый свежий на данный TLS релиз операционной системы Ubuntu - **Ubuntu 22.04 LTS**. Поверх нее будет привычный (для меня) **LEMP**. Начнем.

Пакеты пакеты, накатил обновления, обычные мелочи:

```bash
apt-get update
apt-get upgrade
```

Для **Ubuntu 22.04 LTS** **PHP 8** устанавливается и без лишних плясок с бубном. Никаких дополнительных репозиториев добавлять не нужно. Версия **Nginx** и **MariaDB** не так интересны. 

> **NOTE**  
> Если все же нужна другая php 
TODO: описать добавление ondrej репозиториев

Далее, необходимо установить нужные пакеты. Все что нужно одной строчкой:

```bash
apt-get install mysql-server mysql-client nginx php-cli php-fpm php-gd php-mysql php-xml php-zip php-imagick php-curl php-mbstring sendmail
```


## Работать под рутом - плохо!
TODO: отдельная статья о том как настроить себе пользователя если есть только рут?

Сейчас я работаю под пользователем root. Так быть не должно. Случайно можно снести пол системы и не заметить этого. Чтобы обезопасить себя от ошибок (и ситему от моих необдуманных действий) сделаю себе отдельного пользователя, который может использовать `sudo`, а пользователя `root` закрою для входа. Так будет безопаснее.

```bash
adduser ubuntu
usermod -aG sudo ubuntu
passwd ubuntu
```

> **NOTE**  
В Ubuntu, чтобы разрешитьпользователю использовать sudo нужно добавить его в группу `sudo`.

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


## Затягиавем гайки
TODO: отдельная статья о том как настроить себе пользователя если есть только рут?

Раз уж я оказался здесь - подкрутим немного настройки sshd. Это будет еще один файл в директории _/etc/ssh/sshd_config.d/_. Его так же можно называть как угодно, но я назову его _2-hardening.conf_

TODO: расписать что кадда строска желает

```conf
LoginGraceTime 20
PasswordAuthentication no
PermitEmptyPasswords no
ChallengeResponseAuthentication no
KerberosAuthentication no
GSSAPIAuthentication no
X11Forwarding no
# MaxAuthTries 3 # боты если прочухают какой юзернейм стучать могут не дать заййти с эь=той коммандой, лучше испольщовать в паре с fail2ban или еще чем

# PermitUserEnvironment no
# AllowAgentForwarding no
# AllowTcpForwarding no
# PermitTunnel no
# DebianBanner no
```

TODO: модно блокировать даже ссщш пользователя в своей хомдире, может пригодиться, это всеравно отдельной статьей - описать

И снова - проверка конфига и перезагрузка sshd:

```bash
sshd -t
systemctl reload sshd
```


## Отступление
TODO: отдельная статья о том как настроить себе пользователя если есть только рут?

Зачем вообще создавать несколько файлов, ведь можно просто редиктировать _/etc/ssh/sshd_config_ и не заморачиваться, спросите вы. Да можно. Но это моэет вызвать проблемы при обновлении системы с Манжаро.Арчь не встречал с убунту посточнно. Оригинальный файл изменен, что же с ним делать? Вспомнили? Обычно такими вопросами просто заваливает убунту когда делаеш релиз апгрейд. использование этого метода изваяляет от этого. к тому же всегда легче найти свои правки т.к. они в отдельном файле.


## Sendmail

На самом деле он не нужен. В документации к Wordpress, в описании функции `wp_mail()` описано, как сделать так, чтобы использовался **PHPMailer** вместо встроенной в PHP функции `mail()`. Разработчики почему-то не используют эту возможность, постоянно жалуясь, что **sendmail** не установлен и письма не ходят. Ну да ладно.

**Sendmail** уже установлен. Остальсь только сконфигурировать его командой:

```bash
sendmailconfig
```

Отвечаем `Y` на все вопросы. Должно сработать. По крайней мере письма пойдут. Попытаются.

Кстати о попытке. Можно для проверки отправить письмо и посмотреть работает ли отсылка на самом деле. Для этого я набросал простенький скриптик на PHP.

```bash
<?php 
ini_set( 'display_errors', 1 );
error_reporting( E_ALL );

$from = "emailtest@YOURDOMAIN";
$to = "YOUREMAILADDRESS";

$subject = "PHP Mail Test script";
$message = "This is a test to check the PHP Mail functionality";
$headers = "From:" . $from;

mail($to,$subject,$message, $headers);
echo "Test email sent";
?>
```

В переменную `$from` можно писать что угодно, в `$to` я вставил почтовый адрес, полученный ма сервисе Mailinator. Не обязательно использовать его, просто выбрал тот, который первым вспомнился. Для таких тестов сервисы, предоставляющие одноразовые адреса подходят отлично - нет лишних проверок и спам-блока. Если письмо дошло - ты его увидишь.  Если не дошло - в _/var/spool/mail/_ можно проверить нет ли там вообщений о bоunce. Там же можно почитать и причину.

> **NOTE**  
> Сейчас я не заморачиваюсь с правальной настройкой отсылки почты. Скорее всего для этого будут использоваться сторонние сервисы типа **Sendgrid**. Для них будет своя настройка.

если имейл не пошел пошел долго или с ошибками искать в syslog "unqualified hostname unknown; sleeping for retry unqualified hostname" и если есть 
хостнейм должен совпадать с доменом или в /etc/hosts и добавляем 127.0.0.1 домен.смотрящий.на.сайт


## Группа и пользователь

Уже извечтно что проектов будет не один и даже не два, так что учинываем это сразу.  
Для изоляции каждый сайт будет размещен под своим пользователем, каждому разработчику будет предостевлен доступ только к файлам своего проекта. Так же будет доступ по `sFTP`.Доступа по `ssh` не будет ни у кого (кроме меня, му-ха-ха).  
Для начала нам понадобиться группа, в которую мы будем помущать всех пользователей, которым будет разрешен доступ по `sFTP`.

```bash
groupadd sftp-users
```

Группу можно называть как угодно, главное, запомнить для чего она создавалась. И еще одно- эта группа долдна быть основной группой пользователя (primary group).

Далее - сам пользователь. Основной группой будет группа `sftp-users` - по другому не работает. Как оболочку поставлю заглушку `nologin`, которая будет отображать сообщение при попытке подключения. Ну и пароль, да.

```bash
useradd -g sftp-users -m -s /sbin/nologin wp-test
usermod -a -G www-data wp-test
passwd wp-test
```

>**NOTE**  
>условимся, что пароль будет _password_ просто для простоты демонстрации, но в реальности лучше использовать что-то типа `pwgen -1 -s 64`, чтобы получить более взломо-стойкий пароль. Можно и так `openssl rand -hex 32`

TODO: если пулл для каждого пользователя будет запущен под этим пользователем то запихивать пользователя в группу ввв-дата будет не нужно? или нужно из-за энжджина?

Имя пользователя выбираем по тому же принципу - главное запомнить что там. Ну или записывать где-то. Что, наверно, надёжнее.

>**NOTE**  
>Для демонтрации имя пользователя будет `wp-test`


## sFTP

Для sFTP ставить ничего не нужно. Все уже и так установлено. Нужно только подправить конфиг SSH сервера.

>**NOTE**  
>В тех редких случаях, когда SSH все таки не установлен, установить его можно командой `apt-get install openssh-server`

Нужный нам конфиг находиться в файвле _/etc/ssh/sshd_config_. Воспользуемся возможностью использовать инклюды и не будем трогать основной файл.Вместо этого создадим _/etc/ssh/sshd_config.d/10-sftp.conf_ и впишем нужные опции туда.

```conf
# Subsystem sftp /usr/lib/openssh/sftp-server
Match group sftp-users
ChrootDirectory %h
X11Forwarding no
AllowTcpForwarding no
ForceCommand internal-sftp
```

TODO: расшивфровать каждую опцию

```bash
chown root:root /home/wp-test/
chmod 755 /home/wp-test/
mkdir /home/wp-test/{uploads,wordpress}
chown wp-test:www-data /home/wp-test/uploads
chown wp-test:www-data /home/wp-test/wordpress
# обеденить строки выше глобом?
# выставлять шоуппу по умолчанию как группу влядеющую корневоц папкой https://serverfault.com/questions/816330/set-ownership-on-uploaded-files
chmod g+s /home/wp-test/{uploads,wordpress}
```

TODO: човнить на рута нужно потому что иначе чрут не сработает. Корень для чьрута должен быть под пользователем рут, осе остальные файлы можно отдать пользователю системы
Заметте, что директория пользователя не пренадлежит пользователю. Это необзодимо для работы sftp. В прочьем, файлы и директории внутри доманшей пользователю принядледать мошгут. 

> _man sshd_config_:  
>All components of the pathname must be root-owned directories that are not writable by any other user or group

Затем перезапустим ssh сервис и убедимся, что он поднялся нормально:

```bash
sshd -t
systemctl reload sshd
```

TODO: Задание на А+ - ключь для логина по sftp потому как по папролю таки зашквар) не забыть удалить пароль у существующего пользователя и отключить логин по паролю

Добавляем ключь разработчика сюда.

```bash
mkdir /home/wp-test/.ssh
touch /home/wptest1/.ssh/authorized_keys
chown -R wp-test:sftp-users /home/wptest1/.ssh
vim /home/wptest1/.ssh/authorized_keys
```

TODO: проверит сфтп


если разрабо хочет по паролю? Пошлите его, это достаточно опастно чтобы  так делать, но если уже совсем совсем надо:

_/etc/ssh/sshd_config.d/100-allow.password.conf_

```conf
ChallengeResponseAuthentication yes

Match User wp-test
PasswordAuthentication yes
```

но лучше упираться до последнего. рано ли поздно пароль подберут.

Received message too long 1416128883
Ensure the remote shell produces no output for non-interactive sessions.
https://unix.stackexchange.com/questions/61580/sftp-gives-an-error-received-message-too-long-and-what-is-the-reason

## Настрока базы

Теперь, когда пользователь настроен можно преступить к подготовке базы для CMS. Нам потребуется создать базу, пользователя и выдать этому пользователю права на управление базой. Сейчас нам нужно оказаться в консоли MySQL под пользователем root.

```bash
mysql
```

Далее, создадим базу, пользователя и выдадим ему права:

```sql
CREATE DATABASE `wp-test`;
CREATE USER 'wp-test'@'localhost' IDENTIFIED BY 'password';
GRANT ALL ON `wp-test`.* TO 'wp-test'@'localhost'; #WITH GRANT OPTION
FLUSH PRIVILEGES;
exit;
```
TODO: расшивфровать каждую опцию и что значит локалхост и процеентик если бы ьыл или адрес и что значат звездочки (надо ли тут уже на отдельную статью тянет)
TODO: !!! https://www.strongdm.com/blog/mysql-create-user-manage-access-privileges-how-to


## Wordpress cli и установка CMS

https://wp-cli.org/#installing

```bash
wget https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
php wp-cli.phar --info
chmod +x wp-cli.phar
mv wp-cli.phar /usr/local/bin/wp
```

TODO: тут нужно бутьпод определенным пользователм лиш бы не рут иначе задолбает и неизвестно где вылезет еще в идеале тем же кем и будтет редактироватсья а затем просто вернуть ему запрет на логин. пробую под рутом с --allow-root а там поглядим

чтоюы отдавать команды как другой пользователь но при жтом не логинитья в него воспольщуемся sudo с указанием имени пользователя

если мы просто сделаем су в пользователя вп-тест то ситема нам не даст этого сдеоать сославшись на то что пльзщватель не доступен `This account is currently not available.` чтобы обойти это испольщзуес

```bash
sudo -u wp-test -s /bin/bash
# или
sudo su wp-test -s /bin/bash
```

если выполнить последнюю команду без судо и влогиниться как пользователь опять же получим ов\\ответ о том что пользователь недоступен.

```bash
# cd /home/wp-test/wordpress
cd ~/wordpress
wp core download
# make couple of spaces pefore conmmand so bash wont savw this command in history
wp core config --dbhost=localhost --dbname=wp-test --dbuser=wp-test --dbpass=password

# wp coonfig FS_METHOD ssh2
wp config set FS_METHOD direct
# wp config set WP_MEMORY_LIMIT 128M #500M
# wp config WP_DEBUG_DISPLAY false
# wp config set WP_DEBUG true
# wp config set WP_DEBUG_LOG true
# wp config set FORCE_SSL_ADMIN true



# chmod 644 wp-config.php
# make couple of spaces pefore conmmand so bash wont savw this command in history
wp core install --url=wp-test.wp.local --title="Your Blog Title" --admin_name=admin --admin_password="password" --admin_email=you@example.com

# chmod 775 wp-content/uploads/
# chmod 775 wp-content/plugins/
# chmod 775 wp-content/themes/
# ??? chown -R wp-test:www-data ./
# чекнуть владельца на всякий, по идее човнить не нужно будет
```

проверим все ли нормально установилось.

```bash
wp server --htost=0.0.0.0 --port=80
```

шаг опциональный. это поднимит временный сервер можно будет посмотреть как все заработало. перое обращение может быть он очень долгим

если нужно сменить урл

```bash
wp option update home 'http://wp-test.wp.local'
wp option update siteurl 'http://wp-test.wp.local'
```

```bash
wp user reset-password admin
```

новый пароль уйдет на почту админа

http://wp-test.local/wp-admin/site-health.php?tab=debug
infp filesystem permisiions
админка очычно на /wp-login



## Настрока web сервера

FIXME: попробовать свой конфиг, смешно но вордрессовский похоже не сильно хочет работаь 

настраиваем простой nginx
кофиг взят из фоффициальной кокументайии по вордперессу
https://wordpress.org/support/article/nginx/

_/etc/nginx/sites-available/wp-test.conf_

```nginx
client_max_body_size 10M;

upstream php {
        server unix:/var/run/php/php-fpm.sock;
        # server 127.0.0.1:9000;
}

#server {
#        server_name www.wp-test.local;
#        return 301 $scheme://wp-test.local$request_uri;
#}

server {
        server_name wp-test.local;
        listen 80;
        root /home/wp-test/wordpress;
        index index.php;
 
        location = /favicon.ico {
                log_not_found off;
                access_log off;
        }
 
        location = /robots.txt {
                allow all;
                log_not_found off;
                access_log off;
        }
 
        location / {
                try_files $uri $uri/ /index.php$is_args$args;
        }
 
        location ~ \.php$ {
                #NOTE: You should have "cgi.fix_pathinfo = 0;" in php.ini
                include fastcgi.conf;
                fastcgi_intercept_errors on;
                fastcgi_pass php;
        }
 
        location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
                expires max;
                log_not_found off;
        }
}
```

!!! client_max_body_size 100M;

```bash
ln -s /etc/nginx/sites-available/wp-test.conf /etc/nginx/sites-enabled
nginx -t
nginx -s reload
```



## Сертификаты для https

TODO: отдельной статьей как прикрутить ссл к нгиникс в авто режиме сертботом к голому энджину

```bash
apt-get install certbot python3-certbot-nginx

cp /etc/nginx/sites-available/wp-test.conf /etc/nginx/sites-available/wp-test.conf.bak

certbot --nginx
# certbot --nginx -d example.com -d www.example.com
```

настроить авто обновление сеттов как у меня в заметках

```bash
certbot renew --nginx
```

```conf
SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

0 */12 * * * root certbot -q renew --nginx
```



## htpasswd

TODO: добавить к описанрию установки htpasswd или не добавлять и нписать что если нужно ставить
https://www.digitalocean.com/community/tutorials/how-to-set-up-password-authentication-with-nginx-on-ubuntu-14-04




## Синопсис

TODO: какие домены, какие учетки, как логиниться, где что анходиться

---

## Extra: FTP сервер (нахрен фтп, вот честно)

Когда сфтп не вариант


чтобы получить желанные 755
local_umask=002
file_open_mode=0755

и серт для него. но вп поддерживает сфтп пожтому это не должно быть вариантом большенство времени. на п\описывать?

## Extra: Firewall

sudo ufw allow ssh
sudo ufw enable
sudo ufw status


## Extra: clean history 

history -c && exit

## тюнинг энджина
TODO: отдельной статьей?

 гзип
 кеши

https://stackoverflow.com/questions/66356142/time-out-504-gateway-time-out-nginx-in-wordpress-php-fpm
только две последние по 60 как бы не пришлось провнять со всеми и ставить 300
кое где и 600 приходилось ставить

просто дамп, как подбирались - я хз) эмпирически
https://gist.github.com/denji/8359866

_/etc/nginx/conf.d/tuning.conf_
...
client_max_body_size 128M;
proxy_buffer_size   128k;
proxy_buffers   4 256k;
proxy_busy_buffers_size   256k;
client_max_body_size 100M;
sendfile on;
tcp_nopush on;
tcp_nodelay on;

client_body_timeout 10;
send_timeout 2;
keepalive_timeout 30;
keepalive_requests 100000;

server_tokens off;

gzip on;
# gzip_static on;
gzip_min_length 10240;
gzip_comp_level 1;
gzip_vary on;
gzip_disable msie6;
gzip_proxied expired no-cache no-store private auth;
reset_timedout_connection on;
gzip_types
        # text/html is always compressed by HttpGzipModule
        text/css
        text/javascript
        text/xml
        text/plain
        text/x-component
        application/javascript
        application/x-javascript
        application/json
        application/xml
        application/rssxml
        application/atom+xml
        font/truetype
        font/opentype
        application/vnd.ms-fontobject
        image/svg+xml;
...


## резделение пулов - не отдельная статься

## тюнинг пыха
TODO: отдельной статьей?

_/etc/php/${php-version}/fpm/php.ini_

```ini
max_execution_time = 600
max_input_time = 600
post_max_size = 100M
upload_max_filesize = 3G
memory_limit = 128M
```

_/etc/php/7.4/fpm/pool.d/www.conf_
```ini
pm.max_children = 50
pm.start_servers = 10
pm.min_spare_servers = 10
pm.max_spare_servers = 20
pm.max_requests = 1000
```

ОСНОВНОЦ РЫЧАГ - КОЛИЧЕСТВО ЗАПРОСОВ НА ПРОЦЕСС????

динамик для шареда с раздельнми пулами
СТАТИК МОД для прода!!!
как то ж можно высчитывать чтобы в память вписывалось

смотрим логи - просит больше процессов - даем, смотрим чтобы не переполнило память, базансируем. так же кол во реквестов на процесс тоже можно подкрутить

разделить пулы
безопастность и ресурсы распрделение
каждыц пул от имени пользователя - влядельца проектом
у каждого пула свой сокет?
персональных пхп ини?
время исполнения
конфиги брать из нашего сервака с пометкой "может приголится"
тамауты скорее всего пришлось увеличивать изза плохо ходищих писам


## квоты на файловую систему

НАДО!!!

## sendmail + multiple domains

https://gist.github.com/adamstac/7462202
https://www.techrepublic.com/article/configure-it-quick-set-up-sendmail-to-host-multiple-domains/

отсылка почты тормозит все и занимает много времени
так же где смотреть если бансы
https://serverfault.com/questions/58363/my-unqualified-host-name-foo-bar-unknown-problem
/var/spool/mail
 unable to qualify my own domain name (localhost) -- using short name


## Экстра настройка фрзмера заливаемых файлов

https://www.tecmint.com/increase-file-upload-size-in-php/
https://www.tecmint.com/limit-file-upload-size-in-nginx/
https://www.cloudways.com/blog/increase-media-file-maximum-upload-size-in-wordpress/


## дополнительное чтение

https://wp-cli.org/
https://wordpress.org/support/