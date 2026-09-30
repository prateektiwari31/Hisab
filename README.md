# SplitEase 💸

> A full-stack expense-sharing app. Create groups, add shared expenses, and instantly see who owes whom, with smart debt simplification and email notifications.

![Java](https://img.shields.io/badge/Java-21-orange)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-4.1-brightgreen)
![React](https://img.shields.io/badge/React-18-61DAFB)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-Neon-336791)
![Docker](https://img.shields.io/badge/Docker-ready-2496ED)

## 📌 About

SplitEase removes the awkwardness of tracking shared money (trips, flatmates, dinners). Add an expense, choose how to split it, and SplitEase keeps every member's balance up to date. When it's time to settle, it calculates a small set of payments that clears all debts.

## ✨ Features

- 🔐 **Authentication:** register / login with stateless **JWT** and Spring Security
- 👥 **Groups:** create groups, add or remove members, search users by keyword
- 🧾 **Expenses:** add, view, edit, and delete shared expenses
- ➗ **Three split types:** `EQUAL`, `PERCENTAGE`, and `EXACT` amounts
- 📊 **Balances:** per-member net balance (who is owed / who owes)
- 🔁 **Debt simplification:** greedy algorithm that reduces the number of settlement transactions
- ✅ **Settle up:** record payments between members
- 📧 **Email notifications:** async emails for new expenses and received settlements
- 📖 **Swagger / OpenAPI docs** for every endpoint
- ⚠️ Global exception handling and request validation with clean error responses

## 🛠️ Tech Stack

| Layer        | Technology                                                      |
|--------------|-----------------------------------------------------------------|
| Backend      | Java 21, Spring Boot 4.1, Spring Web MVC                        |
| Security     | Spring Security, JWT (jjwt 0.12.3), stateless sessions          |
| Database     | PostgreSQL (hosted on Neon), Spring Data JPA / Hibernate        |
| Email        | Spring Mail (Gmail SMTP), async via a dedicated thread executor |
| API Docs     | springdoc-openapi (Swagger UI)                                  |
| Frontend     | React 18, Vite, React Router 6, Axios, Tailwind CSS             |
| Build/Deploy | Maven, Docker (multi-stage), Render                             |

## 📁 Project Structure

```
Splitease/
├── backend/                          # Spring Boot REST API
│   ├── src/main/java/com/splitease/splitease/
│   │   ├── config/                   # Security, CORS, Swagger, async config
│   │   ├── controller/               # Auth, Group, Expense, Balance controllers
│   │   ├── dto/                      # Request / response objects
│   │   ├── exception/                # Global exception handler
│   │   ├── model/                    # JPA entities (User, ExpenseGroup, Expense, ExpenseSplit)
│   │   ├── repository/               # Spring Data JPA repositories
│   │   ├── security/                 # JwtService, JwtAuthFilter
│   │   └── service/                  # Business logic (Auth, Group, Expense, Balance, Notification)
│   ├── src/main/resources/application.yaml
│   ├── Dockerfile
│   └── pom.xml
├── frontend/                         # React + Vite client
│   └── src/
│       ├── api/                      # Axios client + endpoint functions
│       ├── components/               # Modals, Navbar, Banner
│       ├── context/                  # AuthContext
│       └── pages/                    # Login, Register, Dashboard, GroupDetail
└── README.md
```

## 🚀 Getting Started

### Prerequisites

- Java 21+
- Node.js 18+ and npm
- A PostgreSQL database (local or cloud, e.g. [Neon](https://neon.tech))
- A Gmail account with an [App Password](https://support.google.com/accounts/answer/185833) (only needed for email notifications)

### 1. Clone the repository

```bash
git clone https://github.com/prateektiwari31/Splitease.git
cd Splitease
```

### 2. Run the backend

Create a `.env` file inside `backend/` (it is already git-ignored):

```env
DB_PASSWORD=your_database_password
MAIL_USERNAME=your_email@gmail.com
MAIL_APP_PASSWORD=your_gmail_app_password
PORT=8081
```

Then update the datasource URL and username in `backend/src/main/resources/application.yaml` to point to **your** database, and start the server:

```bash
cd backend
./mvnw spring-boot:run        # Windows: mvnw.cmd spring-boot:run
```

The API runs at **http://localhost:8081** and Swagger UI is at **http://localhost:8081/swagger-ui.html**.

Hibernate is set to `ddl-auto: update`, so tables are created automatically on first run.

### 3. Run the frontend

Open `frontend/src/api/client.js` and point the API to your local backend:

```js
export const BASE_URL = 'http://localhost:8081'
```

```bash
cd frontend
npm install
npm run dev
```

The app opens at **http://localhost:5173**.

### 4. Run with Docker (backend)

```bash
cd backend
docker build -t splitease-backend .
docker run -p 8081:8081 --env-file .env splitease-backend
```

## 📡 API Reference

All endpoints except `/api/auth/**` and Swagger require the header:
`Authorization: Bearer <jwt_token>`

### Auth

| Method | Endpoint             | Description                      |
|--------|----------------------|----------------------------------|
| POST   | `/api/auth/register` | Register a new user              |
| POST   | `/api/auth/login`    | Login and receive a JWT          |
| GET    | `/api/auth/me`       | Get the currently logged-in user |

### Groups

| Method | Endpoint                                 | Description       |
|--------|------------------------------------------|-------------------|
| POST   | `/api/groups`                            | Create a group    |
| GET    | `/api/groups`                            | List my groups    |
| GET    | `/api/groups/{groupId}`                  | Get group details |
| POST   | `/api/groups/{groupId}/members`          | Add a member      |
| DELETE | `/api/groups/{groupId}/members/{userId}` | Remove a member   |
| GET    | `/api/groups/users/search?keyword=`      | Search users      |

### Expenses and settlement

| Method | Endpoint                           | Description         |
|--------|------------------------------------|---------------------|
| POST   | `/api/groups/{groupId}/expenses`   | Add an expense      |
| GET    | `/api/groups/{groupId}/expenses`   | List group expenses |
| GET    | `/api/groups/expenses/{expenseId}` | Get one expense     |
| PUT    | `/api/groups/expenses/{expenseId}` | Update an expense   |
| DELETE | `/api/groups/expenses/{expenseId}` | Delete an expense   |
| POST   | `/api/groups/{groupId}/settle`     | Record a settlement |

### Balances

| Method | Endpoint                               | Description                                 |
|--------|----------------------------------------|---------------------------------------------|
| GET    | `/api/groups/{groupId}/balances`       | Net balance of every member                 |
| GET    | `/api/groups/{groupId}/simplify-debts` | Small set of payments to settle the group   |

### Example: add an expense

```http
POST /api/groups/1/expenses
Authorization: Bearer <token>
Content-Type: application/json

{
  "description": "Dinner",
  "totalAmount": 1200.00,
  "paidByUserId": 1,
  "splitType": "EQUAL",
  "participants": [1, 2, 3]
}
```

## 🧠 How Debt Simplification Works

1. Compute each member's **net balance** (total paid minus total owed).
2. Split members into **creditors** (positive) and **debtors** (negative).
3. Sort both lists by amount, largest first.
4. Repeatedly match the largest debtor with the largest creditor, settle `min(credit, debt)`, and move on when a balance reaches zero.

This greedy approach keeps the number of payments low, and never needs more than *n − 1* payments for *n* members.

## 🔒 Security Notes

- Passwords are hashed before storage.
- Stateless JWT authentication; the frontend attaches the token to every request and redirects to `/login` on a `401`.
- Database password and mail credentials are read from environment variables.

## 🗺️ Roadmap

- [ ] Unit and integration tests for services and controllers
- [ ] Role-based access (e.g. group admin vs member)
- [ ] Restrict CORS to the deployed frontend origin
- [ ] Use `BigDecimal` for money instead of `Double`
- [ ] Docker Compose for one-command local setup
- [ ] Expense categories and export to CSV

## 🤝 Contributing

1. Fork the repository
2. Create a branch: `git checkout -b feature/your-feature`
3. Commit: `git commit -m "Add your feature"`
4. Push: `git push origin feature/your-feature`
5. Open a Pull Request

## 👨‍💻 Author

**Prateek Tiwari**
GitHub: [@prateektiwari31](https://github.com/prateektiwari31)
