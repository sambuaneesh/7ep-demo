| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---|---|---|---|
| Authentication | POST | /login | Validates user credentials; returns access granted/denied messages. |
| Authentication | POST | /register | Registers a new user with password policy enforcement. |
| Library | POST | /registerbook | Registers a book by title. |
| Library | POST | /registerborrower | Registers a borrower by name. |
| Library | POST | /lend | Lends a book to a borrower; checks registration and availability. |
| Library | GET | /book | Lists all books or searches by id/title. |
| Library | GET | /listavailable | Lists books that are not currently on loan. |
| Library | GET | /borrower | Lists/searches borrowers by id or name. |
| Mathematics | POST | /math | Adds two integers and returns the sum. |
| Mathematics | POST | /fibonacci | Computes Fibonacci(n) using selected algorithm. |
| Mathematics | POST | /ackermann | Computes the Ackermann function with algorithm choice. |
| Persistence/DB Admin | GET | /flyway | Database maintenance: clean, migrate, or clean+migrate based on action param. |
| Web Console | GET | /console/* | H2 database web console UI. |