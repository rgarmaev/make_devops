# Лекция 3: Bash-скриптинг и автоматизация в Linux 🐧

## Содержание
1. [Введение в Bash-скриптинг](#введение-в-bash-скриптинг)
2. [Переменные и типы данных](#переменные-и-типы-данных)
3. [Условные операторы](#условные-операторы)
4. [Циклы](#циклы)
5. [Функции](#функции)
6. [Работа с аргументами](#работа-с-аргументами)
7. [Обработка ошибок](#обработка-ошибок)
8. [Практические примеры](#практические-примеры)
9. [Практическое задание](#практическое-задание)

---

## Введение в Bash-скриптинг 📜

Bash (Bourne Again Shell) — это не только командная оболочка, но и мощный язык программирования для автоматизации задач в Linux.

### Почему Bash важен для DevOps?

- **Автоматизация рутинных задач** — бэкапы, мониторинг, деплой
- **Инфраструктура как код** — скрипты для настройки серверов
- **CI/CD пайплайны** — многие этапы пишутся на bash
- **Прототипирование** — быстрая проверка идей

### Структура bash-скрипта

```bash
#!/bin/bash
# Первая строка — shebang, указывает какой интерпретатор использовать

# Это комментарий
echo "Hello, World!"  # вывод текста
```

### Создание и запуск скрипта

```bash
# 1. Создаем файл
nano myscript.sh

# 2. Добавляем shebang и код
#!/bin/bash
echo "My first script"

# 3. Делаем исполняемым
chmod +x myscript.sh

# 4. Запускаем
./myscript.sh
# или
bash myscript.sh
```

## Переменные и типы данных 📦

### Объявление переменных

```bash
#!/bin/bash

# Присваивание (без пробелов вокруг =)
name="John"
age=25
is_devops=true

# Использование переменных (знак $)
echo "Name: $name"
echo "Age: $age"
echo "Is DevOps: $is_devops"

# Фигурные скобки для явного указания
echo "${name} is $age years old"

# Команда в переменную
current_date=$(date)
current_dir=`pwd`  # устаревший синтаксис, лучше использовать $()
echo "Date: $current_date"
echo "Dir: $current_dir"
```

### Константы

```bash
#!/bin/bash

# readonly - неизменяемая переменная
readonly PI=3.14159
PI=3.14  # Ошибка! readonly variable

# declare -r тоже создает константу
declare -r GITHUB_URL="https://github.com"
```

### Специальные переменные

```bash
#!/bin/bash
echo "Название скрипта: $0"
echo "Первый аргумент: $1"
echo "Второй аргумент: $2"
echo "Все аргументы: $@"
echo "Количество аргументов: $#"
echo "PID текущего процесса: $$"
echo "Код возврата последней команды: $?"
```

### Массивы

```bash
#!/bin/bash

# Объявление массива
fruits=("apple" "banana" "orange")
numbers=(1 2 3 4 5)

# Доступ к элементам
echo "Первый фрукт: ${fruits[0]}"
echo "Все фрукты: ${fruits[@]}"
echo "Количество: ${#fruits[@]}"

# Добавление элемента
fruits+=("grape")

# Цикл по массиву
for fruit in "${fruits[@]}"; do
    echo "I like $fruit"
done

# Ассоциативные массивы (требуют Bash 4+)
declare -A user
user[name]="John"
user[age]=30
user[city]="New York"
echo "User: ${user[name]}, ${user[age]}, ${user[city]}"
```

## Условные операторы 🔀

### Конструкция if-else

```bash
#!/bin/bash

# Базовый синтаксис
if [ условие ]; then
    # команды
elif [ другое_условие ]; then
    # другие команды
else
    # команды по умолчанию
fi

# Примеры
age=18

if [ $age -ge 18 ]; then
    echo "You are an adult"
else
    echo "You are a minor"
fi
```

### Операторы сравнения

**Для чисел:**
```bash
-eq  # равно (equal)
-ne  # не равно (not equal)
-gt  # больше (greater than)
-lt  # меньше (less than)
-ge  # больше или равно (greater or equal)
-le  # меньше или равно (less or equal)

# Пример
if [ $count -gt 10 ]; then
    echo "Count > 10"
fi
```

**Для строк:**
```bash
=    # строки равны
!=   # строки не равны
-z   # строка пустая (zero length)
-n   # строка не пустая (non-zero)

# Пример
name="John"
if [ -n "$name" ]; then
    echo "Name is not empty"
fi

if [ "$name" = "John" ]; then
    echo "Hello, John!"
fi
```

**Файловые операторы:**
```bash
-f file  # существует и это файл
-d dir   # существует и это директория
-e file  # существует
-r file  # доступен для чтения
-w file  # доступен для записи
-x file  # доступен для выполнения
-s file  # не пустой
-nt      # новее чем (newer than)
-ot      # старше чем (older than)

# Примеры
if [ -f "/etc/passwd" ]; then
    echo "File exists"
fi

if [ -d "/home/user" ]; then
    echo "Directory exists"
fi

if [ -r "config.txt" ] && [ -w "config.txt" ]; then
    echo "We can read and write config"
fi
```

### Логические операторы

```bash
&&  # И (AND)
||  # ИЛИ (OR)
!   # НЕ (NOT)

# Примеры
if [ $age -ge 18 ] && [ $has_license = "yes" ]; then
    echo "You can drive"
fi

if [ $day = "saturday" ] || [ $day = "sunday" ]; then
    echo "It's weekend!"
fi

if [ ! -f "error.log" ]; then
    echo "No errors found"
fi
```

### Конструкция case

```bash
#!/bin/bash

case $1 in
    start)
        echo "Starting service..."
        ;;
    stop)
        echo "Stopping service..."
        ;;
    restart|reload)  # несколько вариантов
        echo "Restarting service..."
        ;;
    *)
        echo "Usage: $0 {start|stop|restart}"
        exit 1
        ;;
esac
```

## Циклы 🔄

### Цикл for

```bash
#!/bin/bash

# По списку
for fruit in apple banana orange; do
    echo "Fruit: $fruit"
done

# С диапазоном
for i in {1..5}; do
    echo "Number: $i"
done

# С шагом
for i in {0..10..2}; do
    echo "Even: $i"
done

# C-подобный синтаксис
for ((i=0; i<5; i++)); do
    echo "i = $i"
done

# По файлам
for file in *.txt; do
    echo "Processing $file"
    wc -l "$file"
done

# По выводу команды
for user in $(cut -d: -f1 /etc/passwd); do
    echo "User: $user"
done
```

### Цикл while

```bash
#!/bin/bash

# Счетчик
counter=1
while [ $counter -le 5 ]; do
    echo "Counter: $counter"
    ((counter++))
done

# Чтение файла построчно
while IFS= read -r line; do
    echo "Line: $line"
done < /etc/passwd

# Бесконечный цикл
while true; do
    echo "Press Ctrl+C to stop"
    sleep 1
done

# Чтение вывода команды
ps aux | while read -r line; do
    if [[ $line == *"bash"* ]]; then
        echo "Found bash: $line"
    fi
done
```

### Цикл until

```bash
#!/bin/bash

# until выполняется ПОКА условие ложно
counter=1
until [ $counter -gt 5 ]; do
    echo "Counter: $counter"
    ((counter++))
done

# Ожидание доступности сервера
until ping -c1 google.com &>/dev/null; do
    echo "Waiting for internet..."
    sleep 5
done
echo "Internet is up!"
```

### Управление циклами

```bash
#!/bin/bash

# break - прерывание цикла
for i in {1..10}; do
    if [ $i -eq 5 ]; then
        break
    fi
    echo "i = $i"
done

# continue - переход к следующей итерации
for i in {1..10}; do
    if [ $((i % 2)) -eq 0 ]; then
        continue  # пропускаем четные
    fi
    echo "Odd: $i"
done
```

## Функции 🔧

### Определение функций

```bash
#!/bin/bash

# Способ 1
function greet {
    echo "Hello, $1!"
}

# Способ 2 (более переносимый)
say_hello() {
    local name=$1  # local - локальная переменная
    echo "Hello, $name!"
}

# Вызов функций
greet "John"
say_hello "Jane"

# Функция с возвратом значения
get_sum() {
    local a=$1
    local b=$2
    echo $((a + b))  # возврат через echo
    # return использует только для кодов ошибок (0-255)
}

result=$(get_sum 5 3)
echo "Sum: $result"
```

### Область видимости переменных

```bash
#!/bin/bash

global_var="I'm global"

my_function() {
    local local_var="I'm local"
    global_var="Modified global"
    echo "Inside function: $local_var"
    echo "Inside function: $global_var"
}

my_function
echo "Outside: $global_var"
echo "Outside: $local_var"  # пусто, локальная не видна
```

### Функции с аргументами

```bash
#!/bin/bash

print_info() {
    local name=$1
    local age=$2
    local city=${3:-"Unknown"}  # значение по умолчанию
    
    echo "Name: $name"
    echo "Age: $age"
    echo "City: $city"
}

print_info "John" 30 "New York"
print_info "Jane" 25  # city будет "Unknown"

# Функция с переменным числом аргументов
sum_all() {
    local sum=0
    for num in "$@"; do
        sum=$((sum + num))
    done
    echo $sum
}

total=$(sum_all 1 2 3 4 5)
echo "Total: $total"
```

## Работа с аргументами командной строки 🎯

### Обработка аргументов

```bash
#!/bin/bash

# Простой способ
echo "Script: $0"
echo "First arg: $1"
echo "Second arg: $2"
echo "All args: $@"
echo "Number of args: $#"

# Перебор аргументов
for arg in "$@"; do
    echo "Argument: $arg"
done
```

### Использование getopts для опций

```bash
#!/bin/bash

usage() {
    echo "Usage: $0 [-n name] [-a age] [-h]"
    exit 1
}

while getopts "n:a:h" opt; do
    case $opt in
        n)
            name=$OPTARG
            ;;
        a)
            age=$OPTARG
            ;;
        h)
            usage
            ;;
        \?)
            echo "Invalid option: -$OPTARG"
            usage
            ;;
    esac
done

echo "Name: ${name:-Not provided}"
echo "Age: ${age:-Not provided}"
```

### Расширенная обработка с shift

```bash
#!/bin/bash

while [ $# -gt 0 ]; do
    case $1 in
        --name)
            name=$2
            shift 2
            ;;
        --age)
            age=$2
            shift 2
            ;;
        --verbose)
            verbose=true
            shift
            ;;
        --help)
            echo "Usage: $0 [--name NAME] [--age AGE] [--verbose]"
            exit 0
            ;;
        *)
            echo "Unknown option: $1"
            exit 1
            ;;
    esac
done

echo "Name: $name"
echo "Age: $age"
echo "Verbose: ${verbose:-false}"
```

## Обработка ошибок 🚨

### Проверка кода возврата

```bash
#!/bin/bash

# Каждая команда возвращает код (0 - успех, не 0 - ошибка)
ls /nonexistent
if [ $? -eq 0 ]; then
    echo "Command succeeded"
else
    echo "Command failed with code: $?"
fi

# Более элегантный способ
if mkdir /test 2>/dev/null; then
    echo "Directory created"
else
    echo "Failed to create directory"
fi
```

### Установка опций оболочки

```bash
#!/bin/bash

# Прерывать скрипт при любой ошибке
set -e

# Прерывать при использовании необъявленных переменных
set -u

# Прерывать при ошибке в пайпе
set -o pipefail

# Выводить команды перед выполнением (для отладки)
set -x

# Можно комбинировать
set -euo pipefail

# Временное отключение
set +x  # отключить отладку
```

### Создание функций для обработки ошибок

```bash
#!/bin/bash

error_exit() {
    echo "Error: $1" >&2  # вывод в stderr
    exit 1
}

# Использование
file="config.txt"
[ -f "$file" ] || error_exit "File $file not found"

# trap - перехват сигналов
cleanup() {
    echo "Cleaning up..."
    rm -f /tmp/tempfile
    exit
}

trap cleanup EXIT INT TERM  # выполнить cleanup при выходе
```

## Практические примеры 📋

### Пример 1: Бэкап директории

```bash
#!/bin/bash
set -euo pipefail

# Конфигурация
BACKUP_DIR="/backups"
SOURCE_DIR="/home/user/documents"
RETENTION_DAYS=7
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="backup_${TIMESTAMP}.tar.gz"

# Создание директории для бэкапов
mkdir -p "$BACKUP_DIR"

# Создание архива
echo "Creating backup: $BACKUP_FILE"
tar -czf "${BACKUP_DIR}/${BACKUP_FILE}" "$SOURCE_DIR"

# Проверка результата
if [ $? -eq 0 ]; then
    echo "Backup created successfully"
    ls -lh "${BACKUP_DIR}/${BACKUP_FILE}"
else
    echo "Backup failed" >&2
    exit 1
fi

# Удаление старых бэкапов
echo "Removing backups older than $RETENTION_DAYS days"
find "$BACKUP_DIR" -name "backup_*.tar.gz" -mtime +$RETENTION_DAYS -delete
```

### Пример 2: Мониторинг сервера

```bash
#!/bin/bash

# Цвета для вывода
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

check_cpu() {
    local cpu_usage=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
    echo "CPU Usage: $cpu_usage%"
    
    if (( $(echo "$cpu_usage > 80" | bc -l) )); then
        echo -e "${RED}High CPU usage!${NC}"
    fi
}

check_memory() {
    local mem_usage=$(free | grep Mem | awk '{print ($3/$2) * 100.0}')
    echo "Memory Usage: ${mem_usage}%"
    
    if (( $(echo "$mem_usage > 80" | bc -l) )); then
        echo -e "${RED}High memory usage!${NC}"
    fi
}

check_disk() {
    local disk_usage=$(df -h / | awk 'NR==2 {print $5}' | cut -d'%' -f1)
    echo "Disk Usage: $disk_usage%"
    
    if [ $disk_usage -gt 80 ]; then
        echo -e "${RED}Low disk space!${NC}"
    fi
}

check_services() {
    local services=("nginx" "postgresql" "docker")
    
    for service in "${services[@]}"; do
        if systemctl is-active --quiet "$service"; then
            echo -e "${GREEN}$service is running${NC}"
        else
            echo -e "${RED}$service is not running${NC}"
        fi
    done
}

# Основная функция
main() {
    echo "=== Server Health Check ==="
    echo "Time: $(date)"
    echo "Host: $(hostname)"
    echo "------------------------"
    
    check_cpu
    check_memory
    check_disk
    echo "------------------------"
    check_services
}

main
```

### Пример 3: Деплой приложения

```bash
#!/bin/bash
set -euo pipefail

# Цветной вывод
info() { echo -e "\033[0;32m[INFO]\033[0m $1"; }
error() { echo -e "\033[0;31m[ERROR]\033[0m $1" >&2; }
warn() { echo -e "\033[1;33m[WARN]\033[0m $1"; }

# Конфигурация
APP_NAME="myapp"
REPO_URL="git@github.com:user/${APP_NAME}.git"
DEPLOY_DIR="/var/www/${APP_NAME}"
BACKUP_DIR="/backups/${APP_NAME}"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

# Проверка зависимостей
check_dependencies() {
    local deps=("git" "npm" "pm2")
    for dep in "${deps[@]}"; do
        if ! command -v "$dep" &> /dev/null; then
            error "$dep is not installed"
            exit 1
        fi
    done
}

# Создание бэкапа
create_backup() {
    if [ -d "$DEPLOY_DIR" ]; then
        info "Creating backup..."
        mkdir -p "$BACKUP_DIR"
        tar -czf "${BACKUP_DIR}/${APP_NAME}_${TIMESTAMP}.tar.gz" \
            -C "$(dirname "$DEPLOY_DIR")" "$(basename "$DEPLOY_DIR")"
        info "Backup created: ${BACKUP_DIR}/${APP_NAME}_${TIMESTAMP}.tar.gz"
    fi
}

# Клонирование/обновление кода
update_code() {
    if [ ! -d "$DEPLOY_DIR" ]; then
        info "Cloning repository..."
        git clone "$REPO_URL" "$DEPLOY_DIR"
    else
        info "Updating code..."
        cd "$DEPLOY_DIR"
        git pull origin main
    fi
}

# Установка зависимостей
install_deps() {
    info "Installing dependencies..."
    cd "$DEPLOY_DIR"
    npm ci --production  # чистая установка из lock-файла
}

# Сборка приложения
build_app() {
    info "Building application..."
    cd "$DEPLOY_DIR"
    npm run build
}

# Запуск/перезапуск приложения
restart_app() {
    info "Restarting application..."
    cd "$DEPLOY_DIR"
    
    if pm2 list | grep -q "$APP_NAME"; then
        pm2 reload "$APP_NAME"
    else
        pm2 start npm --name "$APP_NAME" -- start
    fi
    
    pm2 save
}

# Проверка работоспособности
health_check() {
    info "Performing health check..."
    sleep 5  # даем время на запуск
    
    local max_retries=5
    local retry=0
    
    while [ $retry -lt $max_retries ]; do
        if curl -s -f http://localhost:3000/health > /dev/null; then
            info "Health check passed!"
            return 0
        fi
        warn "Health check failed, retrying in 2 seconds... ($((retry+1))/$max_retries)"
        sleep 2
        ((retry++))
    done
    
    error "Health check failed after $max_retries attempts"
    return 1
}

# Откат при неудаче
rollback() {
    warn "Rolling back to previous version..."
    
    # Находим последний бэкап
    local latest_backup=$(ls -t "${BACKUP_DIR}"/*.tar.gz 2>/dev/null | head -1)
    
    if [ -n "$latest_backup" ]; then
        rm -rf "$DEPLOY_DIR"
        tar -xzf "$latest_backup" -C "$(dirname "$DEPLOY_DIR")"
        info "Rollback completed"
        restart_app
    else
        error "No backup found for rollback"
        exit 1
    fi
}

# Основная функция деплоя
deploy() {
    info "Starting deployment of $APP_NAME"
    
    check_dependencies
    create_backup
    update_code
    install_deps
    build_app
    restart_app
    
    if ! health_check; then
        rollback
        exit 1
    fi
    
    info "Deployment completed successfully!"
}

# Обработка ошибок
trap 'error "Deployment failed at line $LINENO"; rollback; exit 1' ERR

# Запуск
deploy
```

## Практическое задание 📝

### Задание 1: Создание скрипта для анализа логов

```bash
#!/bin/bash
# Создайте скрипт, который анализирует файл лога и выводит статистику:

# 1. Создайте тестовый лог-файл
cat > /tmp/test.log << 'EOF'
2024-01-15 10:15:30 INFO Application started
2024-01-15 10:15:31 DEBUG Loading config
2024-01-15 10:15:32 ERROR Database connection failed
2024-01-15 10:15:33 INFO Retry connection
2024-01-15 10:15:35 ERROR Database timeout
2024-01-15 10:15:36 INFO Connected successfully
EOF

# 2. Напишите скрипт log-analyzer.sh, который:
#   - Принимает имя файла как аргумент
#   - Показывает общее количество строк
#   - Группирует по уровням (INFO, DEBUG, ERROR)
#   - Показывает последние 5 ошибок
#   - Находит самые частые сообщения
```

### Задание 2: Скрипт для управления пользователями

```bash
#!/bin/bash
# Создайте скрипт user-manager.sh с функциями:
#   - add_user USERNAME - создает пользователя
#   - delete_user USERNAME - удаляет пользователя
#   - list_users - показывает всех пользователей
#   - backup_homes - создает бэкап домашних директорий
```

### Задание 3: Мониторинг ресурсов

```bash
#!/bin/bash
# Создайте скрипт resource-monitor.sh, который:
#   - Каждые 5 секунд проверяет CPU, память, диск
#   - Записывает в лог, если превышены пороги (CPU>80%, память>80%, диск>90%)
#   - Отправляет уведомление (можно просто echo)
#   - Работает до нажатия Ctrl+C
```

### Задание 4: Парсер конфигов

```bash
#!/bin/bash
# Создайте скрипт config-parser.sh, который:
#   - Читает файл конфигурации в формате KEY=VALUE
#   - Пропускает комментарии (#)
#   - Позволяет получить значение по ключу: ./config-parser.sh config.txt KEY
#   - Позволяет установить новое значение: ./config-parser.sh config.txt KEY VALUE
```

## Дополнительные материалы 📚

- [Bash Guide for Beginners](https://tldp.org/LDP/Bash-Beginners-Guide/html/)
- [Advanced Bash-Scripting Guide](https://tldp.org/LDP/abs/html/)
- [ShellCheck](https://www.shellcheck.net/) — линтер для bash-скриптов
- [Bash Cheat Sheet](https://devhints.io/bash)
- [ExplainShell](https://explainshell.com/) — объяснение команд

---

**Что дальше?** В следующей лекции мы перейдем к сетевой подсистеме Linux — изучим сетевые интерфейсы, настройку сетей, брандмауэры и устранение неполадок. А пока — пишите скрипты и автоматизируйте всё, что автоматизируется! 🚀

> 🐧 **Совет дня:** "Хороший скрипт — это не только рабочий код, но и понятные сообщения об ошибках, обработка нестандартных ситуаций и документация. Всегда представляйте, что ваш скрипт будут читать другие (или вы через полгода)."