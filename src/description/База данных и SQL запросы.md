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
 