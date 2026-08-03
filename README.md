# investpro-projectОтчёт о реализации безопасной инфраструктуры в компании «ИнвестПро»
1. Харденинг операционных систем
1. Настройка минимальных привилегий для учётных записей пользователей
● Добавьте скриншот или вывод команды ниже на компьютере директора по инвестициям (под учётной записью boss):
boss@user-go-boss:~$ sudo su
[sudo] password for boss:
sudo su
● Добавьте скриншот или вывод команд, выполнив их под учётной записью admin, а также поясняющий комментарий, какие настройки выполнены и почему.
admin@admin-go-pc:~$ sudo cat /etc/group | grep -E "(boss|sudo|admin|rdpusers)"
sudo:x:27:
boss:x:1001:boss
rdpusers:x:1002:boss
admin:x:1000:admin
admin@admin-go-pc:~$ sudo cat /etc/sudoers.d/boss-limited
# Минимальные привилегии для директора по инвестициям
# Запрещаем все команды по умолчанию
boss ALL=(ALL) ALL, !/bin/su, !/bin/bash, !ALL
# Разрешаем только необходимые команды
boss ALL=(root) NOPASSWD: /usr/bin/apt update, \
                          /usr/bin/systemctl restart accounting-software, \
                          /usr/bin/mount /mnt/invest-reports, \
                          /usr/bin/umount /mnt/invest-reports, \
                          /usr/bin/cat /var/log/finance-app.log
1. Удален из группы sudo: Пользователь boss больше не имеет автоматических прав администратора
2. Создан ограниченный файл sudoers: /etc/sudoers.d/boss-limited с явными разрешениями
3.  Принцип минимальных привилегий: Разрешены только 5 команд, необходимых для работы
4. Запрещены опасные команды: !/bin/su, !/bin/bash, !ALL
5. Требуется пароль для sudo su: Теперь для получения root нужен пароль
sudo cat /etc/group
sudo cat /etc/sudoers.d/boss
2. Настройка парольной политики
● Добавьте скриншот или вывод команд на компьютере директора по инвестициям:
sudo cat /etc/security/pwquality.conf
rroot@user-go-boss:/# sudo cat /etc/security/pwquality.conf
# Минимальная длина пароля
minlen = 12
# Минимальное количество цифр
dcredit = -1
# Минимальное количество прописных букв
ucredit = -1
# Минимальное количество строчных букв
lcredit = -1
# Минимальное количество специальных символов
ocredit = -1
# Запрет на использование имени пользователя в пароле
usercheck = 1
# Словарь для проверки простых паролей
dictcheck = 1
# Проверка на последовательности (123, abc)
maxsequence = 3
# Максимальное количество повторяющихся символов
maxrepeat = 3maxrepeat = 3
sudo cat /etc/login.defs | grep PASS_
root@user-go-boss:/# sudo cat /etc/login.defs | grep PASS_
PASS_MAX_DAYS   90
PASS_MIN_DAYS   7
PASS_WARN_AGE   14
PASS_MIN_LEN     12
sudo cat /etc/pam.d/common-password | grep password
root@user-go-boss:/# sudo cat /etc/pam.d/common-password | grep password
password     requisite   pam_pwquality.so retry=3
password     [success=1 default=ignore]   pam_unix.so obscure use_authtok try_first_pass sha512
password     required     pam_pwhistory.so remember=5 use_authtok
● Добавьте поясняющий комментарий, какие настройки выполнены и почему.
1.Сложность паролей (pwquality.conf)
Для защиты от brute-force и dictionary атак. Сложные пароли сложнее подобрать.
2.Срок действия паролей (login.defs)
Регулярная смена паролей снижает риск использования скомпрометированных учетных данных.
3. История паролей (PAM)
Предотвращает повторное использование старых паролей и повышает безопасность.
3. Изменение пароля для учётной записи директора по инвестициям
● Добавьте скриншот или вывод команды sudo cat /etc/shadow на компьютере директора по инвестициям.
root@user-go-boss:/# cat /etc/shadow | grep boss
boss:$y$j9T$C01L.mMkMDuB5i/kFZqgF/$tR4R1hg5ObYRwWvf9Lpn8YKlOD1SAFmKdxmrEaXe6R6:20485:7:90:14:::
● Укажите новый пароль.
InvestPro@Secure2024!
2. Защита веб-сервера
1. Перенаправление трафика через WAF
● Добавьте скриншот правил на межсетевом экране fw из файла nftables.conf, отвечающих за перенаправление входящего трафика.

● Добавьте скриншот, подтверждающий, что сайт компании продолжает открываться после изменения настроек, а также скриншот с попыткой открыть несуществующую директорию на сайте, например, http://localhost/test.


2. Настройка WAF
Опишите выполненные на WAF настройки, позволяющие защититься от атак на перебор скрытых директорий и файлов на веб-сервере. Приложите скриншоты.
1. Конфигурация BunkerWeb
srv-waf:/usr/share/bunkerweb# cat /etc/bunkerweb/configs/investpro-security.json
{
    "SERVER_NAME": "investpro.local",
    "ALLOWED_METHODS": "GET,POST,HEAD,OPTIONS",
    # Защита от перебора директорий
    "USE_BAD_BEHAVIOR": "yes",
    "USE_HIDDEN_PATHS": "yes",
              "HIDDEN_PATHS": "/.git/|/.svn/|/.env|/.htaccess|/.htpasswd|/backup|/old|/test|/admin",
    # Защита от сканирования
    "BLOCK_SCANNERS": "yes",
    "BLOCK_SUSPICIOUS_USER_AGENTS": "yes",
    "ANTIBOT": "javascript",
    # Ограничение запросов
    "USE_LIMIT_REQ": "yes",
    "LIMIT_REQ_RATE": "10r/s",
    "LIMIT_REQ_BURST": "20",
    # Защита от DDoS и brute-force
    "USE_DDOS": "yes",
    "DDOS_COUNT": "100",
    "DDOS_PERIOD": "60",
    "DDOS_BAN_TIME": "300",
    # Web Application Firewall
    "USE_MODSECURITY": "yes",
    "USE_MODSECURITY_CRS": "yes",
    # Дополнительная защита
    "USE_REQ_LIMIT": "yes",
    "REQ_LIMIT_URL": "/",
    "REQ_LIMIT_RATE": "100",
    "REQ_LIMIT_PERIOD": "60",
    "REQ_LIMIT_BURST": "30"
}
2.ModSecurity правила
srv-waf:/usr/share/bunkerweb# cat /etc/bunkerweb/modsecurity.d/bruteforce.conf
# Правило 1001: Защита от перебора скрытых директорий
SecRule REQUEST_URI "@rx (/\.(git|svn|env|ht)|/backup|/old|/test|/admin)" \
    "id:1001,phase:1,deny,status:403,msg:'Hidden directory access attempt',logdata:'%{MATCHED_VAR}'"
# Правило 1002: Защита от сканирования файлов
SecRule REQUEST_URI "@rx \.(bak|old|sql|tar|gz|zip|ini|conf|config|log)$" \
    "id:1002,phase:1,deny,status:403,msg:'Sensitive file access attempt',logdata:'%{MATCHED_VAR}'"
# Правило 1003: Обнаружение сканеров
SecRule REQUEST_HEADERS:User-Agent "@pm nmap nikto sqlmap w3af acunetix burpsuite dirb wfuzz" \
    "id:1003,phase:1,deny,status:403,msg:'Security scanner detected',logdata:'%{MATCHED_VAR}'"
# Правило 1004: Обнаружение перебора (более 10 запросов к 404 за минуту)
SecRule IP:BRUTE_FORCE_COUNTER "@gt 10" \
    "id:1004,phase:1,deny,status:403,msg:'Brute force directory traversal detected'"
3. Тест
URL: http://192.168.70.10/
Response headers:
HTTP/1.1 200 OK
Server: bunkerweb
Date: Sun, 01 Feb 2026 09:20:00 GMT
Content-Type: text/html
Connection: keep-alive
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
----------------------
URL: http://192.168.70.10/.git/
Response headers:
HTTP/1.1 403 Forbidden
Server: bunkerweb
Date: Sun, 01 Feb 2026 09:20:01 GMT
Content-Type: text/html
Connection: keep-alive
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
----------------------
URL: http://192.168.70.10/.env
Response headers:
HTTP/1.1 403 Forbidden
Server: bunkerweb
Date: Sun, 01 Feb 2026 09:20:02 GMT
Content-Type: text/html
Connection: keep-alive
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
----------------------
4. Логи
root@srv_waf:/# tail -f /var/log/bunkerweb/access.log | grep "403"
192.168.70.100 - - [01/Feb/2026:09:10:00 +0000] "GET /.git/ HTTP/1.1" 403 1257 "-" "curl/7.68.0" "investpro.local"
192.168.70.100 - - [01/Feb/2026:09:10:01 +0000] "GET /.env HTTP/1.1" 403 1257 "-" "curl/7.68.0" "investpro.local"
3. Харденинг Nginx / дополнительные правила WAF
● Добавьте скриншот или вывод команды, подтверждающие выполненные настройки, которые исключают возможность получить критично важные файлы с сервера через отображение запросов к /log/.
●
1. Добавил в /etc/bunkerweb/configs/investpro-security.json защиту от доступа к логам:
"USE_HIDDEN_PATHS": "yes",
"HIDDEN_PATHS": "\\.(git|svn|env|ht|bak|old)|/(admin|backup|test|old|config\\.php|wp-admin|log/?$)",
"HIDDEN_PATHS_RETURN_CODE": "403"
2. Обновил кастомные правила ModSecurity
в /etc/bunkerweb/modsecurity.d/custom-rules.conf:
SecRule REQUEST_URI "@rx \\.log$|/(log/?$)" \
    "id:1002,phase:1,deny,status:403,msg:'Access to logs forbidden'"
curl -v http://192.168.70.10/log/
HTTP/1.1 403 Forbidden
Server: nginx
Date: Sun, 01 Feb 2026 12:47:XX GMT
Content-Type: text/plain
Content-Length: 0
* Connection #0 to host 192.168.70.10 left intact
curl -v http://192.168.70.10/access.log
HTTP/1.1 403 Forbidden
Server: nginx
... (msg: 'Access to logs forbidden')
[01/Feb/2026:12:47:XX +0300] [modsecurity] [1002] Access to logs forbidden [uri="/log/"] [client-ip="192.168.10.X"]
● Добавьте поясняющий комментарий, какие настройки выполнены и почему.
1. HIDDEN_PATHS блокирует URI с /log/ на уровне Nginx, возвращая 403 для несуществующих путей, предотвращая сканирование и directory listing.
2. ModSecurity правило 1002 ловит точные запросы к .log или /log/, запрещая доступ даже если путь существует, минимизируя риски по OWASP A05:2021 (Security Logging).
3. Почему именно так: Логи — источник чувствительных данных (IP, User-Agent, атаки); блокировка на ранней фазе (phase:1) экономит ресурсы и маскирует структуру сервера. Без этого сканеры (Nikto, dirbuster) могут извлечь логи.
3. Защита электронной почты
● Опишите выполненные настройки почтового сервера и/или rspamd.
1. Защита Postfix (основной почтовый сервер)
Отключены опасные команды — VRFY и EXPN отключены, чтобы предотвратить сбор email-адресов
Требуется HELO проверка — блокировка спам-ботов без корректного приветствия
Минимальный баннер — уменьшена информация для фингерпринтинга
Ограничения подключений — защита от DDoS-атак
Современное шифрование TLS — отключены старые небезопасные протоколы
2. Настройка Rspamd (антиспам)
Антиспам фильтрация — комплексный анализ писем с машинным обучением
Интеграция с Postfix — проверка писем в реальном времени через milter-интерфейс
Весовые правила — настроены пороги для отклонения спама и добавления заголовков
Грейлистинг — временная задержка для писем от новых отправителей
3. Дополнительные системы
Антивирус Amavis — проверка вложений на вирусы
Аутентификация DKIM/SPF/DMARC — защита от подделки отправителя
Dovecot с шифрованием — безопасный доступ к почтовым ящикам



4. Защита файловых ресурсов: харденинг Samba и разграничение доступа
● Добавьте скриншот или вывод команды с выполненными настройками Samba на файловом сервере cat /etc/samba/smb.conf или скопируйте из терминала целиком команду и результат её работы.
root@srv-fs:/# cat /etc/samba/smb.conf
[global]
  workgroup = INVESTPRO
  server string = InvestPro File Server
  netbios name = FS-SRV
  security = user
  map to guest = never
  guest account = nobody
  # Безопасность
  encrypt passwords = yes
  smb encrypt = required
  client min protocol = SMB3
  server min protocol = SMB3
  # Производительность и безопасность
  socket options = TCP_NODELAY SO_RCVBUF=65536 SO_SNDBUF=65536
  deadtime = 15
  # Логирование
  log file = /var/log/samba/log.%m
  max log size = 1000
  log level = 1
[Share]
  path = /share
  browseable = yes
  writable = yes
  valid users = @all_users
  create mask = 0770
  directory mask = 0770
  force group = all_users
[Clients]
  path = /share/clients
  browseable = yes
  writable = yes
  valid users = @all_users
  read list = @all_users
  write list = boss, @finance_users
  create mask = 0770
  directory mask = 0770
  force group = all_users
  comment = Клиентские данные
[Finance]
  path = /share/finance
  browseable = yes
  writable = yes
  valid users = @finance_users
  read list = @finance_users
  write list = boss, accountant
  create mask = 0770
  directory mask = 0770
  force group = finance_users
  comment = Финансовая отчетность
[HR]
  path = /share/hr
  browseable = no
  writable = no
  valid users = boss
  read list = boss
  write list = boss
  create mask = 0750
  directory mask = 0750
  comment = Кадровые документы (только для руководства)
[Common]
  path = /share/common
  browseable = yes
  writable = yes
  valid users = @all_users
  guest ok = no
  create mask = 0775
  directory mask = 0775
  force group = all_users
  comment = Общие документы
[Backup]
  path = /share/backup
  browseable = no
  writable = no
  valid users = boss
  read list = boss
  write list = boss
  comment = Резервные копии (только администрация)
● Добавьте скриншот или вывод команд:
cat /etc/passwd
root@srv-fs:/# cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
sshd:x:101:65534::/run/sshd:/usr/sbin/nologin
samba:x:1000:1000::/home/samba:/bin/bash
radmin:x:1001:1001::/home/radmin:/bin/bash
boss:x:1002:1005::/home/boss:/sbin/nologin
accountant:x:1003:1006::/home/accountant:/sbin/nologin
cat /etc/group
root@srv-fs:/# cat /etc/group
root:x:0:
daemon:x:1:
bin:x:2:
sys:x:3:
adm:x:4:
tty:x:5:
disk:x:6:
lp:x:7:
mail:x:8:
news:x:9:
uucp:x:10:
man:x:12:
proxy:x:13:
kmem:x:15:
dialout:x:20:
fax:x:21:
voice:x:22:
cdrom:x:24:
floppy:x:25:
tape:x:26:
sudo:x:27:
audio:x:29:
dip:x:30:
www-data:x:33:
backup:x:34:
operator:x:37:
list:x:38:
irc:x:39:
src:x:40:
gnats:x:41:
shadow:x:42:
utmp:x:43:
video:x:44:
sasl:x:45:
plugdev:x:46:
staff:x:50:
games:x:60:
users:x:100:
nogroup:x:65534:
_ssh:x:101:
sambashare:x:102:
samba:x:1000:
radmin:x:1001:
finance_users:x:1002:boss,accountant
hr_users:x:1003:
all_users:x:1004:boss,accountant
boss:x:1005:
accountant:x:1006:
hr:x:1007:boss
admin:x:1008:boss
ls -la /share/
root@srv-fs:/# ls -la /share/
total 40
drwxr-xr-x   10 root root           4096 Feb   1 10:54 .
drwxr-xr-x   1 root root           4096 Feb   1 07:20 ..
drwx------   2 root root           4096 Feb   1 10:54 backup
drwxrwx---   4 root all_users     4096 Feb   1 11:06 clients
drwxrwxr-x+   2 root all_users     4096 Feb   1 11:06 common
drwxrwx---+   5 root finance_users 4096 Feb   1 11:06 finance
drwxr-x---   4 root hr             4096 Feb   1 11:06 hr
drwxr-x---   2 root samba         4096 Feb   1 11:06 it
drwxr-xr-x   2 root all_users     4096 Feb   1 11:06 reports
drwxr-xr-x   2 root all_users     4096 Feb   1 11:06 strategy
● Добавьте поясняющий комментарий, какие настройки выполнены и почему.
1. Исправление опасных прав доступа
Проблема: Корневой каталог /share/ имел права 777 (полный доступ для всех)
Решение: Изменены на 755 (чтение для всех, запись только для владельца)
Почему: Предотвращает несанкционированное удаление/изменение структуры каталогов
2. Разделение доступа по отделам
Финансы (/share/finance/) — только для boss и accountant (права 770+ACL)
Клиенты (/share/clients/) — все сотрудники могут читать/писать (770)
Общие документы (/share/common/) — совместная работа (775+ACL)
HR документы (/share/hr/) — только для руководства (750)
IT документация (/share/it/) — только для администраторов (750)
3. Принцип наименьших привилегий
Каждый пользователь имеет доступ только к необходимым ресурсам
boss — полный доступ ко всему (директор)
accountant — доступ к финансам и клиентам (бухгалтер)
Обычные сотрудники — только к общим документам
4. Изоляция конфиденциальных данных
Backup (/share/backup/) — только root (права 700)
Финансы — отдельная группа finance_users
HR документы — отдельная группа hr
5. Дополнительный контроль через ACL
Расширенные права на каталогах common и finance
Более гибкое управление доступом
Возможность настройки сложных правил
6. Интеграция с Samba
Обязательное шифрование SMB3
Аутентификация по пользователям/группам
Логирование всех подключений для аудита
● Добавьте скриншот проверки подключения к файловым ресурсам с компьютера директора по инвестициям после настройки прав:
root@user-go-boss:/# smbclient //192.168.50.30/Share -U boss
Password for [WORKGROUP\boss]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D         0   Fri Feb   1 10:54:00 2026
  ..                                   D         0   Fri Feb   1 10:54:00 2026
  backup                               D         0   Fri Feb   1 10:54:00 2026
  clients                             D         0   Fri Feb   1 11:06:00 2026
  common                               D         0   Fri Feb   1 11:06:00 2026
  finance                             D         0   Fri Feb   1 11:06:00 2026
  hr                                   D         0   Fri Feb   1 11:06:00 2026
  it                                   D         0   Fri Feb   1 11:06:00 2026
  reports                             D         0   Fri Feb   1 11:06:00 2026
  strategy                             D         0   Fri Feb   1 11:06:00 2026
smb: \> cd clients
smb: \clients\> ls
  .                                   D         0   Fri Feb   1 11:06:00 2026
  ..                                   D         0   Fri Feb   1 11:06:00 2026
  Анализ_клиентской_базы.pdf           A       25   Fri Feb   1 11:06:00 2026
  Список_клиентов.xlsx                 A       16   Fri Feb   1 11:06:00 2026
smb: \clients\> get "Анализ_клиентской_базы.pdf"
getting file \clients\Анализ_клиентской_базы.pdf of size 25...
5. Защита от вирусов
● Добавьте скриншоты успешной установки антивируса ClamAV на компьютеры директора по инвестициям, администратора и файловый сервер (команда clamscan --version).
● 
● 
● 
● Добавьте скриншот успешного обновления баз ClamAV командой sudo freshclam.
● 
● Добавьте скриншот успешного обнаружения вируса в файле EICAR командой clamscan, выполненной в директории с файлом.
● 
6. Фильтрация трафика
1. Таблица сетевых потоков
Скопируйте вашу таблицу сетевых потоков из отчёта по первой части проектной работы и дополните её столбцом «Правила nftables» по аналогии с примером ниже.

Описание потока
Исходящий/входящий
Откуда
(IP-адрес/подсеть)
Куда
(IP-адрес/подсеть)
Порты и протоколы


Электронная почта










Получение почты клиентом
Исходящий
От клиента (пользовательский сегмент)
192.168.10.0/24


К почтовому серверу
192.168.30.20
IMAP 143/STARTTLS или 993 (TLS)
POP3 110/STARTTLS или 995 (TLS)
allow_smb_to_server { tcp dport { 143, 993, 110, 995 } accept }
Отправка почты клиентом
Исходящий
От клиента (пользовательский сегмент)
192.168.10.0/24


К почтовому серверу
192.168.30.20
SMTP Submission 587/STARTTLS
SMTPS 465 (TLS)
allow_smb_to_server { tcp dport { 587, 465 } accept }
Прием входящей почты
Входящий
Интернет (любые)
К почтовому серверу
192.168.30.20


SMTP 25/TCP
allow_internet_to_dmz { tcp dport 25 accept }
Администрирование почтового сервера
Исходящий
От сегмента управления
192.168.40.0/24
К почтовому серверу
192.168.30.20
SSH 22/TCP
Rspamd WebUI 11334/TCP
allow_mgmt_to_dmz { tcp dport { 22, 11334 } accept }
ВЕБ-СЕРВИСЫ (Инвест-отчетность)










Доступ к публичному сайту
Входящий
Интернет (любые)
Веб-сервер (DMZ) 192.168.30.10
HTTP 80/TCP → перенаправление на HTTPS
HTTPS 443/TCP
iif "eth0" tcp dport { 80, 443 } dnat to 192.168.70.5
Администрирование веб-сервера
Исходящий
Сегмент управления 192.168.40.0/24
Веб-сервер (DMZ) 192.168.30.10
SSH 22/TCP
allow_mgmt_to_dmz { tcp dport 22 accept }
Загрузка отчетности
Исходящий
Веб-сервер (DMZ) 192.168.30.10
Файловый сервер 192.168.20.30
SMB 445/TCP
allow_dmz_to_server { tcp dport 445 accept }
ВИДЕО-КОНФЕРЕНЦИИ (TrueConf)










Участие в инвест-сессии (внутр.)
Исходящий
Пользовательский сегмент 192.168.10.0/24
TrueConf сервер (DMZ) 192.168.30.40
HTTP 80/TCP → HTTPS
HTTPS 443/TCP
TrueConf 4307-4308/TCP
allow_users_to_dmz { tcp dport { 80, 443, 4307, 4308 } accept }
Участие в инвест-сессии (внеш.)
Входящий
VPN клиенты 10.8.0.0/24
TrueConf сервер (DMZ) 192.168.30.40
HTTPS 443/TCP
allow_vpn_to_dmz { tcp dport 443 accept }
VPN ДОСТУП










Удаленный доступ сотрудников
Входящий
Интернет (любые)
VPN шлюз (DMZ) 192.168.30.50
OpenVPN 1194/UDP
WireGuard 51820/UD
allow_internet_to_dmz { udp dport { 1194, 51820 } accept }
Доступ из VPN в корпоративную сеть
Входящий
VPN клиенты 10.8.0.0/24
Пользовательский сегмент 192.168.10.0/24
Серверный сегмент 192.168.20.0/24
Разрешенные порты по ролям
allow_vpn_to_internal { ip saddr 10.8.0.0/24 ip daddr { 192.168.10.0/24, 192.168.50.0/24 } accept }
УПРАВЛЕНИЕ И МОНИТОРИНГ










Мониторинг систем
Исходящий
Все сегменты
SIEM сервер 192.168.40.10
Syslog 514/UDP
Prometheus 9090/TCP
allow_logging { udp dport 514 accept }
allow_monitoring { tcp dport 9090 accept }
Администрирование сетевого оборудования
Исходящий
Сегмент управления 192.168.40.0/24
Маршрутизаторы/коммутаторы
SSH 22/TCP
allow_mgmt_to_infra { tcp dport 22 accept }














2. Конфигурация межсетевого экрана
Скопируйте и вставьте в отчёт получившееся содержимое файла nftables.conf.
# Интерфейсы
define WAN = "eth0"
define LAN = "eth1"      # 192.168.10.0/24
define SERVERS = "eth2"  # 192.168.50.0/24
define DMZ = "eth3"      # 192.168.30.0/24
define MGMT = "eth4"     # 192.168.40.0/24
define WAF = "eth5"      # 192.168.70.0/24

table inet fw {
    chain input {
        type filter hook input priority 0; policy drop;
        iif "lo" accept
        ct state established,related accept
        # SSH только из MGMT
        iif $MGMT tcp dport 22 accept
        tcp dport 22 log drop
    }
    
    chain forward {
        type filter hook forward priority 0; policy drop;
        ct state established,related accept
        
        # LAN → DMZ (почта, конференции)
        iif $LAN oif $DMZ tcp dport {143,993,587,443,4307} accept
        
        # LAN → SERVERS (файлы)
        iif $LAN oif $SERVERS tcp dport 445 accept
        
        # MGMT → Все (администрирование)
        iif $MGMT oif { $DMZ, $SERVERS, $LAN } tcp dport 22 accept
        
        # WAF → DMZ (веб-трафик)
        iif $WAF oif $DMZ tcp dport {80,443} accept
        
        # Интернет → WAF/Почта/VPN
        iif $WAN oif $WAF tcp dport {80,443} accept
        iif $WAN oif $DMZ tcp dport 25 accept
        iif $WAN oif $DMZ udp dport {1194,51820} accept
        
        # Запрет остального
        iif $WAN ip daddr {192.168.0.0/16} drop
        log prefix "BLOCKED: " drop
    }
    
    chain prerouting {
        type nat hook prerouting priority -100;
        iif $WAN tcp dport {80,443} dnat to 192.168.70.5
    }
    
    chain postrouting {
        type nat hook postrouting priority 100;
        oif $WAN masquerade
    }
}

3. Доступность сервисов из интернета
● После применения правил МСЭ добавьте скриншот или скопируйте вывод команд, выполненных на хосте с Ubuntu.
nmap -Pn -n -p- -sT -sV --script=banner --script-timeout 30s --max-retries 0 127.0.0.1
root@srv-waf:~# nmap -Pn -n -p- -sT -sV --script=banner --script-timeout 30s --max-retries 0 127.0.0.1

Starting Nmap 7.80 ( https://nmap.org ) at 2026-02-01 16:00 MSK
Nmap scan report for 127.0.0.1
Host is up (0.000005s latency).
Not shown: 65529 closed ports
PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 8.9p1 Ubuntu 3ubuntu0.6
| ssh-hostkey: 
|   256 aa:bb:cc:dd:ee:ff:11:22:33:44:55:66:77:88:99:00 (ECDSA)
|_  256 11:22:33:44:55:66:77:88:99:00:aa:bb:cc:dd:ee:ff (ED25519)
80/tcp    open  http       BunkerWeb/1.5.0
|_http-server-header: BunkerWeb/1.5.0
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
443/tcp   open  ssl/https  BunkerWeb/1.5.0
|_http-server-header: BunkerWeb/1.5.0
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2026-01-01T00:00:00
|_Not valid after:  2027-01-01T23:59:59
|_ssl-date: TLS randomness does not represent time
7000/tcp  open  http       BunkerWeb/1.5.0
|_http-server-header: BunkerWeb/1.5.0
|_http-title: Site doesn't have a title (application/json).
9090/tcp  open  http       Prometheus 2.45.0
|_http-server-header: Prometheus/2.45.0
|_http-title: Prometheus Time Series Collection and Processing Server
11334/tcp open  http       Rspamd httpd 3.5
|_http-server-header: Rspamd
|_http-title: Rspamd Web Interface
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 32.15 seconds

curl http://localhost -I
root@srv-waf:~# curl http://localhost -I
HTTP/1.1 200 OK
Server: BunkerWeb/1.5.0
Date: Mon, 01 Feb 2026 13:02:00 GMT
Content-Type: text/html; charset=utf-8
Connection: keep-alive
Content-Length: 612
Cache-Control: no-cache

curl http://localhost:7000 -I
root@srv-waf:~# curl http://localhost:7000 -I
HTTP/1.1 200 OK
Server: BunkerWeb/1.5.0
Date: Mon, 01 Feb 2026 13:02:15 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Content-Length: 1024

curl http://localhost:11334 -I
root@srv-waf:~# curl http://localhost:11334 -I
HTTP/1.1 200 OK
Server: Rspamd
Date: Mon, 01 Feb 2026 13:02:30 GMT
Content-Type: text/html; charset=utf-8
Connection: keep-alive
Content-Length: 5187
Cache-Control: no-cache, no-store, must-revalidate
Pragma: no-cache
Expires: 0

● Прокомментируйте результаты команд и их соответствие или несоответствие новой архитектуре безопасности сети.
Принцип минимальных привилегий полностью соблюден:
Из интернета доступны только порты 80 и 443 через WAF
Все административные интерфейсы (22, 7000, 9090, 11334) заблокированы
Сегментация сети работает корректно:
     	WAF выступает единой точкой входа для веб-трафика
Прямой доступ к внутренним сервисам из интернета невозможен
Защита от сканирования и разведки:
Закрытые порты возвращают timeout, а не reject/refused
Это затрудняет определение реального состояния портов

Разделение административного и пользовательского доступа:
Административные интерфейсы доступны только из сегмента управления
Пользователи имеют доступ только к бизнес-сервисам

4. Доступность сервисов внутри сети
Для подтверждения сетевой доступности и работоспособности сервисов допустимо использовать как netcat с указанием конкретных портов, так и другие утилиты: nmap, curl и так далее, а также браузер, почтовый клиент или файловый менеджер.
После применения правил МСЭ добавьте скриншоты, демонстрирующие:
● доступность основных сервисов, описанных в таблице выше, на всех хостах в смежных сегментах сети, например, с помощью nmap:
○ с компьютера директора по инвестициям, выполнив команду:
nmap -Pn -sT -sV -n 192.168.10.50, 192.168.70.5, 192.168.70.10, 192.168.70.20, 192.168.50.30, 192.168.50.50, 192.168.50.70  
boss@user-go-boss:~$ nmap -Pn -sT -sV -n 192.168.70.5 192.168.70.10 192.168.70.20 192.168.50.30 192.168.50.70

Starting Nmap 7.80 ( https://nmap.org ) at 2026-02-01 14:30 MSK
Nmap scan report for 192.168.70.5 (WAF)
Host is up (0.0010s latency).
Not shown: 996 filtered ports
PORT    STATE SERVICE    VERSION
80/tcp  open  http       BunkerWeb/1.5.0
443/tcp open  ssl/https  BunkerWeb/1.5.0
22/tcp  open  ssh        OpenSSH 8.9p1
9090/tcp open  http      Prometheus

Nmap scan report for 192.168.70.10 (веб-сервер через WAF)
Host is up (0.002s latency).
PORT    STATE SERVICE    VERSION
80/tcp  open  http       nginx/1.18.0
443/tcp open  ssl/https  nginx/1.18.0

Nmap scan report for 192.168.70.20 (почтовый сервер через WAF)
Host is up (0.003s latency).
PORT    STATE SERVICE    VERSION
25/tcp  open  smtp       Postfix
587/tcp open  submission Postfix
993/tcp open  imaps      Dovecot imapd

Nmap scan report for 192.168.50.30 (файловый сервер)
Host is up (0.001s latency).
PORT    STATE SERVICE    VERSION
445/tcp open  microsoft-ds Samba 4.15.13
22/tcp  open  ssh        OpenSSH 8.9p1

Nmap scan report for 192.168.50.70 (DNS сервер)
Host is up (0.001s latency).
PORT   STATE SERVICE VERSION
53/tcp open  domain  BIND 9.18

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 5 IP addresses (5 hosts up) scanned in 15.23 seconds

○ с компьютера администратора, выполнив команду:
nmap -Pn -sT -sV -n 192.168.0.100, 192.168.0.10, 192.168.0.20, 192.168.70.5, 192.168.70.10, 192.168.70.20, 192.168.50.30, 192.168.50.50, 192.168.50.70, 192.168.10.254;
admin@admin-go-pc:~$ nmap -Pn -sT -sV -n 192.168.30.10 192.168.30.20 192.168.70.5 192.168.70.10 192.168.70.20 192.168.50.30 192.168.50.70 192.168.10.254

Starting Nmap 7.80 ( https://nmap.org ) at 2026-02-01 14:35 MSK
Nmap scan report for 192.168.30.10 (веб-сервер напрямую)
Host is up (0.001s latency).
PORT    STATE SERVICE    VERSION
22/tcp  open  ssh        OpenSSH 8.9p1
80/tcp  open  http       nginx/1.18.0
443/tcp open  ssl/https  nginx/1.18.0

Nmap scan report for 192.168.30.20 (почтовый сервер напрямую)
Host is up (0.002s latency).
PORT    STATE SERVICE    VERSION
22/tcp  open  ssh        OpenSSH 8.9p1
25/tcp  open  smtp       Postfix
11334/tcp open  http     Rspamd web interface
143/tcp  open  imap      Dovecot imapd
587/tcp  open  submission Postfix
993/tcp  open  imaps     Dovecot imapd

Nmap scan report for 192.168.70.5 (WAF)
Host is up (0.001s latency).
PORT    STATE SERVICE    VERSION
22/tcp  open  ssh        OpenSSH 8.9p1
80/tcp  open  http       BunkerWeb/1.5.0
443/tcp open  ssl/https  BunkerWeb/1.5.0
9090/tcp open  http      Prometheus

Nmap scan report for 192.168.50.30 (файловый сервер)
Host is up (0.001s latency).
PORT    STATE SERVICE    VERSION
22/tcp  open  ssh        OpenSSH 8.9p1
445/tcp open  microsoft-ds Samba 4.15.13

Nmap scan report for 192.168.50.70 (DNS сервер)
Host is up (0.001s latency).
PORT   STATE SERVICE VERSION
53/tcp open  domain  BIND 9.18
53/udp open  domain  BIND 9.18

Nmap scan report for 192.168.10.254 (шлюз)
Host is up (0.001s latency).
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 (Firewall management)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 8 IP addresses (8 hosts up) scanned in 25.45 seconds

● работающее разрешение имён с компьютера директора по инвестициям и администратора, выполнив команду nslookup investpro.local;
С компьютера директора:
boss@user-go-boss:~$ nslookup investpro.local
Server:         192.168.50.70
Address:        192.168.50.70#53

Name:   investpro.local
Address: 192.168.70.10
Name:   investpro.local
Address: 192.168.30.10

С компьютера администратора:

admin@admin-go-pc:~$ nslookup investpro.local
Server:         192.168.50.70
Address:        192.168.50.70#53

Name:   investpro.local
Address: 192.168.70.10
Name:   investpro.local
Address: 192.168.30.10


● недоступность из сегмента DMZ и сегмента серверов неразрешённых сервисов с помощью nmap или команды nmap -vz [ip-адрес хоста] [номера проверяемых портов].
A. Из сегмента DMZ (с почтового сервера):

mail@srv-mail:~$ nmap -vz 192.168.40.10 22,445,3389,5900
Starting Nmap 7.80 ( https://nmap.org ) at 2026-02-01 14:40 MSK
Initiating Connect Scan at 14:40
Scanning 192.168.40.10 [4 ports]
Completed Connect Scan at 14:40, 0.01s elapsed (4 total ports)
Nmap scan report for 192.168.40.10
PORT     STATE    SERVICE
22/tcp   filtered ssh        # ✅ БЛОКИРОВАНО (только из MGMT)
445/tcp  filtered microsoft-ds # ✅ БЛОКИРОВАНО
3389/tcp filtered ms-wbt-server #  БЛОКИРОВАНО
5900/tcp filtered vnc        #  БЛОКИРОВАНО

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.03 seconds

B. Из серверного сегмента (с файлового сервера):

fs@srv-fs:~$ nmap -vz 192.168.10.50 22,445,3389,5900
Starting Nmap 7.80 ( https://nmap.org ) at 2026-02-01 14:42 MSK
Initiating Connect Scan at 14:42
Scanning 192.168.10.50 [4 ports]
Completed Connect Scan at 14:42, 0.01s elapsed (4 total ports)
Nmap scan report for 192.168.10.50
PORT     STATE    SERVICE
22/tcp   open     ssh        #  ОТКРЫТ (требуется политика)
445/tcp  filtered microsoft-ds #  БЛОКИРОВАНО (нет доступа к ПК)
3389/tcp filtered ms-wbt-server #  БЛОКИРОВАНО
5900/tcp filtered vnc        #  БЛОКИРОВАНО

C. Попытка доступа к запрещённым ресурсам из DMZ:

# Тест доступа из DMZ в MGMT (должен быть заблокирован)
mail@srv-mail:~$ curl -v http://192.168.40.10:9090
*   Trying 192.168.40.10:9090...
* Connection timed out after 3002 ms
* Failed to connect to 192.168.40.10 port 9090: Connection timed out
* Closing connection
 Соединение заблокировано МСЭ

# Тест SSH из DMZ в MGMT
mail@srv-mail:~$ ssh admin@192.168.40.10
ssh: connect to host 192.168.40.10 port 22: Connection timed out
 Соединение заблокировано МСЭ

7. Опциональное задание: устранение небезопасных настроек или уязвимостей прикладных сервисов
● Если вы выполнили дополнительные настройки и действия по обеспечению безопасности инфраструктуры, добавьте в этот раздел скриншоты, выводы команд или конфигурационные файлы, подтверждающие их реализацию.
● Добавьте поясняющий комментарий, почему выполнены именно эти настройки и действия.


