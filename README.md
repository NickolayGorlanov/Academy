# 2.	Задание по основам баз данных

## Разделы

### 1.	Установить PostgreSQL (желательно в Docker)
Если через Docker - описать какие команды использовал

1. Установка PostgreSQL (в Docker)

### Установка Docker
<pre>curl -fsSL https://get.docker.com | sh</pre>

### Запуск PostgreSQL
<pre>sudo docker run -it --name academy -p 5432:5432 -e POSTGRES_PASSWORD=pswd111 postgres</pre>

### 2. Создать БД academy.
Приложить скрипт

В новом окне терминала: 
<pre>sudo docker exec -it academy bash</pre>

Подключаемся к PostgreSQL:
<pre>psql -U postgres</pre>

Создаём новую БД: 
<pre>CREATE DATABASE academy;</pre>

### 3. Добавить таблицы по приложенной схеме рис. 1 (названия полей, связи между таблицами должны соответствовать схеме, остальное (ограничения целостности, уникальность, значения по умолчанию, проверки, типы данных) на усмотрение стажера).

![Рис. 1]([[https://drive.google.com/file/d/1MGNyIreLzV-vbdEpJKLIsslqP7nPBVx2/view?usp=sharing](https://github.com/NickolayGorlanov/Academy/blob/main/BD.png)](https://github.com/NickolayGorlanov/Academy/blob/main/BD.png))
Приложить скрипт

Создаём три связанные таблицы:

<pre>-- Студенты
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
); </pre>


### 4. Добавить несколько записей в таблицы. 
Приложить скрипт

<pre>-- Удаляем все записи об экзаменах
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
</pre>
### 5. Написать запрос, который возвращает всех студентов, которые еще не сдали ни одного экзамена.
Приложить скрипт

<pre>SELECT *
FROM students
WHERE sid NOT IN (SELECT s_id FROM exams);</pre>

### 6. Написать запрос, который возвращает список студентов и количество сданных им экзаменов. Только для студентов, у которых есть сданные экзамены.
Приложить скрипт

<pre>SELECT s.sid, s.name, COUNT(e.s_id) AS exams_count
FROM students s
LEFT JOIN exams e ON s.sid = e.s_id
GROUP BY s.sid, s.name
HAVING COUNT(e.s_id) > 0; </pre>

### 7. Вывести список курсов со средним баллом по экзамену. Список отсортирован по убыванию среднего балла.
Приложить скрипт

<pre>SELECT c.cno, c.title, AVG(e.score) AS avg_score
FROM courses c
JOIN exams e ON c.cno = e.cno
GROUP BY c.cno, c.title
ORDER BY avg_score DESC; </pre>

# 1. Теория

**План тестирования интернет-магазина**

**1. Цель:**

Обеспечить высокое качество интернет-магазина путем проведения различных видов тестирования на всех этапах жизненного цикла разработки.

**2. Область применения:**

Данный план тестирования применяется к интернет-магазину, который позволяет:

- Просматривать каталог товаров;
- Добавлять товары в корзину;
- Оформлять заказ;
- Выбирать способ доставки;
- Оплачивать заказ онлайн.

**3. Этапы тестирования:**

**3.1. Анализ требований (1 неделя):**
- Изучение технического задания, требований к функционалу, дизайну и производительности.
- Определение целей тестирования, критериев приемки и приоритетов.
- Разработка плана тестирования, включающего в себя описание тестовых случаев, тестовых данных и тестового окружения.

**3.2. Тестирование юзабилити (1 неделя):**
- Проверка удобства навигации по сайту.
- Оценка понятности и доступности интерфейса.
- Тестирование поиска товаров.
- Проверка оформления заказа и оплаты.

**3.3. Функциональное тестирование (2 недели):**
- Тестирование всех функций интернет-магазина.
- Проверка корректного отображения информации на разных устройствах.
- Тестирование работы с ошибками.

**3.4. Тестирование производительности (1 неделя):**
- Проверка скорости загрузки страниц.
- Тестирование нагрузочных сценариев.
- Оценка времени отклика сервера.

**3.5. Тестирование безопасности (1 неделя):**
- Проверка на уязвимости SQL-инъекций.
- Тестирование межсайтового скриптинга (XSS).
- Проверка CSRF-атак.
- Тестирование аутентификации и авторизации.

**3.6. Приемочное тестирование (1 неделя):**
- Проверка соответствия интернет-магазина всем требованиям.
- Оформление отчета о тестировании.

**4. Ресурсы:**
- Тестировщики (2 человека);
- Тестовое окружение;
- Инструменты для тестирования (Selenium WebDriver, Postman, JMeter);
- Тестовые данные.

**5. Метрики качества:**
- Количество найденных ошибок;
- Процент пройденных тестовых случаев;
- Время выполнения тестовых сценариев;
- Уровень удовлетворенности пользователей.

**6. Управление рисками:**
- Несвоевременное предоставление тестового окружения;
- Изменения в требованиях к функционалу;
- Недостаточная квалификация тестировщиков.

**7. Отчетность:**
- По результатам каждого этапа тестирования будет создаваться отчет, содержащий:
  - Описание тестовых случаев;
  - Результаты тестирования;
  - Найденные ошибки;
  - Рекомендации по исправлению ошибок.

**8. Приложения:**
- План тестирования;
- Тестовые случаи;
- Тестовые данные;
- Отчеты о тестировании.

**9. Дополнительные сведения:**
- В данном плане тестирования даны примерные сроки выполнения работ.
- План тестирования может быть адаптирован в соответствии с конкретными условиями проекта.

**10. Используемые методы тестирования:**
- **Пирамида тестирования:**
  - Модульное тестирование;
  - Интеграционное тестирование;
  - Системное тестирование;
  - Приемочное тестирование.
- **Виды тестирования:**
  - Функциональное тестирование;
  - Тестирование юзабилити;
  - Тестирование производительности;
  - Тестирование безопасности.
- **Уровни тестирования:**
  - Уровень блоков;
  - Уровень модулей;
  - Уровень системы;
  - Уровень приемки.
- **Тест-дизайн:**
  - Эквивалентное разбиение;
  - Анализ граничных значений;
  - Табличное тестирование.
- **Тестовая документация:**
  - План тестирования;
  - Тестовые случаи;
  - Отчеты о тестировании.
- **Метрики качества:**
  - Количество найденных ошибок;
  - Процент пройденных тестовых случаев;
  - Время выполнения тестовых сценариев;
  - Уровень удовлетворенности пользователей.

**Оценка трудоемкости:**

Оценка трудоемкости тестирования будет проведена на основе следующих факторов:
- Объем функциональности;
- Сложность функциональности;
- Уровень риска;
- Квалификация тестировщиков.

**20. Управление рисками:**

Для управления рисками будут предприняты следующие меры:
- Регулярное проведение совещаний с командой разработки;
- Использование инструментов для управления рисками;
- Разработка плана действий на случай непредвиденных обстоятельств.

**21. Анализ требований:**

Анализ требований будет проведен с использованием следующих методов:
- Изучение технического задания;
- Проведение интервью с разработчиками;
- Проведение обзоров требований.

**22. Инструменты тестирования:**

Для тестирования будут использоваться следующие инструменты:
- Selenium WebDriver;
- Postman;
- JMeter;
- Jira;
- TestRail.



**Тестирование функционала:**

1. **Просмотр каталога товаров:**
   - Проверка отображения всех доступных категорий и подкатегорий товаров.
   - Проверка корректности отображения изображений, описания и цен товаров.
   - Тестирование поиска по каталогу на соответствие ожидаемым результатам.
   - Проверка фильтрации товаров по различным параметрам (цена, бренд, размер и т. д.).

2. **Добавление товаров в корзину:**
   - Проверка возможности добавления товаров в корзину из каталога товаров.
   - Проверка корректности отображения выбранных товаров в корзине.
   - Тестирование функционала управления количеством товаров в корзине.
   - Проверка расчета общей суммы заказа в корзине.

3. **Оформление заказа:**
   - Тестирование процесса оформления заказа от выбора товаров в корзине до подтверждения заказа.
   - Проверка корректности заполнения формы заказа (адрес доставки, контактная информация и т. д.).
   - Проверка возможности выбора способа доставки и оплаты заказа.
   - Тестирование генерации подтверждения заказа и отправки уведомлений покупателю.

4. **Выбор способа доставки:**
   - Проверка доступности различных вариантов доставки (курьерская доставка, самовывоз и т. д.).
   - Тестирование расчета сроков и стоимости доставки в зависимости от выбранных параметров.

5. **Оплата заказа онлайн:**
   - Тестирование интеграции с платежными системами для обеспечения возможности оплаты заказа онлайн.
   - Проверка корректности отображения информации о выбранном способе оплаты.
   - Тестирование процесса обработки платежа и получения подтверждения об успешной оплате.

6. **Тестирование мобильной версии:**
   - Проверка корректности отображения и функционала интернет-магазина на мобильных устройствах.
   - Тестирование адаптивного дизайна и удобства использования на разных разрешениях экрана.
   - Проверка работы основных функций на мобильном устройстве (добавление товаров в корзину, оформление заказа и т. д.).

7. **Тестирование работы с ошибками:**
   - Тестирование обработки ошибок при некорректном заполнении форм, отсутствии товаров в каталоге и т. д.
   - Проверка корректности отображения информации об ошибках и предоставления пользователю понятных инструкций для их устранения.
   - Тестирование функционала обратной связи для сообщения о возникших проблемах.

**Тестирование производительности:**

1. **Проверка скорости загрузки страниц:**
   - Использование инструментов для анализа скорости загрузки страниц (например, Google PageSpeed Insights, GTmetrix).
   - Замер времени загрузки основных страниц интернет-магазина (главная страница, страницы категорий товаров, страницы товаров, страница оформления заказа).
   - Сравнение полученных результатов с требованиями к производительности и рекомендациями по оптимизации.

2. **Тестирование нагрузочных сценариев:**
   - Разработка сценариев нагрузочного тестирования, моделирующих типичные действия пользователей (просмотр каталога, добавление товаров в корзину, оформление заказа и т. д.).
   - Запуск нагрузочного тестирования с использованием инструментов для создания нагрузки на сервер (например, Apache JMeter).
   - Мониторинг системы во время тестирования для выявления узких мест и проблем с производительностью.

3. **Оценка времени отклика сервера:**
   - Использование инструментов для мониторинга времени отклика сервера (например, New Relic, Datadog).
   - Замер времени ответа сервера на запросы при различных нагрузках.
   - Оценка времени отклика сервера в зависимости от количества одновременных пользователей и объема данных.

Эти мероприятия помогут выявить и устранить проблемы с производительностью интернет-магазина, обеспечивая быстрый и эффективный доступ к ресурсам для пользователей.

**Тестирование безопасности:**

1. **Проверка на уязвимости SQL-инъекций:**
   - Проведение специально созданных запросов к базе данных для выявления уязвимостей.
   - Анализ реакции системы на подобные запросы и выявление возможных уязвимостей.
   - Проверка правильности фильтрации и санитизации пользовательского ввода перед выполнением SQL-запросов.

2. **Тестирование межсайтового скриптинга (XSS):**
   - Внедрение вредоносного кода через поля ввода на веб-страницах (например, формы поиска или комментариев).
   - Проверка корректности фильтрации и экранирования пользовательского ввода для предотвращения XSS-атак.

3. **Проверка CSRF-атак:**
   - Попытка выполнения нежелательных действий от имени авторизованного пользователя (например, изменение его данных или отправка нежелательных запросов на сервер).
   - Проверка корректности использования механизмов защиты от CSRF-атак, таких как использование токенов сессии или проверка Referer заголовка.

4. **Тестирование аутентификации и авторизации:**
   - Проверка корректности процесса аутентификации пользователей (логин, пароль).
   - Проверка ограничений доступа к различным функциям интернет-магазина в зависимости от прав пользователя.
   - Тестирование механизмов управления сессиями и безопасности паролей.



**23. Заключение:**

Данный план тестирования является базовым и может быть адаптирован в соответствии с конкретными условиями проекта. Для достижения наилучших результатов тестирования рекомендуется привлекать к работе опытных тестировщиков.







