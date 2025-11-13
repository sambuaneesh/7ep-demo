| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---|---|---|---|
| Authentication | POST | /demo/register | Register a new user; validates inputs and password quality; returns registration status. |
| Authentication | POST | /demo/login | Authenticate user credentials; returns access granted/denied. |
| Library | POST | /demo/registerbook | Register a new book title into the catalog. |
| Library | POST | /demo/registerborrower | Register a new borrower. |
| Library | POST | /demo/lend | Lend a book to a borrower; enforces registration and availability. |
| Library | GET | /demo/book | List all books or search by id/title; returns JSON-like results or error messages. |
| Library | GET | /demo/borrower | List all borrowers or search by id/name; returns JSON-like results or error messages. |
| Library | GET | /demo/listavailable | List books available for lending (not currently on loan). |
| Mathematics | POST | /demo/math | Add two integers with validation and overflow checks. |
| Mathematics | POST | /demo/fibonacci | Compute Fibonacci number; supports multiple algorithms. |
| Mathematics | POST | /demo/ackermann | Compute Ackermann function; supports regular or tail-recursive algorithms. |
| Admin/DB | GET | /demo/flyway | Run Flyway DB actions: action=clean, action=migrate, or default clean+migrate. |
| Admin/DB | GET | /demo/console/* | H2 database web console (developer/admin use). |