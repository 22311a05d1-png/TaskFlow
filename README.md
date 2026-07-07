# ⚡ TaskFlow

TaskFlow is a modern, light-weight, full-stack Task Management application. It enables administrators to assign direct tasks to team members, track progress through an interactive dashboard, and filter/manage tasks efficiently. 

This repository consists of an **Express/PostgreSQL backend API** and a **React/Vite frontend client**.

---

## 📋 Table of Contents
1. [Project Overview](#-project-overview)
2. [Features Implemented](#-features-implemented)
3. [Setup Instructions](#-setup-instructions)
4. [Environment Variables](#-environment-variables)
5. [Assumptions Made](#-assumptions-made)
6. [Known Limitations](#-known-limitations)

---

## 🔍 Project Overview

TaskFlow is designed to streamline day-to-day operations by providing a clear distinction between administrative controls and team-member task tracking.

* **Backend**: Node.js, Express, PostgreSQL (with node-postgres/`pg`), JWT-based authentication, and express-validator checks.
* **Frontend**: React (Vite), React Router v7, Vanilla CSS, Lucide icons, Axios, and React Hot Toast notifications.

---

## ✨ Features Implemented

### 🔒 Authentication & Authorization
* **JWT-based Security**: Users are authenticated using JSON Web Tokens (stored client-side).
* **Role-Based Access Control (RBAC)**: Supports `admin` and `member` roles.
* **Route Protection**: Private pages require valid JWT authentication, enforced via the [ProtectedRoute](file:///f:/Taskflow/frontend/src/components/ProtectedRoute.jsx) component and backend [auth.js](file:///f:/Taskflow/backend/src/middleware/auth.js) middleware.

### 📊 Admin Task Allocation
* **System-wide Visibility**: Administrators can see a complete list of users under [Users.jsx](file:///f:/Taskflow/frontend/src/pages/Users.jsx).
* **Assign Direct Tasks**: Admins can allocate tasks to any user with specific priorities (`low`, `medium`, `high`, `urgent`) and a mandatory due date.
* **Global Oversight**: Admins view all tasks in the system under the unified Tasks page.
* **Task Deletion**: Only admins can delete tasks (`DELETE /api/user-tasks/:id`).

### 🧑 Member Task Workspace
* **Personalized Dashboard**: Regular members view only the tasks assigned to them.
* **Status Updates**: Members can mark their assigned tasks as completed or reopen them (shifting status between `pending` and `completed`).

### 📈 Aggregated Insights & Sorting
* **Interactive Statistics**: Computes counts of Total, Pending, and Completed tasks, along with overdue tasks (due date in the past and status is not completed).
* **Urgent Tasks List**: Automatically highlights pending tasks that are marked as `urgent` or due within the next 3 days.
* **Unified Sorting**: Unified task lists are sorted by priority (urgent ➔ high ➔ medium ➔ low), followed by due date, and finally creation time.

---

## 🚀 Setup Instructions

### 1. Database Setup
1. Ensure PostgreSQL is installed and running on your machine.
2. Create a new database, for example, `projectmanagement`.
3. Initialize the schema using the [schema.sql](file:///f:/Taskflow/backend/src/db/schema.sql) file:
   ```bash
   psql -U your_postgres_user -d projectmanagement -f backend/src/db/schema.sql
   ```

### 2. Backend Setup
1. Navigate to the backend directory:
   ```bash
   cd backend
   ```
2. Install the node packages:
   ```bash
   npm install
   ```
3. Create a `.env` file in the `backend/` root directory (see [Environment Variables](#-environment-variables) below).
4. Start the backend server in development mode:
   ```bash
   npm run dev
   ```
   *The server will run by default on `http://localhost:5000`.*

### 3. Frontend Setup
1. Navigate to the frontend directory:
   ```bash
   cd ../frontend
   ```
2. Install the frontend dependencies:
   ```bash
   npm install
   ```
3. Create a `.env` file in the `frontend/` root directory (see [Environment Variables](#-environment-variables) below).
4. Run the Vite development server:
   ```bash
   npm run dev
   ```
   *The client will start by default on `http://localhost:5173`.*

---

## ⚙️ Environment Variables

### Backend Configuration
Create a `.env` file inside the [backend](file:///f:/Taskflow/backend) directory:

```env
PORT=5000
DATABASE_URL=postgresql://<username>:<password>@localhost:5432/<database_name>
JWT_SECRET=your_jwt_secret_key_here
NODE_ENV=development
```

* **`DATABASE_URL`**: PostgreSQL connection string (URL-encode special characters in passwords if applicable).
* **`JWT_SECRET`**: A strong string used to sign JSON Web Tokens.
* **`NODE_ENV`**: Determines express error stack display (`development` or `production`).

---

### Frontend Configuration
Create a `.env` file inside the [frontend](file:///f:/Taskflow/frontend) directory:

```env
VITE_API_URL=http://localhost:5000/api
```

* **`VITE_API_URL`**: Base URL of the running backend API.

---

## 💡 Assumptions Made

* **Architecture Simplification**: The system was successfully migrated away from complex workspace/project-based tasks (represented by the deleted tables: `projects`, `project_members`, `tasks`) to a simplified **standalone user task model** governed by the [user_tasks](file:///f:/Taskflow/backend/src/db/schema.sql#L25-L37) table.
* **Status Lifecycle**: The database enforces a CHECK constraint restricting task status to either `pending` or `completed` (rather than legacy statuses such as `todo`, `in_progress`, or `done`).
* **Due Date Enforcement**: Providing a due date is mandatory when creating tasks via the admin interface to prevent open-ended scheduling.
* **Auditing/Triggers**: A PostgreSQL trigger `set_user_tasks_updated_at` automatically updates the `updated_at` column whenever a row is modified in the `user_tasks` table.

---

## ⚠️ Known Limitations

1. **In-Progress Tasks Hardcoded**: In [taskController.js](file:///f:/Taskflow/backend/src/controllers/taskController.js#L16), the `in_progress_tasks` count returned to the dashboard stats is hardcoded to `0` because the database schema only stores task statuses as `pending` or `completed`.
2. **Client-side User Management Access**: While the "All Users" navigation link is hidden from regular members in the [Sidebar](file:///f:/Taskflow/frontend/src/components/Sidebar.jsx#L32-L39), there is no client-side route guard preventing members from manually entering `http://localhost:5173/admin/users` in their browser. However, any attempt to assign a task as a member will be rejected by the backend's [rbac.js](file:///f:/Taskflow/backend/src/middleware/rbac.js) middleware with a `403 Forbidden` response.
3. **Legacy UI References**: The frontend [MyTasks.jsx](file:///f:/Taskflow/frontend/src/pages/MyTasks.jsx) retains references to `task.task_type === 'project_task'` and displays `'Personal Task'` as the project name because it was built to fallback when project-level features were removed.
4. **No Self-Assignment or Task Edits for Members**: Regular members cannot create, edit, or delete task details (like titles, descriptions, due dates, or priorities); they are restricted to updating the status of tasks assigned to them.
5. **No Password Recovery**: Users cannot reset or recover forgotten passwords via the application.
