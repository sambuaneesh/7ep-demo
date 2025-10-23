| Component Name | HTTP Method | Endpoint Path | Brief Description |
| :--- | :--- | :--- | :--- |
| Authentication | POST | `/demo/register` | Registers a new user with a username and password. |
| Authentication | POST | `/demo/login` | Authenticates an existing user with a username and password. |
| Library | POST | `/demo/registerbook` | Adds a new book title to the library catalog. |
| Library | POST | `/demo/registerborrower` | Adds a new borrower to the library system. |
| Library | POST | `/demo/lend` | Records a book being loaned to a registered borrower. |
| Library | GET | `/demo/book` | Searches for or lists all books. Can filter by `id` or `title` parameters. |
| Library | GET | `/demo/books` | Lists all available books in the library. |
| Library | GET | `/demo/borrower` | Searches for or lists all borrowers. Can filter by `id` or `name` parameters. |
| Library | GET | `/demo/listavailable` | Returns a JSON array of all books not currently on loan. |
| Mathematics | POST | `/demo/math` | Adds two integers provided as parameters. |
| Mathematics | POST | `/demo/ackermann` | Calculates the Ackermann function for two non-negative integers. |
| Mathematics | POST | `/demo/fibonacci` | Calculates a number in the Fibonacci sequence. |
| System Administration | GET | `/demo/db` | Manages the database state via an `action` parameter (e.g., `clear`). |
| System Administration | GET | `/demo/flyway` | Resets the database to a clean state by running Flyway migrations. |
| System Administration | GET | `/demo/console/*` | Exposes the web-based H2 database management console. |