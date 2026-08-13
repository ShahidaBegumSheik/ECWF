# ECWF Module 1

## Architecture

Four FastAPI microservices use one shared MySQL database container:

| Service | Port | Responsibility |
|---|---:|---|
| Authentication Service | 8001 | Registration, OTP, login/logout, refresh, forgot/reset/change password, Email OTP MFA, Google OAuth,internal token validation |
| User Service | 8002 | User profile, account settings, individual dashboard |
| Tenant Admin Service | 8003 | Tenant settings, invitations, memberships |
| Notification Service | 8004 | SMTP email, OTP/invitation delivery, delivery history |
| MySQL | 3307 | One shared `ecwf_db` database |

## Environment files

The root `.env` contains common infrastructure/security values only. Each service has its own `.env` and `.env.example` for service-specific configuration.

## Start

Docker commands:

```powershell
docker compose down -v
docker compose build
docker compose up -d
docker compose ps
```

Swagger:

- Authentication: http://localhost:8001/docs
- User: http://localhost:8002/docs
- Tenant Admin: http://localhost:8003/docs
- Notification: http://localhost:8004/docs

For Swagger manual testing in User/Tenant Admin services, the protected APIs support both the normal `access_token` HttpOnly cookie and Swagger Bearer `Authorize`. The application uses HttpOnly cookies.

## Database

```powershell
docker compose exec db mysql -uecwf_user -pecwf_password ecwf_db
```

Each service has a separate Alembic version table while sharing `ecwf_db`:

- `alembic_version_auth`
- `alembic_version_user`
- `alembic_version_tenant`
- `alembic_version_notification`

## External authentication scope

This version supports **Google OAuth only** as the external identity provider.

## Execution Steps

### 1. Prerequisites

Make sure the following are installed and running:

- Docker Desktop
- Git
- Python 3.12 (if services are also run locally)
- VS Code or another code editor

### 2. Open the Project

Open PowerShell or the VS Code terminal and move to the project root directory:

```powershell
cd <path-to-ECWF-project>
```

The project root should contain `docker-compose.yml`.

### 3. Configure Environment Files

Create the required `.env` files from the corresponding `.env.example` files.

Provide the required values for the database connection, JWT secret, internal API key, SMTP/email configuration, Google OAuth configuration, and microservice URLs.

### 4. Build the Docker Images

```powershell
docker compose build
```

For a complete rebuild without cache:

```powershell
docker compose build --no-cache
```

### 5. Start the Application

```powershell
docker compose up -d
```

### 6. Check the Running Containers

```powershell
docker compose ps
```

Verify that the database and all required application services are running.

### 7. View Logs

All services:

```powershell
docker compose logs -f
```

Authentication Service:

```powershell
docker compose logs -f authentication-service
```

User Service:

```powershell
docker compose logs -f user-service
```

Tenant Admin Service:

```powershell
docker compose logs -f tenant-admin-service
```

Notification Service:

```powershell
docker compose logs -f notification-service
```

Press `Ctrl + C` to stop following the logs.

### 8. Run Alembic Migrations Manually

If migrations need to be executed manually:

```powershell
docker compose exec authentication-service alembic upgrade head
docker compose exec user-service alembic upgrade head
docker compose exec tenant-admin-service alembic upgrade head
docker compose exec notification-service alembic upgrade head
```

### 9. Open Swagger UI

Authentication Service:

```text
http://localhost:8001/docs
```

User Service:

```text
http://localhost:8002/docs
```

Tenant Admin Service:

```text
http://localhost:8003/docs
```

Notification Service:

```text
http://localhost:8004/docs
```

### 10. Access Protected APIs in Swagger

1. Login through the Authentication Service.
2. Obtain the access token for Swagger testing.
3. Open the Swagger UI of the required service.
4. Click **Authorize**.
5. Enter the access token in the Bearer authentication field.
6. Click **Authorize**.
7. Execute the protected endpoints.

For normal browser-based application access, authentication tokens are stored in HTTP-only cookies.

### 11. Access Service Containers

```powershell
docker compose exec authentication-service sh
docker compose exec user-service sh
docker compose exec tenant-admin-service sh
docker compose exec notification-service sh
```

Use `exit` to leave a container shell.

### 12. Stop the Application

```powershell
docker compose down
```

### 13. Start the Application Again

```powershell
docker compose up -d
```

### 14. Rebuild After Code or Dependency Changes

```powershell
docker compose down
docker compose build --no-cache
docker compose up -d
```

Verify:

```powershell
docker compose ps
```

Check logs if required:

```powershell
docker compose logs -f
```
