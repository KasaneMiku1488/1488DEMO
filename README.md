# 1488DEMO
Это удобный гайд для настройки сети Linux витруальных машин


Руководство по настройке сетевой инфраструктуры (Модуль 1 и Модуль 2) 

Модуль 1: Базовая настройка 

1. Произведите базовую настройку устройств 
<img width="489" height="612" alt="изображение" src="https://github.com/user-attachments/assets/6e72e7e2-ba97-45ef-a788-e14ade2b0ae5" />

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
<img width="837" height="416" alt="изображение" src="https://github.com/user-attachments/assets/e02eb8cc-97ac-4925-946f-de1a8080ce15" />

```text
#deb cdrom:[Debian GNU/Linux 13.1.0 ...

```
СДЕЛАТЬ НА ISP, HQ-RTR И BR-RTR СТРОГО, ИНАЧЕ ИНЕТ НЕ БУДЕТ РАБОТАТЬ
Редактируем sysctl.conf  
```bash
nano /etc/sysctl.d/sysctl.conf
```
```bash
net.ipv4.ip_farward=1
```
Выходим с файла: Ctrl+X, y, Enter

Применяем правила sysctl
```bash
sysctl --system
```
---

Сделаем динамическую трансляцию адресов (проделать также на HQ- RTR, BR-RTR!)
```bash
nano /etc/nftables.conf
```
Приводим файл к такому виду (делаем маскарадинг)
<img width="975" height="625" alt="изображение" src="https://github.com/user-attachments/assets/ff3ac892-56fd-4bc8-b943-95b7a607fb08" />
```bash
table ip nat {
         chain postrouting {
               type nat hook postrouting priority 100; policy accept
               meta l4proto { gre, ipip, ospf } counter return
               masquerade
         }
}
```
ОБЯЗАТЕЛЬНО ПРОВЕРИТЬ ПРАВИЛЬНОСТЬ НАПИСАНИЯ!!! ОДНА ОШИБКА И ТЫ ОШИБСЯ 
Перезагружаем	службу	сети	(делать	рекомендую	почаще	на	всех устройствах, если какие-то проблемы)
```bash
systemctl restart networking 
```
В случае, если какие-то IP-адреса пропадают — перезагружаем службы NetworkManager и networking по очереди:
```bash
systemctl restart NetworkManager 
```
```bash
systemctl restart networking 
```
---

2. Доступ к сети Интернет (ISP)  

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



Проверяем IP-адреса:

```bash
ip -br a

```
<img width="977" height="126" alt="изображение" src="https://github.com/user-attachments/assets/718a57ec-c6ca-4dca-a88e-83681bb4f064" />



> В случае, если какие-то IP-адреса пропадают — перезагружаем службы `NetworkManager` и `networking` по очереди, либо полностью перезагружаем устройство (`reboot`).
> 
> 

### Редактирование `/etc/network/interfaces` на HQ-RTR

Открываем файл:

```bash
nano /etc/network/interfaces
```

Приводим к следующему виду:

```
source /etc/network/interfaces.d/*

# Loopback
auto lo
iface lo inet loopback

auto ens3
iface ens3 inet static
  address 172.16.1.2/28
  gateway 172.16.1.1

post-up nft -f /etc/nftables.conf
```

Выходим: `Ctrl+X`, `y`, `Enter`.

### Перезагружаем сеть и проверяем

```bash
systemctl restart networking
```

```bash
ip -br a
```
<img width="975" height="204" alt="изображение" src="https://github.com/user-attachments/assets/5eb5c058-f737-45cc-8525-9775d3892786" />

Попробуем пингануть с ISP HQ-RTR
<img width="974" height="259" alt="изображение" src="https://github.com/user-attachments/assets/198a06dd-caf1-43d1-858e-f309026fae50" />
А также в обратную сторону
<img width="977" height="298" alt="изображение" src="https://github.com/user-attachments/assets/ee2a37dc-2acc-4d3d-97e8-b96cc86861f3" />
Проверим также и Интернет
<img width="954" height="278" alt="изображение" src="https://github.com/user-attachments/assets/4eb74038-eca3-4257-8ddc-4c60249a3b1e" />

На BR-RTR:
```bash
nano /etc/network/interfaces
```
<img width="748" height="805" alt="изображение" src="https://github.com/user-attachments/assets/916dbb03-bb1c-48df-b68c-872d1b9018ab" />

```bash
auto ens3
iface ens3 inet static
address 172.16.2.2/28
gateway 172.16.2.1

auto ens4
iface ens4 inet static
address 192.168.200.1/28

post-up nft -f /etc/nftables.conf
```

Перезагружаем службу сети
```bash
systemctl restart networking
```
Проверяем IP-адреса
<img width="936" height="102" alt="изображение" src="https://github.com/user-attachments/assets/6e642b97-6fbe-435f-9413-3669c4d968d2" />

На HQ-SRV:
```bash
nano /etc/network/interfaces
```
<img width="691" height="810" alt="изображение" src="https://github.com/user-attachments/assets/f32a41cf-e5fb-409c-9ee6-da27bdde659e" />
```bash
auto ens3
iface ens3 inet static
address 192.168.100.2/27
gateway 192.168.100.1
```
Перезагружаем службу сети
```bash
systemctl restart networking
```
Проверяем IP-адреса
<img width="952" height="72" alt="изображение" src="https://github.com/user-attachments/assets/716c5672-d1c8-4ceb-861b-0621dc19af69" />

На HQ-CLI (временно, до 9 задания, позже он IP будет получать по DHCP):
```bash
nano /etc/network/interfaces
```
<img width="700" height="814" alt="изображение" src="https://github.com/user-attachments/assets/96a3b062-b789-4c1e-bffb-4cd64a46376d" />
```bash
auto ens3
iface ens3 inet static
address 192.168.100.34/28
gateway 192.168.100.33
```
```bash
systemctl restart networking
```
Проверяем IP-адреса
<img width="954" height="74" alt="изображение" src="https://github.com/user-attachments/assets/daa1caf0-91d8-496e-af4a-3720077e2971" />

На BR-SRV:
```bash
nano /etc/network/interfaces
```
<img width="713" height="804" alt="изображение" src="https://github.com/user-attachments/assets/9a865065-db77-4a54-8f81-154c3d6f23e0" />
```bash
auto ens3
iface ens3 inet static
address 192.168.200.2/28
gateway 192.168.200.1
```
```bash
systemctl restart networking
```
Проверяем IP-адреса
<img width="947" height="121" alt="изображение" src="https://github.com/user-attachments/assets/3012dc65-c12f-47e3-9651-119018cb727a" />
Рекомендую повторно проверить везде IP-адреса. Если что-то отвалилось:
```bash
systemctl restart networking
```
---

## Задание 3 — Создайте локальные учетные записи на серверах HQ-SRV и BR-SRV:

Создаем пользователей sshuser на HQ-SRV и BR-SRV
```bash
useradd -m -s /bin/bash sshuser -u 2026 -U 
```
Даем привилегии sudo
```bash
usermod -aG sudo sshuser 
```
Ставим пароль P@$$word на пользователя sshuser
<img width="569" height="132" alt="изображение" src="https://github.com/user-attachments/assets/fd7565e6-a3c2-4990-8f95-95fff57e2be4" />
Заходим в visudo
И на HQ-SRV и на BR-SRV ОБЯЗАТЕЛЬНО в конце файла visudo
добавляем строчку
```bash
sshuser         ALL=(ALL:ALL)         NOPASSWD:ALL
```
После слова sshuser нажимаем TAB, пишем ALL=(ALL:ALL), затем жмем пробел и пишем NOPASSWD:ALL
<img width="903" height="821" alt="изображение" src="https://github.com/user-attachments/assets/9af306ec-9703-428a-bf40-46f7c708ac3c" />

Проверяем наших пользователей sshuser
```bash
su - sshuser 
```
Обратим внимание, что при логине пароль не требуется
<img width="408" height="42" alt="изображение" src="https://github.com/user-attachments/assets/d66630ff-09d2-4749-ac91-46d33a52cc25" />
Повышаем права через sudo –i, видим что также все работает без пароля
<img width="313" height="90" alt="изображение" src="https://github.com/user-attachments/assets/d2c2ea62-0023-454e-bf86-ccad403ba1e7" />

Создаем пользователей net_admin на HQ-RTR и BR-RTR
```bash
useradd -m -s /bin/bash net_admin -U 
```
Даем привилегии sudo

```bash
usermod -aG sudo net_admin 
```
Ставим пароль P@ssw0rd на пользователя net_admin
<img width="792" height="173" alt="изображение" src="https://github.com/user-attachments/assets/55ca1ba4-5be5-4ead-b478-97d135a1b40c" />
Заходим в visudo
И на HQ-RTR и на BR-RTR ОБЯЗАТЕЛЬНО в конце файла visudo
добавляем строчку
```bash
sshuser         ALL=(ALL:ALL)         NOPASSWD:ALL
```
<img width="786" height="811" alt="изображение" src="https://github.com/user-attachments/assets/73ce3bf4-32f5-4517-8498-3301ffc2a00a" />
Также все работает без пароля
<img width="560" height="159" alt="изображение" src="https://github.com/user-attachments/assets/aeb4447c-a337-4c7f-8954-806e64789c91" />


---

## Задание 4 — Безопасный удалённый доступ (SSH)

**Требования:**
- Порт: **2026**
- Разрешить только пользователя `sshuser`
- Максимум **2** попытки входа
- Баннер: `Authorized access only`

### Установка SSH-сервера на HQ-SRV и BR-SRV

```bash
apt install openssh-server -y
```

### Редактирование конфигурации SSH

```bash
nano /etc/ssh/sshd_config
```

Изменяем / добавляем строки:

```
Port 2026
AllowUsers sshuser
MaxAuthTries 2
Banner /etc/ssh_banner
```

### Создание баннера

```bash
nano /etc/ssh_banner
```

Содержимое файла:

```
*******************************
*   Authorized access only    *
*******************************
```

Повторяем на BR-SRV.

### Перезапуск SSH

```bash
systemctl restart sshd
```

### Проверка подключения (с HQ-CLI на HQ-SRV)

```bash
ssh sshuser@192.168.100.2 -p 2026
```

<!-- Вставьте скриншот с отображением баннера при подключении -->

Баннер должен отобразиться корректно. Вводим пароль — соединение устанавливается.

---

## Задание 5 — GRE-туннель между HQ-RTR и BR-RTR

**Цель:** Связать офисы HQ и BR защищённым туннелем.

| Параметр | HQ-RTR | BR-RTR |
|---|---|---|
| Локальный IP (внешний) | `172.16.1.2` | `172.16.2.2` |
| Удалённый IP | `172.16.2.2` | `172.16.1.2` |
| IP туннеля | `10.10.0.1/30` | `10.10.0.2/30` |

### Настройка на HQ-RTR

Запускаем `nmtui`:

```bash
nmtui
```

В интерфейсе:
1. «Редактировать подключения» → «Добавить»
2. Тип: **Туннель IP**
3. Имя профиля: `gre`, Устройство: `tun1`
4. Режим: **GRE**
5. Родительский интерфейс: `ens3`
6. Локальный IP: `172.16.1.2`
7. Удалённый IP: `172.16.2.2`
8. Конфигурация IPv4 → Вручную: `10.10.0.1/30`

<!-- Вставьте скриншот nmtui с заполненными параметрами -->

### Добавляем строчку в `/etc/network/interfaces` на HQ-RTR

```bash
nano /etc/network/interfaces
```

В конце файла добавляем:

```
post-up ip link set gre0 up
```

### Перезапуск сети и проверка на HQ-RTR

```bash
systemctl restart networking
```

```bash
ip -br a
```

> Интерфейс `gre0` должен быть в статусе `UNKNOWN`, а `tun1` — получить IP `10.10.0.1/30`.

<!-- Вставьте скриншот вывода `ip -br a` -->

### Изменение TTL туннельного интерфейса GRE на HQ-RTR

```bash
nmcli connection modify gre ip-tunnel.ttl 64
```

---

### Настройка на BR-RTR

Запускаем `nmtui`:

```bash
nmtui
```

1. «Редактировать подключения» → «Добавить»
2. Тип: **Туннель IP**
3. Имя профиля: `gre`, Устройство: `tun1`
4. Режим: **GRE**
5. Родительский интерфейс: `ens3`
6. Локальный IP: `172.16.2.2`
7. Удалённый IP: `172.16.1.2`
8. Конфигурация IPv4 → Вручную: `10.10.0.2/30`

### Добавляем строчку в `/etc/network/interfaces` на BR-RTR

```bash
nano /etc/network/interfaces
```

В конце файла:

```
post-up ip link set gre0 up
```

### Перезапуск сети и проверка на BR-RTR

```bash
systemctl restart networking
```

```bash
ip -br a
```

> `gre0` — статус `UNKNOWN`, `tun1` — IP `10.10.0.2/30`.

<!-- Вставьте скриншот вывода `ip -br a` -->

### Изменение TTL на BR-RTR

```bash
nmcli connection modify gre ip-tunnel.ttl 64
```

> **Частая ошибка:** если появляется ошибка, проверьте в `nmtui` — после слова `gre` не должно быть пробела.

### Проверка связности туннеля

С BR-RTR пингуем HQ-RTR:

```bash
ping 10.10.0.1
```

<!-- Вставьте скриншот успешного пинга -->

С HQ-RTR пингуем BR-RTR:

```bash
ping 10.10.0.2
```

<!-- Вставьте скриншот успешного пинга -->

---

## Задание 6 — Динамическая маршрутизация (OSPF / FRR)

**Требования:**
- Протокол: OSPF (link-state)
- Только на интерфейсах туннеля (`tun1`)
- Маршрутизаторы обмениваются маршрутами только друг с другом
- Защита паролем

### Установка FRR на HQ-RTR и BR-RTR

```bash
apt install frr -y
```

---

### Настройка HQ-RTR

#### Шаг 1 — Включить OSPF в конфигурации FRR

```bash
nano /etc/frr/daemons
```

Находим строку `ospfd=no` и меняем на:

```
ospfd=yes
```

#### Шаг 2 — Перезапустить FRR

```bash
systemctl restart frr
```

#### Шаг 3 — Войти в CLI FRR

```bash
vtysh
```

#### Шаг 4 — Настройка OSPF

```
hq-rtr.au-team.irpo# conf t
```

```
hq-rtr.au-team.irpo(config)# router ospf
```

```
hq-rtr.au-team.irpo(config-router)# router-id 1.1.1.1
```

Объявляем локальные сети и туннель:

```
hq-rtr.au-team.irpo(config-router)# no passive-interface default
hq-rtr.au-team.irpo(config-router)# network 192.168.100.0/27 area 0
hq-rtr.au-team.irpo(config-router)# network 192.168.100.32/28 area 0
hq-rtr.au-team.irpo(config-router)# network 10.10.0.0/30 area 0
```

Включаем аутентификацию для зоны:

```
hq-rtr.au-team.irpo(config-router)# area 0 authentication
```

Переходим к настройке туннельного интерфейса:

```
hq-rtr.au-team.irpo(config-router)# int tun1
```

```
hq-rtr.au-team.irpo(config-if)# no ip ospf passive
```

```
hq-rtr.au-team.irpo(config-if)# no ip ospf network broadcast
```

Настраиваем аутентификацию:

```
hq-rtr.au-team.irpo(config-if)# ip ospf authentication
hq-rtr.au-team.irpo(config-if)# ip ospf authentication-key password
```

#### Шаг 5 — Сохранить конфигурацию

```
hq-rtr.au-team.irpo(config-if)# exit
hq-rtr.au-team.irpo(config)# exit
hq-rtr.au-team.irpo# wr
```

---

### Настройка BR-RTR

#### Шаг 1 — Включить OSPF

```bash
nano /etc/frr/daemons
```

```
ospfd=yes
```

#### Шаг 2 — Перезапустить FRR

```bash
systemctl restart frr
```

#### Шаг 3 — Войти в CLI FRR

```bash
vtysh
```

#### Шаг 4 — Настройка OSPF

```
br-rtr.au-team.irpo# conf t
```

```
br-rtr.au-team.irpo(config)# router ospf
```

```
br-rtr.au-team.irpo(config-router)# router-id 2.2.2.2
```

```
br-rtr.au-team.irpo(config-router)# no passive-interface default
br-rtr.au-team.irpo(config-router)# network 192.168.200.0/28 area 0
br-rtr.au-team.irpo(config-router)# network 10.10.0.0/30 area 0
```

```
br-rtr.au-team.irpo(config-router)# area 0 authentication
```

```
br-rtr.au-team.irpo(config-router)# int tun1
```

```
br-rtr.au-team.irpo(config-if)# no ip ospf passive
```

```
br-rtr.au-team.irpo(config-if)# no ip ospf network broadcast
```

```
br-rtr.au-team.irpo(config-if)# ip ospf authentication
br-rtr.au-team.irpo(config-if)# ip ospf authentication-key password
```

#### Шаг 5 — Сохранить конфигурацию

```
br-rtr.au-team.irpo(config-if)# exit
br-rtr.au-team.irpo(config)# exit
br-rtr.au-team.irpo# wr
```

---

### Перезагрузка и проверка

Перезагружаем оба маршрутизатора:

```bash
reboot
```

После загрузки входим в FRR и проверяем соседство:

```bash
vtysh
```

На HQ-RTR:

```
hq-rtr.au-team.irpo# show ip ospf neighbor
```

<!-- Вставьте скриншот — должен отображаться сосед 2.2.2.2 в состоянии Full -->

На BR-RTR:

```
br-rtr.au-team.irpo# show ip ospf neighbor
```

<!-- Вставьте скриншот — должен отображаться сосед 1.1.1.1 в состоянии Full -->

Проверяем связность — пингуем BR-SRV с HQ-SRV:

```bash
ping 192.168.200.2
```

<!-- Вставьте скриншот успешного пинга -->

> **Если серверы не видят друг друга**, выполните на обоих маршрутизаторах:
> ```bash
> systemctl restart frr
> ```

---

## Задание 9 — DHCP-сервер на HQ-RTR

**Требования:**
- Подсеть VLAN 200 (`192.168.100.32/28`)
- Сервер DHCP — HQ-RTR
- Клиент — HQ-CLI
- Шлюз: `192.168.100.33`
- DNS-сервер: `192.168.100.2` (HQ-SRV)
- DNS-суффикс: `au-team.irpo`
- Адрес маршрутизатора исключить из выдачи

### Установка DHCP-сервера на HQ-RTR

```bash
apt install isc-dhcp-server -y
```

### Указываем интерфейс для DHCP

```bash
nano /etc/default/isc-dhcp-server
```

Находим строку `INTERFACESv4=""` и меняем на:

```
INTERFACESv4="vlan200"
```

### Настройка DHCP

```bash
nano /etc/dhcp/dhcpd.conf
```

В начале файла меняем строки:

```
# было:
option domain-name "example.org";
option domain-name-servers ns1.example.org, ns2.example.org;

# стало:
option domain-name "au-team.irpo";
option domain-name-servers 192.168.100.2;
```

В конце файла добавляем:

```
subnet 192.168.100.32 netmask 255.255.255.240 {
  range 192.168.100.34 192.168.100.47;
  option routers 192.168.100.33;
  option domain-name-servers 192.168.100.2;
  option domain-name "au-team.irpo";
  default-lease-time 600;
  max-lease-time 7200;
}
```

> **Важно:** перед `range` и `option` — 2 пробела.

**Пояснение параметров:**
- `subnet` — сеть, в которой работает DHCP
- `range` — диапазон выдаваемых адресов
- `option domain-name-servers` — DNS-серверы
- `option domain-name` — суффикс доменного имени
- `option routers` — шлюз по умолчанию
- `default-lease-time` / `max-lease-time` — время аренды адреса в секундах

### Перезапуск DHCP

```bash
systemctl restart isc-dhcp-server
```

### Настройка HQ-CLI на получение адреса по DHCP

```bash
nano /etc/network/interfaces
```

Меняем статическую настройку:

```
# Было (статика):
iface ens3 inet static
  address 192.168.100.34/28
  gateway 192.168.100.33

# Стало (DHCP):
auto ens3
iface ens3 inet dhcp
```

> **P.S.** В `nmtui` рекомендуется удалить `Wired connection 1`, иначе интерфейс получит два IP-адреса.

```bash
systemctl restart networking
```

### Проверка выдачи адреса

```bash
ip -br a
```

<!-- Вставьте скриншот — ens3 должен получить адрес из диапазона 192.168.100.34–47 -->

> Если выдался адрес `192.168.100.35` — значит в `nmtui` не было удалено лишнее подключение; это не критично.

---

## Задание 10 — DNS-сервер на HQ-SRV (dnsmasq)

**Требования:**
- Основной DNS — HQ-SRV
- Разрешение имён в адреса и обратно (согласно таблице)
- DNS-пересылка на публичный сервер (`77.88.8.7`, `77.88.8.3` или другой)

> **Если не скачивается dnsmasq:**
> ```bash
> apt-get update
> ```

### Установка dnsmasq на HQ-SRV

```bash
apt install dnsmasq -y
```

### Конфигурация dnsmasq

```bash
nano /etc/dnsmasq.conf
```

Добавляем в начало или конец файла:

```
domain=au-team.irpo
listen-address=192.168.100.2
no-resolv
no-hosts
address=/hq-rtr.au-team.irpo/192.168.100.1
address=/hq-srv.au-team.irpo/192.168.100.2
address=/hq-cli.au-team.irpo/192.168.100.34
address=/br-rtr.au-team.irpo/192.168.200.1
address=/br-srv.au-team.irpo/192.168.200.2
address=/isp.au-team.irpo/172.16.1.1
server=77.88.8.7
```

<!-- Вставьте скриншот конфига dnsmasq -->

### Настройка resolv.conf на HQ-SRV

```bash
nano /etc/resolv.conf
```

Прописываем:

```
nameserver 127.0.0.1
```

### Проверка

Проверяем интернет с разрешением DNS:

```bash
ping ya.ru
```

Проверяем разрешение внутренних имён:

```bash
ping br-rtr.au-team.irpo
```

<!-- Вставьте скриншот успешного пинга по доменному имени -->

Если пингуется — DNS преобразует имена в IP-адреса корректно.

---

## Задание 11 — Настройка часового пояса

Выполняется на всех устройствах: **ISP, HQ-RTR, BR-RTR, HQ-SRV, HQ-CLI, BR-SRV**.

### Проверить текущий часовой пояс

```bash
timedatectl
```

<!-- Вставьте скриншот вывода timedatectl -->

### Посмотреть список доступных поясов

```bash
ls /usr/share/zoneinfo/
```

### Посмотреть список регионов

```bash
ls /usr/share/zoneinfo/Asia
```

### Установить часовой пояс (пример — Красноярск)

```bash
timedatectl set-timezone Asia/Krasnoyarsk
```

### При необходимости — изменить дату и время вручную

```bash
timedatectl set-time "2025-05-10 15:53:00"
```

### Проверить результат

```bash
timedatectl
```

<!-- Вставьте скриншот — Time zone должен быть Asia/Krasnoyarsk -->

> Повторить на всех устройствах: ISP, HQ-RTR, BR-RTR, HQ-SRV, HQ-CLI, BR-SRV.

---

<details>
<summary>📋 Таблица IP-адресации</summary>

| Устройство | IP-адрес | Шлюз | Сеть |
|---|---|---|---|
| ISP | DHCP | — | Интернет |
| ISP | 172.16.1.1/28 | — | ISP → HQ-RTR |
| ISP | 172.16.2.1/28 | — | ISP → BR-RTR |
| HQ-RTR | 172.16.1.2/28 | 172.16.1.1 | ISP → HQ-RTR |
| HQ-RTR | 192.168.100.1/27 | — | VLAN100 (HQ-SRV) |
| HQ-RTR | 192.168.100.33/28 | — | VLAN200 (HQ-CLI) |
| HQ-RTR | 192.168.100.49/29 | — | VLAN999 (управление) |
| HQ-SRV | 192.168.100.2/27 | 192.168.100.1 | VLAN100 |
| HQ-CLI | DHCP | 192.168.100.33 | VLAN200 |
| BR-RTR | 172.16.2.2/28 | 172.16.2.1 | ISP → BR-RTR |
| BR-RTR | 192.168.200.1/28 | — | BR-RTR → BR-SRV |
| BR-SRV | 192.168.200.2/28 | 192.168.200.1 | BR-RTR → BR-SRV |

</details>

