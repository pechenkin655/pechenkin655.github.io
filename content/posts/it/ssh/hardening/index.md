---
date: 2017-04-09T10:58:08-04:00
description: "???"
featured_image: "/hero/system.png"
images: ["/hero/system.png"]
tags: ["linux", "system administration"]
categories: "it"
title: "Укрепляем ssh"
comment: false
---

## Затягиавем гайки
TODO: отдельная статья о том как настроить себе пользователя если есть только рут?

Раз уж я оказался здесь - подкрутим немного настройки sshd. Это будет еще один файл в директории _/etc/ssh/sshd_config.d/_. Его так же можно называть как угодно, но я назову его _2-hardening.conf_

TODO: расписать что кадда строска желает

```ini
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
TODO: отдельная статься о директориях типа conf.d
ssh, nginx, php используют как минимум, механизм инклюдов

Зачем вообще создавать несколько файлов, ведь можно просто редиктировать _/etc/ssh/sshd_config_ и не заморачиваться, спросите вы. Да можно. Но это моэет вызвать проблемы при обновлении системы с Манжаро.Арчь не встречал с убунту посточнно. Оригинальный файл изменен, что же с ним делать? Вспомнили? Обычно такими вопросами просто заваливает убунту когда делаеш релиз апгрейд. использование этого метода изваяляет от этого. к тому же всегда легче найти свои правки т.к. они в отдельном файле.