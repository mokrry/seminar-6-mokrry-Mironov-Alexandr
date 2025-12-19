[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/qd5dHcvr)
# Семинар 6: SQL-запросы к библиотечной системе и системе поездов

## Описание

В этом задании вам предстоит написать SQL-запросы к двум базам данных: библиотечной системе и системе поездов. Исходные
схемы баз данных были созданы в предыдущих заданиях. Вам предоставлены готовые Docker-compose файлы и SQL-таблицы,
поэтому вам не нужно заниматься их созданием.

## Инструкция по выполнению

1. Клонируйте ваш репозиторий, созданный GitHub Classroom.
2. Запустите контейнеры с базой данных с помощью Docker-compose в каталоге `/docker`.
3. Подключитесь к базе данных с помощью DataGrip. Обратите внимание на порты, которые используются в Docker-compose
   файле.
4. Выполните SQL-запросы к предоставленным таблицам и сохраните их в файле `answers.md` в каталоге `/src`.
5. Зафиксируйте изменения (commit) и отправьте их в репозиторий (push).

> Если каких-то значений не будет хватать в БД, то добавьте их сами.

## Задание

### Часть 1: Select

Возьмите схему библиотеки, созданную в предыдущем задании:

- **Reader** (ID, LastName, FirstName, Address, BirthDate)
- **Book** (ISBN, Title, Author, PagesNum, PubYear, PubName)
- **Publisher** (PubName, PubAdress)
- **Category** (CategoryName, ParentCat)
- **Copy** (ISBN, CopyNumber, ShelfPosition)
- **Borrowing** (ReaderNr, ISBN, CopyNumber, ReturnDate)
- **BookCat** (ISBN, CategoryName)

Напишите SQL-запросы:

* Показать все названия книг вместе с именами издателей.
* В какой книге наибольшее количество страниц?
* Какие авторы написали более 5 книг?
* В каких книгах более чем в два раза больше страниц, чем среднее количество страниц для всех книг?
* Какие категории содержат подкатегории?
* У какого автора (предположим, что имена авторов уникальны) написано максимальное количество книг?
* Какие читатели забронировали все книги (не копии), написанные "Марком Твеном"?
* Какие книги имеют более одной копии?
* ТОП 10 самых старых книг
* Перечислите все категории в категории “Спорт” (с любым уровнем вложености).

### Часть 2: Insert / Update / Delete

Используйте БД выше. Напишите SQL-запросы для следующих действий:

* Добавьте запись о бронировании читателем ‘Василеем Петровым’ книги с ISBN 123456 и номером копии 4.
* Удалить все книги, год публикации которых превышает 2000 год.
* Измените дату возврата для всех книг категории "Базы данных", начиная с 01.01.2016, чтобы они были в заимствовании на
  30 дней дольше (предположим, что в SQL можно добавлять числа к датам).

### Часть 3:

Рассмотрим следующую реляционную схему:

* Student( MatrNr, Name, Semester )
* Check( MatrNr, LectNr, ProfNr, Note )
* Lecture( LectNr, Title, Credit, ProfNr )
* Professor( ProfNr, Name, Room )

Опишите на русском языке результаты следующих запросов:

```sql
-- Запрос 1
SELECT s.Name, s.MatrNr FROM Student s
WHERE NOT EXISTS (
SELECT * FROM Check c WHERE c.MatrNr = s.MatrNr AND c.Note >= 4.0 ) ;
```

```sql
-- Запрос 2
( SELECT p.ProfNr, p.Name, sum(lec.Credit)
FROM Professor p, Lecture lec
WHERE p.ProfNr = lec.ProfNr
GROUP BY p.ProfNr, p.Name)
UNION
( SELECT p.ProfNr, p.Name, 0
FROM Professor p
WHERE NOT EXISTS (
SELECT * FROM Lecture lec WHERE lec.ProfNr = p.ProfNr ));
```

```sql
-- Запрос 3
SELECT s.Name, p.Note
FROM Student s, Lecture lec, Check c
WHERE s.MatrNr = c.MatrNr AND lec.LectNr = c.LectNr AND c.Note >= 4
AND c.Note >= ALL (
SELECT c1.Note FROM Check c1 WHERE c1.MatrNr = c.MatrNr )
```

## Формат сдачи работы

- Все SQL-запросы должны быть записаны в файл `/src/answers.md`. Формат указывается в каждом задании в файле.
- После выполнения задания, зафиксируйте изменения (`git commit -m "Seminar 6 completed"`)
- Отправьте их в ваш репозиторий (`git push origin main`)