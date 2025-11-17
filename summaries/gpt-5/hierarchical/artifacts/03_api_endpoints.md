| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---|---|---|---|
| Authentication – RegisterServlet | POST | /demo/register | Register a user with password-strength enforcement; returns a typed registration result. |
| Authentication – LoginServlet | POST | Not specified (servlet mapping not in summary) | Authenticate a user by validating credentials; returns access granted/denied. |
| Library – LibraryRegisterBookServlet | POST | /demo/registerbook | Register a new book in the catalog; prevents duplicates. |
| Library – LibraryRegisterBorrowerServlet | POST | /demo/registerborrower | Register a new borrower; prevents duplicates. |
| Library – LibraryLendServlet | POST | Not specified (servlet mapping not in summary) | Lend a book to a borrower enforcing registration and availability; records loan date. |
| Library – LibraryBookListSearchServlet | GET | Not specified (servlet mapping not in summary) | List all books or search by id/title; returns a simple JSON-like payload. |
| Library – LibraryBorrowerListSearchServlet | GET | Not specified (servlet mapping not in summary) | List all borrowers or search by id/name; returns a simple JSON-like payload. |
| Library – LibraryBookListAvailableServlet | GET | Not specified (servlet mapping not in summary) | List only available (not checked-out) books. |
| Persistence – DbServlet (clean) | GET | /demo/flyway?action=clean | Trigger Flyway clean to wipe the database schema. |
| Persistence – DbServlet (migrate) | GET | /demo/flyway?action=migrate | Trigger Flyway migrate to apply database migrations. |
| Persistence – DbServlet (clean-and-migrate) | GET | /demo/flyway | Trigger a combined clean-and-migrate (default when no action provided). |
| Mathematics – MathServlet | POST | Not specified (servlet mapping not in summary) | Sum two integers and return the result. |
| Mathematics – FibServlet | POST | Not specified (servlet mapping not in summary) | Compute Fibonacci(n) using the selected algorithm (iterative or recursive). |
| Mathematics – AckServlet | POST | Not specified (servlet mapping not in summary) | Compute the Ackermann function using recursive or iterative/trampolined variant. |