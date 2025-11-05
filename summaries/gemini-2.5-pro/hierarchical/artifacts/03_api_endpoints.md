| Component Name | HTTP Method | Endpoint Path | Brief Description |
| :--- | :--- | :--- | :--- |
| Library Management | POST | `/registerbook` | Registers a new book into the library's catalog. |
| Library Management | POST | `/registerborrower` | Registers a new borrower into the system. |
| Library Management | POST | `/lend` | Lends a book to a registered borrower and records the loan. |
| Library Management | GET | `/listavailablebooks` | Retrieves a list of all books that are currently available for lending. |
| Library Management | GET | `/searchbooks` | Searches for books by ID or title, or lists all books in the catalog. |
| Library Management | GET | `/searchborrowers` | Searches for borrowers by ID or name, or lists all registered borrowers. |
| User Authentication | POST | `/register` | Handles new user registration, including password strength validation. |
| User Authentication | POST | `/login` | Authenticates an existing user's credentials and logs them in. |
| Educational Demonstrations (Mathematics) | POST | `/fib` | Calculates a Fibonacci number using a user-selected algorithm. |
| Educational Demonstrations (Mathematics) | POST | `/ack` | Calculates an Ackermann function value using a user-selected algorithm. |
| Educational Demonstrations (Mathematics) | POST | `/math` | Performs a basic arithmetic (addition) calculation on two numbers. |
| Database Management | GET | `/db` | Manages the application database with actions like clean and migrate. |