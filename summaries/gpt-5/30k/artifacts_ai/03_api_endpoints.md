| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---|---|---|---|
| Admin/DB Management | GET | /flyway | Reset database: clean + migrate (cleanAndMigrateDatabase) |
| Admin/DB Management | GET | /flyway?action=clean | Clean all schemas via Flyway (cleanDatabase) |
| Admin/DB Management | GET | /flyway?action=migrate | Apply Flyway migrations (migrateDatabase) |
| Admin/DB Management | GET | /console/* | H2 Database console web UI (development use) |
| Authentication | POST | /register | Register a new user; enforces password policy and uniqueness |
| Authentication | POST | /login | Authenticate user credentials; returns access granted/denied |
| Library | POST | /registerbook | Register a new book title |
| Library | POST | /registerborrower | Register a new borrower |
| Library | POST | /lend | Lend a book to a borrower; validates existence and not already checked out |
| Library | GET | /book | List/search books by id or title; lists all when no query provided |
| Library | GET | /borrower | List/search borrowers by id or name; lists all when no query provided |
| Library | GET | /listavailable | List available books (in current model: books never loaned) |
| Mathematics | POST | /math | Add two integers with validation and overflow checks |
| Mathematics | POST | /fibonacci | Compute Fibonacci(n); supports algorithm selection |
| Mathematics | POST | /ackermann | Compute Ackermann(m,n); supports algorithm selection |