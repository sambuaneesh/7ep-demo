| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---|---|---|---|
| Authentication | POST | `/demo/register` | Registers a new user. |
| Authentication | POST | `/demo/login` | Authenticates a user. |
| Library Management | POST | `/demo/registerbook` | Adds a new book title. |
| Library Management | POST | `/demo/registerborrower` | Adds a new borrower. |
| Library Management | POST | `/demo/lend` | Records a book loan. |
| Library Management | GET | `/demo/books` | Lists all books or searches by title. Returns HTML. |
| Library Management | GET | `/demo/book` | Lists or searches for books. Returns JSON. |
| Library Management | GET | `/demo/listavailable` | Lists all available (un-loaned) books. Returns JSON. |
| Library Management | GET | `/demo/borrowers` | Lists all borrowers or searches by name. Returns HTML. |
| Library Management | GET | `/demo/borrower` | Lists or searches for borrowers. Returns JSON. |
| Mathematics | POST | `/demo/math` | Adds two integers. |
| Mathematics | POST | `/demo/ack`, `/ackermann` | Calculates the Ackermann function. |
| Mathematics | POST | `/demo/fib`, `/fibonacci` | Calculates a Fibonacci number. |
| Administration | GET | `/demo/db`, `/flyway` | Manages the database schema for testing. Wipes and rebuilds. |
| Administration | GET | `/console/*` | Exposes the H2 database management console. |