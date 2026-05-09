OUTER JOIN или внешнее соединение позволяет возвратить все строки одной или двух таблиц, которые участвуют в соединении.

Outer Join имеет следующий формальный синтаксис:

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4|`SELECT` `столбцы`<br><br>`FROM` `таблица1`<br><br>    `{``LEFT``\|``RIGHT``\|``FULL``} [``OUTER``]` `JOIN` `таблица2` `ON` `условие1`<br><br>    `[{``LEFT``\|``RIGHT``\|``FULL``} [``OUTER``]` `JOIN` `таблица3` `ON` `условие2]...`|

Перед оператором JOIN указывается одно из ключевых слов LEFT, RIGHT или FULL, которые определяют тип соединения:

- LEFT: выборка будет содержать все строки из первой или левой таблицы
    
- RIGHT: выборка будет содержать все строки из второй или правой таблицы
    
- FULL: выборка будет содержать все строки из обеих таблиц
    

Перед оператором JOIN может указываться ключевое слово OUTER, но его применение необязательно. После JOIN указывается присоединяемая таблица, а затем идет условие соединения после оператора ON.

К примеру, возьмем следующие таблицы:

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13<br><br>14<br><br>15<br><br>16<br><br>17<br><br>18<br><br>19<br><br>20<br><br>21<br><br>22|`CREATE` `TABLE` `Products`<br><br>`(`<br><br>    `Id SERIAL` `PRIMARY` `KEY``,`<br><br>    `ProductName` `VARCHAR``(30)` `NOT` `NULL``,`<br><br>    `Company` `VARCHAR``(20)` `NOT` `NULL``,`<br><br>    `ProductCount` `INTEGER` `DEFAULT` `0,`<br><br>    `Price` `NUMERIC` `NOT` `NULL`<br><br>`);`<br><br>`CREATE` `TABLE` `Customers`<br><br>`(`<br><br>    `Id SERIAL` `PRIMARY` `KEY``,`<br><br>    `FirstName` `VARCHAR``(30)` `NOT` `NULL`<br><br>`);`<br><br>`CREATE` `TABLE` `Orders`<br><br>`(`<br><br>    `Id SERIAL` `PRIMARY` `KEY``,`<br><br>    `ProductId` `INTEGER` `NOT` `NULL` `REFERENCES` `Products(Id)` `ON` `DELETE` `CASCADE``,`<br><br>    `CustomerId` `INTEGER` `NOT` `NULL` `REFERENCES` `Customers(Id)` `ON` `DELETE` `CASCADE``,`<br><br>    `CreatedAt` `DATE` `NOT` `NULL``,`<br><br>    `ProductCount` `INTEGER` `DEFAULT` `1,`<br><br>    `Price` `NUMERIC` `NOT` `NULL`<br><br>`);`|

И соединим таблицы Orders и Customers:

|   |   |
|---|---|
|1<br><br>2<br><br>3|`SELECT` `FirstName, CreatedAt, ProductCount, Price, ProductId`<br><br>`FROM` `Orders` `LEFT` `JOIN` `Customers`<br><br>`ON` `Orders.CustomerId = Customers.Id;`|

Таблица Orders является первой или левой таблицей, а таблица Customers - правой таблицей. Поэтому, так как здесь используется выборка по левой таблице, то вначале будут выбираться все строки из Orders, а затем к ним по условию `Orders.CustomerId = Customers.Id` будут добавляться связанные строки из Customers.

![Левостороннее соединение OUTER JOIN в PostgreSQL](https://metanit.com/sql/postgresql/pics/8.6.png)

По вышеприведенному результату может показаться, что левостороннее соединение аналогично INNER Join, но это не так. Inner Join объединяет строки из дух таблиц при соответствии условию. Если одна из таблиц содержит строки, которые не соответствуют этому условию, то данные строки не включаются в выходную выборку. Left Join выбирает все строки первой таблицы и затем присоединяет к ним строки правой таблицы. К примеру, возьмем таблицу Customers и добавим к покупателям информацию об их заказах:

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9|`-- INNER JOIN`<br><br>`SELECT` `FirstName, CreatedAt, ProductCount, Price`<br><br>`FROM` `Customers` `JOIN` `Orders`<br><br>`ON` `Orders.CustomerId = Customers.Id;`<br><br>`--LEFT JOIN`<br><br>`SELECT` `FirstName, CreatedAt, ProductCount, Price`<br><br>`FROM` `Customers` `LEFT` `JOIN` `Orders`<br><br>`ON` `Orders.CustomerId = Customers.Id;`|

![INNER join vs left join в PostgreSQL](https://metanit.com/sql/postgresql/pics/8.7.png)

Изменим в примере выше тип соединения на правостороннее:

|   |   |
|---|---|
|1<br><br>2<br><br>3|`SELECT` `FirstName, CreatedAt, ProductCount, Price, ProductId`<br><br>`FROM` `Orders` `RIGHT` `JOIN` `Customers`<br><br>`ON` `Orders.CustomerId = Customers.Id;`|

Теперь будут выбираться все строки из Customers, а к ним уже будет присоединяться связанные по условию строки из таблицы Orders:

![OUTER JOIN в PostgreSQL](https://metanit.com/sql/postgresql/pics/8.8.png)

Поскольку один из покупателей из таблицы Customers не имеет связанных заказов из Orders, то соответствующие столбцы, которые берутся из Orders, будут иметь значение NULL.

Полное соединение (FULL JOIN) объединяет обе таблицы:

|   |   |
|---|---|
|1<br><br>2<br><br>3|`SELECT` `FirstName, CreatedAt, ProductCount, Price, ProductId`<br><br>`FROM` `Orders` `FULL` `JOIN` `Customers`<br><br>`ON` `Orders.CustomerId = Customers.Id;`|

Используем левостороннее соединение для добавления к заказам информации о пользователях и товарах:

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5|`SELECT` `Customers.FirstName, Orders.CreatedAt,`<br><br>       `Products.ProductName, Products.Company`<br><br>`FROM` `Orders`<br><br>`LEFT` `JOIN` `Customers` `ON` `Orders.CustomerId = Customers.Id`<br><br>`LEFT` `JOIN` `Products` `ON` `Orders.ProductId = Products.Id;`|

![Left Join и соединение таблиц в PostgreSQL](https://metanit.com/sql/postgresql/pics/8.9.png)

И также можно применять более комплексные условия с фильтрацией и сортировкой. Например, выберем все заказы с информацией о клиентах и товарах по тем товарам, у которых цена больше 55000, и отсортируем по дате заказа:

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7|`SELECT` `Customers.FirstName, Orders.CreatedAt,`<br><br>       `Products.ProductName, Products.Company`<br><br>`FROM` `Orders`<br><br>`LEFT` `JOIN` `Customers` `ON` `Orders.CustomerId = Customers.Id`<br><br>`LEFT` `JOIN` `Products` `ON` `Orders.ProductId = Products.Id`<br><br>`WHERE` `Products.Price > 55000`<br><br>`ORDER` `BY` `Orders.CreatedAt;`|

![LEFT JOIN с фильтрацией и сортировкой в PostgreSQL](https://metanit.com/sql/postgresql/pics/8.10.png)

Или выберем всех пользователей из Customers, у которых нет заказов в таблице Orders:

|   |   |
|---|---|
|1<br><br>2<br><br>3|`SELECT` `FirstName` `FROM` `Customers`<br><br>`LEFT` `JOIN` `Orders` `ON` `Customers.Id = Orders.CustomerId`<br><br>`WHERE` `Orders.CustomerId` `IS` `NULL``;`|

Также можно комбинировать Inner Join и Outer Join:

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6|`SELECT` `Customers.FirstName, Orders.CreatedAt,`<br><br>       `Products.ProductName, Products.Company`<br><br>`FROM` `Orders`<br><br>`JOIN` `Products` `ON` `Orders.ProductId = Products.Id` `AND` `Products.Price > 45000`<br><br>`LEFT` `JOIN` `Customers` `ON` `Orders.CustomerId = Customers.Id`<br><br>`ORDER` `BY` `Orders.CreatedAt;`|

Вначале по условию к таблице Orders через Inner Join присоединяется связанная информация из Products, затем через Outer Join добавляется информация из таблицы Customers.

### Cross Join

Cross Join или перекрестное соединение создает набор строк, где каждая строка из одной таблицы соединяется с каждой строкой из второй таблицы. Например, соединим таблицу заказов Orders и таблицу покупателей Customers:

|   |   |
|---|---|
|1|`SELECT` `*` `FROM` `Orders` `CROSS` `JOIN` `Customers;`|

![CROSS JOIN в PostgreSQL](https://metanit.com/sql/postgresql/pics/8.11.png)

Если в таблице Orders 3 строки, а в таблице Customers то же три строки, то в результате перекрестного соединения создается 3 * 3 = 9 строк вне зависимости, связаны ли данные строки или нет.

При неявном перекрестном соединении можно опустить оператор CROSS JOIN и просто перечислить все получаемые таблицы:

|     |                                          |
| --- | ---------------------------------------- |
| 1   | `SELECT` `*` `FROM` `Orders, Customers;` |