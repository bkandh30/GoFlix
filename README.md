# GoFlix

A JSON API which is used to retrieve and manage information about movies using `Golang`.

## Table of Contents

- [Features](#features)
- [API Endpoints](#api-endpoints)
- [API Testing Guide](./TESTING.md)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Clone the Repository](#1-clone-the-repository)
  - [Configuration (Environment Variables)](#2-configuration-environment-variables)
  - [Database Migrations](#3-database-migrations)
  - [Install Dependencies](#4-install-dependencies)
- [Usage](#usage)
  - [Makefile](#makefile)
  - [Running the Application](#running-the-application)
  - [Building for Production](#building-for-production)
  - [Checking Application Metrics](#checking-application-metrics)

## Features

- Full CRUD functionality for movies.
- User registration and authentication.
- JWT-based authentication tokens.
- Secure password handling with bcrypt.
- Background tasks for sending emails (e.g., account activation).
- Structured logging and error handling.
- Database migrations support.
- Graceful shutdown.
- CORS support.
- Application metrics endpoint.

## API Endpoints

| Method | URL Pattern               | Action                                          |
| ------ | ------------------------- | ----------------------------------------------- |
| GET    | /v1/healthcheck           | Show application health and version information |
| GET    | /v1/movies                | Show the details of all movies                  |
| POST   | /v1/movies                | Create a new movie                              |
| GET    | /v1/movies/:id            | Show the details of a specific movie            |
| PATCH  | /v1/movies/:id            | Update the details of a specific movie          |
| DELETE | /v1/movies/:id            | Delete a specific movie                         |
| POST   | /v1/users                 | Register a new user                             |
| PUT    | /v1/users/activated       | Activate a specific user                        |
| PUT    | /v1/users/password        | Update the password for a specific user         |
| POST   | /v1/tokens/authentication | Generate a new authentication token             |
| POST   | /v1/tokens/activation     | Generate a new activation token                 |
| POST   | /v1/tokens/password-reset | Generate a new password-reset token             |
| GET    | /debug/vars               | Display application metrics                     |

## Getting Started

Follow these instructions to get the project up and running on your local machine for development and testing purposes.

### Prerequisites

You will need to have the following tools installed on your system:

- **Go**: Version 1.24 or newer.
- **PostgreSQL**: A running instance of the PostgreSQL database.
- **`golang-migrate/migrate`**: A CLI tool for managing database migrations. [Installation instructions](https://github.com/golang-migrate/migrate/tree/master/cmd/migrate#installation).
- **`hey`**: A tool for performing CORS request simulation on the API. [Installation instructions](https://github.com/rakyll/hey#installation).

### 1. Clone the Repository

```bash
git clone
cd goflix
```

### 2. Configuration (Environment Variables)

The application is configured using environment variables. Create a `.env` file in the root of the project by copying the example file:

```bash
cp .env.example .env
```

Now, open the `.env` file and fill in the values for your local environment.

Note: Some default values are also given in `.envExample` file that were used by me,

Also create a `.envrc` file in the root of the project using:

```bash
touch .envrc
```

In this file, add the following variable and it's corresponding value:

```bash
export GOFLIX_DB_DSN=
```

### 3. Database Migrations

Migrations are used to manage the database schema. They are located in the `/migrations` directory.

**To apply all up migrations:**

This will create the necessary tables (`movies`, `users`, etc.) in your database.

```bash
migrate -path ./migrations -database "$DB_DSN" up
```

**To roll back the last migration:**

```bash
migrate -path ./migrations -database "$DB_DSN" down 1
```

### 4. Install Dependencies

This project uses Go Modules to manage dependencies. To ensure you have all the necessary packages for reproducible builds, you can vendor them.

```bash
# Tidy ensures the go.mod file matches the source code.
go mod tidy

# Vendor copies all dependencies into the /vendor directory.
go mod vendor
```

You can see the vendored packages with:

```bash
# List the top-level vendored directories
tree -L 1 ./vendor/
```

## Usage

### Makefile

A `Makefile` is included to streamline common tasks.

| Command                  | Description                              |
| ------------------------ | ---------------------------------------- |
| `make run/api`           | Runs the API application                 |
| `make build/api`         | Builds the API application binary        |
| `make db/psql`           | Connect to the database using psql       |
| `make db/migrations/new` | Create a new database migration          |
| `make db/migrations/up`  | Apply all up database migrations         |
| `make tidy`              | Format code and tidy module dependencies |
| `make audit`             | Run quality control checks and tests     |

### Running the Application

To start the API server, use the `make` command:

```bash
make run
```

The server will be running on the port specified in your `.env` file (e.g., `http://localhost:4000`).

### Building for Production

To create a single, self-contained binary for deployment:

```bash
make build
```

This will create a binary named `goflix` in the project root. You can then run it directly:

```bash
./goflix
```

### Checking Application Metrics

The application exposes an endpoint with useful runtime metrics. You can view them by navigating to:

```
GET /debug/vars
```

To simulate load and see how the metrics change, you can use a tool like `hey`:

```bash
# Send 10,000 requests with a concurrency of 100.
hey -n 10000 -c 100 "http://localhost:4000/v1/healthcheck"
```

While that is running, check the `http://localhost:4000/debug/vars` endpoint in your browser to see metrics like the total number of active goroutines and requests.

## Authentication & Authorization

### Bearer Token Authentication

The API uses Bearer token authentication. Include the token in the Authorization header:

```
Authorization: Bearer <your-token>
```

### Permission System

The API implements a role-based permission system with the following permissions:

- `movies:read` - Required to view movies
- `movies:write` - Required to create, update, or delete movies

### User Account States

- **Anonymous Users**: Can register and request tokens
- **Authenticated Users**: Have valid tokens but may need activation
- **Activated Users**: Can access protected resources based on permissions

## Rate Limiting

The API implements IP-based rate limiting:

- Default: 2 requests per second with a burst of 4 requests
- Configurable via environment variables
- Automatic cleanup of inactive clients after 3 minutes

## Email Functionality

The application sends automated emails for:

- **Welcome emails** when users register
- **Account activation** tokens
- **Password reset** tokens

Email templates support both plain text and HTML formats and are embedded in the application binary.

## API Usage Examples

### User Registration Flow

1. **Register a new user:**

```bash
curl -X POST http://localhost:4000/v1/users \
  -H "Content-Type: application/json" \
  -d '{"name": "John Doe", "email": "john@example.com", "password": "password123"}'
```

2. **Activate account** (use token from welcome email):

```bash
curl -X PUT http://localhost:4000/v1/users/activated \
  -H "Content-Type: application/json" \
  -d '{"token": "activation-token-from-email"}'
```

3. **Get authentication token:**

```bash
curl -X POST http://localhost:4000/v1/tokens/authentication \
  -H "Content-Type: application/json" \
  -d '{"email": "john@example.com", "password": "password123"}'
```

4. **Access protected resources:**

```bash
curl -X GET http://localhost:4000/v1/movies \
  -H "Authorization: Bearer your-auth-token"
```

## Middleware Features

The API includes several middleware layers:

- **Panic Recovery**: Gracefully handles panics and returns 500 errors
- **Rate Limiting**: IP-based request limiting
- **CORS Support**: Configurable cross-origin resource sharing
- **Authentication**: Bearer token validation
- **Metrics Collection**: Request/response metrics via `/debug/vars`
- **Structured Logging**: JSON-formatted application logs
