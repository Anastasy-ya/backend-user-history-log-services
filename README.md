# postgresqlProject
_Изучение облачной базы данных и синтаксиса postgresql_

## Используется облачная база данных postgresql neon.tech

_Для подключения необходим .env файл в корневой директории приложения:_

```
USER=<your_login>

HOST=<your-host>

DATABASE=<db-name>

PASSWORD=<password>

PORT=5432
```


Строка, содержащая данные для подключения, располагается в Dashboard панели управления

```
postgres://alex:AbC123dEf@ep-cool-darkness-123456.us-east-2.aws.neon.tech/dbname
           ^    ^         ^                                               ^
     role -|    |         |- hostname                                     |- database
                |
                |- password

```

[Документация](https://neon.tech/docs/get-started-with-neon/connect-neon "Переход на сайт neon.tech")

### Скрипт для создания SQL-таблицы и заполнения ее рандомными значениями: 

```
-- Создание таблицы users
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    first_name CHAR(3) NOT NULL,
    last_name CHAR(3) NOT NULL,
    age INT CHECK (age BETWEEN 18 AND 100),
    gender CHAR(1) CHECK (gender IN ('m', 'f')),
    problems BOOLEAN
);

-- Функция для генерации случайной строки заданной длины
CREATE OR REPLACE FUNCTION random_string(length INT) RETURNS TEXT AS $$
DECLARE
    result TEXT := '';
    i INT;
BEGIN
    FOR i IN 1..length LOOP
        result := result || CHR(65 + FLOOR(RANDOM() * 26)::INT);
    END LOOP;
    RETURN result;
END;
$$ LANGUAGE plpgsql;

-- Функция для генерации случайного возраста
CREATE OR REPLACE FUNCTION random_age() RETURNS INT AS $$
BEGIN
    RETURN FLOOR(RANDOM() * 83) + 18;
END;
$$ LANGUAGE plpgsql;

-- Функция для генерации случайного пола
CREATE OR REPLACE FUNCTION random_gender() RETURNS CHAR(1) AS $$
BEGIN
    RETURN CASE WHEN RANDOM() < 0.5 THEN 'm' ELSE 'f' END;
END;
$$ LANGUAGE plpgsql;

-- Функция для генерации случайного значения для проблемы
CREATE OR REPLACE FUNCTION random_problems() RETURNS BOOLEAN AS $$
BEGIN
    RETURN RANDOM() < 0.5;
END;
$$ LANGUAGE plpgsql;

-- Функция для вставки случайного пользователя
CREATE OR REPLACE FUNCTION insert_random_person(num_users INT) RETURNS VOID AS $$
DECLARE
    i INT;
BEGIN
    FOR i IN 1..num_users LOOP
        INSERT INTO users (first_name, last_name, age, gender, problems)
        VALUES (random_string(3), random_string(3), random_age(), random_gender(), random_problems());
    END LOOP;
END;
$$ LANGUAGE plpgsql;

-- Вставка пользователей
SELECT insert_random_person(1000);
```

### Cкрипт для создания связанной таблицы с историей изменения первой таблицы

```
CREATE TABLE user_changes (
  action_id SERIAL PRIMARY KEY,
  user_id INTEGER NOT NULL,
  action_date TIMESTAMP NOT NULL,
  action VARCHAR(255) NOT NULL,
  FOREIGN KEY (user_id) REFERENCES users(id)
    ON UPDATE CASCADE
    ON DELETE CASCADE
);
```

## Запуск

1. Склонировать репозиторий:
```
git clone git@github.com:Anastasy-ya/backend-user-history-log-services.git
```

2. Перейти в папку проекта
```
cd backend-user-history-log-services
```

3. Инициализировать и обновить подмодули:
```
git submodule init
git submodule update
```

4. Перейти в каждую подпапку, установить зависимости:
```
npm install
```
5. Запустить проект в каждой подпапке:
```
npm build 
npm start
```

## Проверка postgresqlProject

### get users:

GET http://localhost:5432

### create user:

POST http://localhost:5432/create-user
<br>
```
{
    "first_name": "S6E", //string(3) required
    "last_name": "ITT", //string(3) required
    "age": 90, //int(18-100) required
    "gender": "f", //char 'm'/'f' required
    "problems": false //boolean false/true required
}
```

### change user:
_id required_

PATCH http://localhost:5432/update-user?id=200
<br>
```
{
    "first_name": "S6E", //string(3)
    "last_name": "ITT", //string(3)
    "age": 18, //int(18-100)
    "gender": "f", //char 'm'/'f'
    "problems": false //boolean false/true
}
```

### 404:

GET http://localhost:5432/sdfg

## Проверка history-log-users-server




## Планы по улучшению:
 - Написать тесты
 - Рефакторинг
 - Валидация входных значений при помощи express-validator
