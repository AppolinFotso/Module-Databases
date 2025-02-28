# Project Brief: E-Commerce Database

In this project, you will work with an e-commerce database. The database has products that consumers can buy from different suppliers. Customers can create an order and add several products in one order.

## Learning Objectives

- Use SQL queries to retrieve specific data from a database
- Draw a database schema to visualize relationships between tables
- Label database relationships defined by the `REFERENCES` keyword in `CREATE TABLE` commands

## Requirements

### Setup

To prepare your environment, open a terminal and create a new database called `cyf_ecommerce`:

```sql
createdb cyf_ecommerce
```

Import the file [`cyf_ecommerce.sql`](./cyf_ecommerce.sql) in your newly created database:

```sql
psql -d cyf_ecommerce -f cyf_ecommerce.sql
```

### Understand the schema

Open the file `cyf_ecommerce.sql` in VSCode and examine the SQL code. Take a piece of paper and draw the database with the different relationships between tables (as defined by the REFERENCES keyword in the CREATE TABLE commands). Identify the foreign keys and make sure you understand the full database schema.

Don't skip this step. You may one day [be asked at interview](https://monzo.com/blog/2022/03/23/demystifying-the-backend-engineering-interview-process) to draw a database schema. Sketching systems is a valuable skill for back end developers and worth practising. If you're interested in systems design, you may also want to take a course on Udemy.

You can even [draw relationship diagrams](https://mermaid.js.org/syntax/entityRelationshipDiagram.html) on [GitHub](https://docs.github.com/en/get-started/writing-on-github/working-with-advanced-formatting/creating-diagrams):

```mermaid
erDiagram
    customers {
id SERIAL PK
name VARCHAR(50) NOT NULL
address VARCHAR(120)
city VARCHAR(30)
country VARCHAR(20)
    }
    customers ||--o{ orders : places
    orders {
        id SERIAL PK
    order_date DATE NOT NULL
    order_reference VARCHAR(10) NOT NULL
    customer_id INT FK
    }
    orders ||--|{ order_items : contains

order_items {
id SERIAL PK
order_id INT NOT NULL FK
product_id INT NOT NULL FK
supplier_id INT NOT NULL FK
quantity INT NOT NULL
}

products {
id SERIAL PK
product_name VARCHAR(100) NOT NULL}

products }o--o{ product_availability : available

product_availability {
prod_id integer PK, FK
supp_id integer PK, FK
unit_price integer not null
}

suppliers }|..|{ product_availability : supplies

suppliers {
id SERIAL PK
supplier_name VARCHAR(100) NOT NULL
country VARCHAR(20) NOT NULL
}
product_availability }o--o{ order_items : in
```

### Query Practice

Write SQL queries to complete the following tasks:

- [ ] List all the products whose name contains the word "socks"

```sql
SELECT * FROM products
WHERE product_name ILIKE '%socks%';

```

- [ ] List all the products which cost more than 100 showing product id, name, unit price, and supplier id

```sql
SELECT pa.prod_id, p.product_name, pa.supp_id, pa.unit_price FROM products as p
FULL OUTER JOIN product_availability as pa ON p.id = pa.prod_id WHERE unit_price > 100;
```

- [ ] List the 5 most expensive products

```sql
SELECT pa.prod_id, p.product_name, pa.supp_id, pa.unit_price FROM products as p
FULL OUTER JOIN product_availability as pa ON p.id = pa.prod_id ORDER BY unit_price DESC
LIMIT 5;
```

- [ ] List all the products sold by suppliers based in the United Kingdom. The result should only contain the columns product_name and supplier_name

```sql
SELECT  p.product_name, s.supplier_name
FROM (SELECT * FROM suppliers FULL OUTER JOIN product_availability as pa ON suppliers.id = pa.supp_id ) as s FULL OUTER JOIN products as p ON s.prod_id = p.id
WHERE country ILIKE 'United Kingdom';
```

- [ ] List all orders, including order items, from customer named Hope Crosby

```sql
SELECT o.id as order_id, o.product_name, c.name
FROM (SELECT i.product_name, o.customer_id, o.id FROM (SELECT p.product_name, i.order_id FROM products as p FULL OUTER JOIN order_items as i ON p.id = i.product_id ) as i FULL OUTER JOIN orders as o ON o.id = i.order_id) as o
FULL OUTER JOIN customers as c ON o.customer_id = c.id
WHERE name ILIKE 'Hope Crosby';
```

- [ ] List all the products in the order ORD006. The result should only contain the columns product_name, unit_price, and quantity

```sql
SELECT product_name, unit_price, quantity FROM (SELECT * FROM (SELECT * FROM product_availability as p FULL OUTER JOIN order_items as i ON p.prod_id = i.product_id) as i FULL OUTER JOIN products as p ON i.prod_id = p.id) as p
FULL OUTER JOIN orders as o ON p.order_id = o.id
WHERE order_reference = 'ORD006';
```

- [ ] List all the products with their supplier for all orders of all customers. The result should only contain the columns name (from customer), order_reference, order_date, product_name, supplier_name, and quantity

```sql
SELECT c.name, o.order_reference, o.order_date, o.product_name, o.supplier_name, o.quantity
FROM (SELECT * FROM (SELECT * FROM (SELECT * FROM (SELECT * FROM suppliers as s FULL OUTER JOIN product_availability as pa ON s.id = pa.supp_id) as pa FULL OUTER JOIN products as p ON pa.prod_id = p.id) as p FULL OUTER JOIN order_items as i ON p.prod_id = i.product_id) as i FULL OUTER JOIN orders as o ON i.order_id = o.id) as o
FULL OUTER JOIN customers as c ON o.customer_id = c.id;
```

## Acceptance Criteria

- [ ] The `cyf_ecommerce` database is imported and set up correctly
- [ ] The database schema is drawn correctly to visualize relationships between tables
- [ ] The SQL queries retrieve the correct data according to the tasks listed above
- [ ] The pull request with the answers to the tasks is opened on the `main` branch of the `E-Commerce` repository
