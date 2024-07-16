*Read this in [Russian](README.rus.md)*

# backend-user-history-log-services
_Learning postgresql_

## Postgresql neon.tech data base is using

## Running

1. Clone:
```
git clone git@github.com:Anastasy-ya/backend-user-history-log-services.git
```

2. Navigate to folder
```
cd backend-user-history-log-services
```

3. To initialize and update submodules:
```
git submodule init
git submodule update
```

4. To navigate to each subfolder and install dependencies:
```
npm install
```
5. To run the project in each subfolder:
```
git pull origin main
npm run build (except postgresqlProject)
npm run start
```

## POSTMAN collection for testing 
https://github.com/Anastasy-ya/BigDB/blob/main/servers.postman_collection.json

## postgresqlProject

JavaScript - Express.js - cors - node-postgres - express-rate-limit

### Ссылка:
1.https://github.com/Anastasy-ya/postgresqlProject
<br>
_Works with the table <persons> returning a list of users,
creates a new one or updates an existing one,
sending the change log to <person_changes>_


### get users:

GET http://localhost:3001

### create user:

POST http://localhost:3001/create-user
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

PATCH http://localhost:3001/update-user?id=200
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


## history-log-users-server

TypeScript - Express.js - cors - node-postgres - express-rate-limit

### Ссылка:
https://github.com/Anastasy-ya/history-log-users-server
<br>
_Works with the table <person_changes>_
_and returns the change history of the <persons> table_

### get users:

GET http://localhost:5432

### get one user:

GET http://localhost:5432/user?id=200

### 404:

GET http://localhost:5432/sdfg


## bigDB

TypeScript - Nest.js - TypeORM - node-postgres

_Works with a database containing over 1 million records._
_Returns the count of users with Problems: true and changes the flag to false._

### Ссылка:
1.https://github.com/Anastasy-ya/BigDB
<br>

### reset problems:

PATCH http://localhost:3000/users/reset-problems

### 404:

GET http://localhost:3000/404


The .env file is required in the root directory of the application for the connection:

```
USER=<your_login>

HOST=<your-host>

DATABASE=<db-name>

PASSWORD=<password>

PORT=5432
```


The connection string is located in the Dashboard control panel

```
postgres://alex:AbC123dEf@ep-cool-darkness-123456.us-east-2.aws.neon.tech/dbname
           ^    ^         ^                                               ^
     role -|    |         |- hostname                                     |- database
                |
                |- password

```

[Docs](https://neon.tech/docs/get-started-with-neon/connect-neon "Переход на сайт neon.tech")

### The script to create an SQL table and populate it with random values: 

```
-- Create table users
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    first_name CHAR(3) NOT NULL,
    last_name CHAR(3) NOT NULL,
    age INT CHECK (age BETWEEN 18 AND 100),
    gender CHAR(1) CHECK (gender IN ('m', 'f')),
    problems BOOLEAN
);

-- The function to generate a random string.
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

-- The function to generate a random age.
CREATE OR REPLACE FUNCTION random_age() RETURNS INT AS $$
BEGIN
    RETURN FLOOR(RANDOM() * 83) + 18;
END;
$$ LANGUAGE plpgsql;

-- The function to generate a random gender.
CREATE OR REPLACE FUNCTION random_gender() RETURNS CHAR(1) AS $$
BEGIN
    RETURN CASE WHEN RANDOM() < 0.5 THEN 'm' ELSE 'f' END;
END;
$$ LANGUAGE plpgsql;

-- The function to generate a random value for a problem
CREATE OR REPLACE FUNCTION random_problems() RETURNS BOOLEAN AS $$
BEGIN
    RETURN RANDOM() < 0.5;
END;
$$ LANGUAGE plpgsql;

-- The function to insert a random user
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

-- insert users
SELECT insert_random_person(1000);
```

### The script to create a related table that tracks the change history of the primary table

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
