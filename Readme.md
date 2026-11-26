# Student Management API

This project showcases a role-based REST API for managing student accounts.
 to build this : Node.js, Express, MongoDB, Mongoose, and JWT authentication are used

## Features

- Student registration and login
- JWT-based authentication
- Admin and student authorization
- Student profile self-service
- Admin CRUD operations for student accounts
- Password hashing with bcrypt
- Request body size limits, CORS, Helmet security headers, and login rate limiting
- MongoDB duplicate-value and validation error handling



## Project Structure

```text
task7/
|-- src/
|   |-- config/
|   |   `-- db.js              # MongoDB connection
|   |-- middleware/
|   |   |-- auth.js            # JWT verification
|   |   |-- authorize.js       # Role authorization
|   |   `-- errors.js          # API error handling
|   |-- models/
|   |   `-- User.js            # User schema and password hashing
|   |-- routes/
|   |   |-- auth.js            # Registration, login, and current user
|   |   `-- students.js        # Student and admin operations
|   |-- app.js                 # Express application setup
|   `-- server.js              # Application entry point
|-- scripts/
|   `-- createAdmin.js         # Creates an initial admin account
|-- .env.example
|-- package.json
`-- README.md
```

## Prerequisites

- Node.js 18.11 or newer
- npm
- A MongoDB database (local or MongoDB Atlas)

## Installation

1. Open a terminal in the project directory.

2. Install dependencies:

   ```bash
   npm install
   ```

3. Create the environment file:

   ```bash
   copy .env.example .env
   ```

  
4. Update `.env` with your own values:

   ```env
   PORT=3000
   MONGODB_URI=mongodb://127.0.0.1:27017/student_management
   JWT_SECRET=replace-with-a-random-secret-at-least-32-characters
   JWT_EXPIRES_IN=1d

   ADMIN_NAME=System Administrator
   ADMIN_EMAIL=admin@example.com
   ADMIN_PASSWORD=replace-with-a-strong-password
   ```

   `JWT_SECRET` must contain at least 32 characters. `ADMIN_*` variables are used only by the admin provisioning command.

## Create an Admin Account

Before starting the API for the first time, configure the `ADMIN_*` values in `.env`, then run:

```bash
npm run create-admin
```

The command:

- Requires `MONGODB_URI`, `JWT_SECRET`, and all `ADMIN_*` variables.
- Creates only one account at a time.
- Rejects an admin email that already belongs to an account.
- Stores the password using bcrypt.

Public registration always creates a student account. There is no public endpoint for assigning the admin role.

## Run the API

Development mode:

```bash
npm run dev
```

Production-style start:

```bash
npm start
```

The API listens on `http://localhost:3000` by default. A health check is available at:

```text
GET http://localhost:3000/
```

## Authentication

### Register

```text
POST /api/auth/register
```

Example body:

```json
{
  "name": "Student Name",
  "email": "student@example.com",
  "password": "strong-password",
  "phone": "9876543210",
  "course": "MCA",
  "dateOfBirth": "2003-05-15",
  "address": "Ahmedabad"
}
```

Registration returns the new public user object. Students must provide `name`, `email`, and a password of at least 8 characters.

### Login

```text
POST /api/auth/login
```

Example body:

```json
{
  "email": "student@example.com",
  "password": "strong-password"
}
```

The login response contains a JWT and the user object. Login attempts are limited to 10 requests per 15 minutes per client.

### Use a Token

Send the token in the `Authorization` header:

```text
Authorization: Bearer <token>
```

### Get the Current Account

```text
GET /api/auth/me
```

This route requires authentication and returns the account associated with the JWT.

## API Routes

### Student Self-Service

Both routes require a valid student token.

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/students/me` | Read the authenticated student's profile |
| `PATCH` | `/api/students/me` | Update `name`, `phone`, `course`, `dateOfBirth`, or `address` |

Students cannot change their email, enrollment number, password, or role through `/api/students/me`.

Example profile update:

```bash
curl -X PATCH http://localhost:3000/api/students/me ^
  -H "Authorization: Bearer <student-token>" ^
  -H "Content-Type: application/json" ^
  -d "{\"phone\":\"9876543210\",\"course\":\"MCA\"}"
```

### Admin Student Management

All routes in this section require an admin token.

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/students` | List student accounts |
| `POST` | `/api/students` | Create a student account |
| `GET` | `/api/students/:id` | Read one student account |
| `PATCH` | `/api/students/:id` | Update allowed student fields |
| `DELETE` | `/api/students/:id` | Delete a student account |

The editable and creatable student fields are:

- `name`
- `email`
- `password`
- `phone`
- `course`
- `enrollmentNo`
- `dateOfBirth`
- `address`

Admin updates cannot change a student's password or role. Passwords are never returned in API responses.

## Response and Error Handling

The API uses JSON responses and common HTTP status codes:

- `200 OK` - Successful request
- `201 Created` - Account or resource created
- `204 No Content` - Student deleted
- `400 Bad Request` - Invalid input or JSON
- `401 Unauthorized` - Missing, invalid, or expired token
- `403 Forbidden` - User lacks the required role
- `404 Not Found` - Route or resource not found
- `409 Conflict` - Duplicate email or enrollment number
- `413 Payload Too Large` - Request body exceeds 10 KB
- `500 Internal Server Error` - Unexpected server error

## Security Notes

- Keep `.env` out of source control.
- Use a unique, random `JWT_SECRET` of at least 32 characters.
- Use a strong MongoDB password and restrict database network access.
- Use HTTPS when deploying publicly.
- Rotate secrets that are exposed or committed accidentally.
