TCP IP
процессы потоки
grpc
protobuf
http
rest
postgres
join
index
типы индексов
select index селективные индексы
transactions
уровни изоляции
агрегация
nosql
redis
kafka
docker


- логирование: zap, log/slog, logrus,
- unit-тесты: testify, для снепшот тестов рекомендую cupaloy
- драйвер к бд: pgx v5 самый топ для постгрес
- ORM редко юзают, но если сильно нужно то GORM самый попсовый, а bun хорош тем что на базе pgx
- squirel (попросту белка) это уже не орм, а sql билдер, важно не путать