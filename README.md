# CafeSync

[![Test and Deploy](https://github.com/kutaysahk/cafesync/actions/workflows/test-and-deploy.yml/badge.svg)](https://github.com/kutaysahk/cafesync/actions/workflows/test-and-deploy.yml)
![Coverage](https://img.shields.io/badge/coverage-92.3%25-brightgreen)
![Python](https://img.shields.io/badge/python-3.10-blue)
![FastAPI](https://img.shields.io/badge/FastAPI-async-009688)

A small but production-leaning cafe ordering system with four roles
(admin, barista, user, viewer), four authentication methods, full RBAC,
and end-to-end browser tests.
https://cafesync-jke6.onrender.com

## Live demo

https://cafesync-jke6.onrender.com — create an account to see.
the operations dashboard, or sign up as a fresh user to order from the menu.

## What's in the box

### Authentication
- **Password** — bcrypt-hashed, with timing-safe checks
- **TOTP** — Google Authenticator compatible (RFC 6238)
- **Backup codes** — 10 per user, single-use, regeneratable
- **Passkeys** — WebAuthn / FIDO2 via the `webauthn` library

### Authorization
- Four roles: `admin`, `barista`, `user`, `viewer`
- Server-side checks on every endpoint (UI hiding is not security)
- Admin dashboard for dynamic role changes
- Last-admin and self-modification safeguards

### Hardening
See [SECURITY.md](SECURITY.md) for the full threat model.
- Rate limits on auth routes (5/min login, 3/min signup)
- CSRF tokens on every mutating request
- CSP with per-request nonces
- Strict cookies (httponly + secure + samesite in production)
- HSTS, X-Frame-Options, X-Content-Type-Options, Referrer-Policy
- Generic 500 responses (no stack-trace info leakage)

### Tests
- **143 unit tests, 92.3% coverage** (`pytest --cov`)
- **Playwright end-to-end** that walks through every feature in one
  sequential story (signup → order → admin promote → barista serve →
  viewer hidden controls → 2FA setup → backup-code login → reuse rejected)

### CI/CD
- GitHub Actions runs tests on every push
- Render auto-deploys ONLY after tests pass

## Local development

```bash
# install deps
pip install -r requirements-dev.txt
playwright install chromium

# .env (see .env.example or below)
cp .env.example .env

# start the server
uvicorn main:app --reload
```

Required `.env` variables:
```
SESSION_SECRET=<random 32+ char string>
ADMIN_USERNAME=********
ADMIN_PASSWORD=********
```

## Running tests

```bash
pytest --cov                # unit + coverage
pytest tests/e2e/ --headed  # end-to-end, watch the browser
```

For Playwright tests, uvicorn must be running in another terminal.

## Architecture

```
main.py              FastAPI app + middleware (auth gate, telemetry, security)
security.py          Rate limiter, CSRF, security headers, CSP nonces
auth_utils.py        Password hashing, TOTP, backup codes
models.py            SQLAlchemy models
roles.py             The 4-role definitions
routers/
  auth.py            Login, signup, logout
  twofa.py           TOTP setup + challenge flow
  passkeys.py        WebAuthn register + authenticate
  orders.py          POST orders, GET queue, PUT complete
  users.py           Admin user management
  telemetry.py       Metrics + recent logs
templates/           Jinja2 HTML
static/              CSS, JS, menu data
tests/
  unit/              79 fast tests, no browser
  e2e/               1 sequential Playwright journey
```

## Deployment

Deployed to Render via the `render.yaml` Blueprint. Deploys are triggered
only by the GitHub Actions workflow after tests pass (Render's
auto-deploy is intentionally disabled).
