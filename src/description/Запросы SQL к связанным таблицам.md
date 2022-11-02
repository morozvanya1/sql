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
