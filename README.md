
TRIGGER AND VIEW
📌 XI RPL 1
📌 Anggota Kelompok
* Inka Dayu Ningtyas
* Muhammad Wildan Zulfahmi
* Talita Azaria

  Setting environment for using XAMPP for Windows.
Lenovo@DESKTOP-I7M33TA c:\users\lenovo\downloads\xampp_
# mysql -u root
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 86
Server version: 10.4.32-MariaDB mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> CREATE DATABASE online_shop3;
Query OK, 1 row affected (0.006 sec)

MariaDB [(none)]> USE online_shop3;
Database changed
MariaDB [online_shop3]> CREATE TABLE customers(
    ->     id INT PRIMARY KEY AUTO_INCREMENT,
    ->     name VARCHAR(100),
    ->     balance DECIMAL(10,2) DEFAULT 0
    -> );
Query OK, 0 rows affected (0.128 sec)

MariaDB [online_shop3]> CREATE TABLE products(
    ->     id INT PRIMARY KEY AUTO_INCREMENT,
    ->     name VARCHAR(100),
    ->     stock INT,
    ->     price DECIMAL(10,2)
    -> );
Query OK, 0 rows affected (0.495 sec)

MariaDB [online_shop3]> CREATE TABLE orders(
    ->     id INT PRIMARY KEY AUTO_INCREMENT,
    ->     customer_id INT,
    ->     total DECIMAL(10,2),
    ->     created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    -> );
Query OK, 0 rows affected (0.178 sec)

MariaDB [online_shop3]> CREATE TABLE order_details(
    ->     id INT PRIMARY KEY AUTO_INCREMENT,
    ->     order_id INT,
    ->     product_id INT,
    ->     quantity INT,
    ->     subtotal DECIMAL(10,2)
    -> );
Query OK, 0 rows affected (0.540 sec)

MariaDB [online_shop3]> CREATE TABLE logs(
    ->     id INT PRIMARY KEY AUTO_INCREMENT,
    ->     message TEXT,
    ->     created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    -> );
Query OK, 0 rows affected (0.236 sec)

MariaDB [online_shop3]> DELIMITER //
MariaDB [online_shop3]>
MariaDB [online_shop3]> CREATE TRIGGER before_insert_order_details
    -> BEFORE INSERT ON order_details
    -> FOR EACH ROW
    -> BEGIN
    ->     DECLARE product_price DECIMAL(10,2);
    ->
    ->     SELECT price INTO product_price
    ->     FROM products
    ->     WHERE id = NEW.product_id;
    ->
    ->     SET NEW.subtotal = product_price * NEW.quantity;
    -> END;
    -> //
Query OK, 0 rows affected (0.029 sec)

MariaDB [online_shop3]> CREATE TRIGGER after_insert_order_details
    -> AFTER INSERT ON order_details
    -> FOR EACH ROW
    -> BEGIN
    ->     UPDATE products
    ->     SET stock = stock - NEW.quantity
    ->     WHERE id = NEW.product_id;
    -> END;
    -> //
Query OK, 0 rows affected (0.015 sec)

MariaDB [online_shop3]> CREATE TRIGGER update_order_total
    -> AFTER INSERT ON order_details
    -> FOR EACH ROW
    -> BEGIN
    ->     UPDATE orders
    ->     SET total = (
    ->         SELECT SUM(subtotal)
    ->         FROM order_details
    ->         WHERE order_id = NEW.order_id
    ->     )
    ->     WHERE id = NEW.order_id;
    -> END;
    -> //
Query OK, 0 rows affected (0.020 sec)

MariaDB [online_shop3]> CREATE TRIGGER after_delete_order_details
    -> AFTER DELETE ON order_details
    -> FOR EACH ROW
    -> BEGIN
    ->     UPDATE products
    ->     SET stock = stock + OLD.quantity
    ->     WHERE id = OLD.product_id;
    -> END;
    -> //
Query OK, 0 rows affected (0.010 sec)

MariaDB [online_shop3]> CREATE TRIGGER log_update_product
    -> AFTER UPDATE ON products
    -> FOR EACH ROW
    -> BEGIN
    ->     INSERT INTO logs(message)
    ->     VALUES (
    ->         CONCAT(
    ->             'produk', OLD.name, OLD.stock,
    ->             'diubah menjadi', NEW.name, NEW.stock
    ->         )
    ->     );
    -> END;
    -> //
Query OK, 0 rows affected (0.013 sec)

MariaDB [online_shop3]> CREATE TRIGGER log_new_order
    -> AFTER INSERT ON orders
    -> FOR EACH ROW
    -> BEGIN
    ->     INSERT INTO logs(message)
    ->     VALUES (
    ->         CONCAT('order baru dengan id ', NEW.id)
    ->     );
    -> END;
    -> //
Query OK, 0 rows affected (0.011 sec)

MariaDB [online_shop3]>
MariaDB [online_shop3]> DELIMITER ;
MariaDB [online_shop3]> CREATE VIEW view_order_details AS
    -> SELECT
    ->     o.id AS order_id,
    ->     c.name AS customer_name,
    ->     p.name AS product_name,
    ->     od.quantity,
    ->     od.subtotal
    -> FROM orders o
    -> JOIN customers c ON c.id = o.customer_id
    -> JOIN order_details od ON od.order_id = o.id
    -> JOIN products p ON p.id = od.product_id;
Query OK, 0 rows affected (0.024 sec)

MariaDB [online_shop3]> CREATE VIEW view_customer_total AS
    -> SELECT
    ->     c.id,
    ->     c.name,
    ->     SUM(o.total) AS total_spent
    -> FROM customers c
    -> JOIN orders o ON c.id = o.customer_id
    -> GROUP BY c.id, c.name;
Query OK, 0 rows affected (0.004 sec)

MariaDB [online_shop3]> INSERT INTO customers (name, balance) VALUES
    -> ('Budi', 1000000),
    -> ('Andi', 500000),
    -> ('Siti', 1200000),
    -> ('Rina', 800000);
Query OK, 4 rows affected (0.024 sec)
Records: 4  Duplicates: 0  Warnings: 0

MariaDB [online_shop3]> INSERT INTO products (name, stock, price) VALUES
    -> ('Mouse', 10, 50000),
    -> ('Keyboard', 5, 150000),
    -> ('Headset', 8, 200000),
    -> ('Monitor', 7, 2000000),
    -> ('Flashdisk', 20, 75000);
Query OK, 5 rows affected (0.004 sec)
Records: 5  Duplicates: 0  Warnings: 0

MariaDB [online_shop3]> INSERT INTO orders (customer_id, total) VALUES
    -> (1, 0),
    -> (2, 0),
    -> (3, 0),
    -> (4, 0);
Query OK, 4 rows affected (0.008 sec)
Records: 4  Duplicates: 0  Warnings: 0

MariaDB [online_shop3]> INSERT INTO order_details (order_id, product_id, quantity)
    -> VALUES
    -> (1, 1, 2),
    -> (2, 1, 2),
    -> (3, 4, 1),
    -> (4, 5, 2);
Query OK, 4 rows affected (0.051 sec)
Records: 4  Duplicates: 0  Warnings: 0

MariaDB [online_shop3]> SELECT * FROM customers;
+----+------+------------+
| id | name | balance    |
+----+------+------------+
|  1 | Budi | 1000000.00 |
|  2 | Andi |  500000.00 |
|  3 | Siti | 1200000.00 |
|  4 | Rina |  800000.00 |
+----+------+------------+
4 rows in set (0.001 sec)

MariaDB [online_shop3]> SELECT * FROM products;
+----+-----------+-------+------------+
| id | name      | stock | price      |
+----+-----------+-------+------------+
|  1 | Mouse     |     6 |   50000.00 |
|  2 | Keyboard  |     5 |  150000.00 |
|  3 | Headset   |     8 |  200000.00 |
|  4 | Monitor   |     6 | 2000000.00 |
|  5 | Flashdisk |    18 |   75000.00 |
+----+-----------+-------+------------+
5 rows in set (0.000 sec)

MariaDB [online_shop3]> SELECT * FROM orders;
+----+-------------+------------+---------------------+
| id | customer_id | total      | created_at          |
+----+-------------+------------+---------------------+
|  1 |           1 |  100000.00 | 2026-04-08 19:18:19 |
|  2 |           2 |  100000.00 | 2026-04-08 19:18:19 |
|  3 |           3 | 2000000.00 | 2026-04-08 19:18:19 |
|  4 |           4 |  150000.00 | 2026-04-08 19:18:19 |
+----+-------------+------------+---------------------+
4 rows in set (0.000 sec)

MariaDB [online_shop3]> SELECT * FROM order_details;
+----+----------+------------+----------+------------+
| id | order_id | product_id | quantity | subtotal   |
+----+----------+------------+----------+------------+
|  1 |        1 |          1 |        2 |  100000.00 |
|  2 |        2 |          1 |        2 |  100000.00 |
|  3 |        3 |          4 |        1 | 2000000.00 |
|  4 |        4 |          5 |        2 |  150000.00 |
+----+----------+------------+----------+------------+
4 rows in set (0.000 sec)

MariaDB [online_shop3]> SELECT * FROM order_details;
+----+----------+------------+----------+------------+
| id | order_id | product_id | quantity | subtotal   |
+----+----------+------------+----------+------------+
|  1 |        1 |          1 |        2 |  100000.00 |
|  2 |        2 |          1 |        2 |  100000.00 |
|  3 |        3 |          4 |        1 | 2000000.00 |
|  4 |        4 |          5 |        2 |  150000.00 |
+----+----------+------------+----------+------------+
4 rows in set (0.000 sec)

MariaDB [online_shop3]> SELECT * FROM logs;
+----+--------------------------------------------+---------------------+
| id | message                                    | created_at          |
+----+--------------------------------------------+---------------------+
|  1 | order baru dengan id 1                     | 2026-04-08 19:18:19 |
|  2 | order baru dengan id 2                     | 2026-04-08 19:18:19 |
|  3 | order baru dengan id 3                     | 2026-04-08 19:18:19 |
|  4 | order baru dengan id 4                     | 2026-04-08 19:18:19 |
|  5 | produkMouse10diubah menjadiMouse8          | 2026-04-08 19:18:40 |
|  6 | produkMouse8diubah menjadiMouse6           | 2026-04-08 19:18:40 |
|  7 | produkMonitor7diubah menjadiMonitor6       | 2026-04-08 19:18:40 |
|  8 | produkFlashdisk20diubah menjadiFlashdisk18 | 2026-04-08 19:18:40 |
+----+--------------------------------------------+---------------------+
8 rows in set (0.002 sec)

MariaDB [online_shop3]> SELECT * FROM view_order_details;
+----------+---------------+--------------+----------+------------+
| order_id | customer_name | product_name | quantity | subtotal   |
+----------+---------------+--------------+----------+------------+
|        1 | Budi          | Mouse        |        2 |  100000.00 |
|        2 | Andi          | Mouse        |        2 |  100000.00 |
|        3 | Siti          | Monitor      |        1 | 2000000.00 |
|        4 | Rina          | Flashdisk    |        2 |  150000.00 |
+----------+---------------+--------------+----------+------------+
4 rows in set (0.004 sec)

MariaDB [online_shop3]> SELECT * FROM view_customer_total;
+----+------+-------------+
| id | name | total_spent |
+----+------+-------------+
|  1 | Budi |   100000.00 |
|  2 | Andi |   100000.00 |
|  3 | Siti |  2000000.00 |
|  4 | Rina |   150000.00 |
+----+------+-------------+
4 rows in set (0.003 sec)

MariaDB [online_shop3]>
