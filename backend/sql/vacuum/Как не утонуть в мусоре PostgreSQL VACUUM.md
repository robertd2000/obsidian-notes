https://habr.com/ru/companies/otus/articles/865850/

Привет, Хабр! Сегодня поговорим о VACUUM в PostgreSQL — штуке, которая спасает базы данных от захламления.

PostgreSQL использует MVCC для управления транзакциями. То есть каждая операция вставки, обновления или удаления оставляет после себя версию строки. Старые версии остаются в таблице, пока VACUUM их не зачистит.

Причины использовать VACUUM:

1. **Сбор мусора:** удаленные или устаревшие строки все еще занимают место на диске.
    
2. **Обновление статистики:** если статистика не обновлена, оптимизатор запросов начинает вести себя, как водитель без навигатора — блуждает в потёмках.
    
3. **Предотвращение переполнения xid:** старые транзакции могут привести к переполнению транзакционных идентификаторов (xid).
    

Дополнительно стоит отметить, что VACUUM — это не плацебо. Он не ускорит запросы напрямую, но создаст условия, при которых база будет работать стабильно.

#### Виды VACUUM

1. **VACUUM:** удаляет мусорные строки, но не возвращает место в ОС.
    
2. **VACUUM FULL:** полностью реорганизует таблицу, возвращает место в ОС, но блокирует доступ к таблице.
    
3. **AUTOVACUUM:** фоновый процесс, который автоматически запускает VACUUM, чтобы не приходилось делать это вручную.
    

**Прочие опции VACUUM:**

**VERBOSE:** выводит подробный отчет о процессе очистки.

Пример вывода:

```
INFO:  vacuuming "public.sales_data"
INFO:  scanned index "sales_idx" to remove 1200 row versions
DETAIL:  CPU 0.05s/0.10u sec elapsed 0.20 sec.
INFO:  "sales_data": removed 1200 dead row versions in 500 pages
DETAIL:  CPU 0.02s/0.03u sec elapsed 0.08 sec.
INFO:  "sales_data": found 300000 removable, 400000 nonremovable row versions in 20000 out of 50000 pages
DETAIL:  0 dead row versions cannot be removed yet.
INFO:  index "sales_idx" now contains 500000 live row versions in 2000 pages
DETAIL:  0 index row versions were removed.
INFO:  "sales_data": free space map contains 3000 pages
DETAIL:  CPU 0.03s/0.02u sec elapsed 0.07 sec.
```

Каждая строка описывает определеный этап работы команды. Например:

1. **Начало очистки:** `INFO: vacuuming "public.sales_data"` — VACUUM начинает обрабатывать таблицу `sales_data` из схемы `public`.
    
2. **Работа с индексом:** `scanned index "sales_idx" to remove 1200 row versions` — индекс `sales_idx` был просмотрен, чтобы удалить 1200 устаревших версий строк.
    
3. **Удаление данных:** `removed 1200 dead row versions in 500 pages` — в 500 страницах было найдено и удалено 1200 «мертвых» строк.
    
4. **Анализ таблицы:** `found 300000 removable, 400000 nonremovable row versions` — из 300 тысяч «мертвых» строк 100 тысяч пока не могут быть удалены (например, используются другими транзакциями).
    
5. **Индексация:** `index "sales_idx" now contains 500000 live row versions` — индекс теперь содержит 500 тысяч актуальных строк.
    
6. **Карта свободного места:** `free space map contains 3000 pages` — указывает количество страниц, доступных для повторного использования.
    

**FREEZE:** используется для агрессивной заморозки всех кортежей в таблице.

**ANALYZE:** комбинирует очистку с обновлением статистики таблицы.

Для аналитических баз чаще всего нас интересует AUTOVACUUM, так как данные обычно обрабатываются пакетами, и ручной VACUUM может быть неудобен.

Для аналитических баз чаще всего нас интересует AUTOVACUUM, т.к данные обычно обрабатываются пакетами, и ручной VACUUM может быть неудобен.

#### Настраиваем AUTOVACUUM

AUTOVACUUM работает по дефолту, но требует настройки.

`autovacuum_vacuum_threshold` и `autovacuum_vacuum_scale_factor` определяют, когда AUTOVACUUM сработает.

- `autovacuum_vacuum_threshold` — минимальное количество изменённых строк для запуска.
    
- `autovacuum_vacuum_scale_factor` — доля измененных строк от общего числа.
    

Пример:

```
ALTER TABLE your_table  SET (autovacuum_vacuum_threshold = 500,       autovacuum_vacuum_scale_factor = 0.05);
```

Здесь VACUUM запустится, если в таблице изменилось более 500 строк или 5% от общего числа.

Если вы используете слишком высокие значения, мусор будет копиться, как снег у неубранного подъезда. Если слишком низкие — VACUUM будет гоняться за каждым изменением, и вы получите больше нагрузки, чем пользы.

`autovacuum_analyze_threshold` и `autovacuum_analyze_scale_factor`: аналогичные параметры для анализа статистики. Анализ полезен для оптимизатора запросов.

```
ALTER TABLE your_table  SET (autovacuum_analyze_threshold = 250,       autovacuum_analyze_scale_factor = 0.02);
```

Все то же самое: балансируйте между частотой запуска и полезностью обновлений.

`autovacuum_freeze_max_age`: один из самых важных параметров — определяет, как долго строки могут оставаться без очистки до запуска полного VACUUM.

```
SET autovacuum_freeze_max_age = 200000000;
```

Для аналитических баз с большими объемами данных этот параметр стоит держать под контролем. Если его игнорировать, можно попасть в ситуацию, когда AUTOVACUUM начнет фризить данные в самый неподходящий момент.

`autovacuum_work_mem`: определяет, сколько памяти AUTOVACUUM может использовать.

```
SET autovacuum_work_mem = '512MB';
```

Больше памяти — быстрее работает. Но если на сервере мало оперативки, можно устроить гонку за ресурсы между VACUUM и вашими приложениями.

`maintenance_work_mem`:общий объем памяти для операций VACUUM и ANALYZE.

```
SET maintenance_work_mem = '512MB';
```

Этот параметр напрямую влияет на скорость обработки больших таблиц. Не жалейте памяти для больших баз.

Если у вас есть индексы GIN, то VACUUM выполняет дополнительную задачу: завершает все ожидающие операции добавления в индекс. Полезно для оптимизации работы сложных запросов.

Пример:

```
VACUUM (VERBOSE, ANALYZE) your_table;
```

Этот запрос одновременно очистит данные и обновит статистику для оптимизатора запросов.

#### Пример настройки для аналитической базы

Скажем, есть таблица `sales_data` с миллиардами записей, обновляемая ежедневно. Вот как можно настроить AUTOVACUUM:

```
ALTER TABLE sales_data  SET (autovacuum_vacuum_threshold = 1000,       autovacuum_vacuum_scale_factor = 0.01,       autovacuum_analyze_threshold = 500,       autovacuum_analyze_scale_factor = 0.005,       autovacuum_freeze_max_age = 1000000000,       autovacuum_work_mem = '1GB',       maintenance_work_mem = '2GB');
```

Эти настройки запускают VACUUM и ANALYZE чаще, чтобы оптимизировать производительность. Не забывайте регулярно проверять, как они работают на практике.

#### Проблемы и их решение

1. **VACUUM не успевает обрабатывать данные.** Увеличьте `autovacuum_max_workers`:
    
    ```
    SET autovacuum_max_workers = 10;
    ```
    
2. **Высокая нагрузка на диск.** Ограничьте `autovacuum_cost_limit`:
    
    ```
    SET autovacuum_cost_limit = 200;
    ```
    
3. **Тормоза из‑за VACUUM FULL.** Используйте `pg_repack` для реорганизации без блокировок:
    
    ```
    pg_repack -d your_database -t your_table
    ```
    

Дополнительно, чтобы избежать неожиданностей, можно настроить мониторинг AUTOVACUUM через метрики`pg_stat_activity` или `pg_stat_user_tables`.

---

Удачи в настройке и не забывайте делиться своими кейсами в комментариях!

В заключение рекомендую обратить внимание на открытые уроки, которые пройдут в Otus в рамках курса «Системный аналитик. Advanced»:

- 18 декабря: Сбор требований. Развенчание мифов. [Узнать подробнее](https://otus.pw/pbXR/)
    
- 13 января: Как снизить риски, связанные с требованиями к IT-системе. [Узнать подробнее](https://otus.pw/5IUe/)
    

Теги:

- [vacuum](https://habr.com/ru/search/?target_type=posts&order=relevance&q=[vacuum])
- [PostgreSQL](https://habr.com/ru/search/?target_type=posts&order=relevance&q=[PostgreSQL])
- [базы данных](https://habr.com/ru/search/?target_type=posts&order=relevance&q=[%D0%B1%D0%B0%D0%B7%D1%8B+%D0%B4%D0%B0%D0%BD%D0%BD%D1%8B%D1%85])

Хабы:

- [Блог компании OTUS](https://habr.com/ru/companies/otus/articles/)
- [PostgreSQL](https://habr.com/ru/hubs/postgresql/)