| Component Name          | HTTP Method | Endpoint Path       | Brief Description                                                 |
| ----------------------- | ----------- | ------------------- | ----------------------------------------------------------------- |
| Authentication Service  | `POST`      | `/register`         | Registers a new user.                                             |
| Authentication Service  | `POST`      | `/login`            | Authenticates a user.                                             |
| Library Service         | `POST`      | `/registerbook`     | Registers a new book.                                             |
| Library Service         | `POST`      | `/registerborrower` | Registers a new borrower.                                         |
| Library Service         | `POST`      | `/lend`             | Lends a book to a borrower.                                       |
| Library Service         | `GET`       | `/book`             | Searches for a book by ID/title, or lists all books.              |
| Library Service         | `GET`       | `/borrower`         | Searches for a borrower by ID/name, or lists all borrowers.       |
| Library Service         | `GET`       | `/listavailable`    | Lists all books that are not currently on loan.                   |
| Mathematics Service     | `POST`      | `/math`             | Adds two integers.                                                |
| Mathematics Service     | `POST`      | `/ackermann`        | Calculates Ackermann's function using a specified algorithm.      |
| Mathematics Service     | `POST`      | `/fibonacci`        | Calculates the nth Fibonacci number.                              |
| Database Administration | `GET`       | `/flyway`           | Manages the database schema (clean, migrate, or both).            |
| Database Administration | `ANY`       | `/console/*`        | Provides access to the H2 Database Console.                       |