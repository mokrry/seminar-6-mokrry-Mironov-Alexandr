# Ответы на задания семинара 6

## Часть 1: Select

Поскольку в этой части задания в пунктах 3, 4, 10 возвращалась пустая таблица. 
в library.init были вставлены новые инсёрты в конец файла.

### Вопрос 1
Показать все названия книг вместе с именами издателей.

```sql
SELECT b.Title, b.Publisher_Name
FROM Book b;
```

### Вопрос 2
В какой книге наибольшее количество страниц?

```sql
SELECT b.ISBN, b.Title, b.Number_of_pages
FROM Book b
WHERE b.Number_of_pages = (SELECT MAX(Number_of_pages) FROM Book);
```

### Вопрос 3
Какие авторы написали более 5 книг?

```sql
SELECT b.Author, COUNT(*) AS books_count
FROM Book b
GROUP BY b.Author
HAVING COUNT(*) > 5;
```

### Вопрос 4
В каких книгах более чем в два раза больше страниц, чем среднее количество страниц для всех книг?

```sql
SELECT b.ISBN, b.Title, b.Number_of_pages
FROM Book b
WHERE b.Number_of_pages > 2 * (SELECT AVG(Number_of_pages) FROM Book);
```

### Вопрос 5
Какие категории содержат подкатегории?

```sql
SELECT DISTINCT parent.CategoryName
FROM Category parent
WHERE EXISTS (
  SELECT 1
  FROM Category child
  WHERE child.ParentCat = parent.CategoryName
);
```

### Вопрос 6
У какого автора (предположим, что имена авторов уникальны) написано максимальное количество книг?

```sql
WITH author_counts AS (
  SELECT Author, COUNT(*) AS books_count
  FROM Book
  GROUP BY Author
)
SELECT Author, books_count
FROM author_counts
WHERE books_count = (SELECT MAX(books_count) FROM author_counts);
```

### Вопрос 7
Какие читатели забронировали все книги (не копии), написанные "Марком Твеном"?

```sql
SELECT r.ID, r.LastName, r.FirstName
FROM Reader r
WHERE NOT EXISTS (
  SELECT 1
  FROM Book b
  WHERE b.Author = 'Марк Твен'
    AND NOT EXISTS (
      SELECT 1
      FROM Borrowing br
      WHERE br.ID = r.ID
        AND br.ISBN = b.ISBN
    )
);
```

### Вопрос 8
Какие книги имеют более одной копии?

```sql
SELECT b.ISBN, b.Title, COUNT(*) AS copies_count
FROM Copy c
JOIN Book b ON b.ISBN = c.ISBN
GROUP BY b.ISBN, b.Title
HAVING COUNT(*) > 1;
```

### Вопрос 9
ТОП 10 самых старых книг

```sql
SELECT b.ISBN, b.Title, b.Year
FROM Book b
ORDER BY b.Year ASC
LIMIT 10;
```

### Вопрос 10
Перечислите все категории в категории "Спорт" (с любым уровнем вложености).

```sql
WITH RECURSIVE cats AS (
  SELECT CategoryName, ParentCat
  FROM Category
  WHERE CategoryName = 'Спорт'

  UNION ALL

  SELECT c.CategoryName, c.ParentCat
  FROM Category c
  JOIN cats ON c.ParentCat = cats.CategoryName
)
SELECT CategoryName
FROM cats;

```

## Часть 2: Insert / Update / Delete

### Вопрос 1
Добавьте запись о бронировании читателем 'Василеем Петровым' книги с ISBN 123456 и номером копии 4.

```sql
-- 1) Создаём читателя, если его ещё нет
INSERT INTO Reader (LastName, FirstName, Address, BirthDate)
SELECT 'Петров', 'Василий', 'Адрес неизвестен', '2000-01-01'
WHERE NOT EXISTS (
  SELECT 1 FROM Reader
  WHERE LastName = 'Петров' AND FirstName = 'Василий'
);

-- 2) Создаём книгу, если её ещё нет (минимально валидные поля)
INSERT INTO Book (ISBN, Title, Author, Number_of_pages, Year, Publisher_Name)
SELECT '123456', 'Книга 123456', 'Неизвестный автор', 100, 2000, NULL
WHERE NOT EXISTS (
  SELECT 1 FROM Book WHERE ISBN = '123456'
);

-- 3) Создаём копию, если её ещё нет
INSERT INTO Copy (ISBN, CopyNumber, Position)
SELECT '123456', 4, 'Shelf-X'
WHERE NOT EXISTS (
  SELECT 1 FROM Copy WHERE ISBN = '123456' AND CopyNumber = 4
);

-- 4) Добавляем бронирование (ReturnDate можно NULL)
INSERT INTO Borrowing (ID, ISBN, CopyNumber, ReturnDate)
VALUES (
  (SELECT ID FROM Reader WHERE LastName = 'Петров' AND FirstName = 'Василий' LIMIT 1),
  '123456',
  4,
  NULL
);

```

### Вопрос 2
Удалить все книги, год публикации которых превышает 2000 год.

```sql
DELETE FROM Book
WHERE Year > 2000;
```

### Вопрос 3
Измените дату возврата для всех книг категории "Базы данных", начиная с 01.01.2016, чтобы они были в заимствовании на 30 дней дольше (предположим, что в SQL можно добавлять числа к датам).

```sql
UPDATE Borrowing br
SET ReturnDate = br.ReturnDate + 30
WHERE br.ReturnDate IS NOT NULL
  AND br.ReturnDate >= DATE '2016-01-01'
  AND br.ISBN IN (
    SELECT bc.ISBN
    FROM BookCategory bc
    WHERE bc.CategoryName = 'Базы данных'
  );
```

## Часть 3: Интерпретация запросов

### Запрос 1

```sql
SELECT s.Name, s.MatrNr FROM Student s
WHERE NOT EXISTS (
SELECT * FROM Check c WHERE c.MatrNr = s.MatrNr AND c.Note >= 4.0 ) ;
```

**Опишите на русском языке результат запроса выше:**
Запрос выводит студентов (имя и MatrNr), для которых не существует ни одной записи в таблице Check с их MatrNr, где оценка Note ≥ 4.0.
То есть это студенты, у которых:

- либо вообще нет ни одной сдачи/оценки в Check,

- либо все их оценки строго меньше 4.0.

### Запрос 2
```sql
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

**Опишите на русском языке результат запроса выше:**
Запрос возвращает список всех преподавателей (ProfNr, Name) и число:

- если преподаватель ведёт лекции — сумму кредитов (Credit) по всем его лекциям,

- если преподаватель не ведёт ни одной лекции — возвращает 0.
UNION объединяет оба набора в один итоговый список.

### Запрос 3
```sql
SELECT s.Name, p.Note
FROM Student s, Lecture lec, Check c
WHERE s.MatrNr = c.MatrNr AND lec.LectNr = c.LectNr AND c.Note >= 4
AND c.Note >= ALL (
SELECT c1.Note FROM Check c1 WHERE c1.MatrNr = c.MatrNr )
```

**Опишите на русском языке результат запроса выше:**

В запросе, судя по смыслу, опечатка: p.Note должно быть c.Note.

По смыслу запрос выводит:

- имя студента

 - его максимальную оценку (Note) среди всех его записей в Check,

но только для тех студентов, у кого эта максимальная оценка ≥ 4.
Если у студента несколько записей с одинаковой максимальной оценкой, он может появиться несколько раз.