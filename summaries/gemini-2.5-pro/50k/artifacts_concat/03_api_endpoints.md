| Component Name | HTTP Method | Endpoint Path | Brief Description |
| :--- | :--- | :--- | :--- |
| Authentication | POST | /demo/register | Registers a new user. |
| Authentication | POST | /demo/login | Authenticates an existing user. |
| Library | POST | /demo/registerbook | Registers a new book in the library. |
| Library | POST | /demo/registerborrower | Registers a new borrower. |
| Library | POST | /demo/lend | Creates a loan record for a book and borrower. |
| Library | GET | /demo/library-books-available | Lists all books that are not currently on loan. |
| Library | GET | /demo/library-books-search | Searches for books by ID or title. |
| Library | GET | /demo/library-borrowers-search | Searches for borrowers by ID or name. |
| Mathematics | POST | /demo/math | Performs simple addition of two integers. |
| Mathematics | POST | /demo/fibonacci | Calculates a Fibonacci number using a selected algorithm. |
| Mathematics | POST | /demo/ackermann | Calculates an Ackermann number using a selected algorithm. |
| System Administration | GET | /demo/flyway | Resets and migrates the database schema (for testing). |
| System Administration | GET/POST | /demo/console/* | Provides a web-based console for the H2 database. |