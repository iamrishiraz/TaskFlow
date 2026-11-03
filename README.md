# TaskFlow

A team task manager where you create projects, add teammates, and assign tasks. It's a React frontend with a Spring Boot backend, and it's deployed online.

**Live app:** https://task-flow-three-alpha.vercel.app

The backend runs on a free Render instance, which goes to sleep when nobody has used it for a while. The first request after that can take up to a minute, so if the first login or signup seems stuck, give it a moment.

## What it does

**Accounts**
- Sign up with name, email and password
- Login uses JWT tokens
- Pages are protected, so logged-out users get sent to the login page

**Projects**
- Anyone can create a project and becomes its admin
- Admins add teammates by searching their email, and can remove them later
- Each person sees only the projects they belong to

**Tasks**
- A task has a title, description, due date and priority (low, medium, high)
- Status moves from To Do to In Progress to Done
- Tasks past their due date are flagged as overdue
- Tasks are assigned to project members

**Dashboard**
- Counts for total, in-progress, completed and overdue tasks
- A tasks-per-user chart for admins
- A list of recent tasks

### Who can do what

| Action | Admin | Member |
|---|---|---|
| Create tasks | Yes | No |
| Edit or delete tasks | Yes | No |
| Add or remove members | Yes | No |
| Update task status | Yes | Only on tasks assigned to them |
| View project tasks and members | Yes | Yes |

## Built with

- **Frontend:** React 18, React Router 6, Axios, Vite
- **Backend:** Spring Boot 3.2, Spring Security, Spring Data JPA
- **Auth:** JWT (jjwt 0.12.3)
- **Database:** PostgreSQL in production, in-memory H2 for local development

## Hosting

| Part | Service |
|---|---|
| Frontend | Vercel |
| Backend | Render (Docker) |
| Database | Neon (PostgreSQL) |

## Running it locally

You need Node.js 18+, Java 21 and Maven 3.9+.

**Backend**

```bash
cd backend
mvn spring-boot:run
```

It starts on http://localhost:8080. With no extra setup it uses an in-memory H2 database, so the data resets every time you restart it.

**Frontend**

```bash
cd frontend
cp .env.example .env
npm install
npm run dev
```

It starts on http://localhost:5173 and talks to the backend through `VITE_API_URL` in `.env` (the example file points to `http://localhost:8080/api`).

## Deploying your own copy

**1. Database (Neon).** Create a project and note the host, database name, user and password.

**2. Backend (Render).** Create a Web Service from this repo with the root directory set to `backend` and the runtime set to Docker. Set the health check path to `/health` and add these environment variables:

```
DATABASE_URL=jdbc:postgresql://<host>/<database>?sslmode=require
DB_USERNAME=<user>
DB_PASSWORD=<password>
DB_DRIVER=org.postgresql.Driver
HIBERNATE_DIALECT=org.hibernate.dialect.PostgreSQLDialect
JWT_SECRET=<a long random string, 32+ characters>
ALLOWED_ORIGINS=https://<your-frontend-url>
```

`DATABASE_URL` has to start with `jdbc:postgresql://`. The plain `postgresql://` link that Neon shows won't work with Spring Boot.

**3. Frontend (Vercel).** Import the repo with the root directory set to `frontend`, and add:

```
VITE_API_URL=https://<your-backend-url>/api
```

This value is read at build time, so redeploy the frontend if you change it. The `vercel.json` file makes page refreshes work with React Router.

**4.** Put the Vercel URL into `ALLOWED_ORIGINS` on Render (no trailing slash), and you're done.

## API

All endpoints except register, login and `/health` need a `Bearer` token.

**Auth**

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/auth/register` | Create an account |
| POST | `/api/auth/login` | Log in and get a token |

**Users**

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/users/search?email=` | Find a user by email |
| GET | `/api/users/me` | Current user |

**Projects**

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/projects` | Projects the user belongs to |
| POST | `/api/projects` | Create a project |
| GET | `/api/projects/:id` | Project details |
| GET | `/api/projects/:id/members` | List members |
| POST | `/api/projects/:id/members` | Add a member (admin) |
| DELETE | `/api/projects/:id/members/:userId` | Remove a member (admin) |

**Tasks**

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/tasks/my` | Tasks assigned to me |
| GET | `/api/tasks/project/:projectId` | Tasks in a project |
| GET | `/api/tasks/:id` | One task |
| POST | `/api/tasks` | Create a task (admin) |
| PATCH | `/api/tasks/:id` | Update a task |
| DELETE | `/api/tasks/:id` | Delete a task (admin) |

**Dashboard**

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/dashboard` | Stats and tasks per user |

## Project layout

```
backend/
├── src/main/java/com/taskmanager/
│   ├── controller/      REST endpoints
│   ├── model/           JPA entities
│   ├── repository/      database access
│   ├── dto/             request and response objects
│   ├── security/        JWT and Spring Security
│   └── config/          security config, error handling
├── src/main/resources/
│   └── application.properties
├── Dockerfile
└── pom.xml

frontend/
├── src/
│   ├── api/
│   │   └── client.js    Axios setup and API calls
│   ├── components/      shared layout
│   ├── context/         auth state
│   ├── pages/           login, register, dashboard, projects, tasks
│   └── utils/           badge helpers
├── vercel.json
└── package.json
```