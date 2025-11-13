| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---|---|---|---|
| Authentication | POST | /demo/login | Validate user credentials (username, password); returns access granted/denied or input errors. |
| Authentication | POST | /demo/register | Register a new user with password policy enforcement; returns registration status and feedback. |
| Library | POST | /demo/registerbook | Register a new book by title; returns status (success/duplicate/missing title). |
| Library | POST | /demo/registerborrower | Register a new borrower by name; returns status (success/duplicate/missing name). |
| Library | POST | /demo/lend | Lend a book to a borrower; returns status (success/not registered/checked out). |
| Library | GET | /demo/book | List/search books by id or title (mutually exclusive); returns JSON-like list or errors. |
| Library | GET | /demo/listavailable | List books not currently on loan; returns JSON-like list or “No books exist in the database”. |
| Library | GET | /demo/borrower | List borrowers (JSON list) used by UI for selection/autocomplete. |
| Mathematics | POST | /demo/math | Add two integers; returns sum or “Error: only accepts integers”. |
| Mathematics | POST | /demo/fibonacci | Compute Fibonacci(n) via selected algorithm; returns result or “Error: only accepts integers”. |
| Mathematics | POST | /demo/ackermann | Compute Ackermann(m,n) via selected algorithm; returns result or “Error: only accepts integers”. |
| Persistence/DB Admin | GET | /demo/flyway | Run Flyway action (action=clean|migrate|default→clean+migrate); returns operation result. |
| H2 Console | GET | /demo/console/* | Embedded H2 database web console. |