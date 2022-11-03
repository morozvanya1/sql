# Запросы SQL к связанным таблицам

## Связи между таблицами
### Задание 1
create table author (
    author_id INT PRIMARY KEY AUTO_INCREMENT, 
    name_author VARCHAR(50)
);

### Задание 2
insert into author (name_author) values ('Булгаков М.А.'), 
('Достоевский Ф.М.'), 
('Есенин С.А.'), 
('Пастернак Б.Л.');

### Задание 3
CREATE TABLE book (
    book_id INT PRIMARY KEY AUTO_INCREMENT, 
    title VARCHAR(50), 
    author_id INT NOT NULL, 
    genre_id INT, 
    price DECIMAL(8,2), 
    amount INT, 
    FOREIGN KEY (author_id)  REFERENCES author (author_id), 
    FOREIGN KEY (genre_id)  REFERENCES genre (genre_id) 
);

### Задание 4
CREATE TABLE book (
    book_id INT PRIMARY KEY AUTO_INCREMENT, 
    title VARCHAR(50), 
    author_id INT NOT NULL, 
    genre_id INT, 
    price DECIMAL(8,2), 
    amount INT, 
    FOREIGN KEY (author_id)  REFERENCES author (author_id) ON DELETE CASCADE, 
    FOREIGN KEY (genre_id)  REFERENCES genre (genre_id) ON DELETE SET NULL
);

### Задание 5
insert into book (title, author_id, genre_id, price, amount)
values ('Стихотворения и поэмы', 3, 2, '650.00', 15),
('Черный человек', 3, 2, '570.20', 6),
('Лирика', 4, 2, '518.99', 2);

## Запросы на выборку, соединение таблиц
### Соединение INNER JOIN
SELECT title, name_genre, price
FROM 
    book INNER JOIN genre
    ON book.genre_id = genre.genre_id
where book.amount > 8 order by price desc;

### Внешнее соединение LEFT и RIGHT OUTER JOIN
SELECT name_genre 
FROM genre LEFT JOIN book ON genre.genre_id = book.genre_id
where book.genre_id is null;

### Перекрестное соединение CROSS JOIN
select name_city, name_author, (DATE_ADD('2020-01-01', INTERVAL FLOOR(RAND() * 365) DAY)) as 'Дата'
from city, author order by name_city, (DATE_ADD('2020-01-01', INTERVAL FLOOR(RAND() * 365) DAY)) desc;

### Запросы на выборку из нескольких таблиц
select name_genre, title, name_author from genre g 
join book b on g.genre_id = b.genre_id
join author a on a.author_id = b.author_id where name_genre = 'Роман' order by title;

### Запросы для нескольких таблиц с группировкой
SELECT name_author, sum(amount) AS 'Количество'
FROM 
    author left join book
    on author.author_id = book.author_id
GROUP BY name_author having sum(amount) < 10 or sum(amount) is null
ORDER BY sum(amount);

### Запросы для нескольких таблиц со вложенными запросами
select name_author from author join book
    on author.author_id = book.author_id
GROUP BY name_author
HAVING count(distinct genre_id) = 1;

### Вложенные запросы в операторах соединения
SELECT title, name_author, name_genre, price, amount
FROM 
    author 
    INNER JOIN book ON author.author_id = book.author_id
    INNER JOIN genre ON  book.genre_id = genre.genre_id
GROUP BY title, name_author, price, amount, genre.genre_id
HAVING genre.genre_id IN
         (SELECT query_in_1.genre_id
          FROM 
              (
                SELECT genre_id, SUM(amount) AS sum_amount
                FROM book
                GROUP BY genre_id
               ) query_in_1
          INNER JOIN 
              (SELECT genre_id, SUM(amount) AS sum_amount
                FROM book
                GROUP BY genre_id
                ORDER BY sum_amount DESC
                LIMIT 1
               ) query_in_2
          ON query_in_1.sum_amount = query_in_2.sum_amount
         )
order by title;

### Операция соединение, использование USING()
SELECT book.title as 'Название', name_author as 'Автор', sum(book.amount) + sum(supply.amount) as 'Количество'
FROM 
    author 
    INNER JOIN book USING (author_id)   
    INNER JOIN supply ON book.title = supply.title 
                         and author.name_author = supply.author
where supply.price = book.price
group by book.title, name_author;

## Запросы корректировки, соединение таблиц
### Запросы на обновление, связанные таблицы
UPDATE book
     INNER JOIN author ON author.author_id = book.author_id
     INNER JOIN supply ON book.title = supply.title 
                         and supply.author = author.name_author
SET book.amount = book.amount + supply.amount,
    book.price = (book.amount * book.price + supply.amount * supply.price) / (book.amount + supply.amount),
    supply.amount = 0   
WHERE book.price != supply.price;

### Запросы на добавление, связанные таблицы
insert into author (name_author)
SELECT supply.author
FROM 
    author 
    RIGHT JOIN supply on author.name_author = supply.author
WHERE name_author IS Null;

### Запрос на добавление, связанные таблицы
insert into book (title, author_id, price, amount)
SELECT title, author_id, price, amount
FROM 
    author 
    INNER JOIN supply ON author.name_author = supply.author
WHERE amount <> 0;

### Запрос на обновление, вложенные запросы
update book
set genre_id = 
      (
       SELECT genre_id 
       FROM genre 
       WHERE name_genre = 'Поэзия'
      )
WHERE title = 'Стихотворения и поэмы';

update book
set genre_id = 
      (
       SELECT genre_id 
       FROM genre 
       WHERE name_genre = 'Приключения'
      )
WHERE title = 'Остров сокровищ';

### Каскадное удаление записей связанных таблиц
DELETE FROM author
WHERE author_id in (select author_id from book group by author_id having sum(amount) < 20);

### Удаление записей главной таблицы с сохранением записей в зависимой
DELETE FROM genre
WHERE genre_id in (select genre_id from book group by genre_id having count(genre_id) < 4);

### Удаление записей, использование связанных таблиц
DELETE FROM author
USING 
    author 
    INNER JOIN book ON author.author_id = book.author_id
    INNER JOIN genre using(genre_id)
WHERE genre.name_genre = 'Поэзия';

## База данных «Интернет-магазин книг», запросы на выборку
### Запросы на основе трех и более связанных таблиц
SELECT buy_book.buy_id, title, price, buy_book.amount
FROM 
    client 
    INNER JOIN buy ON client.client_id = buy.client_id
    INNER JOIN buy_book ON buy_book.buy_id = buy.buy_id
    INNER JOIN book ON buy_book.book_id=book.book_id
WHERE name_client = 'Баранов Павел'
order by buy_book.buy_id, book.title;

### Задание 1
select name_author, title, count(buy_book.book_id) as 'Количество' from book 
left join buy_book using (book_id)
join author using (author_id)
group by name_author, title, buy_book.book_id 
order by name_author, title;

### Задание 2
select name_city, count(buy.client_id) as 'Количество' from client 
join buy using (client_id)
join city using (city_id)
group by name_city, buy.client_id
order by count(buy.client_id) desc, name_city;

### Задание 3
select buy_id, date_step_end
from buy_step join step using (step_id)
where date_step_end is not null and step_id = 1;

### Задание 4
select buy_book.buy_id, name_client, sum(buy_book.amount * book.price) as 'Стоимость' from buy_book
join buy using (buy_id)
join book using (book_id)
join client using (client_id)
group by buy_book.buy_id, name_client
order by buy_book.buy_id;

### Задание 5
select buy_id, name_step from buy_step
join step using (step_id)
where date_step_end is null and date_step_beg is not null
order by buy_id;

### Задание 6
select buy_id, datediff(date_step_end, date_step_beg) as 'Количество_дней',
GREATEST(datediff(date_step_end, date_step_beg) - city.days_delivery, 0) as 'Опоздание'
from buy_step join buy using (buy_id)
join step using (step_id)
join client using (client_id)
join city using (city_id)
where name_step = 'Транспортировка' and date_step_end is not null
order by buy_id;

### Задание 7
select distinct name_client from buy_book join buy using (buy_id)
join client using (client_id)
join book using (book_id)
join author using (author_id)
where name_author = 'Достоевский Ф.М.'
order by name_client;

### Задание 8
select name_genre, sum(buy_book.amount) as 'Количество'
from buy_book join book using (book_id)
join genre using (genre_id)
group by name_genre
order by sum(buy_book.amount) desc limit 1;

### Задание 9
select name_genre, sum(buy_book.amount) as 'Количество'
from buy_book join book using (book_id)
join genre using (genre_id)
group by name_genre
HAVING SUM(buy_book.amount) = 
    (SELECT MAX(sum_amount) 
            FROM 
            (SELECT SUM(buy_book.amount) AS sum_amount
                FROM book 
             INNER JOIN buy_book USING(book_id)
             GROUP BY genre_id) query_in)
order by sum(buy_book.amount) desc;

### Задание 10
SELECT YEAR(date_payment) as Год, MONTHNAME(date_payment) as Месяц, sum(amount * price) as 'Сумма'
FROM buy_archive
group by YEAR(date_payment), MONTHNAME(date_payment)
union
SELECT YEAR(buy_step.date_step_end) as Год, MONTHNAME(buy_step.date_step_end) as Месяц, sum(buy_book.amount * price) as 'Сумма'
FROM book 
    INNER JOIN buy_book USING(book_id)
    INNER JOIN buy USING(buy_id) 
    INNER JOIN buy_step USING(buy_id)
    INNER JOIN step USING(step_id)                  
WHERE  date_step_end IS NOT Null and name_step = "Оплата"
group by YEAR(buy_step.date_step_end), MONTHNAME(buy_step.date_step_end)
order by Месяц, Год;

### Задание 11
select title, sum(Количество) as Количество, sum(Сумма) as Сумма from (
    SELECT title, sum(buy_archive.amount) as Количество, sum(buy_archive.price * buy_archive.amount) as Сумма
    FROM buy_archive join book using (book_id)
    group by title
    union all
    select title, sum(buy_book.amount) as Количество, sum(book.price * buy_book.amount) as Сумма
    from book join buy_book using (book_id)
    join buy using (buy_id)
    join buy_step using (buy_id)
    join step USING(step_id)  
    where date_step_end is not null and name_step = "Оплата"
    group by title
) query_in 
group by title order by Сумма desc;

## База данных «Интернет-магазин книг», запросы корректировки
### 