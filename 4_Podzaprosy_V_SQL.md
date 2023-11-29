## Тема 4. ПОДЗАПРОСЫ В SQL

## Введение в SQL подзапросы

SQL подзапросы - это мощный инструмент в языке SQL, который позволяет выполнять операции на основе результатов других SQL запросов. Они позволяют вам создавать более сложные запросы, комбинируя данные из разных таблиц или выполняя операции над подмножествами данных. 
### Примеры SQL запросов

#### Пример: Выборка компаний и имен заказчиков

```sql
SELECT company_name
FROM suppliers
WHERE country IN (SELECT country FROM customers)


SELECT DISTINCT suppliers.company_name
FROM suppliers
JOIN customers USING(country)
```

#### Пример: Суммирование товаров по категориям и использование подзапроса в LIMIT

```sql
SELECT category_name, SUM(units_in_stock)
FROM products
INNER JOIN categories ON products.category_id = categories.category_id
GROUP BY category_name
ORDER BY SUM(units_in_stock) DESC
LIMIT (SELECT MIN(product_id) + 4 FROM products)
```

#### Пример: Вычисление среднего количества товаров в наличии и выбор товаров, у которых количество в наличии больше среднего

```sql

SELECT AVG(units_in_stock)
FROM products


SELECT product_name, units_in_stock
FROM products
WHERE units_in_stock > (SELECT AVG(units_in_stock) FROM products)
ORDER BY units_in_stock
```

### WHERE EXISTS

`WHERE EXISTS` - это условие, которое используется для проверки наличия результатов подзапроса. 

#### Пример: Выбор компаний и имен заказчиков, которые делали заказы в определенный период

```sql

SELECT company_name, contact_name
FROM customers
WHERE EXISTS (SELECT customer_id FROM orders
              WHERE customer_id = customers.customer_id AND
              freight BETWEEN 50 AND 100);


SELECT company_name, contact_name
FROM customers
WHERE EXISTS (SELECT customer_id FROM orders
              WHERE customer_id = customers.customer_id AND
               orderdate BETWEEN '1995-02-01' AND '1995-02-15');


SELECT companyname, contactname
FROM customers
WHERE NOT EXISTS (SELECT customer_id FROM orders
               WHERE customerid = customers.customerid AND
                orderdate BETWEEN '1995-02-01' AND '1995-02-15');
```

### Подзапросы с квантификаторами ANY и ALL

Подзапросы с квантификаторами `ANY` и `ALL` позволяют сравнивать значение из основного запроса с набором значений, возвращаемых подзапросом.

#### Пример: Выбор компаний заказчиков, которые сделали заказы на более чем 40 единиц товаров

```sql

SELECT DISTINCT company_name
FROM customers
JOIN orders USING(customer_id)
JOIN order_details USING(order_id)
WHERE quantity > 40;

SELECT DISTINCT company_name
FROM customers
WHERE customer_id = ANY(SELECT customer_id FROM orders
                       JOIN order_details USING(order_id)
                       WHERE quantity > 40);
```

#### Пример: Выбор товаров, количество которых больше среднего по заказам

```sql

SELECT AVG(quantity)
FROM order_details;


SELECT DISTINCT product_name, quantity
FROM products
JOIN order_details USING(product_id)
WHERE quantity > (SELECT AVG(quantity) FROM order_details);
```

#### Пример: Выбор товаров, количество которых больше среднего значения по каждой категории товаров

```sql

SELECT AVG(quantity)
FROM order_details
GROUP BY product_id;


SELECT DISTINCT product_name, quantity
FROM products
JOIN order_details USING(product_id)
WHERE quantity > ALL
      (SELECT AVG(quantity)
       FROM order_details
       GROUP BY product_id)
ORDER BY quantity;
```

