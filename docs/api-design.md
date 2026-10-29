# My E-commerce: API Design

**SDLC stage:** Design
**Style:** REST over HTTP, JSON request and response bodies

All client requests go through the gateway at `http://localhost:8080`.

---

## 1. Conventions

### Authentication
Protected endpoints need this header:

```
Authorization: Bearer <JWT token>
```

### Common status codes

| Code | Meaning | Used when |
|---|---|---|
| 200 OK | Success | Reading or updating data |
| 201 Created | New resource created | Register, add product, place order |
| 204 No Content | Success, nothing to return | Delete |
| 400 Bad Request | Invalid input | Validation fails, empty cart |
| 401 Unauthorized | Not logged in | Missing, invalid, or expired token; wrong login |
| 403 Forbidden | Logged in but not allowed | Customer calls an admin endpoint |
| 404 Not Found | Resource does not exist | Unknown product or order id |
| 409 Conflict | Conflicts with current state | Duplicate email, not enough stock |

### Error response format (same in every service)

```json
{
  "timestamp": "2026-10-28T10:15:30",
  "status": 409,
  "error": "Conflict",
  "message": "Email already registered"
}
```

---

## 2. user-service (public path `/api/users`)

| Method | Path | Access | Story | Success |
|---|---|---|---|---|
| POST | `/api/users/register` | Public | US-01 | 201 |
| POST | `/api/users/login` | Public | US-02 | 200 |

### POST /api/users/register

Request:
```json
{
  "name": "Kavi",
  "email": "kavi@example.com",
  "password": "MyPass@123"
}
```

Response 201:
```json
{
  "id": 1,
  "name": "Kavi",
  "email": "kavi@example.com",
  "role": "CUSTOMER"
}
```

Errors: 400 (invalid email, password shorter than 8 characters, missing fields), 409 (email already registered).
The response never contains the password.

### POST /api/users/login

Request:
```json
{
  "email": "kavi@example.com",
  "password": "MyPass@123"
}
```

Response 200:
```json
{
  "token": "eyJhbGciOiJIUzI1NiJ9...",
  "tokenType": "Bearer",
  "expiresInSeconds": 3600
}
```

Errors: 401 with the message `Invalid credentials` for both a wrong email and a wrong password.

---

## 3. product-service (public path `/api/products`)

| Method | Path | Access | Story | Success |
|---|---|---|---|---|
| GET | `/api/products` | Public | US-03 | 200 |
| GET | `/api/products/{id}` | Public | US-03 | 200 |
| POST | `/api/products` | ADMIN | US-04 | 201 |
| PUT | `/api/products/{id}` | ADMIN | US-04 | 200 |
| DELETE | `/api/products/{id}` | ADMIN | US-04 | 204 |

### GET /api/products

Optional query parameters: `name` (partial match) and `category`.

Example: `GET /api/products?name=phone&category=Electronics`

Response 200 (an empty list `[]` when nothing matches):
```json
[
  {
    "id": 1,
    "name": "Smart Phone X",
    "description": "6 GB RAM, 128 GB storage",
    "price": 14999.00,
    "stock": 25,
    "category": "Electronics"
  }
]
```

### POST /api/products (ADMIN)

Request:
```json
{
  "name": "Smart Phone X",
  "description": "6 GB RAM, 128 GB storage",
  "price": 14999.00,
  "stock": 25,
  "category": "Electronics"
}
```

Response 201: the created product, as shown above.

Errors: 400 (name missing, price not greater than 0, negative stock), 401 (no token), 403 (not an admin).

### PUT /api/products/{id} (ADMIN)
Same request body as POST. Response 200 with the updated product. Errors: 400, 401, 403, 404.

### DELETE /api/products/{id} (ADMIN)
Response 204. Errors: 401, 403, 404.

### Internal endpoints (service-to-service only)

These live under `/internal/products`. The gateway does **not** route them, so clients cannot reach them. Only order-service calls them, directly.

| Method | Path | Purpose |
|---|---|---|
| GET | `/internal/products?ids=1,2,3` | Get name, price, and stock for several products |
| POST | `/internal/products/reserve` | Check stock for all items and reduce it in one step |

POST `/internal/products/reserve` request:
```json
{
  "items": [
    { "productId": 1, "quantity": 2 },
    { "productId": 4, "quantity": 1 }
  ]
}
```

Response 200: the products with their current names and prices.
Response 409: if any product has insufficient stock. In this case **no stock is reduced for any item** (all or nothing, using a database transaction).
Response 404: if any product does not exist.

---

## 4. order-service (public paths `/api/cart` and `/api/orders`)

All order-service endpoints need a valid CUSTOMER token. The user id always comes from the token, never from the request.

| Method | Path | Story | Success |
|---|---|---|---|
| POST | `/api/cart/items` | US-05 | 201 |
| GET | `/api/cart` | US-05 | 200 |
| DELETE | `/api/cart/items/{productId}` | US-05 | 204 |
| POST | `/api/orders` | US-06 | 201 |
| GET | `/api/orders` | US-07 | 200 |
| GET | `/api/orders/{id}` | US-07 | 200 |

### POST /api/cart/items

Request:
```json
{ "productId": 1, "quantity": 2 }
```

Behavior: if the product is already in the cart, its quantity is increased.
Response 201:
```json
{ "productId": 1, "quantity": 2 }
```
Errors: 400 (quantity below 1), 401, 404 (product does not exist).

### GET /api/cart

Response 200:
```json
{
  "items": [
    {
      "productId": 1,
      "name": "Smart Phone X",
      "price": 14999.00,
      "quantity": 2,
      "subtotal": 29998.00
    }
  ],
  "totalAmount": 29998.00
}
```

### DELETE /api/cart/items/{productId}
Response 204. Errors: 401, 404 (item not in cart).

### POST /api/orders

No request body. The order is created from the logged-in customer's cart.

Response 201:
```json
{
  "orderId": 7,
  "status": "PLACED",
  "totalAmount": 29998.00,
  "createdAt": "2026-10-28T10:20:00",
  "items": [
    {
      "productId": 1,
      "productName": "Smart Phone X",
      "price": 14999.00,
      "quantity": 2
    }
  ]
}
```

Errors: 400 (cart is empty), 401, 409 (not enough stock; the message names the product).

### GET /api/orders
Response 200: a list of the customer's own orders, latest first, with `orderId`, `status`, `totalAmount`, and `createdAt`.

### GET /api/orders/{id}
Response 200: the full order as shown above. Response 404 if the order does not exist **or belongs to another customer** (so that order ids of other users are not revealed).

---

## 5. Design Decisions

| Decision | Reason |
|---|---|
| Nouns in URLs (`/products`, not `/getProducts`) | REST convention: the URL names the resource, the HTTP method names the action |
| User id taken from the JWT, not from the request body | A customer cannot place an order or read a cart for someone else by changing a number |
| 404 instead of 403 for another user's order | Does not confirm that the order id exists |
| `/internal/**` endpoints for service-to-service calls | Keeps stock-changing operations out of reach of clients |
| One `reserve` call that checks and reduces stock together | Avoids a gap between checking and reducing, and gives all-or-nothing behavior |
| Same error format everywhere | Clients handle errors in one way |
| Request and response DTOs instead of exposing entities | The password hash and internal fields never leave the service |
