# 1488DEMO
Это удобный гайд для настройки сети Linux витруальных машин


Руководство по настройке сетевой инфраструктуры (Модуль 1 и Модуль 2) 

Модуль 1: Базовая настройка 

1. Произведите базовую настройку устройств 

Зададим имена устройствам полным доменным именем:

* 
**На ISP**:


```bash
hostnamectl set-hostname isp.au-team.irpo; exec bash

```


* 
**На HQ-RTR**:


```bash
hostnamectl set-hostname hq-rtr.au-team.irpo; exec bash

```


* 
**На BR-RTR**:


```bash
hostnamectl set-hostname br-rtr.au-team.irpo; exec bash

```


* 
**На HQ-SRV**:


```bash
hostnamectl set-hostname hq-srv.au-team.irpo; exec bash

```


* 
**На HQ-CLI**:


```bash
hostnamectl set-hostname hq-cli.au-team.irpo; exec bash

```


* 
**На BR-SRV**:


```bash
hostnamectl set-hostname br-srv.au-team.irpo; exec bash

```



Редактируем репозиторий через редактор `nano`:

```bash
nano /etc/apt/sources.list

```

Ставим комментарий на первой строчке символом `#`:

```text
#deb cdrom:[Debian GNU/Linux 13.1.0 ...

```



---

2. Доступ к сети Интернет (ISP) и настройка VLAN (HQ) 

Настройка ISP 

Проведем базовую настройку сети через конфигурационный файл `/etc/network/interfaces`:

```bash
nano /etc/network/interfaces

```

```text
auto lo
iface lo inet loopback

auto ens3
iface ens3 inet dhcp

auto ens4
iface ens4 inet static
address 172.16.1.1/28

auto ens5
iface ens5 inet static
address 172.16.2.1/28

post-up nft -f /etc/nftables.conf

```

Выходим с файла: `Ctrl+X`, `y`, `Enter`.

Редактируем `sysctl.conf` для включения маршрутизации:

```bash
nano /etc/sysctl.d/sysctl.conf

```

```text
net.ipv4.ip_forward=1

```

Применяем правила `sysctl`:

```bash
sysctl --system

```

Делаем маскарадинг в `nftables`:

```bash
nano /etc/nftables.conf

```



Перезагружаем службу сети:

```bash
systemctl restart networking

```

Проверяем IP-адреса:

```bash
ip -br a

```



> В случае, если какие-то IP-адреса пропадают — перезагружаем службы `NetworkManager` и `networking` по очереди, либо полностью перезагружаем устройство (`reboot`).
> 
> 

Настройка HQ-RTR 

Настроим сразу VLAN из задания 4.
Сначала скачиваем пакет vlan (после появления доступа в интернет):

```bash
apt install vlan

```

> Если пакет не скачивается, проверяем `/etc/resolv.conf`, он должен иметь вид `nameserver 8.8.8.8`.
> 
> 
> 

Редактируем интерфейсы:

```bash
nano /etc/network/interfaces

```

Приводим конфиг к такому виду:

```text
auto lo
iface lo inet loopback

auto ens3
iface ens3 inet static
address 172.16.1.2/28
gateway 172.16.1.1

auto ens4.100
iface ens4.100 inet static
address 192.168.100.1/27
vlan_raw_device ens4

auto ens5.200
iface ens5.200 inet static
address 192.168.100.33/28
vlan_raw_device ens5

auto ens6.999
iface ens6.999 inet static
address 192.168.100.49/29
vlan_raw_device ens6

post-up nft -f /etc/nftables.conf

```

Далее включаем форвардинг в `sysctl` и настраиваем маскарадинг в `nftables`:

```bash
nano /etc/sysctl.d/sysctl.conf
# Добавить: net.ipv4.ip_forward=1
sysctl --system
nano /etc/nftables.conf

```



Выполняем перезагрузку службы и проверяем IP-адреса:

```bash
systemctl restart networking
ip -br a

```

Проверяем пинги до ISP, обратно и в Интернет:

```bash
ping 172.16.1.1
ping ya.ru

```



#### Настройка остальных узлов

**На BR-RTR**:

```bash
nano /etc/network/interfaces

```

Включаем `ip_forward`, настраиваем `nftables`, перезагружаем сеть и проверяем IP.

```bash
nano /etc/sysctl.d/sysctl.conf
sysctl --system
nano /etc/nftables.conf
systemctl restart networking
ip -br a

```

**На HQ-SRV**:

```bash
nano /etc/network/interfaces

```



```bash
systemctl restart networking
ip -br a

```

> Если возникает ошибка, используем `reboot`.
> 
> 

**На HQ-CLI (временно статика)**:

```bash
nano /etc/network/interfaces

```



```bash
systemctl restart networking
ip -br a

```



**На BR-SRV**:

```bash
nano /etc/network/interfaces

```



```bash
systemctl restart networking
ip -br a

```



---

3. Локальные учетные записи на серверах 

**Пользователь `sshuser` на HQ-SRV и BR-SRV**:
Создаем пользователя и даем привилегии `sudo`:

```bash
useradd -m -s /bin/bash sshuser -u 2026 -U
usermod -aG sudo sshuser

```

Ставим пароль `P@$$word`:

```bash
passwd sshuser

```

Настраиваем беспарольный `sudo` в `visudo` на обоих серверах:

```bash
visudo

```

ОБЯЗАТЕЛЬНО в конце файла добавляем строчку:

```text
sshuser ALL=(ALL:ALL) NOPASSWD:ALL

```

Проверяем вход без пароля:

```bash
su - sshuser
sudo -i

```

**Пользователь `net_admin` на HQ-RTR и BR-RTR**:
Аналогично создаем `net_admin` с паролем `P@ssw0rd` и добавляем беспарольный `sudo`:

```bash
useradd -m -s /bin/bash net_admin -U
usermod -aG sudo net_admin
passwd net_admin
visudo

```

В конце файла добавляем:

```text
net_admin ALL=(ALL:ALL) NOPASSWD:ALL

```

Проверяем работу пользователя.

---

4. Безопасный удаленный доступ (SSH) на HQ-SRV и BR-SRV 

Устанавливаем SSH-сервер:

```bash
apt install openssh-server

```

Редактируем конфигурацию:

```bash
nano /etc/ssh/sshd_config

```

Раскомментируем и изменим параметры: порт 2026, разрешаем только `sshuser`, ограничиваем попытки, задаем баннер:

```text
Port 2026
AllowUsers sshuser
MaxAuthTries 2
Banner /etc/ssh_banner

```

Создадим баннер `/etc/ssh_banner`:

```bash
nano /etc/ssh_banner

```

Вписываем текст в рамке из звездочек:

```text
*********************************
* *
* Authorized access only    *
* *
*********************************

```

Перезагружаем SSH на серверах:

```bash
systemctl restart ssh

```

Проверяем подключение с HQ-CLI:

```bash
ssh sshuser@192.168.100.2 -p 2026

```



---

5. IP-туннель (GRE) между HQ и BR 

**На HQ-RTR**:
Заходим в `nmtui`:

```bash
nmtui

```

Выбираем `Новое подключение` -> `Туннель IP` и настраиваем параметры GRE туннеля (Локальный и Удаленный IP, конфигурация IPv4).
В настройках `Маршрутизация` добавляем маршрут к `ens3` BR-SRV.


В конце файла `/etc/network/interfaces` добавляем строчку для поднятия интерфейса GRE.
Перезапускаем службу сети и проверяем `tun1`.
Изменяем TTL интерфейса GRE на 64.

**На BR-RTR**:
Производим аналогичную настройку в `nmtui` (создаем туннель, прописываем маршруты). Добавляем интерфейс GRE в `interfaces`, перезапускаем сеть, проверяем `tun1` и изменяем TTL на 64.
Проверяем доступность туннеля и сетей пингами.

---

6. Настройка DHCP для сети HQ-CLI 

Установим DHCP-сервер на HQ-RTR:

```bash
apt install isc-dhcp-server -y
nano /etc/default/isc-dhcp-server

```

Вписываем в `INTERFACESv4` значение `ens5.200`.
Настроим сам DHCP:

```bash
nano /etc/dhcp/dhcpd.conf

```

Добавляем в конец настройку пула (время аренды увеличиваем, чтобы IP не менялся).


```bash
systemctl restart isc-dhcp-server

```

**На HQ-CLI** изменяем `interfaces` на получение IP по DHCP:

```bash
nano /etc/network/interfaces

```

```text
auto ens3.200
iface ens3.200 inet dhcp
vlan_raw_device ens3

```

Удаляем старое подключение `Wired connection 1` в `nmtui`.
Перезапускаем сеть и проверяем IP:

```bash
systemctl restart networking
ip -br a

```

---

7. DNS-инфраструктура (dnsmasq) 

Устанавливаем и настраиваем `dnsmasq`:

```bash
apt install dnsmasq -y
nano /etc/dnsmasq.conf

```



Прописываем локальный DNS:

```bash
nano /etc/resolv.conf
# Добавить: nameserver 127.0.0.1

```

Перезапускаем службу и проверяем:

```bash
systemctl restart dnsmasq
ping ya.ru
ping br-rtr.au-team.irpo

```

---

8. Настройка часового пояса 

Повторяем на всех устройствах (выберем Красноярск):

```bash
timedatectl list-timezones
timedatectl set-timezone Asia/Krasnoyarsk

```

---

Модуль 2 

1. Настройка контроллера домена Samba DC (BR-SRV) 

**Внимание:** В DHCP HQ-RTR нужно поменять IP DNS сервера на BR-SRV:

```bash
nano /etc/dhcp/dhcpd.conf
# option domain-name-server 192.168.200.2;
systemctl restart isc-dhcp-server

```



**Установка Samba на BR-SRV**:

```bash
apt update
apt install -y samba smbclient winbind libnss-winbind krb5-user net-tools

```

В синем окне настройки вводим `AU-TEAM.IRPO`, `br-srv.au-team.irpo`.
Переименовываем конфиг, останавливаем службы и создаем домен:

```bash
mv /etc/samba/smb.conf /etc/samba/smb.conf.bak
systemctl stop smbd nmbd winbind
systemctl stop samba-ad-dc
samba-tool domain provision --use-rfc2307 --interactive

```

Везде жмем `ENTER` до `Administrator`, пароль вводим `P@ssw0rd`.
Запускаем службы и копируем krb5.conf:

```bash
systemctl start smbd nmbd winbind
systemctl start samba-ad-dc
cp /var/lib/samba/private/krb5.conf /etc/krb5.conf

```

Создаем пользователей в домене.


**Ввод HQ-CLI в домен**:

```bash
apt update && apt install -y realmd sssd sssd-tools libnss-sss libpam-sss adcli packagekit
nano /etc/resolv.conf # Вписать nameserver 192.168.200.2 и удалить старый
realm join -v au-team.irpo # Ввести пароль P@ssw0rd
pam-auth-update --enable mkhomedir
realm deny --all
realm permit -g hq@au-team.irpo

```

Выдаем права и проверяем:

```bash
nano /etc/sudoers.d/hq-users
# Вписать: %hq@au-team.irpo ALL=(ALL) /usr/bin/cat, /usr/bin/grep, /usr/bin/id
su - hquser1@au-team.irpo
sudo id

```

---

2. Служба сетевого времени Chrony 

**ISP**:

```bash
apt install chrony -y
nano /etc/chrony/chrony.conf

```

Комментируем строки `pool` и добавляем:

```text
server ntp1.vniiftri.ru iburst prefer
local stratum 5
allow 0.0.0.0/0

```

```bash
systemctl restart chronyd

```



**Офисы HQ и BR (все остальные устройства)**:
Устанавливаем chrony и настраиваем синхронизацию с локальными маршрутизаторами:

```bash
apt install chrony -y
nano /etc/chrony/chrony.conf
# Комментируем pool

```

Для офиса HQ сервер — `172.16.1.1 iburst`, для BR — `172.16.2.1 iburst`.

```bash
systemctl restart chronyd

```



---

3. Конфигурация Ansible (BR-SRV) 

На **BR-SRV**:

```bash
apt install ansible sshpass -y
mkdir -p /etc/ansible
nano /etc/ansible/ansible.cfg

```

Вписываем:

```text
[defaults]
host_key_checking=False

```

Редактируем инвентарь `nano /etc/ansible/hosts`.


На **HQ-RTR, BR-RTR и HQ-CLI** устанавливаем `openssh-server`, создаем пользователя `sshuser` (если не создан) и выдаем права.
Проверка на BR-SRV:

```bash
ansible all -m ping

```

---

4. Веб-приложение в Docker (BR-SRV) 

На **BR-SRV**:

```bash
apt install docker.io docker-compose -y

```

Извлекаем образы из `Additional.iso` в `/root`.

```bash
docker image load -i /root/Additional/docker/site_latest.tar
docker image load -i /root/Additional/docker/mariadb_latest.tar
docker images
mkdir testapp
cd testapp/
nano docker-compose.yaml

```

Поднимаем контейнеры и проверяем доступ с HQ-CLI `http://192.168.200.2:8080`:

```bash
docker-compose -f docker-compose.yaml up -d
docker ps

```

---

5. Веб-приложение на HQ-SRV (Apache) 

На **HQ-SRV** устанавливаем стек:

```bash
apt update
apt install apache2 mariadb-server mariadb-client php8.4 php8.4-mysqli -y

```

Копируем дамп из извлеченной папки:

```bash
cp /root/Additional/web/dump.sql /tmp
mysql -u root

```

В MariaDB выполняем:

```sql
CREATE DATABASE webdb;
CREATE USER 'web'@'localhost' IDENTIFIED BY 'P@ssw0rd';
GRANT ALL PRIVILEGES ON webdb.* TO 'web'@'localhost';
FLUSH PRIVILEGES;
EXIT;

```

Импортируем БД и настраиваем директорию сайта:

```bash
mysql -u root webdb < /tmp/dump.sql
mkdir -p /var/www/html
cp /root/Additional/web/index.php /var/www/html
chown -R www-data:www-data /var/www/html
chmod -R 755 /var/www/html
nano /var/www/html/index.php
rm -f /var/www/html/index.html
systemctl restart apache2

```



Указываем DNS HQ-SRV в DHCP HQ-RTR и проверяем доступ к сайту.

---

6. NAT (Трансляция портов) 

Меняем порт Apache на **HQ-SRV**:

```bash
nano /etc/apache2/sites-available/000-default.conf # Сменить на <VirtualHost *:8080>
nano /etc/apache2/ports.conf # Сменить на Listen 8080
systemctl restart apache2

```



Вносим правила `dnat` в `nftables` на **HQ-RTR** и **BR-RTR**, перезагружаем сеть.


Проверяем с **ISP**:

```bash
ssh sshuser@172.16.1.2 -p 2026
ssh sshuser@172.16.2.2 -p 2026

```

---

7. Nginx Обратный прокси (ISP) 

Устанавливаем Nginx на **ISP**:

```bash
apt install nginx apache2-utils -y
nano /etc/nginx/sites-available/proxy

```



```bash
ln -s /etc/nginx/sites-available/proxy /etc/nginx/sites-enabled
nano /etc/nginx/nginx.conf
# Раскомментируем: server_names_hash_bucket_size 64;

```



Прописываем хосты и перезапускаем nginx:

```bash
nano /etc/hosts
# 172.16.1.1 web.au-team.irpo
# 172.16.2.1 docker.au-team.irpo
systemctl restart nginx

```

Проверяем доступ по URL.

---

8. Web-based аутентификация 

Создаем файл паролей на **ISP**:

```bash
htpasswd -bc /etc/nginx/.htpasswd WEB P@ssw0rd

```

Добавляем правила в конфигурацию `proxy` и перезапускаем nginx:

```bash
nano /etc/nginx/sites-enabled/proxy
systemctl restart nginx

```

При доступе к сайту вводим пользователя `WEB` и пароль `P@ssw0rd`.

---

9. Установка Яндекс Браузера (HQ-CLI) 

На **HQ-CLI** скачиваем `Yandex.deb` и устанавливаем:

```bash
apt install ./Yandex.deb -y

```

Запускаем с параметром `no-sandbox`:

```bash
yandex-browser-stable --no-sandbox

```
