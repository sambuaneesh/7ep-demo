| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---|---|---|---|
| Authentication | POST | /login | Authenticate a user with username and password; returns “access granted/denied”. |
| Authentication | POST | /register | Register a new user; validates password strength and uniqueness; returns registration status. |
| Library | POST | /registerbook | Register a new book by title; returns status (e.g., SUCCESS, ALREADY_REGISTERED_BOOK). |
| Library | POST | /registerborrower | Register a new borrower by name; returns status (e.g., SUCCESS, ALREADY_REGISTERED_BORROWER). |
| Library | POST | /lend | Lend a book to a borrower (book title, borrower name); returns lending status. |
| Library | GET | /book | List all books or search by id or title; returns JSON array or message. |
| Library | GET | /listavailable | List books currently available for borrowing; returns JSON array or message. |
| Library | GET | /borrower | List all borrowers or search by id or name; returns JSON array or message. |
| Mathematics | POST | /math | Add two integers (item_a, item_b); returns sum or an error message. |
| Mathematics | POST | /fibonacci | Compute the nth Fibonacci number; supports default or tail-recursive algorithms. |
| Mathematics | POST | /ackermann | Compute Ackermann’s function; supports default or tail-recursive algorithm. |
| Persistence/Admin | GET | /flyway | Database control: action=clean, migrate, or default clean+ migrate; returns operation status. |
| Database Console | GET | /console | H2 database web console UI. |