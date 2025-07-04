# Django Todolist - Docker Compose Instructions

## Передумови
- Docker та Docker Compose встановлені
- Git для роботи з репозиторієм

## Швидкий старт

### 1. Клонування та перехід в папку
```bash
git clone https://github.com/YOUR_USERNAME/devops_todolist_docker_core_task_3_docker_compose.git
cd devops_todolist_docker_core_task_3_docker_compose
```

### 2. Запуск контейнерів
```bash
docker-compose up -d
```

### 3. Доступ до додатка
- Додаток: http://localhost:8000
- API: http://localhost:8000/api/
- Admin: http://localhost:8000/admin/

## Команди управління

### Запуск контейнерів
```bash
# В фоновому режимі
docker-compose up -d

# З виводом логів
docker-compose up

# Перебудова образів
docker-compose up --build
```

### Зупинка контейнерів
```bash
# Зупинка сервісів
docker-compose stop

# Повне видалення контейнерів
docker-compose down

# Видалення контейнерів та volumes (БД буде очищена!)
docker-compose down -v
```

### Моніторинг

#### Статус контейнерів
```bash
docker-compose ps
```

#### Перегляд логів
```bash
# Всі сервіси
docker-compose logs

# Конкретний сервіс
docker-compose logs web
docker-compose logs db

# Логи в реальному часі
docker-compose logs -f web
```

### Django команди

#### Створення суперкористувача
```bash
docker-compose exec web python manage.py createsuperuser
```

#### Міграції
```bash
# Створити міграції
docker-compose exec web python manage.py makemigrations

# Застосувати міграції
docker-compose exec web python manage.py migrate
```

#### Django shell
```bash
docker-compose exec web python manage.py shell
```

### Робота з базою даних

#### Підключення до MySQL
```bash
docker-compose exec db mysql -u todouser -p todolist
```

#### Бекап БД
```bash
docker-compose exec db mysqldump -u todouser -p todolist > backup.sql
```

#### Відновлення БД
```bash
docker-compose exec -T db mysql -u todouser -p todolist < backup.sql
```

## Архітектура

### Контейнери
- **web**: Django додаток (порт 8000)
- **db**: MySQL 8.0 (порт 3306)

### Мережі
- **todolist_network**: Внутрішня мережа для контейнерів

### Volumes
- **mysql_data**: Постійне зберігання даних MySQL

### Environment Variables
- `DB_HOST=db` - хост бази даних
- `DB_NAME=todolist` - назва БД
- `DB_USER=todouser` - користувач БД
- `DB_PASSWORD=todopass` - пароль БД
- `DB_PORT=3306` - порт БД

## Усунення проблем

### 1. Контейнери не запускаються
```bash
# Перевірити статус
docker-compose ps

# Переглянути логи
docker-compose logs

# Перезапустити
docker-compose restart
```

### 2. Помилки підключення до БД
```bash
# Перевірити чи працює MySQL
docker-compose logs db

# Перевірити змінні оточення
docker-compose exec web env | grep DB_

# Перезапустити тільки web сервіс
docker-compose restart web
```

### 3. Помилки міграцій
```bash
# Виконати міграції вручну
docker-compose exec web python manage.py migrate

# Переглянути статус міграцій
docker-compose exec web python manage.py showmigrations
```

### 4. Порти зайняті
Якщо порти 8000 або 3306 зайняті, змініть у docker-compose.yml:
```yaml
ports:
  - "8080:8000"  # Замість 8000:8000
  - "3307:3306"  # Замість 3306:3306
```

### 5. Очистка даних
```bash
# Видалити всі дані (увага: БД буде очищена!)
docker-compose down -v
docker-compose up -d
```

## Розробка

### Зміни в коді
После изменений в коде:
```bash
# Перезапустити web контейнер
docker-compose restart web

# Або пересобрать образ
docker-compose up --build web
```

### Додавання нових пакетів
1. Оновіть requirements.txt
2. Пересоберіть образ:
```bash
docker-compose build web
docker-compose up -d
```

## Тестування

### Перевірка працездатності
```bash
# Перевірити доступність API
curl http://localhost:8000/api/

# Перевірити головну сторінку
curl http://localhost:8000/

# Перевірити статус контейнерів
docker-compose ps
```

### Здоров'я БД
```bash
# Підключитися до MySQL
docker-compose exec db mysql -u todouser -p todolist

# Переглянути таблиці
docker-compose exec db mysql -u todouser -p todolist -e "SHOW TABLES;"
```

## Безпека

### Для продакшену
- Змініть паролі в docker-compose.yml
- Прибрати порт 3306 якщо не потрібен зовнішній доступ
- Використовуйте Docker secrets для паролів

### Приклад безпечних налаштувань
```yaml
environment:
  - MYSQL_ROOT_PASSWORD_FILE=/run/secrets/db_root_password
  - MYSQL_PASSWORD_FILE=/run/secrets/db_password
secrets:
  db_root_password:
    file: ./secrets/db_root_password.txt
  db_password:
    file: ./secrets/db_password.txt
```