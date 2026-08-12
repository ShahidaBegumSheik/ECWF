# ECWF Module 1

## Architecture

Four FastAPI microservices use one shared MySQL database container:

| Service | Port | Responsibility |
|---|---:|---|
| Authentication Service | 8001 | Registration, OTP, login/logout, refresh, forgot/reset/change password, Email OTP MFA, Google OAuth, Google external identity linking, internal token validation |
| User Service | 8002 | User profile, account settings, individual dashboard, user-centric organization context |
| Tenant Admin Service | 8003 | Tenant setup, settings, multi-department hierarchy, invitations, memberships, RBAC, department/workspace scope, temporary grants, audit/compliance |
| Notification Service | 8004 | SMTP email, OTP/invitation delivery, development redirect, delivery history |
| API Gateway | 8000 | Nginx routing only; not a business microservice |
| MySQL | 3307 | One shared `ecwf_db` database |

## Workflows

- Individual registration uses a personal email, OTP verification and HttpOnly flow cookie.
- Organization registration uses a business email + organization name; successful OTP verification bootstraps the tenant and Tenant Admin membership through HTTPX.
- Access/refresh JWTs are stored only in HttpOnly cookies and are not returned in normal login JSON.
- Forgot-password OTP -> verified reset cookie -> reset-password flow is preserved.
- Registration/password-reset/MFA OTP values are kept in signed short-lived HttpOnly flow cookies, not in database OTP tables.
- Email OTP MFA is preserved, including setup, verification, resend, login challenge and disable.
- Google login/link/unlink/logout is preserved and moved into Authentication Service.
- Google OAuth state is single-use and stored in the shared DB (`oauth_states`) instead of requiring Redis.
- Tenant invitations require an already registered active verified individual user and authenticated acceptance/rejection.
- Tenant user management, department hierarchy, tenant isolation, RBAC, workspace-scoped permissions and temporary access grants are preserved.
- Audit is no longer a separate service; all services write the shared `audit_logs` table and Tenant Admin exposes audit list/export APIs.
- Notification remains a separate microservice.

## HTTPX AsyncClient communication

Services use one lifespan-scoped `httpx.AsyncClient` when another service's business capability is needed:

- Authentication -> User: profile bootstrap
- Authentication -> Tenant Admin: organization bootstrap / membership context
- Authentication -> Notification: OTP/welcome email
- User -> Authentication: token validation / full-name update / OAuth provider lookup / password change
- User -> Tenant Admin: organization membership context
- Tenant Admin -> Authentication: registered-user validation
- Tenant Admin -> Notification: invitation email

## Environment files

The root `.env` contains common infrastructure/security values only. Each service has its own `.env` and `.env.example` for service-specific configuration.

## Start

Used Docker volume for the first run:

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

For Swagger manual testing in User/Tenant Admin services, the protected APIs support both the normal `access_token` HttpOnly cookie and Swagger Bearer `Authorize`. The application itself should continue using HttpOnly cookies.

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

This version supports **Google OAuth only** as the external identity provider. Google login, callback, explicit link/unlink, duplicate-link protection, OAuth state validation and Google identity mapping supported.
