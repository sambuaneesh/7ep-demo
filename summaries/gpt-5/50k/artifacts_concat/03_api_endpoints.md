| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---|---|---|---|
| Authentication | POST | /demo/register | Register a new user (username, password). |
| Authentication | POST | /demo/login | Authenticate user (username, password). |
| Library | POST | /demo/registerbook | Register a new book (book title). |
| Library | POST | /demo/registerborrower | Register a new borrower (name). |
| Library | POST | /demo/lend | Lend a book to a borrower. |
| Library | GET | /demo/book | List/search books by id or title; lists all when none provided. |
| Library | GET | /demo/listavailable | List books currently available (not on loan). |
| Library | GET | /demo/borrower | List/search borrowers by id or name; lists all when none provided. |
| Mathematics | POST | /demo/math | Add two integers. |
| Mathematics | POST | /demo/fibonacci | Compute Fibonacci number (algorithm choice supported). |
| Mathematics | POST | /demo/ackermann | Compute Ackermann function (algorithm choice supported). |
| Persistence/Admin | GET | /demo/flyway | Run Flyway DB actions: clean, migrate, or clean+migrate (via action param). |
| Persistence/Admin | GET | /demo/console | H2 database web console (development). |
| Web UI | GET | /demo/index.html | Home/landing page. |
| Web UI | GET | /demo/library.html | Library/admin UI with forms for auth and library operations. |
| Web UI | GET | /demo/endpointcatalog.html | Endpoint catalog UI to invoke APIs. |
| Web UI | GET | /demo/dbhelp.html | Database help page (H2 connection info). |