# Database-group-assignment

Group members:
Jasper,
Peter, 
Tsholo.

# Digital Library System README

## Overview
We, as a group, have developed a practical digital library system designed to support various functionalities such as book-selling websites, recommendation platforms, and the digital interface for physical library operations.

## Database Setup

### 1. Create Database
```sql
CREATE DATABASE bookstore;
USE bookstore;
```

### 2. Create Reference Tables

#### Country Table
```sql
CREATE TABLE country (
    country_id INT AUTO_INCREMENT PRIMARY KEY,
    country_name VARCHAR(100) NOT NULL
);
```

#### Book Language Table
```sql
CREATE TABLE book_language (
    language_id INT AUTO_INCREMENT PRIMARY KEY,
    language_name VARCHAR(50) NOT NULL
);
```

#### Publisher Table
```sql
CREATE TABLE publisher (
    publisher_id INT AUTO_INCREMENT PRIMARY KEY,
    publisher_name VARCHAR(100) NOT NULL,
    publisher_email VARCHAR(100),
    publisher_phone VARCHAR(20)
);
```

#### Author Table
```sql
CREATE TABLE author (
    author_id INT AUTO_INCREMENT PRIMARY KEY,
    author_first_name VARCHAR(50) NOT NULL,
    author_last_name VARCHAR(50) NOT NULL,
    author_email VARCHAR(100),
    author_bio TEXT
);
```

#### Address Status Table
```sql
CREATE TABLE address_status (
    status_id INT AUTO_INCREMENT PRIMARY KEY,
    status_name VARCHAR(30) NOT NULL
);
```

#### Shipping Method Table
```sql
CREATE TABLE shipping_method (
    method_id INT AUTO_INCREMENT PRIMARY KEY,
    method_name VARCHAR(50) NOT NULL,
    cost DECIMAL(6,2) NOT NULL
);
```

#### Order Status Table
```sql
CREATE TABLE order_status (
    status_id INT AUTO_INCREMENT PRIMARY KEY,
    status_name VARCHAR(30) NOT NULL
);
```

### 3. Create Main Entity Tables

#### Address Table
```sql
CREATE TABLE address (
    address_id INT AUTO_INCREMENT PRIMARY KEY,
    street_number VARCHAR(20),
    street_name VARCHAR(100) NOT NULL,
    city VARCHAR(100) NOT NULL,
    state_province VARCHAR(100),
    postal_code VARCHAR(20) NOT NULL,
    country_id INT NOT NULL,
    FOREIGN KEY (country_id) REFERENCES country(country_id)
);
```

#### Customer Table
```sql
CREATE TABLE customer (
    customer_id INT AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    phone VARCHAR(20),
    registration_date DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

#### Customer Address (Junction Table)
```sql
CREATE TABLE customer_address (
    customer_id INT NOT NULL,
    address_id INT NOT NULL,
    status_id INT NOT NULL,
    PRIMARY KEY (customer_id, address_id),
    FOREIGN KEY (customer_id) REFERENCES customer(customer_id),
    FOREIGN KEY (address_id) REFERENCES address(address_id),
    FOREIGN KEY (status_id) REFERENCES address_status(status_id)
);
```

#### Book Table
```sql
CREATE TABLE book (
    book_id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    isbn VARCHAR(20) UNIQUE,
    publication_date DATE,
    price DECIMAL(6,2) NOT NULL,
    publisher_id INT,
    language_id INT,
    page_count INT,
    description TEXT,
    FOREIGN KEY (publisher_id) REFERENCES publisher(publisher_id),
    FOREIGN KEY (language_id) REFERENCES book_language(language_id)
);
```

#### Book Author (Junction Table)
```sql
CREATE TABLE book_author (
    book_id INT NOT NULL,
    author_id INT NOT NULL,
    PRIMARY KEY (book_id, author_id),
    FOREIGN KEY (book_id) REFERENCES book(book_id),
    FOREIGN KEY (author_id) REFERENCES author(author_id)
);
```

#### Customer Order Table
```sql
CREATE TABLE cust_order (
    order_id INT AUTO_INCREMENT PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date DATETIME DEFAULT CURRENT_TIMESTAMP,
    shipping_method_id INT NOT NULL,
    destination_address_id INT NOT NULL,
    status_id INT NOT NULL,
    FOREIGN KEY (customer_id) REFERENCES customer(customer_id),
    FOREIGN KEY (shipping_method_id) REFERENCES shipping_method(method_id),
    FOREIGN KEY (destination_address_id) REFERENCES address(address_id),
    FOREIGN KEY (status_id) REFERENCES order_status(status_id)
);
```

#### Order Line Table
```sql
CREATE TABLE order_line (
    order_id INT NOT NULL,
    book_id INT NOT NULL,
    quantity INT NOT NULL,
    price DECIMAL(6,2) NOT NULL,
    PRIMARY KEY (order_id, book_id),
    FOREIGN KEY (order_id) REFERENCES cust_order(order_id),
    FOREIGN KEY (book_id) REFERENCES book(book_id)
);
```

#### Order History Table
```sql
CREATE TABLE order_history (
    history_id INT AUTO_INCREMENT PRIMARY KEY,
    order_id INT NOT NULL,
    status_id INT NOT NULL,
    status_date DATETIME DEFAULT CURRENT_TIMESTAMP,
    notes TEXT,
    FOREIGN KEY (order_id) REFERENCES cust_order(order_id),
    FOREIGN KEY (status_id) REFERENCES order_status(status_id)
);
```

### 4. Create User Roles

#### Admin Role: Jasper
```sql
CREATE USER 'jasper'@'localhost' IDENTIFIED BY 'J@sp3r_S3cure!';
GRANT ALL PRIVILEGES ON bookstore.* TO 'jasper'@'localhost';
```

#### Sales Role: Tsholo
```sql
CREATE USER 'tsholo'@'localhost' IDENTIFIED BY 'Tsh0l0_S@les2024';
GRANT SELECT, INSERT, UPDATE ON bookstore.customer TO 'tsholo'@'localhost';
GRANT SELECT, INSERT, UPDATE ON bookstore.cust_order TO 'tsholo'@'localhost';
GRANT SELECT ON bookstore.book TO 'tsholo'@'localhost';
FLUSH PRIVILEGES;  -- This is in the case of older versions of My SQL. Just in case.
```

### 5. Insert Sample Data

#### Countries, Book Languages, Publishers, Authors, Address Statuses, Shipping Methods, Order Statuses, Addresses, Customers, Customer Addresses, Books, Orders, Order Lines, Order History

(See provided SQL commands for detailed data insertion)

### 6. Create Indexes

```sql
CREATE INDEX idx_book_title ON book(title);
CREATE INDEX idx_customer_email ON customer(email);
CREATE INDEX idx_order_date ON cust_order(order_date);
```

### 7. Create Views

#### Book Details View
```sql
CREATE VIEW book_details AS
SELECT b.book_id, b.title, b.isbn, 
    GROUP_CONCAT(CONCAT(a.author_first_name, ' ', a.author_last_name)) AS authors
FROM book b
JOIN book_author ba ON b.book_id = ba.book_id
JOIN author a ON ba.author_id = a.author_id
GROUP BY b.book_id;
```

### 8. Relational Data Integrity Checks

#### Verify Book-Author Relationships (Many-to-Many)
```sql
SELECT b.title, CONCAT(a.author_first_name, ' ', a.author_last_name) AS author_name
FROM book b
JOIN book_author ba ON b.book_id = ba.book_id
JOIN author a ON ba.author_id = a.author_id
ORDER BY b.title;
```

#### Verify Customer-Order Relationships (One-to-Many)
```sql
SELECT o.order_id, CONCAT(c.first_name, ' ', c.last_name) AS customer_name, o.order_date
FROM cust_order o
JOIN customer c ON o.customer_id = c.customer_id;
```

#### Verify Order-Book Relationships (Many-to-Many via Order Line)
```sql
SELECT o.order_id, b.title, ol.quantity, ol.price
FROM cust_order o
JOIN order_line ol ON o.order_id = ol.order_id
JOIN book b ON ol.book_id = b.book_id;
```

### 9. Security & Permissions Test

#### Check Jasper’s Privileges
```sql
SHOW GRANTS FOR 'jasper'@'localhost';
```

### 10. Customer Operations

#### Customer Looking for a Book
```sql
SELECT title, isbn, price 
FROM book 
WHERE title LIKE '%Harry Potter%';
```

#### Customer Checking Order History
```sql
SELECT o.order_id, o.order_date, os.status_name, sm.method_name AS shipping_method
FROM cust_order o
JOIN order_status os ON o.status_id = os.status_id
JOIN shipping_method sm ON o.shipping_method_id = sm.method_id
WHERE o.customer_id = 1; -- Replace with actual customer_id
```

### 11. Customer Addres

#### Updating Customer Address (Sales Role Test)
```sql
UPDATE address
SET street_name = 'New Street Name'
WHERE address_id = 1;
```

## Conclusion

This repository contains a comprehensive digital library system setup, including database schema, sample data, user roles, views, and integrity checks, designed to facilitate efficient library operations across various digital platforms.

---
