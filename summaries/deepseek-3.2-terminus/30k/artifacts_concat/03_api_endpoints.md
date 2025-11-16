```markdown
| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---------------|-------------|---------------|-------------------|
| Authentication | POST | /demo/register | User registration |
| Authentication | POST | /demo/login | User authentication |
| Library Management | POST | /demo/registerbook | Register new book |
| Library Management | POST | /demo/registerborrower | Register new borrower |
| Library Management | POST | /demo/lend | Lend book to borrower |
| Library Management | POST | /demo/lendbook | Alternative book lending endpoint |
| Library Management | GET | /demo/book | Search books by ID/title |
| Library Management | GET | /demo/borrower | Search borrowers by ID/name |
| Library Management | GET | /demo/listavailable | List all available books |
| Mathematics | POST | /demo/math | Basic arithmetic operations |
| Mathematics | POST | /demo/fib | Fibonacci calculations |
| Mathematics | POST | /demo/fibonacci | Alternative Fibonacci endpoint |
| Mathematics | POST | /demo/ack | Ackermann function calculations |
| Mathematics | POST | /demo/ackermann | Alternative Ackermann endpoint |
| Database Management | GET | /demo/flyway | Database management (clean/migrate) |
| Database Management | GET | /demo/console | H2 database console |
| Database Management | GET | /demo/db | Database management operations |
| Reporting | GET | /reports/bdd/cucumber-html-reports/overview-features.html | BDD test reports |
| Reporting | GET | /reports/dependency-check-report.html | Dependency vulnerability reports |
| Reporting | GET | /reports/zap_report.html | Security scanning reports |
```