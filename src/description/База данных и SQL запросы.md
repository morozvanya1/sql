# База данных и SQL запросы

## База данных «Тестирование», запросы на выборку
### Задание 1
select name_student, date_attempt, result
from attempt join subject using (subject_id)
join student using (student_id)
where name_subject = 'Основы баз данных'
order by result desc;

### Задание 2
select name_subject, count(attempt.subject_id) as 'Количество', ROUND(avg(result),2) as 'Среднее' from attempt
right join subject using (subject_id)
group by name_subject, attempt.subject_id
order by avg(result) desc;

### Задание 3
SELECT name_student, result
FROM student
    INNER JOIN attempt USING(student_id)
WHERE result = (
         SELECT MAX(result) 
         FROM attempt
      )
ORDER BY name_student;

### Задание 4
select name_student, name_subject, DATEDIFF(max(date_attempt), min(date_attempt)) as Интервал
from attempt join student using (student_id)
join subject using (subject_id)
group by name_student, name_subject having count(name_student) > 1
order by Интервал;

### Задание 5
select name_subject, count(distinct(student_id)) as Количество
from attempt right join subject using (subject_id)
group by name_subject, attempt.subject_id
order by Количество desc, name_subject;

### Задание 6
select question_id, name_question from question join subject using (subject_id)
where name_subject = 'Основы баз данных' ORDER BY RAND() limit 3;

### Задание 7
select name_question, name_answer, if (is_correct = 1, 'Верно', 'Неверно') as Результат
from testing join answer using (answer_id)
join question on question.question_id = testing.question_id
where attempt_id = 7;

### Задание 8
SELECT name_student, name_subject, date_attempt,  ROUND((SUM(answer.is_correct)/ 3) * 100 ,2) AS Результат
FROM attempt
    JOIN student USING(student_id)
    JOIN subject USING(subject_id)
    JOIN testing USING(attempt_id)
    JOIN answer USING(answer_id)
GROUP BY name_student, name_subject, date_attempt 
ORDER BY name_student ASC, date_attempt DESC;

### Задание 9
SELECT name_subject, 
       CONCAT(LEFT(name_question, 30), '...') AS Вопрос, 
       COUNT(t.answer_id) AS Всего_ответов, 
       ROUND(SUM(is_correct) / COUNT(t.answer_id) * 100, 2) AS Успешность
  FROM subject
       JOIN question USING(subject_id)
       JOIN testing t USING(question_id)
       LEFT JOIN answer a USING(answer_id)
 GROUP BY 1, 2
 ORDER BY 1, 4 DESC, 2;
 
 ## База данных «Тестирование», запросы корректировки
 ### Задание 1
 insert into attempt (student_id, subject_id, date_attempt, result)
 select 1, 2, now(), null;
 
 ### Задание 2
 insert into testing (attempt_id, question_id, answer_id)
 select attempt_id, question_id, null from attempt join question using (subject_id)
 where attempt_id = (select max(attempt_id) from attempt)
 order by RAND() limit 3;
 
 ### Задание 3
 UPDATE attempt
     SET result = (SELECT ROUND(SUM(is_correct)/3*100, 0)
         FROM answer INNER JOIN testing ON answer.answer_id = testing.answer_id
         WHERE attempt_id = 8)
     WHERE attempt_id = 8;
     
### Задание 4
delete from attempt where date_attempt < '2020-05-01';

## База данных «Абитуриент», запросы на выборку
### Задание 1
select name_enrollee from enrollee
join program_enrollee using (enrollee_id)
join program using (program_id)
where name_program = 'Мехатроника и робототехника'
order by name_enrollee;

### Задание 2
select name_program from program join program_subject using (program_id)
join subject using (subject_id)
where name_subject = 'Информатика'
order by name_program desc;

### Задание 3
select name_subject, count(enrollee_id) as Количество, max(result) as Максимум, min(result) as Минимум, ROUND(avg(result), 1) as Среднее 
from subject join enrollee_subject using (subject_id)
group by 1
order by 1;

### Задание 4
select distinct name_program from program join program_subject using (program_id)
where min_result >= 40
order by 1;

### Задание 5
select distinct name_program from program join program_subject using (program_id)
group by 1
having min(min_result) >= 40
order by 1;

### Задание 6
select name_program, plan from program 
where plan in (select max(plan) from program);

### Задание 7
select name_enrollee, if(sum(bonus) is null, 0, sum(bonus)) as Бонус 
from enrollee left join enrollee_achievement using (enrollee_id)
left join achievement using (achievement_id)
group by 1
order by 1;

### Задание 8
SELECT name_department, name_program, plan, Количество, ROUND(Количество/plan, 2) AS Конкурс
FROM department
    INNER JOIN program USING(department_id)
    INNER JOIN 
            (SELECT program_id, COUNT(enrollee_id) AS Количество
            FROM  program_enrollee
            GROUP BY program_id) AS temp USING(program_id)
ORDER BY Конкурс DESC;

### Задание 9
SELECT name_program
    FROM program
        INNER JOIN program_subject USING(program_id)
        INNER JOIN subject USING(subject_id)
        WHERE name_subject IN ("Информатика", "Математика")
GROUP BY name_program
HAVING COUNT(name_program) = 2
ORDER BY name_program ASC;

### Задание 10
