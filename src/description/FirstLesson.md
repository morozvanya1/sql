# Основы реляционной модели и SQL

## Создание таблицы
create table book(
    book_id INT PRIMARY KEY AUTO_INCREMENT, 
    title VARCHAR(50),
    author varchar(30),
    price DECIMAL(8, 2),
    amount int
);

## Вставка записи в таблицу
insert into book values ("1","Мастер и Маргарита","Булгаков М.А.","670.99","3");

insert into book values ("2","Белая гвардия","Булгаков М.А.","540.50","5"),
("3","Идиот","Достоевский Ф.М.","460.00","10"),
("4","Братья Карамазовы","Достоевский Ф.М.","799.01","2");

## Выборка данных
### Выборка всех данных из таблицы
select * from book;

### Выборка отдельных столбцов
select author, title, price from book;

### Выборка новых столбцов и присвоение им новых имен
select title as 'Название', author as 'Автор' from book;

### Выборка данных с созданием вычисляемого столбца
select title, amount, amount * 1.65 as 'pack' from book;

### Выборка данных, вычисляемые столбцы, математические функции
select title, author, amount, ROUND(price * 0.7, 2) as new_price from book;

### Выборка данных, вычисляемые столбцы, логические функции
select author, title, 
    ROUND(if (author = 'Булгаков М.А.', price + price * 0.1, 
             if (author = 'Есенин С.А.', price + price * 0.05, price)), 2) as new_price
from book;

### Выборка данных по условию
select author, title, price from book where amount < 10;

### Выборка данных, логические операции
select title, author, price, amount from book 
where (price < 500 or price > 600) and (price * amount) >= 5000;

### Выборка данных, операторы BETWEEN, IN
select title, author from book 
where (price between 540.50 and 800) and amount in (2, 3, 5, 7);

### Выборка данных с сортировкой
select author, title from book
where amount >= 2 and amount <= 14
order by author desc, title;

### Выборка данных, оператор LIKE
select title, author from book
where title like '%_ _%' and author like '%С.%'
order by title;

## Запросы, групповые операции
### Выбор уникальных элементов столбца
select distinct amount from book;

### Выборка данных, групповые функции SUM и COUNT
select author as 'Автор', count(author) as 'Различных_книг', sum(amount) as 'Количество_экземпляров'
from book group by author;

### Выборка данных, групповые функции MIN, MAX и AVG
select author, min(price) as 'Минимальная_цена', max(price) as 'Максимальная_цена', 
avg(price) as 'Средняя_цена' from book group by author;

### Выборка данных c вычислением, групповые функции
select author, sum(price * amount) as 'Стоимость',
ROUND((sum(price * amount) * 0.18 / (1 + 0.18)), 2) as 'НДС',
ROUND((sum(price * amount) / (1 + 0.18)), 2) as 'Стоимость_без_НДС'
from book group by author;

### Вычисления по таблице целиком
select ROUND(min(price), 2) as 'Минимальная_цена', 
ROUND(max(price), 2) as 'Максимальная_цена',
ROUND(avg(price), 2) as 'Средняя_цена' from book;

### Выборка данных по условию, групповые функции
select ROUND(avg(price), 2) as 'Средняя_цена', 
ROUND(sum(price * amount), 2) as 'Стоимость'
from book where amount between 5 and 14;

### Выборка данных по условию, групповые функции, WHERE и HAVING
select author, ROUND(sum(price * amount), 2) as 'Стоимость'
from book where title not in ('Идиот', 'Белая гвардия')
group by author having sum(price * amount) > 5000
order by sum(price * amount) desc;

## Вложенные запросы
### Вложенный запрос, возвращающий одно значение
select author, title, price from book
where price <= (select avg(price) from book)
order by price desc;

### Использование вложенного запроса в выражении
select author, title, price from book 
where price <= (select min(price) + 150 from book)
order by price;

### Вложенный запрос, оператор IN
select author, title, amount from book
where amount in (
    select amount from book group by amount having count(amount) = 1
);

### Вложенный запрос, операторы ANY и ALL
select author, title, price from book
where price < ALL(select max(price) from book group by author);

### Вложенный запрос после SELECT
select title, author, amount,
((select max(amount) from book) - amount) as 'Заказ'
from book where amount not in (select max(amount) from book);

## Запросы корректировки данных
### Создание пустой таблицы
create table supply(
    supply_id INT PRIMARY KEY AUTO_INCREMENT, 
    title VARCHAR(50),
    author varchar(30),
    price DECIMAL(8, 2),
    amount int
);

### Добавление записей в таблицу
insert into supply values ("1","Лирика","Пастернак Б.Л.","518.99","2"),
("2","Черный человек","Есенин С.А.","570.20","6"),
("3","Белая гвардия","Булгаков М.А.","540.50","7"),
("4","Идиот","Достоевский Ф.М.","360.80","3");

### Добавление записей из другой таблицы
insert into book (title, author, price, amount)
select title, author, price, amount from supply 
where author not in ('Булгаков М.А.', 'Достоевский Ф.М.');

### Добавление записей, вложенные запросы
insert into book (title, author, price, amount)
select title, author, price, amount from supply 
where author not in (select author from book);

### Запросы на обновление
update book set price = price - price * 0.1
where amount between 5 and 10;

### Запросы на обновление нескольких столбцов
update book set buy = IF(buy > amount, amount, buy),
price = IF(buy = 0, price - price * 0.1, price);

### Запросы на обновление нескольких таблиц 
update book, supply
set book.amount = book.amount + supply.amount, 
book.price = (book.price + supply.price) / 2
where supply.title = book.title;

### Запросы на удаление
delete from supply where author in (select author from book group by author having sum(amount) > 10);

### Запросы на создание таблицы
CREATE TABLE ordering AS
select author, title,
(select avg(amount) from book) as amount
from book where amount < (select avg(amount) from book);

## Таблица "Командировки", запросы на выборку
### Задание 1
select name, city, per_diem, date_first, date_last
from trip where name like '%а %'
order by date_last desc;

### Задание 2
