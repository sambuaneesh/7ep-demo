
| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---|---|---|---|
| Library Book List Available | GET | /books/available | Lists all available books for borrowing |
| Library Register Book | POST | /books/register | Registers a new book in the library catalog |
| Library Register Borrower | POST | /borrowers/register | Registers a new borrower in the system |
| Library Lend Book | POST | /loans/lend | Processes book lending to registered borrowers |
| Library Borrower List Search | GET | /borrowers/search | Searches for borrowers by ID or name |
| Library Book List Search | GET | /books/search | Searches for books by ID or title |
| Authentication Register | POST | /auth/register | Handles new user registration with password validation |
| Authentication Login | POST | /auth/login | Processes user authentication and login |
| Persistence Database | GET | /db/admin | Manages database operations (clean, migrate, reset) |
| Mathematics Fibonacci | POST | /math/fibonacci | Calculates Fibonacci numbers using various algorithms |
| Mathematics Basic Operations | POST | /math/calculate | Performs basic arithmetic calculations |
| Mathematics Ackermann | POST | /math/ackermann | Calculates Ackermann function values |