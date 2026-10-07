# VulnBank — Intentionally Vulnerable Web App (Training Lab)

**FOR AUTHORIZED CLASSROOM/LAB USE ONLY.** Do not deploy this on a public
network, shared hosting, or anywhere reachable from the internet. It contains
deliberate, serious security flaws (including remote command execution) by design.

## What this is

A small banking-style web app (login, user profiles, search, admin panel,
file download, network tool) built specifically to be penetration tested.
It is not based on DVWA or Juice Shop — different bugs, different code, so
students can't just reuse cheat sheets from those.

## Quick start (Docker — recommended)

Each student/pair should run their own instance so nobody can interfere with
another student's session or crash a shared target.

```bash
cd vulnapp
docker compose up --build
```

The app will be available at `http://localhost:5000`.

To stop: `docker compose down` (this also wipes the container's DB state —
run `docker compose up --build` again for a clean environment).

## Quick start (without Docker)

Requires Python 3.10+.

```bash
cd vulnapp
pip install -r requirements.txt
python init_db.py      # creates vulnbank.db with seed users
python app.py           # runs on http://0.0.0.0:5000
```

Note: the command-injection vulnerability specifically calls the system
`ping` binary. On Linux, install it if missing: `sudo apt install iputils-ping`.
Without it, that one finding will error out gracefully but the rest of the
app is unaffected.

## Seed accounts

| Username | Password         | Role  |
|----------|------------------|-------|
| alice    | Password123!     | user  |
| bob      | letmein1         | user  |
| carol    | sunshine22       | user  |
| admin    | SupErSecret!2024 | admin |

Students can also register their own accounts via `/register`.

## Scope for students

Test only this application, on the host/port it's running on. In-scope
features: login, registration, password reset, search, user profiles,
file download, admin panel, admin network tool.

Out of scope: the underlying host OS beyond what the app itself exposes,
denial-of-service attacks against the server, and anything outside this
application entirely.

## Deliverable

A penetration test report covering each vulnerability found: title,
severity, affected endpoint, steps to reproduce, evidence, and a
remediation recommendation. Ask your instructor for the report template.
