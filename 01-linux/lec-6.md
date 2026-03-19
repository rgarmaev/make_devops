

# Лекция 6: Управление службами с systemd 🚀

## Содержание
1. [Введение в systemd](#введение-в-systemd)
2. [Основы управления службами](#основы-управления-службами)
3. [Структура unit-файлов](#структура-unit-файлов)
4. [Создание своего unit-файла](#создание-своего-unit-файла)
5. [Работа с логами (journald)](#работа-с-логами-journald)
6. [Таймеры вместо cron](#таймеры-вместо-cron)
7. [Целевые состояния (targets)](#целевые-состояния-targets)
8. [Практическое задание](#практическое-задание)

---

## Введение в systemd 🏗️

### Что такое systemd?

**systemd** — это система инициализации и менеджер служб для Linux, ставший стандартом де-факто в большинстве дистрибутивов.

```bash
# Проверить, использует ли ваша система systemd
ps -p 1
# Если вы видите "systemd" — значит используется systemd
#   PID TTY          TIME CMD
#     1 ?        00:00:03 systemd
```

### Почему systemd важен для DevOps?

- **Управление сервисами** — запуск, остановка, перезапуск приложений
- **Автозапуск** — контроль что стартует при загрузке
- **Мониторинг** — systemd следит за процессами и перезапускает их при падении
- **Логирование** — централизованные логи через journald
- **Изоляция** — ограничение ресурсов для сервисов
- **Зависимости** — управление порядком запуска

### Основные компоненты systemd

```bash
systemd                      # главный процесс (PID 1)
systemctl                    # основная команда управления
journalctl                   # просмотр логов
systemd-analyze              # анализ загрузки
systemd-tmpfiles             # управление временными файлами
systemd-timesyncd            # синхронизация времени
systemd-resolved             # DNS-резолвер
systemd-networkd             # управление сетью
```

### Где живут unit-файлы?

```bash
# Системные unit-файлы (от пакетов)
/lib/systemd/system/          # Debian/Ubuntu
/usr/lib/systemd/system/      # CentOS/RHEL

# Пользовательские unit-файлы (приоритет выше)
/etc/systemd/system/           # системные override
/etc/systemd/system/multi-user.target.wants/  # симлинки для автозапуска

# Пользовательские unit-файлы (для конкретного пользователя)
~/.config/systemd/user/        # юзер-специфичные службы
```

## Основы управления службами 🎮

### Базовые команды systemctl

```bash
# Просмотр статуса
systemctl status nginx         # статус конкретной службы
systemctl status --lines=20 nginx  # показать 20 строк лога

systemctl is-active nginx      # проверить, активна ли (yes/no)
systemctl is-enabled nginx     # проверить, включена ли автозагрузка
systemctl is-failed nginx      # проверить, упала ли служба

# Управление службой
sudo systemctl start nginx     # запустить
sudo systemctl stop nginx      # остановить
sudo systemctl restart nginx   # перезапустить
sudo systemctl reload nginx    # перечитать конфиг (без остановки)
sudo systemctl reload-or-restart nginx  # reload если можно, иначе restart

# Автозагрузка
sudo systemctl enable nginx    # добавить в автозагрузку
sudo systemctl disable nginx   # убрать из автозагрузки
sudo systemctl enable --now nginx  # включить и сразу запустить

# Более жесткие команды
sudo systemctl kill nginx      # послать сигнал процессу
sudo systemctl kill -s SIGKILL nginx  # конкретный сигнал
sudo systemctl kill --kill-who=main nginx  # убить только главный процесс
```

### Просмотр всех служб

```bash
# Все активные службы
systemctl list-units --type=service

# Все службы (включая неактивные)
systemctl list-units --type=service --all

# Все установленные службы (с состоянием автозагрузки)
systemctl list-unit-files --type=service

# Поиск службы по имени
systemctl list-units --type=service | grep nginx

# Недавние действия со службами
systemctl list-units --state=failed  # упавшие службы
systemctl list-units --state=running # только запущенные
```

### Управление зависимостями

```bash
# Посмотреть зависимости службы
systemctl list-dependencies nginx
systemctl list-dependencies --reverse nginx  # кто зависит от этой службы

# Маскировка (полное отключение)
sudo systemctl mask nginx       # запретить запуск (создает симлинк в /dev/null)
sudo systemctl unmask nginx     # разрешить снова

# Сброс упавшей службы
sudo systemctl reset-failed nginx  # сбросить счетчик падений
```

## Структура unit-файлов 📄

### Базовый синтаксис

Unit-файлы состоят из секций и директив в формате `INI`:

```ini
[Unit]
Description=My Service
Documentation=https://example.com
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/myapp
User=myuser
Group=mygroup

[Install]
WantedBy=multi-user.target
```

### Секция [Unit]

Общая информация и зависимости:

```ini
[Unit]
Description=Краткое описание службы
Documentation=man:myservice(8)  # ссылка на документацию
Documentation=https://example.com

# Порядок запуска
After=network.target              # запустить ПОСЛЕ network.target
Before=nginx.service              # запустить ДО nginx
Requires=postgresql.service       # требует postgresql (жесткая зависимость)
Wants=redis.service                # хочет redis (мягкая зависимость)

# Конфликты
Conflicts=apache2.service          # не может работать одновременно с apache2

# Условия запуска
ConditionPathExists=/etc/myapp.conf  # запускать только если файл существует
ConditionPathIsDirectory=/var/lib/myapp  # только если директория существует
ConditionHost=server01              # только на определенном хосте
ConditionVirtualization=!docker     # не запускать в Docker
```

### Секция [Service]

Настройки самого сервиса:

```ini
[Service]
# Тип сервиса (ВАЖНО!)
Type=simple              # (по умолчанию) процесс не форкается
Type=forking             # процесс форкается (демоны)
Type=oneshot             # одноразовая задача (запустился и завершился)
Type=notify              # уведомляет systemd о готовности (через sd_notify)
Type=dbus                # готов, когда появляется на D-Bus
Type=idle                # запускается, когда нечем заняться

# Команды запуска/остановки
ExecStart=/usr/bin/myapp --config /etc/myapp.conf
ExecStartPre=/bin/chown -R myuser:mygroup /var/lib/myapp  # перед запуском
ExecStartPost=/usr/bin/logger "MyApp started"             # после запуска
ExecStop=/usr/bin/kill $MAINPID                            # команда остановки
ExecReload=/bin/kill -HUP $MAINPID                         # команда перезагрузки

# Окружение
Environment=MYAPP_ENV=production
EnvironmentFile=-/etc/default/myapp  # файл с переменными (- игнорирует отсутствие)
WorkingDirectory=/var/lib/myapp       # рабочая директория
RootDirectory=/srv/chroot             # chroot окружение

# Пользователь
User=myuser
Group=mygroup
SupplementaryGroups=docker,www-data   # дополнительные группы

# Ресурсы (ограничения)
CPUQuota=50%                          # не больше 50% CPU
MemoryMax=1G                           # максимум 1GB памяти
MemoryHigh=512M                        # мягкий лимит
TasksMax=100                           # максимум процессов

# Рестарт
Restart=on-failure                      # перезапускать при падении
RestartSec=5s                           # ждать 5 секунд перед рестартом
StartLimitBurst=3                        # максимум попыток рестарта
StartLimitIntervalSec=60                  # за какой период

# Сигналы
KillMode=control-group                   # убить всю группу процессов (по умолч.)
KillMode=mixed                            # послать SIGTERM главному, потом всем
KillMode=process                          # убить только главный процесс
KillSignal=SIGTERM                        # какой сигнал посылать
TimeoutStopSec=30s                        # время на остановку

# Выходные потоки
StandardOutput=journal                   # stdout в journald
StandardError=journal                    # stderr в journald
StandardInput=null
# Другие варианты: file:/path, syslog, kmsg, null, tty
```

### Секция [Install]

Настройки автозагрузки:

```ini
[Install]
# Куда привязать службу
WantedBy=multi-user.target    # стандартный уровень для серверов
RequiredBy=graphical.target   # обязательный для graphical.target
Also=secondary.service        # установить/удалить вместе с этой службой
Alias=myapp.service           # альтернативное имя
```

### Типы сервисов в деталях

**Type=simple** (самый распространенный):
```ini
[Service]
Type=simple
ExecStart=/usr/bin/python3 /opt/app/server.py
# systemd считает службу запущенной сразу после execve()
# Подходит для большинства современных приложений
```

**Type=forking** (для традиционных демонов):
```ini
[Service]
Type=forking
ExecStart=/usr/sbin/nginx
PIDFile=/run/nginx.pid
# systemd ждет, пока процесс форкнется и запишет PID в файл
# Используется для nginx, Apache, sendmail
```

**Type=oneshot** (одноразовые задачи):
```ini
[Service]
Type=oneshot
ExecStart=/usr/bin/backup.sh
RemainAfterExit=yes   # считать активным даже после завершения
# Для скриптов, которые выполняются один раз
# Например: создание директорий, очистка кеша
```

**Type=notify** (современный способ):
```ini
[Service]
Type=notify
ExecStart=/usr/bin/node app.js
# Приложение само уведомляет systemd, что готово
# Требует поддержки sd_notify() в приложении
# Используется в Docker, Podman, современных сервисах
```

## Создание своего unit-файла 🛠️

### Пример 1: Простой Python-скрипт

Создадим простое веб-приложение на Python:

```python
# /opt/myapp/app.py
#!/usr/bin/env python3
from http.server import HTTPServer, BaseHTTPRequestHandler
import time
import sys
import os

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.end_headers()
        self.wfile.write(b"Hello from MyApp! Time: " + 
                        time.strftime("%Y-%m-%d %H:%M:%S").encode())

if __name__ == '__main__':
    host = os.getenv('HOST', '0.0.0.0')
    port = int(os.getenv('PORT', '8000'))
    print(f"Starting server on {host}:{port}")
    sys.stdout.flush()  # важно для буферизации логов
    
    try:
        server = HTTPServer((host, port), Handler)
        server.serve_forever()
    except KeyboardInterrupt:
        print("Shutting down...")
        server.shutdown()
```

Создадим unit-файл:

```bash
sudo nano /etc/systemd/system/myapp.service
```

```ini
[Unit]
Description=My Python Web Application
Documentation=https://example.com/docs
After=network.target
Wants=postgresql.service  # хотим БД, но не обязательно

[Service]
Type=simple
User=www-data
Group=www-data
WorkingDirectory=/opt/myapp
Environment="HOST=0.0.0.0"
Environment="PORT=8000"
EnvironmentFile=-/etc/default/myapp  # дополнительные переменные
ExecStart=/usr/bin/python3 /opt/myapp/app.py
Restart=on-failure
RestartSec=10
StartLimitBurst=5
StartLimitIntervalSec=60

# Безопасность
NoNewPrivileges=yes
PrivateTmp=yes
ProtectSystem=strict
ProtectHome=yes
ReadWritePaths=/var/log/myapp

# Лимиты
CPUQuota=50%
MemoryMax=512M
TasksMax=50

[Install]
WantedBy=multi-user.target
```

Создадим директорию для логов:

```bash
sudo mkdir -p /var/log/myapp
sudo chown www-data:www-data /var/log/myapp
```

Запустим наш сервис:

```bash
sudo systemctl daemon-reload      # перечитать unit-файлы
sudo systemctl enable --now myapp
sudo systemctl status myapp
curl localhost:8000               # проверим
```

### Пример 2: Docker-контейнер через systemd

Часто нужно управлять Docker-контейнерами через systemd:

```ini
[Unit]
Description=My Docker Container
After=docker.service
Requires=docker.service

[Service]
Type=simple
User=root
ExecStartPre=-/usr/bin/docker stop myapp
ExecStartPre=-/usr/bin/docker rm myapp
ExecStart=/usr/bin/docker run --rm --name myapp -p 8080:80 myapp:latest
ExecStop=/usr/bin/docker stop myapp
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

### Пример 3: Одноразовая задача (oneshot)

```ini
[Unit]
Description=Initialize Application Directories
Before=myapp.service

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/local/bin/init-dirs.sh
ExecStart=/bin/chown -R myapp:myapp /var/lib/myapp
User=root

[Install]
WantedBy=multi-user.target
```

## Работа с логами (journald) 📝

### Основы journalctl

```bash
# Просмотр логов
journalctl                      # все логи (может быть много)
journalctl -n 50                 # последние 50 строк
journalctl -f                    # следить в реальном времени (tail -f)

# По конкретной службе
journalctl -u nginx               # логи службы nginx
journalctl -u nginx -f            # следить за логами nginx
journalctl -u nginx --since "1 hour ago"  # за последний час
journalctl -u nginx --since today --until "2024-01-01 12:00"

# По времени
journalctl --since "2024-01-01 00:00:00"
journalctl --since "yesterday"
journalctl --since "1 hour ago" --until "30 minutes ago"

# По приоритету
journalctl -p err                  # только ошибки (и выше)
journalctl -p warning               # предупреждения и выше
# Приоритеты: emerg, alert, crit, err, warning, notice, info, debug

# Вывод
journalctl -u nginx -o json-pretty  # в JSON формате
journalctl -u nginx --output=verbose # детальный вывод
journalctl -u nginx --no-pager       # не использовать pager
```

### Продвинутый поиск

```bash
# Поиск по сообщению
journalctl -u nginx | grep "error"

# По полям
journalctl -u nginx _PID=1234       # логи от конкретного PID
journalctl -u nginx _UID=33          # логи от конкретного UID

# По хосту (для кластера)
journalctl _HOSTNAME=server1

# Комбинации
journalctl -u nginx -u postgresql --since today  # логи двух служб

# Объем логов
journalctl --disk-usage              # сколько места занимают логи
```

### Настройка journald

```bash
# Конфигурация
sudo nano /etc/systemd/journald.conf

# Важные параметры:
SystemMaxUse=1G                    # максимум места на диске
SystemMaxFileSize=100M              # максимальный размер одного файла
MaxRetentionSec=1month              # хранить максимум месяц
Compress=yes                         # сжимать логи
ForwardToSyslog=no                   # не дублировать в syslog

# Применить изменения
sudo systemctl restart systemd-journald

# Очистка старых логов
sudo journalctl --vacuum-size=500M   # оставить только 500МБ
sudo journalctl --vacuum-time=7days  # оставить только за 7 дней
```

## Таймеры вместо cron ⏰

### Что такое systemd-таймеры?

Таймеры — это замена cron с дополнительными возможностями:
- Зависят от других unit'ов
- Можно пропускать пропущенные запуски
- Логирование через journald
- Точный контроль времени

### Структура таймера

Создадим задачу для бэкапа:

**1. Сначала сам сервис (что делать):**
```ini
# /etc/systemd/system/backup.service
[Unit]
Description=Daily Backup Service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh
User=backup
Group=backup
```

**2. Потом таймер (когда делать):**
```ini
# /etc/systemd/system/backup.timer
[Unit]
Description=Run backup daily
Requires=backup.service

[Timer]
# Разные способы задания времени
OnCalendar=daily                    # ежедневно в полночь
OnCalendar=*-*-* 02:00:00           # каждый день в 2 часа ночи
OnCalendar=Mon..Fri 01:00:00        # по будням в час ночи
OnCalendar=Sat,Sun 03:00:00          # по выходным в 3 часа

# Альтернативные способы
OnBootSec=10min                      # через 10 минут после загрузки
OnUnitActiveSec=1d                    # через день после последнего запуска
Persistent=true                        # если пропущен запуск, выполнить сразу

# Точность
AccuracySec=1min                       # точность ±1 минута
RandomizedDelaySec=5min                 # случайная задержка (чтобы не все сразу)

[Install]
WantedBy=timers.target
```

### Управление таймерами

```bash
# Включение и запуск
sudo systemctl enable backup.timer
sudo systemctl start backup.timer

# Просмотр таймеров
systemctl list-timers                  # все активные таймеры
systemctl list-timers --all             # включая неактивные

# Статус конкретного таймера
systemctl status backup.timer

# Проверка следующих запусков
systemctl list-timers backup.timer

# Запустить вручную (игнорируя таймер)
sudo systemctl start backup.service
```

### Пример: очистка логов по таймеру

```ini
# /etc/systemd/system/clean-logs.service
[Unit]
Description=Clean old logs

[Service]
Type=oneshot
ExecStart=/usr/bin/find /var/log -name "*.log" -mtime +30 -delete
ExecStart=/usr/bin/journalctl --vacuum-size=500M
User=root
```

```ini
# /etc/systemd/system/clean-logs.timer
[Unit]
Description=Clean logs weekly

[Timer]
OnCalendar=weekly
Persistent=true

[Install]
WantedBy=timers.target
```

## Целевые состояния (targets) 🎯

### Что такое target?

Target — это группа служб, которая должна быть активна для определенного состояния системы.

```bash
# Основные targets
poweroff.target          # выключение
rescue.target            # однопользовательский режим
multi-user.target        # многопользовательский (без GUI)
graphical.target         # с графическим интерфейсом
reboot.target            # перезагрузка

# Просмотр текущего target
systemctl get-default

# Смена default target
sudo systemctl set-default multi-user.target  # серверный режим
sudo systemctl set-default graphical.target   # для десктопа

# Переключение между targets
sudo systemctl isolate multi-user.target  # перейти в режим без GUI
```

### Создание своего target

```ini
# /etc/systemd/system/devops.target
[Unit]
Description=DevOps Services
Requires=multi-user.target
After=multi-user.target
Wants=docker.service prometheus.service

[Install]
WantedBy=multi-user.target
```

## Мониторинг и отладка systemd 🔍

### Анализ загрузки

```bash
# Время загрузки
systemd-analyze
# Output: Startup finished in 2.345s (kernel) + 3.456s (initrd) + 12.345s (userspace)

# График загрузки
systemd-analyze blame           # кто сколько тормозил
systemd-analyze critical-chain  # критическая цепочка
systemd-analyze plot > boot.svg  # визуализация (открыть в браузере)
```

### Отладка проблем

```bash
# Если служба не запускается
systemctl status failing-service
journalctl -u failing-service -n 50

# Проверка синтаксиса unit-файла
systemd-analyze verify /etc/systemd/system/myapp.service

# Что слушает процесс?
systemctl show myapp.service    # все параметры службы

# Включить отладку для конкретной службы
sudo systemctl edit myapp.service
# Добавить:
[Service]
Environment=SYSTEMD_LOG_LEVEL=debug
```

### Инструменты мониторинга

```bash
# Нагрузка на systemd
systemd-cgtop                    # как top, но для cgroups

# Информация о ресурсах
systemd-cgls                      # дерево cgroups

# Показать все юниты
systemctl list-units --all --full
```

## Практическое задание 📝

### Задание 1: Создание простого сервиса

Создайте сервис для Node.js приложения:

```bash
# 1. Создайте простое приложение
mkdir -p ~/nodeapp
cat > ~/nodeapp/app.js << 'EOF'
const http = require('http');
const server = http.createServer((req, res) => {
    res.writeHead(200);
    res.end('Hello from Node.js service!\n');
});
server.listen(3000, () => {
    console.log('Server running on port 3000');
});
EOF

# 2. Создайте unit-файл
sudo nano /etc/systemd/system/nodeapp.service

# 3. Заполните unit-файл (см. требования ниже)
# Требования:
# - Type=simple
# - User=youruser
# - WorkingDirectory=/home/youruser/nodeapp
# - Restart=on-failure
# - Ограничить память до 256MB
# - Ограничить CPU до 25%

# 4. Запустите сервис
sudo systemctl daemon-reload
sudo systemctl enable --now nodeapp

# 5. Проверьте работу
curl localhost:3000
journalctl -u nodeapp -f
```

### Задание 2: Таймер для бэкапа

Создайте таймер для ежедневного бэкапа:

```bash
# 1. Создайте скрипт бэкапа
sudo nano /usr/local/bin/backup.sh
#!/bin/bash
BACKUP_DIR="/backups/$(date +%Y%m%d)"
mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/etc.tar.gz" /etc
echo "Backup completed at $(date)" >> /var/log/backup.log

sudo chmod +x /usr/local/bin/backup.sh

# 2. Создайте сервис (backup.service)
# Type=oneshot
# ExecStart=/usr/local/bin/backup.sh

# 3. Создайте таймер (backup.timer)
# OnCalendar=daily
# Persistent=true

# 4. Запустите
sudo systemctl enable --now backup.timer

# 5. Проверьте
systemctl list-timers backup.timer
```

### Задание 3: Работа с логами

```bash
# 1. Посмотрите логи SSH за сегодня
journalctl -u ssh --since today

# 2. Найдите все ошибки в логах за последний час
journalctl -p err --since "1 hour ago"

# 3. Посмотрите логи между 10 и 11 утра
journalctl --since "10:00" --until "11:00"

# 4. Выведите логи в JSON и найдите что-нибудь
journalctl -u nginx -o json-pretty | grep -i error

# 5. Очистите логи старше 2 недель
sudo journalctl --vacuum-time=2weeks
```

### Задание 4: Анализ загрузки

```bash
# 1. Проверьте общее время загрузки
systemd-analyze

# 2. Найдите самые медленные службы
systemd-analyze blame | head -10

# 3. Постройте граф зависимостей для ssh
systemctl list-dependencies ssh

# 4. Проверьте, можно ли отключить ненужные службы
systemctl list-unit-files --state=enabled | grep -v systemd
```

### Задание 5: Создание сложного сервиса с зависимостями

Создайте сервис для веб-приложения, который:

```ini
# 1. Зависит от PostgreSQL и Redis
# 2. Запускается после них
# 3. Имеет переменные окружения из файла
# 4. Перезапускается при падении, но не чаще 3 раз за 5 минут
# 5. Ограничен по памяти и CPU
# 6. Пишет логи в отдельный файл
# 7. Имеет ExecStop для корректного завершения
```

## Шпаргалка по systemd 📋

| Команда | Назначение |
|---------|------------|
| `systemctl start service` | Запустить службу |
| `systemctl stop service` | Остановить службу |
| `systemctl restart service` | Перезапустить службу |
| `systemctl reload service` | Перезагрузить конфиг |
| `systemctl enable service` | Включить автозагрузку |
| `systemctl disable service` | Отключить автозагрузку |
| `systemctl status service` | Статус службы |
| `systemctl is-active service` | Проверить активность |
| `systemctl list-units --type=service` | Все службы |
| `journalctl -u service` | Логи службы |
| `journalctl -f` | Следить за логами |
| `systemd-analyze blame` | Время загрузки |
| `systemctl list-timers` | Таймеры |
| `systemctl cat service` | Посмотреть unit-файл |
| `systemctl edit service` | Отредактировать |
| `systemctl daemon-reload` | Перечитать unit-файлы |

## Вопросы для самопроверки 🤔

1. В чем разница между `Type=simple` и `Type=forking`?
2. Что делает директива `Restart=on-failure`?
3. Как посмотреть логи конкретной службы за последний час?
4. Чем systemd-таймеры лучше cron?
5. Что такое target и зачем он нужен?
6. Как ограничить потребление памяти для службы?
7. Что произойдет при `systemctl daemon-reload`?
8. Как сделать так, чтобы служба запускалась после PostgreSQL?

## Дополнительные материалы 📚

- [Systemd Documentation](https://systemd.io/) — официальная документация
- [systemd for Administrators](http://0pointer.de/blog/projects/systemd-for-admins-1.html) — серия статей
- [Creating systemd service files](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/chap-managing_services_with_systemd)
- [Systemd Timers vs Cron](https://systemd.io/TIMERS/)

---

> 🐧 **Совет дня:** "Systemd — это не просто система инициализации, это целая экосистема. Научившись писать свои unit-файлы и работать с journald, вы сможете профессионально управлять любыми сервисами в Linux. Это основа для понимания контейнеризации и оркестрации!"