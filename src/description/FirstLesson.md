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



