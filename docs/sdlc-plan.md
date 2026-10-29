# My E-commerce: SDLC and Sprint Plan

## 1. Methodology: Agile

This project follows the **Agile** methodology. Instead of building the whole system in one long phase, the work is split into short sprints. Each sprint delivers one small, working, tested piece of the system.

**Why Agile:**
- Each microservice can be built, tested, and deployed as its own sprint, which matches the microservices architecture.
- Design mistakes are found early, because working code exists after every sprint.
- Requirements can be adjusted between sprints without redoing the whole project.

## 2. SDLC Stages and Deliverables

| Stage | Activities | Deliverable |
|---|---|---|
| 1. Requirement Analysis | Define actors, user stories, acceptance criteria, scope | `docs/requirements.md` |
| 2. Design | Architecture, database design, API design | `docs/architecture.md`, `docs/database-design.md`, `docs/api-design.md` |
| 3. Implementation | Code each service using feature branches | Source code on GitHub |
| 4. Testing | Unit tests (JUnit, Mockito), API tests (Postman) | Test classes, Postman collection |
| 5. Deployment | Docker packaging, cloud deployment | Live application URL |
| 6. Maintenance | Bug fixes, new features, change history | `CHANGELOG.md`, GitHub Issues |

In Agile, stages 3 to 5 are repeated inside every sprint for the feature being built.

## 3. Sprint Plan

| Sprint | Goal | User Stories | Duration |
|---|---|---|---|
| Sprint 1 | user-service: register and login with JWT | US-01, US-02 | 1-2 weeks |
| Sprint 2 | product-service: product CRUD and search | US-03, US-04 | 1-2 weeks |
| Sprint 3 | order-service: cart and orders | US-05, US-06, US-07 | 1-2 weeks |
| Sprint 4 | api-gateway and service-to-service communication | All | 1-2 weeks |
| Sprint 5 | Docker and cloud deployment | All | 1-2 weeks |

## 4. Task Tracking

- Each task is created as a **GitHub Issue** (for example, "Create register API (US-01)").
- Commit messages reference the user story ID (for example, `feat: add register API (US-01)`).
- A sprint is complete when all its acceptance criteria pass in Postman and the code is pushed to GitHub.

## 5. Definition of Done

A user story is done when:
1. The code is written and follows the layered structure.
2. All acceptance criteria for the story are verified.
3. Unit tests pass.
4. The API is tested in Postman and saved in the collection.
5. The code is committed and pushed to the repository.
