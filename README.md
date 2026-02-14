# Multi-Tenant SaaS Platform with Project & Task Management

## 1. Project Overview

### Description
This project is a production-ready multi-tenant SaaS application that enables organizations (tenants) to manage users, projects, and tasks in a secure and scalable way. The system follows a three-tier architecture consisting of a client layer (React), application layer (Node.js/Express), and data layer (PostgreSQL).

It supports strict role-based access control with Super Admin, Tenant Admin, and User roles, along with audit logging and Docker-based deployment.

### Target Audience
- SaaS startups and enterprises managing multiple organizations.
- Teams needing project and task tracking.
- Developers learning multi-tenant architecture.

---

## 2. Architecture & Design Decisions

### Multi-Tenancy Strategy
**Chosen Approach:** Shared Database with Shared Schema using a `tenant_id` column.

- **Justification:** This approach allows for easy onboarding of new tenants and ensures efficient resource utilization while enforcing strict API-level authorization and role-based access control (RBAC).
- **Isolation:** Data isolation is enforced at the application layer; every database query includes a `tenant_id` filter extracted from the authenticated JWT.

### System Components
1. **Frontend:** React-based single-page application.
2. **Backend:** Node.js (Express) REST API handling auth, logic, and data access.
3. **Database:** PostgreSQL used for persistent storage.
4. **Auth:** JWT-based authentication for securing API endpoints.

### Database Schema (ERD)
The database contains the following core tables, all isolated via `tenant_id` (except strictly global tables):
- **Tenants:** Stores organization details, subscription plans, and limits.
- **Users:** Stores credentials, roles, and tenant association.
- **Projects:** Manage project details and status.
- **Tasks:** Tracks task priority, assignment, and due dates.
- **Audit Logs:** Records user actions, entity types, and IP addresses for security.

---

## 3. Technology Stack

### Frontend
- **Framework:** React 18 (Vite).
- **State/Routing:** React Router DOM v6, Axios.
- **Justification:** Component-based architecture and efficient rendering via virtual DOM.

### Backend
- **Runtime:** Node.js 18.
- **Framework:** Express.js.
- **Auth:** JWT (JSON Web Tokens) & bcrypt.
- **Justification:** Lightweight, high performance for I/O-bound applications, and a large ecosystem.

### Database
- **RDBMS:** PostgreSQL 15.
- **Justification:** ACID compliance and strong support for relational integrity and complex queries.

### Infrastructure
- **Containerization:** Docker & Docker Compose.
- **Justification:** Ensures consistent environments across development and evaluation.

---

## 4. User Roles & Personas

The system supports three distinct user roles:

| Role | Description | Key Responsibilities |
| :--- | :--- | :--- |
| **Super Admin** | System-level administrator with unrestricted access. | Monitor system health, manage tenant registrations, oversee subscriptions. |
| **Tenant Admin** | Organization-level administrator. | Manage tenant users, create projects, enforce subscription limits. |
| **End User** | Regular team member. | View assigned projects, create/update tasks, track progress. |

---

## 5. Project Structure

The project is divided into backend and frontend directories.

```text
project-root/
├── backend/
│   ├── src/
│   │   ├── controllers/   # Request handlers
│   │   ├── middleware/    # Auth & Tenant isolation
│   │   ├── models/        # DB Schemas
│   │   ├── routes/        # API Endpoints
│   │   └── services/      # Business logic
│   ├── migrations/        # Database migrations
│   └── Dockerfile
├── frontend/
│   ├── src/
│   │   ├── components/    # Reusable UI
│   │   ├── pages/         # Dashboard, Login, etc.
│   │   ├── services/      # API communication
│   │   └── context/       # Auth state management
│   └── Dockerfile
├── docs/                  # Documentation
└── docker-compose.yml     # Orchestration

```

---

## 6. Installation & Setup

### Prerequisites

* Node.js v18+
* Docker & Docker Compose
* Git

### Quick Start (Docker)

1. **Clone the repository:**
```bash
git clone [https://github.com/Shalini-vetsa/Build-Multi-Tenant-SaaS-Platform-with-Project-Task-Management.git](https://github.com/Shalini-vetsa/Build-Multi-Tenant-SaaS-Platform-with-Project-Task-Management.git)
cd Multi-Tenant-SaaS-Platform-with-Project-and-Task-Management-System

```


2. **Start services:**
```bash
docker-compose up -d

```


3. **Access the application:**
* **Frontend:** `http://localhost:3000`
* **Backend API:** `http://localhost:5000`
* **Health Check:** `http://localhost:5000/api/health`



### Environment Variables

Configure these in `.env` or `docker-compose.yml`.

**Backend:**

```env
NODE_ENV=development
PORT=5000
DATABASE_URL=postgres://postgres:postgres@database:5432/saas_db
JWT_SECRET=supersecretjwtkey
JWT_EXPIRES_IN=24h

```

**Frontend:**

```env
VITE_API_URL=http://localhost:5000/api

```

---

## 7. Default Credentials

Use the following credentials to test different roles:

| Role | Email | Password |
| --- | --- | --- |
| **Super Admin** | `superadmin@system.com` | `Admin@123` |
| **Tenant Admin** | `admin@demo.com` | `Demo@123` |
| **User** | `user1@demo.com` | `User@123` |
| **User** | `user2@demo.com` | `User@123` |

---

## 8. API Documentation

Authentication requires a JWT token in the header: `Authorization: Bearer <token>`.

### Authentication Module

* **POST** `/api/auth/register-tenant` - Register a new tenant.
* **POST** `/api/auth/login` - Login and receive JWT.
* **GET** `/api/auth/me` - Get current user details.
* **POST** `/api/auth/logout` - Logout user.

### Tenant Module

* **GET** `/api/tenants/:tenantId` - Get tenant details.
* **PUT** `/api/tenants/:tenantId` - Update tenant details.
* **GET** `/api/tenants/` - List all tenants (**Super Admin only**).

### User Module

* **POST** `/api/tenants/:tenantId/users` - Add user to tenant (**Tenant Admin only**).
* **GET** `/api/tenants/:tenantId/users` - List users in tenant.
* **PUT** `/api/users/:userId` - Update user details.
* **DELETE** `/api/users/:userId` - Delete user (**Tenant Admin only**).

### Project Module

* **POST** `/api/projects` - Create a new project.
* **GET** `/api/projects` - List all projects.
* **PUT** `/api/projects/:projectId` - Update project (**Admin or Creator**).
* **DELETE** `/api/projects/:projectId` - Delete project (**Admin or Creator**).

### Task Module

* **POST** `/api/projects/:projectId/tasks` - Create a task.
* **GET** `/api/projects/:projectId/tasks` - List tasks in a project.
* **PATCH** `/api/tasks/:taskId/status` - Update task status.
* **PUT** `/api/tasks/:taskId` - Update task details.

### System / Super Admin Module

* **GET** `/api/projects/all` - List projects across *all* tenants (**Super Admin**).
* **GET** `/api/tasks/all` - List tasks across *all* tenants (**Super Admin**).
* **GET** `/api/health` - System health check (Public).

---

## 9. Security Features

* **Data Filtering:** Middleware enforces `tenant_id` checks on all data queries.
* **Encryption:** Passwords are hashed using bcrypt.
* **Validation:** Input validation on API requests.
* **CORS:** Configured to allow only trusted frontend origins.
* **Audit Logs:** Tracks sensitive actions for traceability.

```

```
