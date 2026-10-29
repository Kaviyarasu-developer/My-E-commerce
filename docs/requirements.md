# My E-commerce: Requirements

**Version:** 1.0
**SDLC stage:** Requirement Analysis
**Methodology:** Agile (sprint-based)

---

## 1. Project Purpose

My E-commerce is an online shopping backend where customers can browse products and place orders, and admins can manage the product catalog. It is built using Spring Boot microservices and MySQL, and it is developed using the Software Development Life Cycle (SDLC) with Agile sprints.

---

## 2. Actors

| Actor | Description |
|---|---|
| **Customer** | A registered user who browses products, manages a cart, and places orders. |
| **Admin** | A privileged user who manages the product catalog. |

---

## 3. Functional Requirements (User Stories)

| ID | User Story |
|---|---|
| US-01 | As a customer, I want to register, so that I can have my own account. |
| US-02 | As a customer, I want to log in, so that I can shop securely. |
| US-03 | As a customer, I want to browse and search products, so that I can find what I want to buy. |
| US-04 | As an admin, I want to add, update, and delete products, so that the catalog stays current. |
| US-05 | As a customer, I want to add products to a cart, so that I can buy several items together. |
| US-06 | As a customer, I want to place an order, so that I can purchase my cart items. |
| US-07 | As a customer, I want to view my past orders, so that I can track what I bought. |

---

## 4. Acceptance Criteria

### US-01: Register
- Registration requires name, email, and password.
- The email must be in a valid format.
- The email must be unique; registering with an existing email returns an error.
- The password must be at least 8 characters long.
- The password is stored in encrypted form (BCrypt), never as plain text.
- On success, the system returns a "registered successfully" response with HTTP status 201.
- New users are assigned the role CUSTOMER by default.

### US-02: Login
- Login requires email and password.
- If the email and password are correct, the system returns a JWT token with HTTP status 200.
- If the email does not exist or the password is wrong, the system returns HTTP status 401 with a generic "invalid credentials" message. It must not reveal which of the two was wrong.
- The JWT token contains the user's identity and role, and it expires after a fixed time (for example, 1 hour).
- Requests to protected APIs without a valid token are rejected with HTTP status 401.

### US-03: Browse and Search Products
- Any user can view the list of products without logging in.
- Each product shows name, description, price, category, and available stock.
- Products can be searched by name (partial match is allowed).
- Products can be filtered by category.
- If no product matches, the system returns an empty list, not an error.

### US-04: Admin Manages Products
- Only a user with the role ADMIN can add, update, or delete products.
- A customer attempting these actions receives HTTP status 403.
- Product name and price are mandatory, and price must be greater than zero.
- Stock cannot be negative.
- Updating or deleting a product that does not exist returns HTTP status 404.

### US-05: Add to Cart
- Only a logged-in customer can use the cart.
- The customer can add a product with a quantity of at least 1.
- Adding a product that is already in the cart increases its quantity.
- The customer can remove an item from the cart.
- The customer can view their cart with the items and the total price.

### US-06: Place Order
- Only a logged-in customer can place an order.
- The cart must not be empty.
- If the stock of any product is not enough, the order is rejected with a clear message.
- After a successful order, the product stock is reduced by the ordered quantity.
- After a successful order, the cart is emptied.
- The order stores the products, quantities, prices at the time of purchase, total amount, and date.

### US-07: View Past Orders
- Only a logged-in customer can view orders.
- A customer sees only their own orders, never another customer's.
- Orders are listed with the latest first.
- A customer can open one order to see its full details.

---

## 5. Non-Functional Requirements

| Category | Requirement |
|---|---|
| **Security** | Passwords are encrypted using BCrypt. APIs are protected using JWT. Role-based access control separates CUSTOMER and ADMIN. |
| **Architecture** | The system follows a microservices architecture. Each service is independently runnable and deployable. |
| **Data** | Each service has its own MySQL database (database per service). |
| **Maintainability** | Code follows a layered structure (Controller, Service, Repository). |
| **Reliability** | APIs return meaningful HTTP status codes and error messages. Input is validated. |
| **Documentation** | All APIs are documented and tested using a Postman collection stored in the repository. |
| **Version control** | Work is tracked using Git with feature branches and meaningful commit messages. |
| **Deployment** | The application is packaged with Docker and deployed to the cloud. |

---

## 6. Out of Scope (Version 1)

- Payment gateway integration
- Email or SMS notifications
- Product reviews and ratings
- Frontend user interface (this version focuses on the backend)
- Discount coupons and offers

---

## 7. Planned Services (to be finalized in the Design stage)

| Service | Covers |
|---|---|
| user-service | US-01, US-02 |
| product-service | US-03, US-04 |
| order-service | US-05, US-06, US-07 |
| api-gateway | Single entry point that routes requests to the services |
