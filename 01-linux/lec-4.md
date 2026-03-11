# Лекция 4: Сетевая подсистема Linux 🌐

## Содержание
1. [Основы сетевого взаимодействия](#основы-сетевого-взаимодействия)
2. [Сетевые интерфейсы](#сетевые-интерфейсы)
3. [Управление IP-адресами](#управление-ip-адресами)
4. [Настройка DNS](#настройка-dns)
5. [Управление маршрутизацией](#управление-маршрутизацией)
6. [Диагностика сети](#диагностика-сети)
7. [Практическое задание](#практическое-задание)

---

## Основы сетевого взаимодействия 📡

### Почему сеть критична для DevOps?

Сеть — это "нервная система" ИТ-инфраструктуры . Как DevOps-инженер, вы будете постоянно сталкиваться с сетевыми взаимодействиями:

- **Микросервисы** общаются между собой по сети
- **Базы данных** доступны по сети
- **Пользователи** подключаются к приложениям через сеть
- **Мониторинг** собирает метрики по сети
- **CI/CD** деплоит артефакты через сеть

### Базовые сетевые концепции

```bash
# Основные компоненты сети:
IP-адрес     # уникальный идентификатор устройства в сети (как почтовый адрес)
Подсеть       # логическое объединение устройств (как район города)
Шлюз (gateway) # выход в другие сети (как КПП)
MAC-адрес     # физический адрес устройства (зашит в сетевую карту)
Порт          # номер "двери" на устройстве (22 для SSH, 80 для HTTP)
```

**Типы IP-адресов:**
- **IPv4**: 32-битные, например `192.168.1.100` (исчерпаны, но всё ещё основные)
- **IPv6**: 128-битные, например `2001:0db8:85a3::8a2e:0370:7334` 

### Модель OSI и стек TCP/IP

Для DevOps достаточно понимать 4 уровня модели TCP/IP:

```bash
4. Прикладной (Application)  # HTTP, SSH, DNS, SMTP
3. Транспортный (Transport)   # TCP, UDP
2. Сетевой (Internet)         # IP, ICMP (ping)
1. Канальный (Link)           # Ethernet, MAC-адреса
```

## Сетевые интерфейсы 🔌

### Что такое сетевой интерфейс?

Сетевой интерфейс — это точка подключения устройства к сети . Это может быть:
- Физический интерфейс (Ethernet-карта, Wi-Fi адаптер)
- Виртуальный интерфейс (для контейнеров, VPN, Docker)
- Петлевой интерфейс (loopback) для самого себя

### Просмотр сетевых интерфейсов

```bash
# Современный способ (iproute2) - РЕКОМЕНДУЕТСЯ
ip link show                    # показать все интерфейсы
ip -brief link show             # краткая информация
ip addr show                    # показать интерфейсы с IP-адресами
ip -brief addr show             # краткая информация с IP

# Традиционный способ (net-tools) - устаревает
ifconfig                        # показать все интерфейсы
ifconfig -a                     # показать включая отключенные

# Информация из файловой системы
ls -l /sys/class/net/           # все интерфейсы в системе
```

**Пример вывода `ip addr`:**
```bash
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP
    inet 192.168.1.100/24 brd 192.168.1.255 scope global dynamic enp0s3
    inet6 fe80::1a2b:3c4d:5e6f:f7g8/64 scope link
```

**Разбор:**
- `lo` — loopback (localhost), всегда `127.0.0.1` 
- `enp0s3` — физический интерфейс (новое именование)
- `UP` — интерфейс активен
- `inet` — IPv4-адрес
- `inet6` — IPv6-адрес
- `/24` — маска подсети (CIDR-нотация)

### Именование сетевых интерфейсов

Современные Linux-системы используют предсказуемые имена:

| Шаблон | Значение | Пример |
|--------|----------|--------|
| `en` | Ethernet | `enp0s3` |
| `wl` | WLAN (Wi-Fi) | `wlp2s0` |
| `ww` | WWAN (мобильный) | `wwp0s29u1u4` |
| `lo` | Loopback | `lo` |

> 💡 Если хотите вернуть старые имена (eth0, eth1), добавьте в `/etc/default/grub`: `GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"` 

## Управление IP-адресами 📍

### Временная настройка IP (до перезагрузки)

```bash
# Добавить IP-адрес
sudo ip addr add 192.168.1.101/24 dev enp0s3

# Удалить IP-адрес
sudo ip addr del 192.168.1.101/24 dev enp0s3

# Включить/выключить интерфейс
sudo ip link set enp0s3 up
sudo ip link set enp0s3 down

# Устаревший способ (ifconfig)
sudo ifconfig enp0s3 192.168.1.101 netmask 255.255.255.0 up
```

### Постоянная настройка (разные дистрибутивы)

#### CentOS / RHEL 6/7 (традиционный способ)

```bash
# /etc/sysconfig/network-scripts/ifcfg-eth0
TYPE=Ethernet
BOOTPROTO=static           # static или dhcp
NAME=eth0
DEVICE=eth0
ONBOOT=yes                 # включить при загрузке
IPADDR=192.168.1.100
NETMASK=255.255.255.0      # или PREFIX=24
GATEWAY=192.168.1.1
DNS1=8.8.8.8
DNS2=8.8.4.4

# Применить изменения
sudo systemctl restart network
```

#### CentOS / RHEL 8/9 (NetworkManager)

```bash
# RHEL 8/9 используют только NetworkManager 
# Просмотр соединений
nmcli con show

# Настройка статического IP
nmcli con mod eth0 ipv4.addresses 192.168.1.100/24
nmcli con mod eth0 ipv4.gateway 192.168.1.1
nmcli con mod eth0 ipv4.dns "8.8.8.8 8.8.4.4"
nmcli con mod eth0 ipv4.method manual

# Активация
nmcli con up eth0
```

#### Ubuntu 16.04 и старше (/etc/network/interfaces)

```bash
# /etc/network/interfaces
# Статический IP
auto eth0
iface eth0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8 8.8.4.4

# DHCP
auto eth0
iface eth0 inet dhcp

# Применить изменения
sudo systemctl restart networking
```

#### Ubuntu 18.04+ (Netplan) ⭐

```yaml
# /etc/netplan/01-netcfg.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: false
      addresses:
        - 192.168.1.100/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 114.114.114.114]

# Применить изменения
sudo netplan apply
```

> ⚠️ **Важно**: YAML чувствителен к отступам! Используйте пробелы, не табуляцию .

### Настройка нескольких IP на одном интерфейсе

```bash
# Временное добавление (iproute2)
sudo ip addr add 192.168.1.200/24 dev eth0
sudo ip addr add 192.168.1.201/24 dev eth0

# CentOS (постоянно)
cp /etc/sysconfig/network-scripts/ifcfg-eth0 /etc/sysconfig/network-scripts/ifcfg-eth0:1
# отредактировать IPADDR в новом файле

# Ubuntu Netplan
addresses:
  - 192.168.1.100/24
  - 192.168.1.101/24
  - 192.168.1.102/24
```

## Настройка DNS 🔍

### Файл /etc/resolv.conf

```bash
# Просмотр текущих DNS-серверов
cat /etc/resolv.conf

# Пример содержимого
nameserver 8.8.8.8
nameserver 8.8.4.4
search example.com local
```

> ⚠️ **Внимание**: Во многих современных системах этот файл генерируется автоматически. Править его вручную бесполезно — изменения перезапишутся .

### Настройка DNS через системные инструменты

```bash
# Через NetworkManager (CentOS/RHEL)
nmcli con mod eth0 ipv4.dns "8.8.8.8 8.8.4.4"
nmcli con up eth0

# Через Netplan (Ubuntu)
# в файле конфигурации:
nameservers:
  addresses: [8.8.8.8, 8.8.4.4]
```

### Проверка DNS-резолвинга

```bash
# nslookup (простой)
nslookup google.com
nslookup google.com 8.8.8.8  # использовать конкретный DNS

# dig (подробный)
dig google.com
dig +short google.com        # только IP-адрес

# host (простой)
host google.com
host -t MX google.com        # MX-записи (почтовые серверы)

# getent (системный резолвер)
getent hosts google.com      # использует /etc/nsswitch.conf
```

## Управление маршрутизацией 🧭

### Просмотр таблицы маршрутизации

```bash
# Современный способ
ip route show
# или
ip route list

# Традиционный способ
route -n
netstat -rn

# Только маршрут по умолчанию
ip route show default
```

**Пример вывода:**
```bash
default via 192.168.1.1 dev eth0 proto dhcp metric 100
192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.100 metric 100
```

**Разбор:**
- `default` — маршрут по умолчанию (для всех адресов вне локальной сети)
- `via 192.168.1.1` — через шлюз 192.168.1.1
- `dev eth0` — через интерфейс eth0
- `192.168.1.0/24` — локальная сеть (доступна напрямую)

### Добавление и удаление маршрутов

```bash
# Временное добавление маршрута
sudo ip route add 10.0.0.0/24 via 192.168.1.254 dev eth0
sudo ip route add default via 192.168.1.1  # маршрут по умолчанию

# Удаление маршрута
sudo ip route del 10.0.0.0/24
sudo ip route del default

# Проверка маршрута до конкретного хоста
ip route get 8.8.8.8
# Вывод: 8.8.8.8 via 192.168.1.1 dev eth0 src 192.168.1.100
```

### Постоянная настройка маршрутов

**CentOS/RHEL:**
```bash
# /etc/sysconfig/network-scripts/route-eth0
10.0.0.0/24 via 192.168.1.254
# или
ADDRESS0=10.0.0.0
NETMASK0=255.255.255.0
GATEWAY0=192.168.1.254
```

**Ubuntu Netplan:**
```yaml
routes:
  - to: 10.0.0.0/24
    via: 192.168.1.254
  - to: default
    via: 192.168.1.1
```

## Диагностика сети 🔧

### Базовые утилиты

#### ping — проверка доступности

```bash
# Базовый ping (бесконечно)
ping 8.8.8.8

# Ограничить количество запросов
ping -c 4 8.8.8.8

# С интервалом 0.2 секунды
ping -i 0.2 -c 10 8.8.8.8

# Разбор ответа:
64 bytes from 8.8.8.8: icmp_seq=1 ttl=117 time=10.4 ms
  ↑                      ↑          ↑          ↑
байт данных         номер пакета  время жизни время ответа
```

#### traceroute — путь пакета

```bash
# Трассировка до хоста
traceroute google.com

# TCP-трассировка (если ICMP заблокирован)
traceroute -T google.com

# Ускоренная версия (Parallel traceroute)
mtr google.com  # интерактивный режим
```

#### ss и netstat — соединения и порты

```bash
# Все TCP-соединения
ss -t
netstat -t

# Все слушающие порты
ss -tln
netstat -tln

# С PID процессами
ss -tlnp
netstat -tlnp

# Расшифровка:
# -t = TCP, -u = UDP, -l = listening, -n = числовые порты, -p = процессы
```

#### nc (netcat) — проверка портов

```bash
# Проверить, открыт ли порт
nc -zv google.com 80
# -z = сканировать без передачи данных, -v = подробно

# Подключиться к порту (ручной ввод)
nc google.com 80
GET / HTTP/1.0
Host: google.com
[Enter][Ctrl+D]
```

### Продвинутая диагностика

#### tcpdump — сниффер трафика

```bash
# Просмотр всего трафика на интерфейсе
sudo tcpdump -i eth0

# Только HTTP-трафик (порт 80)
sudo tcpdump -i eth0 port 80

# Только трафик с конкретного хоста
sudo tcpdump -i eth0 host 192.168.1.100

# Сохранить в файл для анализа в Wireshark
sudo tcpdump -i eth0 -w capture.pcap

# Читать из файла
tcpdump -r capture.pcap
```

#### ethtool — информация о сетевой карте

```bash
# Информация о интерфейсе
ethtool eth0

# Скорость соединения
ethtool eth0 | grep Speed

# Статистика ошибок
ethtool -S eth0
```

### Проверка сетевых настроек

```bash
# Полный чек-лист диагностики:

# 1. Проверить интерфейсы
ip addr show

# 2. Проверить маршруты
ip route show

# 3. Проверить DNS
cat /etc/resolv.conf

# 4. Проверить локальный loopback
ping -c 2 127.0.0.1

# 5. Проверить шлюз
ping -c 2 $(ip route | grep default | awk '{print $3}')

# 6. Проверить внешний DNS
ping -c 2 8.8.8.8

# 7. Проверить разрешение имен
nslookup google.com
```

## Практическое задание 📝

### Задание 1: Исследование сети

```bash
# 1. Выведите все сетевые интерфейсы
ip link show

# 2. Определите свой IP-адрес, маску и шлюз
ip addr show
ip route show default

# 3. Определите MAC-адрес своего интерфейса
ip link show eth0 | grep link/ether

# 4. Проверьте доступность разных узлов
ping -c 4 127.0.0.1
ping -c 4 192.168.1.1  # или адрес вашего шлюза
ping -c 4 8.8.8.8
ping -c 4 google.com
```

### Задание 2: Настройка статического IP

**Для Ubuntu (Netplan):**
```bash
# 1. Создайте бэкап текущей конфигурации
sudo cp /etc/netplan/01-netcfg.yaml /etc/netplan/01-netcfg.yaml.bak

# 2. Отредактируйте конфигурацию
sudo nano /etc/netplan/01-netcfg.yaml
# Установите статический IP (192.168.1.200/24)

# 3. Примените конфигурацию
sudo netplan apply

# 4. Проверьте изменения
ip addr show
ping -c 4 8.8.8.8
```

**Для CentOS/RHEL:**
```bash
# 1. Найдите свой интерфейс
ip link show

# 2. Отредактируйте конфигурацию
sudo nano /etc/sysconfig/network-scripts/ifcfg-eth0
# Установите BOOTPROTO=static и добавьте IPADDR, NETMASK, GATEWAY

# 3. Перезапустите сеть
sudo systemctl restart network

# 4. Проверьте
ip addr show
ping -c 4 8.8.8.8
```

### Задание 3: Диагностика проблем

```bash
# Сценарий: сайт не открывается

# 1. Проверьте локальную сеть
ping -c 4 192.168.1.1

# 2. Проверьте выход в интернет
ping -c 4 8.8.8.8

# 3. Проверьте DNS
nslookup google.com

# 4. Проверьте конкретный порт
nc -zv google.com 80

# 5. Посмотрите маршрут
traceroute google.com

# 6. Проверьте открытые порты на своем компьютере
ss -tln
```

### Задание 4: Работа с несколькими IP

```bash
# 1. Добавьте дополнительный IP на ваш интерфейс
sudo ip addr add 192.168.1.150/24 dev eth0

# 2. Проверьте, что IP добавился
ip addr show eth0

# 3. Попингуйте новый IP с другого компьютера (если есть)
# или просто проверьте, что он виден
ping -c 4 192.168.1.150

# 4. Удалите временный IP
sudo ip addr del 192.168.1.150/24 dev eth0
```

### Задание 5: Анализ трафика

```bash
# 1. Запустите сниффер трафика на 10 секунд
sudo timeout 10 tcpdump -i eth0 -w /tmp/capture.pcap

# 2. Сгенерируйте трафик (в другом окне)
ping -c 5 google.com
curl http://example.com

# 3. Проанализируйте захваченные пакеты
tcpdump -r /tmp/capture.pcap | head -20

# 4. Посмотрите статистику по протоколам
tcpdump -r /tmp/capture.pcap -n | awk '{print $2}' | sort | uniq -c | sort -nr
```

## Шпаргалка по командам 📋

| Команда | Назначение | Пример |
|---------|------------|--------|
| `ip addr show` | Показать IP-адреса | `ip a` |
| `ip link show` | Показать интерфейсы | `ip l` |
| `ip route show` | Показать маршруты | `ip r` |
| `ping` | Проверка доступности | `ping -c 4 8.8.8.8` |
| `traceroute` | Трассировка маршрута | `traceroute google.com` |
| `ss -tln` | Слушающие порты | `ss -tlnp` |
| `nc -zv` | Проверка порта | `nc -zv google.com 80` |
| `nslookup` | DNS-запросы | `nslookup google.com` |
| `dig` | Подробный DNS | `dig google.com` |
| `tcpdump` | Сниффинг трафика | `sudo tcpdump -i eth0` |
| `ethtool` | Информация о NIC | `ethtool eth0` |

## Вопросы для самопроверки 🤔

1. В чем разница между `ip addr show` и `ifconfig`?
2. Что такое маршрут по умолчанию и зачем он нужен?
3. Как проверить, работает ли DNS?
4. Почему в современных системах не рекомендуется редактировать `/etc/resolv.conf`?
5. Как временно добавить IP-адрес на интерфейс?
6. Что означают поля в выводе `ping`?
7. Как узнать, какие порты слушает ваш сервер?
8. В чем разница между TCP и UDP?

## Дополнительные материалы 📚

- [Linux Network Administration course (Coursera)](https://futurex.nelc.gov.sa/index.php/en/node/16477301) — comprehensive course updated May 2025 
- [Rocky Linux Networking Essentials Lab](https://docs.rockylinux.org/10/ko/labs/systems_administration_I/lab5-networking/) — практические упражнения 
- [Сетевые средства Linux. Учебное пособие](https://www.iprbookshop.ru/146397.html) — ISBN 978-5-4497-0930-1, 2025 
- [Linux Networking at Scale](https://linuxhandbook.com/courses/networking-scale/) — продвинутый курс 
- [ExplainShell](https://explainshell.com) — объяснение любых команд

---

**Что дальше?** В следующей лекции мы углубимся в безопасность сети — изучим брандмауэры (iptables/nftables), настройку SSH для безопасности и основы VPN. А пока — экспериментируйте с сетью, ломайте и чините соединения! 🚀

> 🌐 **Совет дня:** "Сеть в Linux — как воздух: не замечаешь, пока она работает, и сразу понимаешь её ценность, когда она ломается. Учитесь диагностировать проблемы быстро и эффективно!"