# GoFlix API Testing Guide

This comprehensive guide provides detailed instructions for testing all GoFlix API endpoints using both Postman and curl commands. The guide follows a logical testing flow from user registration to movie management.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Base URL](#base-url)
- [Authentication Flow](#authentication-flow)
- [Endpoint Testing](#endpoint-testing)
  - [Health Check](#1-health-check)
  - [User Registration](#2-user-registration)
  - [User Activation](#3-user-activation)
  - [Authentication Token](#4-authentication-token)
  - [Movie Management](#5-movie-management)
  - [Token Management](#6-token-management)
  - [Application Metrics](#7-application-metrics)
- [Common Error Responses](#common-error-responses)
- [Testing Tips](#testing-tips)

## Prerequisites

- GoFlix API server running (default: `http://localhost:4000`)
- Postman installed (for GUI testing)
- curl command available (for command-line testing)
- Valid email account (for receiving activation/reset emails)

## Base URL

All API endpoints use the base URL: `http://localhost:4000`

## Authentication Flow

Most endpoints require authentication. Follow this flow:

1. Register a new user
2. Activate the user account (using token from email)
3. Get authentication token
4. Use Bearer token in Authorization header for protected endpoints

## Endpoint Testing

### 1. Health Check

**Purpose**: Check if the API server is running and get system information.

#### Postman

- **Method**: GET
- **URL**: `http://localhost:4000/v1/healthcheck`
- **Headers**: None required

#### curl

```bash
curl -X GET http://localhost:4000/v1/healthcheck
```

**Expected Response** (200 OK):

```json
{
  "status": "available",
  "system_info": {
    "environment": "development",
    "version": "1.0.0"
  }
}
```

---

### 2. User Registration

**Purpose**: Create a new user account. This will send a welcome email with an activation token.

#### Postman

- **Method**: POST
- **URL**: `http://localhost:4000/v1/users`
- **Headers**:
  - `Content-Type: application/json`
- **Body** (raw JSON):

```json
{
  "name": "John Doe",
  "email": "john.doe@example.com",
  "password": "securepassword123"
}
```

#### curl

```bash
curl -X POST http://localhost:4000/v1/users \
  -H "Content-Type: application/json" \
  -d '{
    "name": "John Doe",
    "email": "john.doe@example.com",
    "password": "securepassword123"
  }'
```

**Expected Response** (202 Accepted):

```json
{
  "user": {
    "id": 1,
    "created_at": "2024-01-15T10:30:00Z",
    "name": "John Doe",
    "email": "john.doe@example.com",
    "activated": false
  }
}
```

**Notes**:

- Password must be at least 8 characters
- Email must be unique
- User starts with `movies:read` permission
- Check your email for the activation token

---

### 3. User Activation

**Purpose**: Activate a user account using the token received via email.

#### Postman

- **Method**: PUT
- **URL**: `http://localhost:4000/v1/users/activated`
- **Headers**:
  - `Content-Type: application/json`
- **Body** (raw JSON):

```json
{
  "token": "ACTIVATION_TOKEN_FROM_EMAIL"
}
```

#### curl

```bash
curl -X PUT http://localhost:4000/v1/users/activated \
  -H "Content-Type: application/json" \
  -d '{
    "token": "ACTIVATION_TOKEN_FROM_EMAIL"
  }'
```

**Expected Response** (200 OK):

```json
{
  "user": {
    "id": 1,
    "created_at": "2024-01-15T10:30:00Z",
    "name": "John Doe",
    "email": "john.doe@example.com",
    "activated": true
  }
}
```

---

### 4. Authentication Token

**Purpose**: Get an authentication token for accessing protected endpoints.

#### Postman

- **Method**: POST
- **URL**: `http://localhost:4000/v1/tokens/authentication`
- **Headers**:
  - `Content-Type: application/json`
- **Body** (raw JSON):

```json
{
  "email": "john.doe@example.com",
  "password": "securepassword123"
}
```

#### curl

```bash
curl -X POST http://localhost:4000/v1/tokens/authentication \
  -H "Content-Type: application/json" \
  -d '{
    "email": "john.doe@example.com",
    "password": "securepassword123"
  }'
```

**Expected Response** (201 Created):

```json
{
  "authentication_token": {
    "token": "ABCDEF123456789...",
    "expiry": "2024-01-16T10:30:00Z"
  }
}
```

**Important**: Save the token value for use in subsequent requests!

---

### 5. Movie Management

All movie endpoints require authentication and appropriate permissions.

#### 5.1 List Movies

**Purpose**: Get a paginated list of movies with optional filtering.

#### Postman

- **Method**: GET
- **URL**: `http://localhost:4000/v1/movies`
- **Headers**:
  - `Authorization: Bearer YOUR_AUTH_TOKEN`
- **Query Parameters** (optional):
  - `title`: Filter by movie title
  - `genres`: Filter by genres (comma-separated)
  - `page`: Page number (default: 1)
  - `page_size`: Items per page (default: 20)
  - `sort`: Sort field (id, title, year, runtime, -id, -title, -year, -runtime)

#### curl

```bash
# Basic list
curl -X GET http://localhost:4000/v1/movies \
  -H "Authorization: Bearer YOUR_AUTH_TOKEN"

# With filters and pagination
curl -X GET "http://localhost:4000/v1/movies?title=batman&genres=action,drama&page=1&page_size=10&sort=-year" \
  -H "Authorization: Bearer YOUR_AUTH_TOKEN"
```

**Expected Response** (200 OK):

```json
{
  "movies": [
    {
      "id": 1,
      "title": "The Dark Knight",
      "year": 2008,
      "runtime": "152 mins",
      "genres": ["action", "drama"],
      "version": 1
    }
  ],
  "metadata": {
    "current_page": 1,
    "page_size": 20,
    "first_page": 1,
    "last_page": 1,
    "total_records": 1
  }
}
```

#### 5.2 Get Movie by ID

**Purpose**: Retrieve details of a specific movie.

#### Postman

- **Method**: GET
- **URL**: `http://localhost:4000/v1/movies/1`
- **Headers**:
  - `Authorization: Bearer YOUR_AUTH_TOKEN`

#### curl

```bash
curl -X GET http://localhost:4000/v1/movies/1 \
  -H "Authorization: Bearer YOUR_AUTH_TOKEN"
```

**Expected Response** (200 OK):

```json
{
  "movie": {
    "id": 1,
    "created_at": "2024-01-15T10:30:00Z",
    "title": "The Dark Knight",
    "year": 2008,
    "runtime": "152 mins",
    "genres": ["action", "drama"],
    "version": 1
  }
}
```

#### 5.3 Create Movie

**Purpose**: Add a new movie to the database.
**Required Permission**: `movies:write`

#### Postman

- **Method**: POST
- **URL**: `http://localhost:4000/v1/movies`
- **Headers**:
  - `Authorization: Bearer YOUR_AUTH_TOKEN`
  - `Content-Type: application/json`
- **Body** (raw JSON):

```json
{
  "title": "Inception",
  "year": 2010,
  "runtime": 148,
  "genres": ["sci-fi", "thriller"]
}
```

#### curl

```bash
curl -X POST http://localhost:4000/v1/movies \
  -H "Authorization: Bearer YOUR_AUTH_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Inception",
    "year": 2010,
    "runtime": 148,
    "genres": ["sci-fi", "thriller"]
  }'
```

**Expected Response** (201 Created):

```json
{
  "movie": {
    "id": 2,
    "created_at": "2024-01-15T11:00:00Z",
    "title": "Inception",
    "year": 2010,
    "runtime": "148 mins",
    "genres": ["sci-fi", "thriller"],
    "version": 1
  }
}
```

#### 5.4 Update Movie

**Purpose**: Update an existing movie's information.
**Required Permission**: `movies:write`

#### Postman

- **Method**: PATCH
- **URL**: `http://localhost:4000/v1/movies/2`
- **Headers**:
  - `Authorization: Bearer YOUR_AUTH_TOKEN`
  - `Content-Type: application/json`
- **Body** (raw JSON):

```json
{
  "title": "Inception (Director's Cut)",
  "runtime": 158
}
```

#### curl

```bash
curl -X PATCH http://localhost:4000/v1/movies/2 \
  -H "Authorization: Bearer YOUR_AUTH_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Inception (Directors Cut)",
    "runtime": 158
  }'
```

**Expected Response** (200 OK):

```json
{
  "movie": {
    "id": 2,
    "created_at": "2024-01-15T11:00:00Z",
    "title": "Inception (Director's Cut)",
    "year": 2010,
    "runtime": "158 mins",
    "genres": ["sci-fi", "thriller"],
    "version": 2
  }
}
```

#### 5.5 Delete Movie

**Purpose**: Remove a movie from the database.
**Required Permission**: `movies:write`

#### Postman

- **Method**: DELETE
- **URL**: `http://localhost:4000/v1/movies/2`
- **Headers**:
  - `Authorization: Bearer YOUR_AUTH_TOKEN`

#### curl

```bash
curl -X DELETE http://localhost:4000/v1/movies/2 \
  -H "Authorization: Bearer YOUR_AUTH_TOKEN"
```

**Expected Response** (200 OK):

```json
{
  "message": "movie successfully deleted"
}
```

---

### 6. Token Management

#### 6.1 Request Activation Token

**Purpose**: Request a new activation token if the original one expired.

#### Postman

- **Method**: POST
- **URL**: `http://localhost:4000/v1/tokens/activation`
- **Headers**:
  - `Content-Type: application/json`
- **Body** (raw JSON):

```json
{
  "email": "john.doe@example.com"
}
```

#### curl

```bash
curl -X POST http://localhost:4000/v1/tokens/activation \
  -H "Content-Type: application/json" \
  -d '{
    "email": "john.doe@example.com"
  }'
```

**Expected Response** (202 Accepted):

```json
{
  "message": "an email will be sent to you containing activation instructions"
}
```

#### 6.2 Request Password Reset Token

**Purpose**: Request a password reset token.

#### Postman

- **Method**: POST
- **URL**: `http://localhost:4000/v1/tokens/password-reset`
- **Headers**:
  - `Content-Type: application/json`
- **Body** (raw JSON):

```json
{
  "email": "john.doe@example.com"
}
```

#### curl

```bash
curl -X POST http://localhost:4000/v1/tokens/password-reset \
  -H "Content-Type: application/json" \
  -d '{
    "email": "john.doe@example.com"
  }'
```

**Expected Response** (202 Accepted):

```json
{
  "message": "an email will be sent to you containing password reset instructions"
}
```

#### 6.3 Reset Password

**Purpose**: Reset user password using the token received via email.

#### Postman

- **Method**: PUT
- **URL**: `http://localhost:4000/v1/users/password`
- **Headers**:
  - `Content-Type: application/json`
- **Body** (raw JSON):

```json
{
  "token": "PASSWORD_RESET_TOKEN_FROM_EMAIL",
  "password": "mynewsecurepassword123"
}
```

#### curl

```bash
curl -X PUT http://localhost:4000/v1/users/password \
  -H "Content-Type: application/json" \
  -d '{
    "token": "PASSWORD_RESET_TOKEN_FROM_EMAIL",
    "password": "mynewsecurepassword123"
  }'
```

**Expected Response** (200 OK):

```json
{
  "message": "your password was successfully reset"
}
```

---

### 7. Application Metrics

**Purpose**: View application runtime metrics and statistics.

#### Postman

- **Method**: GET
- **URL**: `http://localhost:4000/debug/vars`
- **Headers**: None required

#### curl

```bash
curl -X GET http://localhost:4000/debug/vars
```

**Expected Response** (200 OK):

```json
{
  "cmdline": ["./api"],
  "memstats": {
    "Alloc": 1234567,
    "TotalAlloc": 2345678,
    "Sys": 3456789,
    "NumGC": 10
  },
  "total_requests_received": 42,
  "total_responses_sent": 42,
  "total_processing_time_μs": 12345,
  "total_responses_sent_by_status": {
    "200": 35,
    "201": 3,
    "404": 2,
    "422": 2
  }
}
```

---

## Common Error Responses

### 400 Bad Request

```json
{
  "error": "invalid JSON format"
}
```

### 401 Unauthorized

```json
{
  "error": "invalid or missing authentication token"
}
```

### 403 Forbidden

```json
{
  "error": "your user account doesn't have the necessary permissions to access this resource"
}
```

### 404 Not Found

```json
{
  "error": "the requested resource could not be found"
}
```

### 409 Conflict

```json
{
  "error": "unable to update the record due to an edit conflict, please try again"
}
```

### 422 Unprocessable Entity

```json
{
  "error": {
    "email": "must be a valid email address",
    "password": "must be at least 8 bytes long"
  }
}
```

### 429 Too Many Requests

```json
{
  "error": "rate limit exceeded"
}
```

### 500 Internal Server Error

```json
{
  "error": "the server encountered a problem and could not process your request"
}
```

---

## Testing Tips

### Postman Collection Setup

1. Create a new Postman collection named "GoFlix API"
2. Set up environment variables:
   - `base_url`: `http://localhost:4000`
   - `auth_token`: (to be populated after authentication)
3. Use `{{base_url}}` and `{{auth_token}}` in your requests

### Environment Variables for Postman

```json
{
  "base_url": "http://localhost:4000",
  "auth_token": "",
  "user_email": "test@example.com",
  "user_password": "testpassword123"
}
```

### Testing Workflow

1. **Start with Health Check**: Ensure the API is running
2. **User Registration Flow**: Register → Activate → Authenticate
3. **Test Movie Operations**: Create → Read → Update → Delete
4. **Test Error Scenarios**: Invalid tokens, missing permissions, etc.
5. **Performance Testing**: Use metrics endpoint to monitor performance

### Rate Limiting Testing

The API implements rate limiting (default: 2 requests/second, burst of 4). To test:

```bash
# Send multiple rapid requests to trigger rate limiting
for i in {1..10}; do
  curl -X GET http://localhost:4000/v1/healthcheck &
done
wait
```

### Permission Testing

- New users get `movies:read` permission by default
- To test `movies:write` operations, you'll need to manually grant permissions in the database:

```sql
-- Grant movies:write permission to user ID 1
INSERT INTO users_permissions
SELECT 1, permissions.id FROM permissions WHERE permissions.code = 'movies:write';
```

### Email Testing

For development, consider using:

- [Mailtrap](https://mailtrap.io/) for email testing
- [MailHog](https://github.com/mailhog/MailHog) for local email testing
- Configure SMTP settings in your `.env` file accordingly

### Debugging Tips

1. **Check logs**: The API provides structured logging
2. **Use metrics endpoint**: Monitor request counts and response times
3. **Verify tokens**: Ensure authentication tokens haven't expired (24-hour lifetime)
4. **Check permissions**: Verify user has required permissions for protected endpoints
5. **Database state**: Check user activation status and permissions in the database

This testing guide should help you thoroughly test all aspects of the GoFlix API. Remember to test both success and failure scenarios to ensure robust API behavior.
