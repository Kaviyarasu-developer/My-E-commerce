# My E-commerce: Database Design

**SDLC stage:** Design
**Database:** MySQL (one database per service)

| Service | Database | Tables |
|---|---|---|
| user-service | `user_db` | `users` |
| product-service | `product_db` | `products` |
| order-service | `order_db` | `cart_items`, `orders`, `order_items` |

---

## 1. user_db

### users

| Column | Type | Constraints | Notes |
|---|---|---|---|
| id | BIGINT | PK, AUTO_INCREMENT | |
| name | VARCHAR(100) | NOT NULL | |
| email | VARCHAR(150) | NOT NULL, UNIQUE | Used to log in |
| password | VARCHAR(255) | NOT NULL | BCrypt hash, never plain text |
| role | VARCHAR(20) | NOT NULL, default `CUSTOMER` | `CUSTOMER` or `ADMIN` |
| created_at | TIMESTAMP | default current time | |

```sql
CREATE DATABASE IF NOT EXISTS user_db;
USE user_db;

CREATE TABLE users (
    id         BIGINT AUTO_INCREMENT PRIMARY KEY,
    name       VARCHAR(100) NOT NULL,
    email      VARCHAR(150) NOT NULL UNIQUE,
    password   VARCHAR(255) NOT NULL,
    role       VARCHAR(20)  NOT NULL DEFAULT 'CUSTOMER',
    created_at TIMESTAMP    NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

---

## 2. product_db

### products

| Column | Type | Constraints | Notes |
|---|---|---|---|
| id | BIGINT | PK, AUTO_INCREMENT | |
| name | VARCHAR(150) | NOT NULL | |
| description | TEXT | | |
| price | DECIMAL(10,2) | NOT NULL | Must be greater than 0 |
| stock | INT | NOT NULL, default 0 | Cannot be negative |
| category | VARCHAR(100) | | Used for filtering |
| created_at | TIMESTAMP | default current time | |
| updated_at | TIMESTAMP | auto-updated | |

```sql
CREATE DATABASE IF NOT EXISTS product_db;
USE product_db;

CREATE TABLE products (
    id          BIGINT AUTO_INCREMENT PRIMARY KEY,
    name        VARCHAR(150)  NOT NULL,
    description TEXT,
    price       DECIMAL(10,2) NOT NULL,
    stock       INT           NOT NULL DEFAULT 0,
    category    VARCHAR(100),
    created_at  TIMESTAMP     NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at  TIMESTAMP     NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    CONSTRAINT chk_price CHECK (price > 0),
    CONSTRAINT chk_stock CHECK (stock >= 0)
);

CREATE INDEX idx_products_category ON products(category);
CREATE INDEX idx_products_name ON products(name);
```

---

## 3. order_db

### cart_items

| Column | Type | Constraints | Notes |
|---|---|---|---|
| id | BIGINT | PK, AUTO_INCREMENT | |
| user_id | BIGINT | NOT NULL | From JWT; not a foreign key (different database) |
| product_id | BIGINT | NOT NULL | Not a foreign key (different database) |
| quantity | INT | NOT NULL | At least 1 |

A user can have only one row per product, so `(user_id, product_id)` is unique.

### orders

| Column | Type | Constraints | Notes |
|---|---|---|---|
| id | BIGINT | PK, AUTO_INCREMENT | |
| user_id | BIGINT | NOT NULL | Not a foreign key (different database) |
| total_amount | DECIMAL(12,2) | NOT NULL | |
| status | VARCHAR(20) | NOT NULL, default `PLACED` | |
| created_at | TIMESTAMP | default current time | |

### order_items

| Column | Type | Constraints | Notes |
|---|---|---|---|
| id | BIGINT | PK, AUTO_INCREMENT | |
| order_id | BIGINT | NOT NULL, FK to `orders(id)` | Same database, so a real foreign key |
| product_id | BIGINT | NOT NULL | Not a foreign key (different database) |
| product_name | VARCHAR(150) | NOT NULL | Copy of the name at purchase time |
| price | DECIMAL(10,2) | NOT NULL | Copy of the price at purchase time |
| quantity | INT | NOT NULL | |

```sql
CREATE DATABASE IF NOT EXISTS order_db;
USE order_db;

CREATE TABLE cart_items (
    id         BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id    BIGINT NOT NULL,
    product_id BIGINT NOT NULL,
    quantity   INT    NOT NULL,
    CONSTRAINT chk_cart_qty CHECK (quantity >= 1),
    CONSTRAINT uq_cart_user_product UNIQUE (user_id, product_id)
);

CREATE TABLE orders (
    id           BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id      BIGINT        NOT NULL,
    total_amount DECIMAL(12,2) NOT NULL,
    status       VARCHAR(20)   NOT NULL DEFAULT 'PLACED',
    created_at   TIMESTAMP     NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE order_items (
    id           BIGINT AUTO_INCREMENT PRIMARY KEY,
    order_id     BIGINT        NOT NULL,
    product_id   BIGINT        NOT NULL,
    product_name VARCHAR(150)  NOT NULL,
    price        DECIMAL(10,2) NOT NULL,
    quantity     INT           NOT NULL,
    CONSTRAINT fk_order_items_order FOREIGN KEY (order_id) REFERENCES orders(id)
);

CREATE INDEX idx_orders_user ON orders(user_id);
```

---

## 4. Relationships

```
user_db                product_db              order_db
+--------+             +----------+            +-------------+
| users  |             | products |            | cart_items  |
+--------+             +----------+            +-------------+
                                               +--------+  1      N  +-------------+
                                               | orders |-----------<| order_items |
                                               +--------+            +-------------+
```

- **Inside `order_db`:** one order has many order items (one-to-many), enforced by a real foreign key.
- **Across databases:** `user_id` and `product_id` are plain numbers. The application checks them through APIs. The database cannot.

---

## 5. Design Decisions

| Decision | Reason |
|---|---|
| `DECIMAL` for money | `FLOAT` and `DOUBLE` give rounding errors; `DECIMAL` is exact |
| Password column of 255 characters | A BCrypt hash is about 60 characters; extra space allows a future change of algorithm |
| `product_name` and `price` copied into `order_items` | Old orders must keep showing what the customer paid, even if the product changes later |
| Unique `email` | Enforces the "no duplicate registration" acceptance criterion at database level |
| `CHECK` constraints | Enforce price greater than 0 and stock not negative at database level, in addition to code validation |
| Indexes on `category`, `name`, `user_id` | These columns are used in searches and filters, so indexes make them faster |
| `ddl-auto` set to `validate` or `none` in later phases | Scripts like these stay the source of truth; we will use them with version control |
