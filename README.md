# GoFlix

A JSON API which is used to retrieve and manage information about movies using `Golang`.

# Routes to Implement

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
| POST   | /v1/tokens/password-reset | Generate a new password-reset token             |
| GET    | /debug/vars               | Display application metrics                     |

<!-- TO ADD -->
<!-- SQL Database Connection Creation -->
<!-- SQL Migration Setup -->
<!-- Page 129 -->

<!-- Add MailTrap Account Setup For SMTP Server setup>


<!-- Add hey command for metrics-->

<!--Explain the make file->
<!--How to run the migrations-->

<!--Add env variables configuration-->
