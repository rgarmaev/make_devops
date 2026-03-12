# Лекция 5: Управление пакетами и программным обеспечением в Linux 📦

## Содержание
1. [Введение в управление пакетами](#введение-в-управление-пакетами)
2. [Менеджеры пакетов](#менеджеры-пакетов)
3. [Работа с APT (Debian/Ubuntu)](#работа-с-apt-debianubuntu)
4. [Работа с YUM/DNF (CentOS/RHEL)](#работа-с-ymdnf-centosrhel)
5. [Сборка из исходников](#сборка-из-исходников)
6. [Управление репозиториями](#управление-репозиториями)
7. [Практическое задание](#практическое-задание)

---

## Введение в управление пакетами 🎯

### Что такое пакет?

Пакет — это архив, содержащий:
- Исполняемые файлы программы
- Библиотеки
- Конфигурационные файлы
- Документацию
- Скрипты установки/удаления
- Информацию о зависимостях

### Почему важно уметь управлять пакетами?

В работе DevOps-инженера вы постоянно будете:
- **Устанавливать ПО** на серверы
- **Обновлять** компоненты инфраструктуры
- **Управлять зависимостями** приложений
- **Автоматизировать** установку через скрипты
- **Создавать** собственные пакеты

### Типы пакетов

```bash
# Основные форматы пакетов:
.deb    # Debian, Ubuntu, Linux Mint
.rpm    # Red Hat, CentOS, Fedora, Rocky Linux
.tar.*  # исходный код или бинарные сборки
.snap   # универсальный формат от Canonical
.flatpak # универсальный формат для десктопов
.AppImage # переносимые приложения
```

## Менеджеры пакетов 🔧

### Семейства менеджеров

| Семейство | Низкий уровень | Высокий уровень | Дистрибутивы |
|-----------|---------------|-----------------|--------------|
| **Debian** | `dpkg` | `apt`, `apt-get` | Ubuntu, Debian, Mint |
| **Red Hat** | `rpm` | `yum`, `dnf` | CentOS, RHEL, Fedora |
| **Arch** | `pacman` | - | Arch, Manjaro |
| **SUSE** | `rpm` | `zypper` | openSUSE, SLES |
| **Gentoo** | `emerge` | - | Gentoo |

### Схема работы менеджера пакетов

```
Репозиторий (интернет)
       ↓
Скачивание пакета и metadata
       ↓
Проверка зависимостей
       ↓
Скачивание зависимостей
       ↓
Установка всех пакетов
       ↓
Запуск post-install скриптов
       ↓
Готово! ✅
```

## Работа с APT (Debian/Ubuntu) 🚀

### Основы dpkg (низкий уровень)

```bash
# Установка .deb файла
sudo dpkg -i package.deb

# Удаление пакета
sudo dpkg -r package_name     # оставляет конфиги
sudo dpkg -P package_name     # полное удаление (purge)

# Информация о пакете
dpkg -l                       # все установленные пакеты
dpkg -l | grep nginx          # поиск по имени
dpkg -s package_name          # статус пакета
dpkg -L package_name          # какие файлы установил пакет
dpkg -S /bin/ls               # какому пакету принадлежит файл

# Пример:
dpkg -L nginx | grep config   # найти конфиги nginx
```

### Работа с APT (высокий уровень)

```bash
# Обновление информации о пакетах
sudo apt update                # обновить список пакетов из репозиториев
sudo apt list --upgradable     # показать доступные обновления

# Установка и удаление
sudo apt install nginx         # установить пакет
sudo apt install -y nginx      # без подтверждения (для скриптов)
sudo apt remove nginx          # удалить (оставить конфиги)
sudo apt purge nginx           # удалить полностью
sudo apt autoremove            # удалить ненужные зависимости

# Обновление системы
sudo apt upgrade               # обновить все пакеты
sudo apt full-upgrade          # обновить с возможным удалением пакетов
sudo apt dist-upgrade          # то же что full-upgrade

# Поиск и информация
apt search nginx               # поиск пакета
apt show nginx                 # показать информацию о пакете
apt list --installed           # все установленные пакеты
apt depends nginx              # зависимости пакета
apt rdepends nginx             # что зависит от этого пакета

# Работа с кэшем
apt clean                      # очистить кэш .deb файлов
apt autoclean                  # очистить устаревшие .deb
```

### Полезные комбинации APT

```bash
# Установить несколько пакетов
sudo apt install -y nginx mysql-server redis-server

# Найти и установить пакет одной командой
sudo apt install -y $(apt search something | grep -i "package" | cut -d/ -f1 | head -5)

# Посмотреть историю APT
less /var/log/apt/history.log

# Установить конкретную версию
sudo apt install nginx=1.18.0-0ubuntu1

# Заблокировать версию (не обновлять)
sudo apt-mark hold nginx
sudo apt-mark unhold nginx
sudo apt-mark showhold
```

### Решение проблем с APT

```bash
# Если сломались зависимости
sudo apt --fix-broken install

# Если что-то не так с кэшем
sudo apt clean
sudo apt update --fix-missing

# Переустановка пакета
sudo apt install --reinstall nginx

# Скачать .deb без установки
sudo apt download nginx
```

## Работа с YUM/DNF (CentOS/RHEL) 📦

### Основы rpm (низкий уровень)

```bash
# Установка .rpm файла
sudo rpm -ivh package.rpm      # i=install, v=verbose, h=progress

# Обновление
sudo rpm -Uvh package.rpm      # U=upgrade

# Удаление
sudo rpm -e package_name

# Информация
rpm -qa                        # все установленные пакеты (query all)
rpm -qa | grep nginx           # поиск по имени
rpm -qi package_name           # информация о пакете
rpm -ql package_name           # какие файлы установил
rpm -qf /bin/ls                # какому пакету принадлежит файл
rpm -qR package_name           # зависимости пакета (Requires)
```

### Работа с YUM (CentOS 6/7)

```bash
# Поиск и установка
yum search nginx               # поиск пакета
yum install nginx              # установка
yum install -y nginx           # без подтверждения

# Удаление
yum remove nginx               # удалить пакет
yum autoremove                 # удалить ненужные зависимости

# Обновление
yum check-update               # проверить обновления
yum update                     # обновить все
yum update nginx               # обновить конкретный пакет

# Информация
yum list installed             # все установленные
yum list available             # доступные для установки
yum info nginx                 # информация о пакете
yum deplist nginx              # зависимости пакета

# Группы пакетов
yum group list                 # список групп
yum group install "Development Tools"  # установить группу
yum group info "Development Tools"     # что входит в группу
```

### Работа с DNF (CentOS 8/9, Fedora)

DNF — следующее поколение YUM (тот же синтаксис, но быстрее и лучше)

```bash
# Основные команды (аналогичны YUM)
dnf search nginx
dnf install nginx
dnf remove nginx
dnf update
dnf list installed
dnf info nginx
dnf provides /bin/ls          # какой пакет дает файл

# Дополнительные возможности
dnf history                    # история действий
dnf reinstall nginx            # переустановка
dnf downgrade nginx            # откат версии
dnf autoremove                 # удалить ненужные зависимости
```

### Решение проблем с YUM/DNF

```bash
# Очистка кэша
yum clean all
dnf clean all

# Если сломались зависимости
yum --skip-broken update
yum --nogpgcheck install package  # игнорировать проверку GPG ключей

# Переустановка rpm базы
sudo rm -f /var/lib/rpm/__db.*
sudo rpm --rebuilddb
```

## Сборка из исходников 🔨

### Зачем собирать из исходников?

- **Нет пакета** для вашей версии ОС
- **Нужны специфические опции** компиляции
- **Максимальная оптимизация** под железо
- **Хотите понять**, как работает программа

### Типичный процесс сборки

```bash
# 1. Установка инструментов сборки
# Ubuntu/Debian
sudo apt install build-essential
sudo apt build-dep nginx       # установить зависимости для сборки nginx

# CentOS/RHEL
sudo yum groupinstall "Development Tools"
sudo yum install yum-utils
sudo yum-builddep nginx

# 2. Скачивание исходников
wget http://nginx.org/download/nginx-1.22.1.tar.gz
tar -xzf nginx-1.22.1.tar.gz
cd nginx-1.22.1

# 3. Конфигурация (настройка опций сборки)
./configure --prefix=/usr/local/nginx \
            --with-http_ssl_module \
            --with-http_v2_module

# 4. Компиляция
make -j$(nproc)                # использовать все ядра CPU

# 5. Установка
sudo make install

# 6. Проверка
/usr/local/nginx/sbin/nginx -V
```

### Часто используемые опции configure

```bash
./configure --help              # показать все опции

# Типичные опции:
--prefix=/usr/local/package    # куда установить
--sysconfdir=/etc/package       # где будут конфиги
--with-feature                  # включить фичу
--without-feature               # выключить фичу
--enable-debug                  # включить отладку
--with-openssl=DIR              # использовать свой OpenSSL
```

### Создание собственного пакета

```bash
# Пример: создание .deb пакета
# 1. Установка инструментов
sudo apt install dh-make debhelper

# 2. Создание структуры
mkdir myapp-1.0
cd myapp-1.0
dh_make --createorig

# 3. Настройка правил сборки
nano debian/rules
nano debian/control

# 4. Сборка пакета
dpkg-buildpackage -us -uc

# 5. Полученный .deb файл будет в родительской директории
```

## Управление репозиториями 📚

### APT-репозитории (Debian/Ubuntu)

```bash
# Файлы репозиториев
/etc/apt/sources.list
/etc/apt/sources.list.d/

# Просмотр текущих репозиториев
cat /etc/apt/sources.list
ls /etc/apt/sources.list.d/

# Добавление репозитория (Ubuntu 18.04+)
sudo add-apt-repository ppa:nginx/stable
sudo add-apt-repository "deb http://archive.ubuntu.com/ubuntu $(lsb_release -sc) main universe"

# Добавление вручную
echo "deb https://packages.docker.com/linux/ubuntu focal stable" | sudo tee /etc/apt/sources.list.d/docker.list

# Добавление GPG ключа (для проверки подлинности)
wget -qO - https://packages.docker.com/gpg | sudo apt-key add -
# Новый способ (Ubuntu 20.04+)
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys KEY_ID

# Обновление после добавления
sudo apt update
```

### YUM/DNF-репозитории (CentOS/RHEL)

```bash
# Файлы репозиториев
/etc/yum.repos.d/

# Просмотр репозиториев
ls /etc/yum.repos.d/
yum repolist
dnf repolist

# Добавление EPEL (Extra Packages for Enterprise Linux)
sudo yum install epel-release
# или вручную
sudo rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

# Добавление репозитория вручную
sudo nano /etc/yum.repos.d/myrepo.repo

# Формат .repo файла:
[myrepo]
name=My Repository
baseurl=https://myrepo.example.com/centos/$releasever/$basearch/
enabled=1
gpgcheck=1
gpgkey=https://myrepo.example.com/RPM-GPG-KEY-myrepo

# Включение/отключение репозиториев
yum --enablerepo=epel install package
yum --disablerepo=base update
```

### Полезные сторонние репозитории

```bash
# EPEL (для CentOS/RHEL) - много дополнительных пакетов
sudo yum install epel-release

# Remi (для PHP, CentOS/RHEL)
sudo yum install https://rpms.remirepo.net/enterprise/remi-release-7.rpm

# Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Kubernetes
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

# PostgreSQL
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
```

## Практическое задание 📝

### Задание 1: Работа с APT

```bash
# 1. Обновите список пакетов
sudo apt update

# 2. Найдите все пакеты, связанные с nginx
apt search nginx | grep -i web

# 3. Установите nginx и curl
sudo apt install -y nginx curl

# 4. Проверьте, какие файлы установил nginx
dpkg -L nginx | head -20

# 5. Посмотрите информацию о пакете
apt show nginx

# 6. Узнайте, какой пакет дает команду curl
dpkg -S $(which curl)

# 7. Удалите nginx, оставив конфиги
sudo apt remove nginx

# 8. Удалите конфиги полностью
sudo apt purge nginx

# 9. Удалите ненужные зависимости
sudo apt autoremove
```

### Задание 2: Работа с YUM/DNF (на CentOS/RHEL)

```bash
# 1. Посмотрите все репозитории
yum repolist

# 2. Найдите пакет для установки веб-сервера
yum search web server

# 3. Установите httpd (Apache)
sudo yum install -y httpd

# 4. Посмотрите информацию о пакете
yum info httpd

# 5. Какие файлы установил httpd?
rpm -ql httpd | head -20

# 6. Удалите httpd
sudo yum remove -y httpd
```

### Задание 3: Добавление репозитория и установка

```bash
# Для Ubuntu:
# 1. Добавьте репозиторий PostgreSQL
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

# 2. Добавьте ключ
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

# 3. Обновите список пакетов
sudo apt update

# 4. Найдите доступные версии PostgreSQL
apt search postgresql | grep -E "^postgresql-[0-9]+"

# 5. Установите PostgreSQL 14
sudo apt install -y postgresql-14

# 6. Проверьте, что сервер запущен
systemctl status postgresql
```

### Задание 4: Сборка из исходников

```bash
# 1. Установите инструменты сборки
sudo apt update
sudo apt install -y build-essential wget

# 2. Скачайте исходники hello (простая программа)
cd /tmp
wget http://ftp.gnu.org/gnu/hello/hello-2.12.tar.gz
tar -xzf hello-2.12.tar.gz
cd hello-2.12

# 3. Сконфигурируйте
./configure --prefix=/tmp/hello-test

# 4. Соберите
make

# 5. Установите в /tmp/hello-test
make install

# 6. Проверьте
/tmp/hello-test/bin/hello --version

# 7. Посмотрите, что установилось
find /tmp/hello-test -type f
```

### Задание 5: Автоматизация установки (скрипт)

Создайте скрипт `install-lamp.sh`, который автоматически устанавливает LAMP-стек (Linux, Apache, MySQL, PHP):

```bash
#!/bin/bash
set -euo pipefail

# Цвета для вывода
RED='\033[0;31m'
GREEN='\033[0;32m'
NC='\033[0m'

log() {
    echo -e "${GREEN}[INFO]${NC} $1"
}

error() {
    echo -e "${RED}[ERROR]${NC} $1"
    exit 1
}

# Определение дистрибутива
if [ -f /etc/debian_version ]; then
    DISTRO="debian"
    PKG_MANAGER="apt"
    APACHE_PKG="apache2"
    MYSQL_PKG="mysql-server"
    PHP_PKG="php libapache2-mod-php php-mysql"
elif [ -f /etc/redhat-release ]; then
    DISTRO="redhat"
    PKG_MANAGER="yum"
    APACHE_PKG="httpd"
    MYSQL_PKG="mysql-server"
    PHP_PKG="php php-mysqlnd"
else
    error "Unsupported distribution"
fi

log "Detected distribution: $DISTRO"

# Обновление пакетов
log "Updating package list..."
if [ "$PKG_MANAGER" = "apt" ]; then
    sudo apt update
else
    sudo yum check-update || true  # yum returns 100 if updates available
fi

# Установка Apache
log "Installing Apache..."
sudo $PKG_MANAGER install -y $APACHE_PKG

# Установка MySQL
log "Installing MySQL..."
sudo $PKG_MANAGER install -y $MYSQL_PKG

# Установка PHP
log "Installing PHP..."
sudo $PKG_MANAGER install -y $PHP_PKG

# Запуск сервисов
log "Starting services..."
if [ "$DISTRO" = "debian" ]; then
    sudo systemctl enable apache2
    sudo systemctl start apache2
    sudo systemctl enable mysql
    sudo systemctl start mysql
else
    sudo systemctl enable httpd
    sudo systemctl start httpd
    sudo systemctl enable mysqld
    sudo systemctl start mysqld
fi

# Создание тестовой PHP-страницы
log "Creating test page..."
echo "<?php phpinfo(); ?>" | sudo tee /var/www/html/info.php

# Проверка
log "Testing Apache..."
if curl -s http://localhost/ | grep -q "Apache"; then
    log "Apache is working"
else
    error "Apache test failed"
fi

log "LAMP stack installation completed!"
log "Test PHP at: http://localhost/info.php"
log "MySQL root password: (set manually with mysql_secure_installation)"
```

## Шпаргалка по командам 📋

### APT (Debian/Ubuntu)
| Команда | Назначение |
|---------|------------|
| `sudo apt update` | Обновить список пакетов |
| `sudo apt upgrade` | Обновить все пакеты |
| `sudo apt install pkg` | Установить пакет |
| `sudo apt remove pkg` | Удалить пакет |
| `sudo apt purge pkg` | Удалить с конфигами |
| `apt search term` | Поиск пакета |
| `apt show pkg` | Информация о пакете |
| `dpkg -L pkg` | Файлы пакета |
| `dpkg -S file` | Какому пакету принадлежит файл |

### YUM/DNF (CentOS/RHEL)
| Команда | Назначение |
|---------|------------|
| `yum install pkg` | Установить пакет |
| `yum remove pkg` | Удалить пакет |
| `yum update` | Обновить все |
| `yum search term` | Поиск пакета |
| `yum info pkg` | Информация о пакете |
| `yum list installed` | Установленные пакеты |
| `rpm -ql pkg` | Файлы пакета |
| `rpm -qf file` | Какому пакету принадлежит файл |

### Сборка из исходников
| Команда | Назначение |
|---------|------------|
| `./configure` | Конфигурация сборки |
| `make` | Компиляция |
| `make install` | Установка |
| `make clean` | Очистка временных файлов |
| `make uninstall` | Удаление (если поддерживается) |

## Вопросы для самопроверки 🤔

1. В чем разница между `apt update` и `apt upgrade`?
2. Что такое зависимости пакета и как их посмотреть?
3. Как найти, какому пакету принадлежит файл `/etc/nginx/nginx.conf`?
4. Чем отличается `yum remove` от `yum autoremove`?
5. Зачем нужны GPG ключи при добавлении репозиториев?
6. Как установить конкретную версию пакета?
7. В чем преимущества и недостатки сборки из исходников?
8. Что такое PPA в Ubuntu и зачем они нужны?

## Дополнительные материалы 📚

- [APT Package Management](https://www.debian.org/doc/manuals/debian-handbook/sect.apt-get.en.html) — официальная документация
- [DNF Documentation](https://dnf.readthedocs.io/en/latest/) — документация DNF
- [Creating Debian Packages](https://www.debian.org/doc/manuals/maint-guide/) — руководство по созданию пакетов
- [RPM Packaging Guide](https://rpm-packaging-guide.github.io/) — руководство по RPM-пакетам
- [GNU Autotools Tutorial](https://www.lrde.epita.fr/~adl/autotools.html) — для понимания сборки из исходников

---

**Что дальше?** В следующей лекции мы перейдем к одной из важнейших тем для DevOps — контейнеризации и Docker. Вы узнаете, как упаковывать приложения в контейнеры и управлять ими! 🐳

> 📦 **Совет дня:** "Менеджер пакетов — лучший друг администратора. Всегда старайтесь использовать официальные пакеты из репозиториев, это упрощает обновление и управление зависимостями. Сборка из исходников — только когда действительно нужно!"