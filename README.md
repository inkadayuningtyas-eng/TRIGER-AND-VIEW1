#  Online Shop Database (MySQL)

## TRIGGER AND VIEW

## XI RPL 1

### Anggota Kelompok

* Inka Dayu Ningtyas
* Muhammad Wildan Zulfahmi
* Talita Azaria

---

## Deskripsi

Project ini adalah implementasi **database sistem toko online sederhana** menggunakan **MySQL / MariaDB**.

Fitur:

* Manajemen pelanggan
* Manajemen produk
* Sistem order
* Logging aktivitas
* Otomatisasi dengan **Trigger**
* Reporting dengan **View**

---

##  Struktur Database

### 1. Database

```sql
CREATE DATABASE online_shop3;
USE online_shop3;
```

---

### 2. Tabel

#### Customers

```sql
CREATE TABLE customers(
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    balance DECIMAL(10,2) DEFAULT 0
);
```

#### Products

```sql
CREATE TABLE products(
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    stock INT,
    price DECIMAL(10,2)
);
```

#### Orders

```sql
CREATE TABLE orders(
    id INT PRIMARY KEY AUTO_INCREMENT,
    customer_id INT,
    total DECIMAL(10,2),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### Order Details

```sql
CREATE TABLE order_details(
    id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT,
    product_id INT,
    quantity INT,
    subtotal DECIMAL(10,2)
);
```

#### Logs

```sql
CREATE TABLE logs(
    id INT PRIMARY KEY AUTO_INCREMENT,
    message TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

##  Trigger

```sql
DELIMITER //

CREATE TRIGGER before_insert_order_details
BEFORE INSERT ON order_details
FOR EACH ROW
BEGIN
    DECLARE product_price DECIMAL(10,2);

    SELECT price INTO product_price
    FROM products
    WHERE id = NEW.product_id;

    SET NEW.subtotal = product_price * NEW.quantity;
END;
//

CREATE TRIGGER after_insert_order_details
AFTER INSERT ON order_details
FOR EACH ROW
BEGIN
    UPDATE products
    SET stock = stock - NEW.quantity
    WHERE id = NEW.product_id;
END;
//

CREATE TRIGGER update_order_total
AFTER INSERT ON order_details
FOR EACH ROW
BEGIN
    UPDATE orders
    SET total = (
        SELECT SUM(subtotal)
        FROM order_details
        WHERE order_id = NEW.order_id
    )
    WHERE id = NEW.order_id;
END;
//

CREATE TRIGGER after_delete_order_details
AFTER DELETE ON order_details
FOR EACH ROW
BEGIN
    UPDATE products
    SET stock = stock + OLD.quantity
    WHERE id = OLD.product_id;
END;
//

CREATE TRIGGER log_update_product
AFTER UPDATE ON products
FOR EACH ROW
BEGIN
    INSERT INTO logs(message)
    VALUES (
        CONCAT(
            'produk', OLD.name, OLD.stock,
            'diubah menjadi', NEW.name, NEW.stock
        )
    );
END;
//

CREATE TRIGGER log_new_order
AFTER INSERT ON orders
FOR EACH ROW
BEGIN
    INSERT INTO logs(message)
    VALUES (
        CONCAT('order baru dengan id ', NEW.id)
    );
END;
//

DELIMITER ;
```

---

##  View

```sql
CREATE VIEW view_order_details AS
SELECT 
    o.id AS order_id,
    c.name AS customer_name,
    p.name AS product_name,
    od.quantity,
    od.subtotal
FROM orders o
JOIN customers c ON c.id = o.customer_id
JOIN order_details od ON od.order_id = o.id
JOIN products p ON p.id = od.product_id;
```

```sql
CREATE VIEW view_customer_total AS
SELECT
    c.id,
    c.name,
    SUM(o.total) AS total_spent
FROM customers c
JOIN orders o ON c.id = o.customer_id
GROUP BY c.id, c.name;
```

---

##  Data 

```sql
INSERT INTO customers (name, balance) VALUES
('Budi', 1000000),
('Andi', 500000),
('Siti', 1200000),
('Rina', 800000);

INSERT INTO products (name, stock, price) VALUES
('Mouse', 10, 50000),
('Keyboard', 5, 150000),
('Headset', 8, 200000),
('Monitor', 7, 2000000),
('Flashdisk', 20, 75000);

INSERT INTO orders (customer_id, total) VALUES
(1, 0),
(2, 0),
(3, 0),
(4, 0);

INSERT INTO order_details (order_id, product_id, quantity)
VALUES 
(1, 1, 2),
(2, 1, 2),
(3, 4, 1),
(4, 5, 2);
```

---

##  Hasil

### Customers

```
Budi, Andi, Siti, Rina
```

### Products

* Stock otomatis berkurang

### Orders

* Total otomatis terisi

### Logs

* Menyimpan aktivitas trigger

---

##  Testing

```sql
SELECT * FROM customers;
SELECT * FROM products;
SELECT * FROM orders;
SELECT * FROM order_details;
SELECT * FROM logs;
SELECT * FROM view_order_details;
SELECT * FROM view_customer_total;
```

---

 Kesimpulan

Database ini menunjukkan implementasi:

* Trigger untuk automasi
* View untuk reporting
* Relasi antar tabel


