| Component Name | HTTP Method | Endpoint Path | Brief Description |
| :--- | :--- | :--- | :--- |
| Authentication & User Management | POST | `/demo/register` | Registers a new user. |
| Authentication & User Management | POST | `/demo/login` | Authenticates a user. |
| Library Management | POST | `/demo/registerbook` | Registers a new book. |
| Library Management | POST | `/demo/registerborrower` | Registers a new borrower. |
| Library Management | POST | `/demo/lend` | Creates a loan record. |
| Library Management | GET | `/demo/library-books-search` | Searches for books by ID or title. |
| Library Management | GET | `/demo/library-borrowers-search` | Searches for borrowers by ID or name. |
| Library Management | GET | `/demo/library-books-available` | Lists all books not currently on loan. |
| Mathematical Calculations | POST | `/demo/math` | Adds two integers. |
| Mathematical Calculations | POST | `/demo/fibonacci` | Calculates a Fibonacci number. |
| Mathematical Calculations | POST | `/demo/ackermann` | Calculates an Ackermann number. |
| System Administration | GET | `/demo/flyway` | Resets/migrates the database schema. |
| System Administration | GET/POST | `/demo/console/*` | Provides a web-based console for the H2 database. |