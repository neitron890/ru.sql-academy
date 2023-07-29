# Оператор HAVING

Мы уже рассматривали запрос получения средней стоимости аренды жилых помещений в зависимости от
типа жилья:

<ERD databaseName="Airbnb" />

```sql
SELECT home_type, AVG(price) as avg_price FROM Rooms
GROUP BY home_type
```

| home_type       | avg_price |
| --------------- | --------- |
| Private room    | 89.4286   |
| Entire home/apt | 148.6667  |
| Shared room     | 40        |

Давайте доработаем этот запрос таким образом, чтобы в итоговой выборке отображались только те группы, чья средняя стоимость больше 50.

Обладая предыдущим опытом, есть большой соблазн попытаться использовать для этих целей оператор `WHERE`. Но при попытке выполнить такой запрос
СУБД неминуемо выдаст ошибку, указав что мы некорректно используем синтаксис `WHERE avg_price > 50`.

```sql
SELECT home_type, AVG(price) as avg_price FROM Rooms
GROUP BY home_type
WHERE avg_price > 50
```

Говоря наперёд, для фильтрации групп мы должны использовать оператор `HAVING`:

```sql
SELECT home_type, AVG(price) as avg_price FROM Rooms
GROUP BY home_type
HAVING avg_price > 50
```

| home_type       | avg_price |
| --------------- | --------- |
| Private room    | 89.4286   |
| Entire home/apt | 148.6667  |

<br />

## Порядок выполнения SQL запроса

Но почему же мы не могли использовать `WHERE` и зачем нужен отдельный оператор для этой цели? Все дело в порядке выполнения SQL запроса.

![Схема порядка выполнения SQL запроса](https://sql-academy.org/static/guidePage/operator-having/sql_query_order_ru.png "Схема порядка выполнения SQL запроса")

Наш первый запрос был неверный, потому что мы пытались использовать поле `avg_price` у образовавшихся групп ещё до их образования,
так как выполнение оператора `WHERE` предшествует группировке.

То есть оператор `WHERE` в момент его исполнения ничего не знает о последующей группировке, он работает только с записями из таблицы.
Так мы, например, с его помощью можем отфильтровать записи таблицы `Rooms` по цене до применения группировки и лишь затем вычислить среднюю стоимость
групп оставшегося жилья:

```sql
SELECT home_type, AVG(price) as avg_price FROM Rooms
WHERE price > 50
GROUP BY home_type
```

| home_type       | avg_price |
| --------------- | --------- |
| Private room    | 96.875    |
| Entire home/apt | 148.6667  |

<br />

## Общая структура запроса с оператором HAVING

```sql
SELECT [константы, агрегатные_функции, поля_группировки]
FROM имя_таблицы
WHERE условия_на_ограничения_строк
GROUP BY поля_группировки
HAVING условие_на_ограничение_строк_после_группировки
ORDER BY условие_сортировки
```

## Пример использования HAVING

Для примера давайте получим минимальную стоимость каждого типа жилья c телевизором. При этом нас интересуют только типы жилья, содержащие как минимум 5 жилых
помещений, относящихся к ним.

Чтобы получить такой результат мы должны:

- Сначала получить все данные из таблицы

  ```sql
  SELECT ... FROM Rooms;
  ```

- Затем выбрать из всех записей таблиц `Room` только интересующие нас, т.е. только жильё с телевизором

  ```sql
  SELECT ... FROM Rooms
  WHERE has_tv = True
  ```

- Затем сгруппировать данные записи о жилых помещений по их типу

  ```sql
  SELECT ... FROM Rooms
  WHERE has_tv = True
  GROUP BY home_type
  ```

- После этого отфильтровать полученных группы по условию, что нас интересуют групп, имеющие как минимум 5 представителей

  ```sql
  SELECT ... FROM Rooms
  WHERE has_tv = True
  GROUP BY home_type
  HAVING COUNT(*) >= 5
  ```

- И под конец, посмотреть что нас просят в задании и, соответственно, добавить вывод необходимой информации. В нашем случае, нам
  необходимо вывести название типа жилья и его минимальную стоимость.
  ```sql
  SELECT home_type, MIN(price) as min_price FROM Rooms
  WHERE has_tv = True
  GROUP BY home_type
  HAVING COUNT(*) >= 5;
  ```