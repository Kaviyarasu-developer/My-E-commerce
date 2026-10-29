# My E-commerce: Architecture

**SDLC stage:** Design

## 1. Architecture Style

The system uses a **microservices architecture**. The application is split into small services. Each service has one responsibility, runs on its own port, and owns its own MySQL database.

## 2. Components

| Component | Port | Database | Responsibility |
|---|---|---|---|
| api-gateway | 8080 | none | Single entry point; routes each request to the right service |
| user-service | 8081 | `user_db` | Registration, login, JWT creation |
| product-service | 8082 | `product_db` | Product catalog, search, stock |
| order-service | 8083 | `order_db` | Cart and orders |

## 3. Architecture Diagram

```
                    +-------------------+
   Client           |    api-gateway    |
 (Postman / app) -> |    (port 8080)    |
                    +---------+---------+
                              |
          +-------------------+-------------------+
          |                   |                   |
          v                   v                   v
 +----------------+  +-----------------+  +----------------+
 |  user-service  |  | product-service |  |  order-service |
 |   (port 8081)  |  |   (port 8082)   |  |   (port 8083)  |
 +-------+--------+  +--------+--------+  +--------+-------+
         |                    ^                    |
         v                    |                    v
   +-----------+              |             +-----------+
   |  user_db  |              +-------------|  order_db |
   |  (MySQL)  |        REST call           |  (MySQL)  |
   +-----------+   (check/reduce stock)     +-----------+
                              |
                        +------------+
                        | product_db |
                        |   (MySQL)  |
                        +------------+
```

## 4. Gateway Routing

| Request path | Routed to |
|---|---|
| `/api/users/**` | user-service (8081) |
| `/api/products/**` | product-service (8082) |
| `/api/cart/**` and `/api/orders/**` | order-service (8083) |

## 5. Communication Between Services

- **Client to services:** HTTP/REST through the gateway.
- **order-service to product-service:** synchronous REST call. When an order is placed, order-service asks product-service for the product price and stock, and asks it to reduce the stock.
- No service reads another service's database directly. Data is only shared through APIs.

## 6. Security Design

- user-service creates a **JWT** after a successful login.
- The client sends the JWT in the header `Authorization: Bearer <token>` with each request.
- Every service validates the JWT itself using a shared secret key, and reads the user id and role (CUSTOMER or ADMIN) from the token.
- Passwords are stored using BCrypt.

## 7. Key Design Decisions

| Decision | Reason |
|---|---|
| Database per service | Services stay independent; one service's schema change does not break another |
| No foreign keys across databases | Different databases cannot enforce them; `order_db` stores `user_id` and `product_id` as plain values |
| Synchronous REST between services | Simplest to understand and debug for version 1 |
| Fixed ports, no service discovery | Keeps version 1 simple; can be added later (for example Eureka) |
| Monorepo | One repository with a folder per service is easier to manage |

## 8. Request Flow Example: Place Order

1. Client sends `POST /api/orders` with the JWT to the gateway.
2. The gateway routes it to order-service.
3. order-service validates the JWT and reads the user id.
4. order-service reads the user's cart from `order_db`.
5. order-service calls product-service to check stock and get current prices.
6. If stock is enough, order-service saves the order in `order_db` and asks product-service to reduce the stock.
7. order-service clears the cart and returns the order details to the client.
