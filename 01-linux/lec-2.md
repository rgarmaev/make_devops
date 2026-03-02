
# Лекция 2: Права доступа, пользователи и управление процессами в Linux 🐧

## Содержание
1. [Пользователи и группы](#пользователи-и-группы)
2. [Права доступа к файлам](#права-доступа-к-файлам)
3. [Управление процессами](#управление-процессами)
4. [Управление службами (systemd)](#управление-службами-systemd)
5. [SSH и удаленный доступ](#ssh-и-удаленный-доступ)
6. [Практическое задание](#практическое-задание)

---

## Пользователи и группы 👥

В Linux каждый процесс и каждый файл принадлежит определенному пользователю и группе. Это основа безопасности системы.

### Типы пользователей

```bash
# Три типа пользователей:
1. root (UID 0)       # суперпользователь, может всё
2. системные (UID 1-999)   # для служб и демонов
3. обычные (UID 1000+)      # пользователи-люди
```

### Управление пользователями

```bash
# Просмотр информации
whoami                    # текущий пользователь
id                        # информация о текущем пользователе
id username               # информация о конкретном пользователе
users                     # кто залогинен в систему
who                       # подробная информация о залогиненных
w                         # еще подробнее + что делают

# Создание пользователей
sudo useradd john                    # создать пользователя
sudo useradd -m -s /bin/bash john    # создать с домашней папкой и bash
sudo adduser john                    # интерактивное создание (Debian/Ubuntu)

# Изменение пользователей
sudo usermod -aG sudo john           # добавить в группу sudo
sudo usermod -s /bin/zsh john        # сменить shell
sudo passwd john                     # сменить/установить пароль

# Удаление пользователей
sudo userdel john                     # удалить пользователя
sudo userdel -r john                  # удалить с домашней папкой

# Переключение между пользователями
su john                               # переключиться на john
su - john                             # переключиться с загрузкой окружения
sudo -i                               # стать root (с загрузкой окружения)
sudo -s                               # стать root (с текущим окружением)
```

### Управление группами

```bash
# Просмотр групп
groups                                # группы текущего пользователя
groups john                           # группы пользователя john
cat /etc/group                        # все группы системы

# Создание и удаление групп
sudo groupadd developers               # создать группу
sudo groupdel developers               # удалить группу

# Добавление/удаление пользователя в группу
sudo gpasswd -a john developers       # добавить в группу
sudo usermod -aG developers john      # альтернативный способ
sudo gpasswd -d john developers       # удалить из группы
```

### Важные системные файлы

```bash
/etc/passwd     # информация о пользователях
/etc/shadow     # зашифрованные пароли (только для root)
/etc/group      # информация о группах
/etc/sudoers    # настройки sudo (редактировать только через visudo!)
```

**Формат /etc/passwd:**
```
username:x:UID:GID:description:/home/dir:/bin/bash
    ↑      ↑  ↑   ↑       ↑           ↑           ↑
   имя   пароль UID  GID   описание   домашняя    shell
          (x значит в /etc/shadow)
```

## Права доступа к файлам 🔐

### Базовая концепция

В Linux каждый файл имеет:
- **Владельца** (user)
- **Группу-владельца** (group)
- **Права доступа** для трех категорий:
  - Владелец (u — user)
  - Группа (g — group)
  - Остальные (o — others)

### Просмотр прав

```bash
ls -l filename

# Пример вывода:
-rw-r--r-- 1 john developers 1024 Mar 15 10:30 file.txt
↑↑↑↑↑↑↑↑↑↑   ↑   ↑       ↑      ↑       ↑        ↑
|  права    кол-во владелец группа размер   дата    имя
тип файла   ссылок
```

**Тип файла (первый символ):**
- `-` — обычный файл
- `d` — директория
- `l` — символическая ссылка
- `c` — символьное устройство
- `b` — блочное устройство

**Права доступа (следующие 9 символов):**
```
rwx rwx rwx
 │   │   │
 │   │   └── остальные (others)
 │   └────── группа (group)
 └────────── владелец (user)

r = read    (чтение)     - = нет права
w = write   (запись)
x = execute (выполнение)
```

### Числовое представление прав (восьмеричное)

```bash
r = 4
w = 2
x = 1

# Суммируем для каждой категории:
rwx = 4+2+1 = 7
rw- = 4+2+0 = 6
r-x = 4+0+1 = 5
r-- = 4+0+0 = 4
-wx = 0+2+1 = 3
-w- = 0+2+0 = 2
--x = 0+0+1 = 1
--- = 0

# Пример: 754
# 7 (владелец): rwx
# 5 (группа):   r-x
# 4 (остальные): r--
```

### Изменение прав (chmod)

```bash
# Символьный способ
chmod u+x file.sh           # добавить execute владельцу
chmod g-w file.txt          # убрать write у группы
chmod o=r file.txt          # установить только чтение для остальных
chmod a+x script.sh         # добавить execute всем (a = all)
chmod u=rwx,g=rx,o=r file   # установить конкретные права

# Числовой способ (самый популярный)
chmod 755 script.sh         # rwxr-xr-x
chmod 644 file.txt          # rw-r--r--
chmod 600 secret.txt        # rw-------
chmod 700 private_dir       # rwx------
chmod 777 bad.sh            # НИКОГДА ТАК НЕ ДЕЛАЙТЕ! (все могут всё)
```

### Изменение владельца (chown, chgrp)

```bash
# Изменение владельца
sudo chown john file.txt                # сменить владельца
sudo chown john:developers file.txt     # сменить владельца и группу
sudo chown :developers file.txt         # сменить только группу

# Рекурсивно для директорий
sudo chown -R john:developers /home/john/project

# Изменение группы (chgrp)
sudo chgrp developers file.txt           # сменить группу
sudo chgrp -R developers /shared/folder  # рекурсивно
```

### Специальные права доступа

```bash
# SUID (4) - выполнение от имени владельца
chmod u+s file              # установить SUID
chmod 4755 file             # SUID + rwxr-xr-x
# Пример: /usr/bin/passwd имеет SUID (обычные пользователи могут менять пароль)

# SGID (2) - выполнение от имени группы
chmod g+s directory         # установить SGID
chmod 2755 directory        # SGID + rwxr-xr-x
# Для директорий: новые файлы наследуют группу директории

# Sticky bit (1) - только владелец может удалять
chmod +t /tmp               # установить sticky bit
chmod 1777 /tmp             # sticky + rwxrwxrwx
# Пример: /tmp имеет sticky bit (все могут создавать, но удалять - только владелец)

# Просмотр специальных прав
ls -l /usr/bin/passwd
# -rwsr-xr-x  # s вместо x означает SUID
```

## Управление процессами ⚙️

### Что такое процесс?

Процесс — это запущенная программа с уникальным идентификатором (PID).

### Просмотр процессов

```bash
# ps (process status)
ps                     # процессы текущего пользователя
ps aux                 # все процессы системы
ps auxf                # в виде дерева
ps -ef                 # все процессы (стандарт System V)
ps -u john             # процессы пользователя john

# top - интерактивный просмотр
top                    # запустить top
htop                   # улучшенная версия (нужно установить)

# В top/htop:
# q - выход
# k - убить процесс (ввести PID)
# u - показать процессы пользователя
# M - сортировка по памяти
# P - сортировка по CPU
```

**Колонки ps aux:**
```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.4 167936 11776 ?        Ss   мар15   0:06 /sbin/init
  ↑          ↑    ↑    ↑     ↑     ↑    ↑       ↑     ↑       ↑       ↑
пользователь PID CPU память вирт. физ. терминал статус старт  время команда
```

### Статусы процессов

```bash
# Основные статусы (STAT):
R (running)      # выполняется
S (sleeping)     # ожидает события
D (uninterruptible sleep) # ожидание диска
Z (zombie)       # зомби (завершен, но не убран родителем)
T (stopped)      # остановлен (Ctrl+Z)
```

### Управление сигналами

```bash
# Отправка сигналов процессам
kill PID                    # отправить SIGTERM (15) - вежливое завершение
kill -9 PID                 # SIGKILL (9) - принудительное завершение
kill -15 PID                # SIGTERM (15)
kill -l                     # список всех сигналов

# Другие способы
pkill firefox               # убить все процессы с именем firefox
killall chrome              # убить все процессы chrome
pkill -u john               # убить все процессы пользователя john
```

### Приоритеты процессов (nice)

```bash
# nice: от -20 (высокий приоритет) до 19 (низкий)
# по умолчанию: 0

# Запуск с приоритетом
nice -n 10 ./heavy-script.sh     # запустить с приоритетом 10 (низкий)

# Изменение приоритета запущенного процесса
renice -n 5 -p 1234               # установить приоритет 5 для PID 1234
renice -n -5 -p 1234              # повысить приоритет (только root)
```

### Управление задачами (job control)

```bash
# Запуск в фоне
long-running-command &            # запустить в фоне
./script.sh &> /dev/null &        # запустить в фоне без вывода

# Управление задачами
jobs                                # показать фоновые задачи
fg %1                               # вернуть задачу 1 на передний план
bg %2                               # продолжить задачу 2 в фоне

# Приостановка и продолжение
Ctrl + Z                            # приостановить текущую задачу
kill %1                             # убить фоновую задачу 1
```

## Управление службами (systemd) 🚀

Большинство современных дистрибутивов используют systemd для управления службами.

### Основные команды systemctl

```bash
# Управление службами
sudo systemctl start nginx          # запустить службу
sudo systemctl stop nginx           # остановить
sudo systemctl restart nginx        # перезапустить
sudo systemctl reload nginx         # перечитать конфиг (без остановки)
sudo systemctl status nginx         # статус службы

# Включение/отключение автозапуска
sudo systemctl enable nginx         # добавить в автозагрузку
sudo systemctl disable nginx        # убрать из автозагрузки
sudo systemctl is-enabled nginx     # проверить, включена ли

# Просмотр всех служб
systemctl list-units --type=service # все запущенные службы
systemctl list-unit-files           # все установленные службы
```

### Просмотр логов (journalctl)

```bash
# Просмотр логов службы
journalctl -u nginx                 # логи службы nginx
journalctl -u nginx -f              # следить за логами в реальном времени
journalctl -u nginx --since "1 hour ago"  # за последний час

# Системные логи
journalctl -b                       # логи текущей загрузки
journalctl -b -1                    # логи предыдущей загрузки
journalctl -f                       # следить за всеми логами
```

### Создание своего systemd unit-файла

```bash
# /etc/systemd/system/myapp.service
[Unit]
Description=My Custom Application
After=network.target

[Service]
Type=simple
User=john
WorkingDirectory=/home/john/app
ExecStart=/usr/bin/python3 /home/john/app/main.py
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target

# После создания файла:
sudo systemctl daemon-reload        # перечитать конфиги
sudo systemctl enable myapp         # включить автозапуск
sudo systemctl start myapp          # запустить
```

## SSH и удаленный доступ 🔑

### Настройка SSH-сервера

```bash
# Установка SSH-сервера
sudo apt install openssh-server     # Debian/Ubuntu
sudo yum install openssh-server     # CentOS/RHEL

# Основные конфиги
/etc/ssh/sshd_config                # конфиг сервера
/etc/ssh/ssh_config                  # конфиг клиента
~/.ssh/                              # личные SSH-ключи
```

### Настройка SSH-ключей

```bash
# Генерация ключей
ssh-keygen -t rsa -b 4096           # создать ключ RSA 4096 бит
ssh-keygen -t ed25519                # создать ключ Ed25519 (рекомендуется)

# Копирование ключа на сервер
ssh-copy-id user@server.com          # скопировать публичный ключ
# Или вручную:
cat ~/.ssh/id_rsa.pub | ssh user@server.com "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

# Структура ~/.ssh/
~/.ssh/
├── id_rsa          # приватный ключ (никому не показывать!)
├── id_rsa.pub      # публичный ключ (можно распространять)
├── authorized_keys # публичные ключи разрешенных клиентов
└── config          # конфигурация клиента
```

### Безопасность SSH

```bash
# Рекомендуемые настройки /etc/ssh/sshd_config
Port 2222                           # сменить порт (не использовать 22)
PermitRootLogin no                    # запретить вход root
PasswordAuthentication no             # только ключи, без паролей
PubkeyAuthentication yes              # разрешить ключи
AllowUsers john devops                # разрешить только этих пользователей
MaxAuthTries 3                        # максимум попыток входа
```

### SSH-туннели и проброс портов

```bash
# Локальный проброс порта
# весь трафик с localhost:8080 уйдет на server.com:80 через SSH
ssh -L 8080:localhost:80 user@server.com

# Удаленный проброс порта
# сервер пробросит свой порт 8080 на наш localhost:3000
ssh -R 8080:localhost:3000 user@server.com

# Динамический проброс (SOCKS прокси)
ssh -D 1080 user@server.com
# теперь можно настроить браузер на SOCKS прокси localhost:1080
```

## Практическое задание 📝

### Задание 1: Пользователи и группы

```bash
# 1. Создайте нового пользователя devops_student
sudo useradd -m -s /bin/bash devops_student
sudo passwd devops_student

# 2. Создайте группы developers и admins
sudo groupadd developers
sudo groupadd admins

# 3. Добавьте пользователя в обе группы
sudo usermod -aG developers,admins devops_student

# 4. Проверьте информацию о пользователе
id devops_student
groups devops_student
```

### Задание 2: Права доступа

```bash
# 1. Создайте структуру
mkdir -p ~/lab/{public,private,shared}

# 2. Создайте файлы с разными правами
echo "Public file" > ~/lab/public/readme.txt
echo "Secret data" > ~/lab/private/secret.txt
echo "Shared file" > ~/lab/shared/common.txt

# 3. Установите права:
#   - public: все могут читать (644)
chmod 644 ~/lab/public/readme.txt

#   - private: только владелец (600)
chmod 600 ~/lab/private/secret.txt

#   - shared: владелец всё, группа чтение, остальные ничего (640)
chmod 640 ~/lab/shared/common.txt

# 4. Создайте скрипт и сделайте его исполняемым
echo '#!/bin/bash
echo "Hello from script"' > ~/lab/script.sh
chmod 755 ~/lab/script.sh

# 5. Проверьте права
ls -la ~/lab/
```

### Задание 3: Процессы

```bash
# 1. Запустите команду в фоне
sleep 300 &

# 2. Найдите её процесс
ps aux | grep sleep
pgrep sleep

# 3. Измените приоритет
renice -n 10 -p $(pgrep sleep)

# 4. Завершите процесс
kill $(pgrep sleep)

# 5. Посмотрите дерево процессов
ps auxf
pstree  # если установлен
```

### Задание 4: SSH

```bash
# 1. Сгенерируйте SSH-ключ
ssh-keygen -t ed25519 -f ~/.ssh/lab_key -N ""

# 2. Посмотрите содержимое публичного ключа
cat ~/.ssh/lab_key.pub

# 3. Если есть доступ к другому серверу, скопируйте ключ
ssh-copy-id -i ~/.ssh/lab_key.pub user@remote-server

# 4. Создайте SSH-конфиг
cat > ~/.ssh/config << EOF
Host lab-server
    HostName 192.168.1.100
    User devops_student
    Port 22
    IdentityFile ~/.ssh/lab_key
EOF

chmod 600 ~/.ssh/config
```

### Задание 5: Systemd

```bash
# 1. Создайте простой скрипт
cat > ~/counter.sh << 'EOF'
#!/bin/bash
while true; do
    echo "$(date): Counter is running" >> /tmp/counter.log
    sleep 60
done
EOF

chmod +x ~/counter.sh

# 2. Создайте systemd unit-файл
sudo tee /etc/systemd/system/counter.service << EOF
[Unit]
Description=Counter Service
After=network.target

[Service]
Type=simple
User=$USER
ExecStart=$HOME/counter.sh
Restart=always

[Install]
WantedBy=multi-user.target
EOF

# 3. Запустите службу
sudo systemctl daemon-reload
sudo systemctl start counter
sudo systemctl status counter

# 4. Посмотрите логи
journalctl -u counter -f
```

## Вопросы для самопроверки 🤔

1. Чем отличается UID 0 от UID 1000?
2. Что означают права 755, 644, 600?
3. Как сделать файл исполняемым для всех?
4. В чем разница между SIGTERM и SIGKILL?
5. Как посмотреть все процессы конкретного пользователя?
6. Что такое SUID и зачем он нужен?
7. Как настроить автозапуск программы при старте системы?
8. Почему нельзя использовать права 777?

## Дополнительные материалы 📚

- [Linux Permissions Guide](https://linuxhandbook.com/linux-file-permissions/)
- [Process Management in Linux](https://www.digitalocean.com/community/tutorials/process-management-in-linux)
- [Systemd Essentials](https://www.digitalocean.com/community/tutorials/systemd-essentials-working-with-services-units-and-the-journal)
- [SSH Essentials](https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys)

---

**Что дальше?** В следующей лекции мы погрузимся в написание bash-скриптов и автоматизацию повседневных задач. А пока — экспериментируйте с пользователями, правами и процессами! 🚀

> 🐧 **Совет дня:** "Всегда используйте минимально необходимые права. Если файл не нужно выполнять — не давайте прав на выполнение. Если не нужно писать — уберите запись. Безопасность строится на мелочах!"