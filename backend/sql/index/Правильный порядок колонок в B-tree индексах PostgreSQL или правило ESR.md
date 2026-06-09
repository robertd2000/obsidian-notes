https://habr.com/ru/articles/911688/

Когда в проекте используется составной B-tree индекс, важно не просто "создать индекс", а сделать это правильно — иначе запросы могут не только не ускориться, но и начать работать медленнее. Возникает логичный вопрос: как выбрать порядок колонок, чтобы индекс действительно работал эффективно? Брутфорсом? По интуиции? По селективности?

В этой статье я расскажу, как подходить к построению составных индексов в PostgreSQL, на что реально влияет порядок колонок. Также разберём простое правило ESR, которое помогает упростить выбор и получать стабильный прирост производительности на всех стендах.

![Леонард Эйлер - считается одним из основателей теории графов благодаря решению задачи о кёнигсбергских мостах](https://habrastorage.org/r/w1560/getpro/habr/upload_files/94d/f2d/0f0/94df2d0f0013dc728b921ea23ad0f016.jpg "Леонард Эйлер - считается одним из основателей теории графов благодаря решению задачи о кёнигсбергских мостах")

[Леонард Эйлер](https://ru.wikipedia.org/wiki/%D0%AD%D0%B9%D0%BB%D0%B5%D1%80,_%D0%9B%D0%B5%D0%BE%D0%BD%D0%B0%D1%80%D0%B4#) - считается одним из основателей теории графов благодаря решению задачи о [кёнигсбергских мостах](https://ru.wikipedia.org/wiki/%D0%97%D0%B0%D0%B4%D0%B0%D1%87%D0%B0_%D0%BE_%D1%81%D0%B5%D0%BC%D0%B8_%D0%BA%D1%91%D0%BD%D0%B8%D0%B3%D1%81%D0%B1%D0%B5%D1%80%D0%B3%D1%81%D0%BA%D0%B8%D1%85_%D0%BC%D0%BE%D1%81%D1%82%D0%B0%D1%85)

Скрытый текст

Я — Дмитрий Денисенко, Sofware Developer. Хочу делиться и рассказывать про интересные моменты на этапах разработки.

---

## Вводная часть - как работает поиск в B-tree индексе

Для понимания того, как правильно создать составной индекс, сначала поймём, что из себя представляет B-tree в PostgreSQL.

Скрытый текст

[](https://github.com/postgres/postgres/tree/master/src/backend/access/nbtree)

B-tree или же многонаправленное сбалансированное дерево (что значит B в B-tree — точно не знает никто: есть версии как `balanced`, `broad` или просто `Bayer`, по имени автора). В PostgreSQL представляет под собой 3 основных элемента:

- Корневой узел
    
- Внутренние узлы
    
- Листовые узлы
    

Для более глубокого понимания корневого узла и внутренних узлов советую почитать информацию на специализированных статьях, у них есть крутые плюшки в виде метаинформации. Но это я думаю нужно людям, которые напрямую работают с БД, оставим это базистам настраивающим эту инфраструктуру на проекте.

Нас же интересуют листовые узлы и логика поиска в них.

Листовой узел (leaf node) — это страница дерева на самом нижнем уровне, содержащая реальные данные индекса. В листовом узле - данные хранятся через Page - специальный блок, который хранит в себе информацию по ключам индекса и TID (Обычно это 8 KB). Сам PostgreSQL хранит листовые ноды в виде двухсвязного списка (будет важно чуть ниже).

Для начала приведём пример как выглядит дерево на 1 колонке индекса

```
-- ТаблицаCREATE TABLE IF NOT EXISTS public.bank_transactions(    id integer NOT NULL DEFAULT nextval('bank_transactions_id_seq'::regclass),    sender_id integer NOT NULL,    receiver_id integer NOT NULL,    amount numeric(12,2) NOT NULL,    transaction_date date NOT NULL,    payment_type text COLLATE pg_catalog."default" NOT NULL DEFAULT 'СБП'::text)
```

Нагенерим данных

```
INSERT INTO bank_transactions (sender_id, receiver_id, amount, transaction_date, payment_type)SELECT    (random() * 100000)::int,                          -- sender_id    (random() * 100000)::int,                          -- receiver_id    ROUND((random() * 50000)::numeric, 2),             -- amount    DATE '2023-01-01' + (random() * 365)::int,         -- transaction_date    (ARRAY['СБП', 'SWIFT', 'ВНУТРЕННИЙ', 'Межбанк'])[floor(random() * 4)::int + 1]  -- payment_typeFROM generate_series(1, 1000000);  
```

Теперь попытаемся найти sender_id = 444

```
-- ЗапросEXPLAIN (ANALYZE, BUFFERS)SELECT * FROM bank_transactionsWHERE bank_transactions.sender_id = 444-- ПланGather  (cost=1000.00..15098.43 rows=11 width=36) (actual time=0.203..47.653 rows=6 loops=1)  Workers Planned: 2  Workers Launched: 2  Buffers: shared hit=7749 read=1140  ->  Parallel Seq Scan on bank_transactions  (cost=0.00..14097.33 rows=5 width=36) (actual time=0.692..20.904 rows=2 loops=3)        Filter: (sender_id = 444)        Rows Removed by Filter: 333331        Buffers: shared hit=7749 read=1140Planning:  Buffers: shared hit=5Planning Time: 0.433 msExecution Time: 47.677 ms
```

Тут мы видим, что SQL просто прошёлся Seq Scan по всей таблице, отбросил кучу строк (Rows Removed by Filter), и в итоге выдал нам 6 строк. Т.е. чтобы найти нужную нам строчку, ему пришлось очень много читать и отбрасывать.

Добавим индекс по `sender_id` и посмотрим, как изменится план.

```
-- ИндексCREATE INDEX idx_sender_id ON bank_transactions (sender_id);-- ПланBitmap Heap Scan on bank_transactions  (cost=4.51..47.49 rows=11 width=36) (actual time=0.036..0.043 rows=6 loops=1)  Recheck Cond: (sender_id = 444)  Heap Blocks: exact=6  Buffers: shared hit=6 read=3  ->  Bitmap Index Scan on idx_sender_id  (cost=0.00..4.51 rows=11 width=0) (actual time=0.032..0.032 rows=6 loops=1)        Index Cond: (sender_id = 444)        Buffers: shared read=3Planning:  Buffers: shared hit=15 read=1Planning Time: 0.725 msExecution Time: 0.061 ms
```

Bitmap Index Scan - находит в B-tree индексе все строки, где sender_id = 5203, получает список TID (указатель на физическое расположение строки в таблице).  
Bitmap Heap Scan — По списку TID идёт в таблицу (heap) и проверят существуют ли такие строчки в таблице.

Немного визуализируем

![Визуализация Bitmap Index Scan по нашей таблице для одиночного индекса](https://habrastorage.org/r/w1560/getpro/habr/upload_files/989/9b1/68c/9899b168c67ebd0693a8d4c8d31473c4.png "Визуализация Bitmap Index Scan по нашей таблице для одиночного индекса")

Визуализация Bitmap Index Scan по нашей таблице для одиночного индекса

Отлично, поняли что и как для индекса с 1 колонкой.  
Теперь поймём как это работает для составных индексов.

В составных индексах — Page хранит ключ индекса уже в виде тех колонок, что мы задаём в индексе.

Теперь, наша задача, найти отправителей за конкретную дату. Сделаем это без индекса (и старый удалим).

```
EXPLAIN (ANALYZE, BUFFERS)SELECT * FROM bank_transactionsWHERE bank_transactions.sender_id = 444	AND bank_transactions.transaction_date = '2023-09-24'--ПланGather  (cost=1000.00..16139.10 rows=1 width=36) (actual time=0.203..54.237 rows=1 loops=1)  Workers Planned: 2  Workers Launched: 2  Buffers: shared hit=7877 read=1012  ->  Parallel Seq Scan on bank_transactions  (cost=0.00..15139.00 rows=1 width=36) (actual time=9.925..26.461 rows=0 loops=3)        Filter: ((sender_id = 444) AND (transaction_date = '2023-09-24'::date))        Rows Removed by Filter: 333333        Buffers: shared hit=7877 read=1012Planning:  Buffers: shared hit=5 dirtied=1Planning Time: 0.443 msExecution Time: 54.256 ms
```

То же самое, полный последовательный просмотр всех строк подряд, чтобы найти нужную.  
Добавим индекс (выбираем по селективности)

```
-- ИндексCREATE INDEX idx_sender_dateON bank_transactions(sender_id, transaction_date);-- ПланIndex Scan using idx_sender_date on bank_transactions  (cost=0.42..8.45 rows=1 width=36) (actual time=0.046..0.046 rows=1 loops=1)  Index Cond: ((sender_id = 444) AND (transaction_date = '2023-09-24'::date))  Buffers: shared hit=1 read=3Planning:  Buffers: shared hit=18 read=1Planning Time: 1.122 msExecution Time: 0.062 ms
```

Тут мы прошлись Index Scan и быстро нашли всё что нам нужно  
Прежде чем будем визуализировать, отмечу, что поиск осуществляется в порядке, заданном в индексе. Это означает, что PostgreSQL использует порядок колонок в индексе слева направо: сначала фильтруется `sender_id`, затем `transaction_date`.

![Визуализация Bitmap Index Scan по нашей таблице для составного индекса](https://habrastorage.org/r/w1560/getpro/habr/upload_files/f02/c30/e7a/f02c30e7a1a80bd36e9f6a52dbcb833d.png "Визуализация Bitmap Index Scan по нашей таблице для составного индекса")

Визуализация Bitmap Index Scan по нашей таблице для составного индекса

Скрытый текст

---

## Практическая часть - ESR как правило при составлении индекса

Нельзя создать универсальный индекс, который одинаково быстро будет работать для всех задач, если сделать универсальным — то это не максимальный прирост скорости.

В реалях разработки, обычно идёт выбор того или иного решения, ведь индекс не воздушный и занимает как память, так и влияет на производительность. Хочу сразу сказать, что составление индекса — дело супер прикладное, решения для одной базы, может не подходить для решения в другой. Но по англо интернету гуляет правило (встречал вскользь и в других русскоязычных статьях на Хабре) ESR (Equality, Sort, Range).

ESR — говорит о том, что в составных ключах для B-tree сначала столбцы с равенством, затем сортировка, затем диапазоны.

## Что такое правило ESR?

> **ESR = Equality → Sort → Range**

При создании составного B-tree индекса:

- **E — Equality**: сначала колонки, по которым в запросе стоит `=`
    
- **S — Sort**: затем те, по которым идёт сортировка (`ORDER BY`)
    
- **R — Range**: и только в конце — колонки с диапазонами (`>`, `<`, `BETWEEN`)
    

В официальной документации в рамках этого правила есть всего 1 абзац

> A multicolumn B-tree index can be used with query conditions that involve any subset of the index's columns, but the index is most efficient when there are constraints on the leading (leftmost) columns. The exact rule is that equality constraints on leading columns, plus any inequality constraints on the first column that does not have an equality constraint, will be used to limit the portion of the index that is scanned. Constraints on columns to the right of these columns are checked in the index, so they save visits to the table proper, but they do not reduce the portion of the index that has to be scanned. For example, given an index on (a, b, c) and a query condition WHERE a = 5 AND b >= 42 AND c < 77, the index would have to be scanned from the first entry with a = 5 and b = 42 up through the last entry with a = 5. Index entries with c >= 77 would be skipped, but they'd still have to be scanned through. This index could in principle be used for queries that have constraints on b and/or c with no constraint on a — but the entire index would have to be scanned, so in most cases the planner would prefer a sequential table scan over using the index.

Давайте покажем на практике

Для начала определим потребность

Из нашей таблицы, мы хотим получить отправителей не за конкретную дату , а за временной промежуток и по типам платежей (Вполне продовская задача для сервиса отчётов банка).

Потребность найдена, теперь представим, что записей не 100 000, а куда больше (огромная продовская бд с историями по банку). Нам приходят и говорят, что запрос слишком медленный и обычный индекс по sender_id работает слишком медленно.

Добавим ещё данных

```
INSERT INTO bank_transactions (sender_id, receiver_id, amount, transaction_date, payment_type)SELECT    (random() * 1_000_000)::int,                  -- sender_id    (random() * 1_000_000)::int,    ROUND((random() * 10000)::numeric, 2),    DATE '2020-01-01' + (random() * 2000)::int,    (ARRAY['СБП', 'SWIFT', 'ВНУТРЕННИЙ', 'Межбанк'])[floor(random() * 4)::int + 1]FROM generate_series(1, 10_000_000);
```

Также добавим 10 000 записей конкретно под каждому sender_id.  
Сразу подчеркну — это сделано исключительно для демонстрации логики работы PostgreSQL под нагрузкой. Я сознательно не моделирую продовую таблицу — нет возможности воспроизвести её объём и ресурсы.  
Да, при небольших объёмах данных PostgreSQL часто выбирает оптимальные планы даже при неоптимальной структуре индексов — оптимизатор успевает всё аккуратно подкрутить, найти хороший план и отработать даже при плохой структуре данных.  
Но это не значит, что в проде всё будет так же. Там — жесткая борьба за ресурсы, кеши не безразмерные, и у планировщика не всегда будет шанс "спасти" косяки в структуре индексов или запросов. Что работает на тесте с 10 строками — вполне может обрушить прод с миллионами. Именно поэтому важно моделировать нагрузку.

```
INSERT INTO bank_transactions (sender_id, receiver_id, amount, transaction_date, payment_type)SELECT    sender_id,    (random() * 1000000)::int,    ROUND((random() * 10000)::numeric, 2),    DATE '2020-01-01' + (random() * 1860)::int,    (ARRAY['СБП', 'SWIFT', 'Межбанк', 'ВНУТРЕННИЙ'])[floor(random() * 4)::int + 1]FROM generate_series(1, 10000) AS sender_id,     generate_series(1, 1000);-- ЗапросEXPLAIN (ANALYZE, BUFFERS)SELECT * FROM bank_transactionsWHERE bank_transactions.sender_id = 444	AND bank_transactions.transaction_date < '2025-05-18'	AND bank_transactions.transaction_date > '2020-04-18'	AND bank_transactions.payment_type = 'СБП'-- ПланSeq Scan on bank_transactions  (cost=0.00..871550.86 rows=3541467 width=35) (actual time=5416.344..6795.770 rows=222 loops=1)  Filter: ((transaction_date < '2025-05-18'::date) AND (transaction_date > '2020-04-18'::date) AND (sender_id = 444) AND (payment_type = 'СБП'::text))  Rows Removed by Filter: 20999762  Buffers: shared hit=258 read=252001Planning:  Buffers: shared hit=3 read=2 dirtied=1Planning Time: 1.006 msExecution Time: 6795.867 ms
```

При увеличении данных мы перестали использовать индекс по sender_id — стало слишком много строк (21 млн) и из них сложно что-то хорошее вычленить по sender_id, поэтому планировщик решил использовать обычный поиск по всем данным.

При выборе порядка колонок в индексе часто ориентируются на селективность - пойдём отталкиваясь от этого критерия.

1. sender_id — поле для компоновки данных, лучше чем смотреть кучу платежей за дату
    
2. transaction_date — когда мы нашли данные по пользователю, можем спокойно итерироваться по датам
    
3. payment_type — самое не селективное поле
    

```
-- ИндексCREATE INDEX idx_sender_date_typeON bank_transactions (sender_id, transaction_date, payment_type);-- ПланIndex Scan using idx_sender_date_type on bank_transactions  (cost=0.56..254.61 rows=83 width=35) (actual time=0.034..0.325 rows=222 loops=1)  Index Cond: ((sender_id = 444) AND (transaction_date < '2025-05-18'::date) AND (transaction_date > '2020-04-18'::date) AND (payment_type = 'СБП'::text))  Buffers: shared hit=230Planning Time: 0.133 msExecution Time: 0.354 ms
```

На первый взгляд — идеально. Но стоит задуматься: действительно ли всё работает так, как мы ожидаем?  
Тут надо подумать, а показывают ли нам то, как это работает изнутри?  
Попробуем составить индекс по ESR (внутри каждой категории — по селективности)

Добавляю индекс без удаления idx_sender_date_type, пусть PostgreSQL сам решит какой индекс брать

```
-- ИндексCREATE INDEX idx_sender_type_dateON bank_transactions (sender_id, payment_type, transaction_date);-- ПланIndex Scan using idx_sender_type_date on bank_transactions  (cost=0.56..246.88 rows=83 width=35) (actual time=0.037..0.184 rows=222 loops=1)  Index Cond: ((sender_id = 444) AND (payment_type = 'СБП'::text) AND (transaction_date < '2025-05-18'::date) AND (transaction_date > '2020-04-18'::date))  Buffers: shared hit=226Planning Time: 0.108 msExecution Time: 0.204 ms
```

Даже при полностью прогретом кешировании, PostgreSQL использует **меньше узлов B-tree** и выполняет индексный проход **быстрее**, если порядок колонок в индексе соответствует порядку ESR  
  
Важно отметить почему так и почему это может выглядеть не очевидно.  
Когда PostgreSQL проходится по индексу, он выбирает сразу нужные и тут нет такого, что он показывает Rows Removed by Filter, он просто сразу берёт нужные и отсекает ненужные.

Визуально это выглядит так:

![Визуальная схема данных в индексе](https://habrastorage.org/r/w1560/getpro/habr/upload_files/b53/d53/c5a/b53d53c5aa33ac839601db3bea894911.png "Визуальная схема данных в индексе")

Визуальная схема данных в индексе

---

## Заключение

Выбор порядка колонок в составном B-tree индексе — это не вкусовщина, а инженерное и архитектурное решение, основанное на логике доступа к данным.  
Правило **ESR (Equality, Sort, Range)** — это не волшебная формула, а рабочий принцип, подтверждённый планами, буферами, временем выполнения и выбором PostgreSQL планировщика (дай бог ему здоровья).  
Продуманная структура составных индексов — важнейший фактор производительности в PostgreSQL. Правило ESR не заменяет анализа, но помогает выстроить эффективную стратегию создания индексов. При росте объёма данных — это разница между мгновенным ответом и минутным ожиданием.  
Рассматривали правило на хороших нагрузках (допускаю, что модель может отличаться от реальных данных, тут всё происходило скриптом и псевдослучайным образом)  
Ну и `EXPLAIN (ANALYZE, BUFFERS)` твой лучший друг, не стоит брать мои слова на веру