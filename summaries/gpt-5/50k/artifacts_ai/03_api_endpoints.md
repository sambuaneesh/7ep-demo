| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---|---|---|---|
| Authentication | POST | /demo/register | Register a new user; validates inputs and returns a detailed RegistrationResult. |
| Authentication | POST | /demo/login | Validate username/password; returns access granted/denied or missing field message. |
| Library | POST | /demo/registerbook | Register a new book title; validates and prevents duplicates. |
| Library | POST | /demo/registerborrower | Register a new borrower; validates and prevents duplicates. |
| Library | POST | /demo/lend | Lend a book to a borrower; checks registration/availability; uses current date. |
| Library | GET | /demo/book | Search books by id or title; lists all if none; returns JSON or error string. |
| Library | GET | /demo/listavailable | List available (not loaned) books as JSON. |
| Library | GET | /demo/borrower | Search borrowers by id or name; returns JSON or error string. |
| Mathematics | POST | /demo/math | Sum two integers with bounds checking; returns sum or error. |
| Mathematics | POST | /demo/fibonacci | Compute Fibonacci(n) using selected algorithm; returns value or error. |
| Mathematics | POST | /demo/ackermann | Compute Ackermann function using selected algorithm; returns value or error. |
| Database Admin | GET | /demo/flyway | Run Flyway actions (clean, migrate, or both); returns status message. |
| Web UI | GET | /demo/ | Main index page with math/algorithm forms. |
| Web UI | GET | /demo/index.html | Static index page (same as root). |
| Web UI | GET | /demo/library.html | Authentication and library management UI. |
| Web UI | GET | /demo/endpointcatalog.html | Interactive endpoint explorer UI. |
| Web UI | GET | /demo/dbhelp.html | H2 console usage instructions. |
| H2 Console | GET | /demo/console/* | Embedded H2 database web console. |