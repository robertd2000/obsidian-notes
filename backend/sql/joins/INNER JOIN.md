Еще одним способом соединения таблиц является использование оператора JOIN или INNER JOIN. Он представляет так называемое внутренее соединение. Его формальный синтаксис:

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6|`SELECT` `столбцы`<br><br>`FROM` `таблица1`<br><br>    `[``INNER``]` `JOIN` `таблица2`<br><br>    `ON` `условие1`<br><br>    `[[``INNER``]` `JOIN` `таблица3`<br><br>    `ON` `условие2]`|

После оператора JOIN идет название второй таблицы, данные которой надо добавить в выборку. Перед JOIN можно указывать необязательный оператор INNER. Его наличие или отсутствие ни на что не влияет. Далее после ключевого слова ON указывается условие соединения. Это условие устанавливает, как две таблицы будут сравниваться. Как правило, для соединения применяется первичный ключ главной таблицы и внешний ключ зависимой таблицы.

Возьмем таблицы с данными из прошлой темы:

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13<br><br>14<br><br>15<br><br>16<br><br>17<br><br>18<br><br>19<br><br>20<br><br>21<br><br>22|`CREATE` `TABLE` `Products`<br><br>`(`<br><br>    `Id SERIAL` `PRIMARY` `KEY``,`<br><br>    `ProductName` `VARCHAR``(30)` `NOT` `NULL``,`<br><br>    `Company` `VARCHAR``(20)` `NOT` `NULL``,`<br><br>    `ProductCount` `INTEGER` `DEFAULT` `0,`<br><br>    `Price` `NUMERIC` `NOT` `NULL`<br><br>`);`<br><br>`CREATE` `TABLE` `Customers`<br><br>`(`<br><br>    `Id SERIAL` `PRIMARY` `KEY``,`<br><br>    `FirstName` `VARCHAR``(30)` `NOT` `NULL`<br><br>`);`<br><br>`CREATE` `TABLE` `Orders`<br><br>`(`<br><br>    `Id SERIAL` `PRIMARY` `KEY``,`<br><br>    `ProductId` `INTEGER` `NOT` `NULL` `REFERENCES` `Products(Id)` `ON` `DELETE` `CASCADE``,`<br><br>    `CustomerId` `INTEGER` `NOT` `NULL` `REFERENCES` `Customers(Id)` `ON` `DELETE` `CASCADE``,`<br><br>    `CreatedAt` `DATE` `NOT` `NULL``,`<br><br>    `ProductCount` `INTEGER` `DEFAULT` `1,`<br><br>    `Price` `NUMERIC` `NOT` `NULL`<br><br>`);`|

Используя JOIN, выберем все заказы и добавим к ним информацию о товарах:

|   |   |
|---|---|
|1<br><br>2<br><br>3|`SELECT` `Orders.CreatedAt, Orders.ProductCount, Products.ProductName`<br><br>`FROM` `Orders`<br><br>`JOIN` `Products` `ON` `Products.Id = Orders.ProductId;`|

Поскольку таблицы могут содержать столбцы с одинаковыми названиями, то при указании столбцов для выборки указывается их полное имя вместе с именем таблицы, например, "Orders.ProductCount".

![JOIN ON в PostgreSQL](https://metanit.com/sql/postgresql/pics/8.4.png)

С помощью псевдонимов, определяемых через оператор AS, можно сократить код:

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4|`SELECT` `O.CreatedAt, O.ProductCount, P.ProductName`<br><br>`FROM` `Orders` `AS` `O`<br><br>`JOIN` `Products` `AS` `P`<br><br>`ON` `P.Id = O.ProductId;`|

Подобным образом мы можем присоединять и другие таблицы. Например, добавим к заказу информацию о покупателе из таблицы Customers:

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4|`SELECT` `Orders.CreatedAt, Customers.FirstName, Products.ProductName`<br><br>`FROM` `Orders`<br><br>`JOIN` `Products` `ON` `Products.Id = Orders.ProductId`<br><br>`JOIN` `Customers` `ON` `Customers.Id=Orders.CustomerId;`|

![Соединение таблиц в PostgreSQL](https://metanit.com/sql/postgresql/pics/8.5.png)

Благодаря соединению таблиц мы можем использовать их столбцы для фильтрации выборки или ее сортировки:

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6|`SELECT` `Orders.CreatedAt, Customers.FirstName, Products.ProductName`<br><br>`FROM` `Orders`<br><br>`JOIN` `Products` `ON` `Products.Id = Orders.ProductId`<br><br>`JOIN` `Customers` `ON` `Customers.Id=Orders.CustomerId`<br><br>`WHERE` `Products.Price > 45000`<br><br>`ORDER` `BY` `Customers.FirstName;`|

Условия после ключевого слова ON могут быть более сложными по составу. Например, выбирем все заказы на товары, производителем которых является Apple.

|                                       |                                                                                                                                                                                                                                                                                                            |
| ------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1<br><br>2<br><br>3<br><br>4<br><br>5 | `SELECT` `Orders.CreatedAt, Customers.FirstName, Products.ProductName`<br><br>`FROM` `Orders`<br><br>`JOIN` `Products` `ON` `Products.Id = Orders.ProductId` `AND` `Products.Company=``'Apple'`<br><br>`JOIN` `Customers` `ON` `Customers.Id=Orders.CustomerId`<br><br>`ORDER` `BY` `Customers.FirstName;` |