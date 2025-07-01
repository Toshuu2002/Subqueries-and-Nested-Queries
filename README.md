# Subqueries-and-Nested-Queries

-- Table: Customers
CREATE TABLE Custom (
    customer_id INTEGER PRIMARY KEY,
    name TEXT NOT NULL,
    city TEXT
);

INSERT INTO Custom (customer_id, name, city) VALUES
(1, 'sonu', 'Delhi'),
(2, 'mihir', 'Mumbai'),
(3, 'tushar', 'Bangalore'),
(4, 'khushi', 'Chennai');

-- Table: Orders
CREATE TABLE Orders (
    order_id INTEGER PRIMARY KEY,
    customer_id INTEGER,
    order_date DATE,
    total_amount REAL,
    FOREIGN KEY (customer_id) REFERENCES Custom(customer_id)
);

INSERT INTO Orders (order_id, customer_id, order_date, total_amount) VALUES
(1001, 1, '2024-07-01', 100000),
(1002, 2, '2024-07-02', 30000),
(1003, 1, '2024-07-05', 5000),
(1004, 3, '2024-07-06', 20000);

-- Table: Products
CREATE TABLE Products (
    product_id INTEGER PRIMARY KEY,
    product_name TEXT NOT NULL,
    price REAL NOT NULL
);

INSERT INTO Products (product_id, product_name, price) VALUES
(101, 'Laptop', 70000),
(102, 'Smartphone', 30000),
(103, 'Tablet', 20000),
(104, 'Headphones', 5000);

-- Table: OrderDetails
CREATE TABLE OrderDetails (
    order_detail_id INTEGER PRIMARY KEY,
    order_id INTEGER,
    product_id INTEGER,
    quantity INTEGER,
    price REAL,
    FOREIGN KEY (order_id) REFERENCES Orders(order_id),
    FOREIGN KEY (product_id) REFERENCES Products(product_id)
);

INSERT INTO OrderDetails (order_detail_id, order_id, product_id, quantity, price) VALUES
(1, 1001, 101, 1, 70000),
(2, 1001, 102, 1, 30000),
(3, 1002, 102, 1, 30000),
(4, 1003, 104, 1, 5000),
(5, 1004, 103, 1, 20000);

SELECT 
    name,
    (SELECT MAX(order_date)
     FROM Orders o
     WHERE o.customer_id = c.customer_id) AS latest_order
FROM Custom c;

SELECT name 
FROM Custom 
WHERE customer_id IN (
    SELECT DISTINCT customer_id FROM Orders
);

SELECT name 
FROM Custom c
WHERE EXISTS (
    SELECT 1 
    FROM Orders o 
    JOIN OrderDetails od ON o.order_id = od.order_id
    WHERE o.customer_id = c.customer_id AND od.product_id = 101
);

SELECT name 
FROM Custom c
JOIN Orders o ON c.customer_id = o.customer_id
WHERE o.total_amount = (
    SELECT MAX(total_amount) FROM Orders
);

SELECT 
    c.name, 
    avg_orders.avg_total
FROM Custom c
JOIN (
    SELECT 
        customer_id, 
        AVG(total_amount) AS avg_total
    FROM Orders
    GROUP BY customer_id
) AS avg_orders ON c.customer_id = avg_orders.customer_id;

SELECT 
    product_name,
    (SELECT COUNT(*) 
     FROM OrderDetails od 
     WHERE od.product_id = p.product_id) AS times_ordered
FROM Products p;
