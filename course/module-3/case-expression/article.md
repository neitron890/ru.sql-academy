# Условная логика, оператор CASE

SQL, подобно многим языкам программирования, позволяет писать условную логику, чтобы в зависимости от набора условий возвращать одно из множества возможных значений. В этой статье мы рассмотрим как это реализуется в SQL с
помощью оператора `CASE`.

## Что такое условная логика?

Под условной логикой понимается наличие у программы нескольких путей выполнения в зависимости от каких-то условий.

Например, в базе данных «Расписание» есть таблица `Student` с полем `birthday`, отражающим дату рождения студента. Допустим,
в выборке необходимо отобразить не саму дату рождения, а текстовое значение «Совершеннолетний» или «Несовершеннолетний» в зависимости от того,
есть ли студенту 18 лет. Это и есть пример условной логики, при которой должно вывестись либо одно значение, либо другое
в зависимости от конкретного условия.

<ERD databaseName="Schedule" />

Реализация такого запроса с помощью `CASE` может выглядеть следующим образом:

```sql
SELECT first_name, last_name,
CASE
  WHEN TIMESTAMPDIFF(YEAR, birthday, NOW()) >= 18 THEN "Совершеннолетний"
  ELSE "Несовершеннолетний"
END AS status
FROM Student
```

| first_name | last_name    | status             |
| ---------- | ------------ | ------------------ |
| Nikolaj    | Sokolov      | Совершеннолетний   |
| Vyacheslav | Eliseev      | Совершеннолетний   |
| Ivan       | Efremov      | Совершеннолетний   |
| Anatolij   | ZHdanov      | Несовершеннолетний |
| Georgij    | Noskov       | Совершеннолетний   |
| Artyom     | Sergeev      | Несовершеннолетний |
| Arina      | Evseeva      | Совершеннолетний   |
| Angelina   | Voroncova    | Совершеннолетний   |
| Ekaterina  | Ustinova     | Совершеннолетний   |
| Raisa      | Lapina       | Совершеннолетний   |
| Leonid     | Ignatov      | Несовершеннолетний |
| Snezhana   | Seliverstova | Совершеннолетний   |
| Semyon     | Biryukov     | Совершеннолетний   |
| Georgij    | Baranov      | Совершеннолетний   |
| YUliya     | Vishnyakova  | Совершеннолетний   |
| Valentina  | Bolshakova   | Совершеннолетний   |
| Leonid     | Kryukov      | Совершеннолетний   |
| Vladislav  | Cvetkov      | Совершеннолетний   |
| Snezhana   | Morozova     | Совершеннолетний   |
| Lyubov     | Borisova     | Совершеннолетний   |
| Anfisa     | Kalashnikova | Совершеннолетний   |
| Anna       | Osipova      | Совершеннолетний   |

## Синтаксис поискового выражения CASE

```sql
CASE
    WHEN условие_1 THEN возвращаемое_значение_1
    WHEN условие_2 THEN возвращаемое_значение_2
    WHEN условие_n THEN возвращаемое_значение_n
    [ELSE возвращаемое_значение_по_умолчанию]
END
```

Если `условие_1` возвращает истинное значение, то выражение `CASE` вернёт `возвращаемое_значение_1`, иначе будет сделана проверка
на `условие_2` и т.д. Если ни одно из предложенный условий не будет выполнено, то вернётся `NULL` или `возвращаемое_значение_по_умолчанию`,
если была использована конструкция `ELSE`.

### Пример

Рассмотрим оператор `CASE` на примере определения этапа школьного образования.

![Этапы школьного образования](https://sql-academy.org/static/guidePage/case-expression/ru_school_education_stages.png "Этапы школьного образования")

```sql
SELECT name,
CASE
  WHEN SUBSTRING(name, 1, INSTR(name, ' ')) IN (10, 11) THEN "Старшая школа"
  WHEN SUBSTRING(name, 1, INSTR(name, ' ')) IN (5, 6, 7, 8, 9) THEN "Средняя школа"
  ELSE "Начальная школа"
END AS stage
FROM Class
```

| name | stage           |
| ---- | --------------- |
| 8 A  | Средняя школа   |
| 8 B  | Средняя школа   |
| 9 C  | Средняя школа   |
| 9 B  | Средняя школа   |
| 9 A  | Средняя школа   |
| 10 B | Старшая школа   |
| 10 A | Старшая школа   |
| 11 B | Старшая школа   |
| 11 A | Старшая школа   |
| 7 A  | Средняя школа   |
| 7 B  | Средняя школа   |
| 6 A  | Средняя школа   |
| 6 B  | Средняя школа   |
| 5 A  | Средняя школа   |
| 5 B  | Средняя школа   |
| 4 A  | Начальная школа |

- Сначала мы извлекаем номер класса из его названия
  ```sql
  SUBSTRING(name, 1, INSTR(name, ' '))
  ```
- Далее мы проверяем вхождение данного номера в список классов, относящихся к «Старшая школа» и «Средняя школа».
- Если номер класса не находится в диапазоне 5–11, мы выводим «Начальная школа».

## Синтаксис простого выражения CASE

Оператор `CASE` также имеет и более простой синтаксис, который схож с поисковым выражением `CASE`, но
является менее гибким. Так он выглядит в общем виде:

```sql
CASE значение
    WHEN сравниваемое_значение_1 THEN возвращаемое_значение_1
    WHEN сравниваемое_значение_2 THEN возвращаемое_значение_2
    WHEN сравниваемое_значение_n THEN возвращаемое_значение_n
    [ELSE возвращаемое_значение_по-умолчанию]
END
```

В этом синтаксисе `значение` в `CASE` поочерёдно сравнивается с переданными значениями в `WHEN` и при совпадении возвращается значение следующее за `THEN`.

Используя этот синтаксис, можно переписать наш предыдущий пример таким образом:

```sql
SELECT name,
CASE SUBSTRING(name, 1, INSTR(name, ' '))
  WHEN 11 THEN "Старшая школа"
  WHEN 10 THEN "Старшая школа"
  WHEN 9 THEN "Средняя школа"
  WHEN 8 THEN "Средняя школа"
  WHEN 7 THEN "Средняя школа"
  WHEN 6 THEN "Средняя школа"
  WHEN 5 THEN "Средняя школа"
  ELSE "Начальная школа"
END AS stage
FROM Class
```

| name | stage           |
| ---- | --------------- |
| 8 A  | Средняя школа   |
| 8 B  | Средняя школа   |
| 9 C  | Средняя школа   |
| 9 B  | Средняя школа   |
| 9 A  | Средняя школа   |
| 10 B | Старшая школа   |
| 10 A | Старшая школа   |
| 11 B | Старшая школа   |
| 11 A | Старшая школа   |
| 7 A  | Средняя школа   |
| 7 B  | Средняя школа   |
| 6 A  | Средняя школа   |
| 6 B  | Средняя школа   |
| 5 A  | Средняя школа   |
| 5 B  | Средняя школа   |
| 4 A  | Начальная школа |