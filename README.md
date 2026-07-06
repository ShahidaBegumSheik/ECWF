# Enterprise Collaboration Workflow Tool

An enterprise collaboration app built with FastAPI and MySQL. It supports role-based task management, workflow approvals, comments, documents, activity/audit logs, notifications, real-time updates, OAuth login, and payment plans..

## Tech Stack

Backend:
- FastAPI, Uvicorn, SQLAlchemy, Alembic, Pydantic v2
- MySQL
- JWT authentication with `python-jose`
- Password hashing with Passlib bcrypt
- Redis-backed optional caching
- SlowAPI rate limiting
- Authlib Google OAuth
- Razorpay/Stripe payment integrations
- FastAPI Pagination
- WebSockets for live notifications
