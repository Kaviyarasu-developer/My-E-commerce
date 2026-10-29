# My E-commerce

An online shopping backend built with Spring Boot microservices and MySQL. Customers can register, browse products, and place orders, and admins can manage the product catalog.

## Tech Stack

- Java, Spring Boot (MVC, Data JPA, Security)
- MySQL (database per service)
- JWT authentication
- Git and GitHub
- Postman
- Docker
- Cloud deployment

## Architecture (planned)

| Service | Responsibility |
|---|---|
| user-service | Registration, login, JWT |
| product-service | Product catalog and search |
| order-service | Cart and orders |
| api-gateway | Single entry point that routes requests |

## Documentation

- [Requirements](docs/requirements.md)
- [SDLC and Sprint Plan](docs/sdlc-plan.md)

## Status

Stage: Requirement Analysis complete. Design in progress.
