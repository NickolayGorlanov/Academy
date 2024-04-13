# 2.	Задание по основам баз данных

## Разделы

### 1.	Установить PostgreSQL (желательно в Docker)
Если через Docker - описать какие команды использовал

1. Установка PostgreSQL (в Docker)

# Установка Docker
curl -fsSL https://get.docker.com | sh

# Запуск PostgreSQL
sudo docker run -it --name academy -p 5432:5432 -e POSTGRES_PASSWORD=pswd111 postgres  

### 2. Создать БД academy.
Приложить скрипт

В новом окне терминала: 
sudo docker exec -it academy bash

Подключаемся к PostgreSQL:
psql -U postgres

Создаём новую БД: 
CREATE DATABASE academy;

### 3. Добавить таблицы по приложенной схеме рис. 1 (названия полей, связи между таблицами должны соответствовать схеме, остальное (ограничения целостности, уникальность, значения по умолчанию, проверки, типы данных) на усмотрение стажера).
Приложить скрипт

Создаём три связанные таблицы:

-- Студенты
CREATE TABLE students (
    sid SERIAL PRIMARY KEY,
    s_id INTEGER UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    start_year INTEGER NOT NULL
);


-- Курсы
CREATE TABLE courses (
    cno SERIAL PRIMARY KEY,
    c_no INTEGER UNIQUE NOT NULL,
    title VARCHAR(255) NOT NULL,
    hours INTEGER NOT NULL
);


-- Экзамены
CREATE TABLE exams (
    s_id INTEGER NOT NULL,
    cno INTEGER NOT NULL,
    score INTEGER NOT NULL,
    FOREIGN KEY (s_id) REFERENCES students (sid),
    FOREIGN KEY (cno) REFERENCES courses (cno)
);


### 4. Добавить несколько записей в таблицы. 
Приложить скрипт

-- Удаляем все записи об экзаменах
DELETE FROM exams;

-- Вставляем случайные данные в таблицу students
INSERT INTO students (s_id, name, start_year)
SELECT 
    subquery.s_id,
    'Student ' || subquery.s_id,
    2020 + (random() * 5)::int 
FROM (
    SELECT 1000 + row_number() OVER () as s_id
    FROM generate_series(1, 100)
) AS subquery
LEFT JOIN students ON subquery.s_id = students.s_id
WHERE students.s_id IS NULL;

-- Вставляем случайные данные в таблицу courses
INSERT INTO courses (c_no, title, hours)
SELECT 
    subquery.c_no,
    'Course ' || subquery.c_no,
    20 + (random() * 30)::int 
FROM (
    SELECT 2000 + row_number() OVER () as c_no
    FROM generate_series(1, 5) -- Ограничиваем количество добавляемых курсов до 5
) AS subquery
LEFT JOIN courses ON subquery.c_no = courses.c_no
WHERE courses.c_no IS NULL;

-- Добавляем новые экзамены только для студентов, которые еще не сдали ни одного экзамена
INSERT INTO exams (s_id, cno, score)
SELECT 
    students.sid,
    courses.cno,
    (random() * 100)::int 
FROM (
    SELECT sid FROM students
    WHERE sid NOT IN (SELECT DISTINCT s_id FROM exams)
    LIMIT 80 -- Ограничиваем количество добавляемых студентов
) AS students
CROSS JOIN LATERAL (
    SELECT cno FROM courses ORDER BY random() LIMIT 5
) AS courses;

### 5. Написать запрос, который возвращает всех студентов, которые еще не сдали ни одного экзамена.
Приложить скрипт

SELECT *
FROM students
WHERE sid NOT IN (SELECT s_id FROM exams);

### 6. Написать запрос, который возвращает список студентов и количество сданных им экзаменов. Только для студентов, у которых есть сданные экзамены.
Приложить скрипт

SELECT s.sid, s.name, COUNT(e.s_id) AS exams_count
FROM students s
LEFT JOIN exams e ON s.sid = e.s_id
GROUP BY s.sid, s.name
HAVING COUNT(e.s_id) > 0;

### 7. Вывести список курсов со средним баллом по экзамену. Список отсортирован по убыванию среднего балла.
Приложить скрипт

SELECT c.cno, c.title, AVG(e.score) AS avg_score
FROM courses c
JOIN exams e ON c.cno = e.cno
GROUP BY c.cno, c.title
ORDER BY avg_score DESC;






