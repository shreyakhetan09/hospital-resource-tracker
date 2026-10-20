# Hospital Resource Tracker

A Flask web app that lets hospitals publish **live capacity** — available beds,
ICU beds, oxygen, and vaccine doses — so patients and families can find care
fast during a surge. Built during the COVID-19 second wave, when the hardest
part of getting treatment was simply *knowing where capacity existed*.

<p>
  <img alt="Python" src="https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white" />
  <img alt="Flask" src="https://img.shields.io/badge/Flask-000000?style=flat&logo=flask&logoColor=white" />
  <img alt="SQLAlchemy" src="https://img.shields.io/badge/SQLAlchemy-D71F00?style=flat&logo=sqlalchemy&logoColor=white" />
  <img alt="SQLite" src="https://img.shields.io/badge/SQLite-003B57?style=flat&logo=sqlite&logoColor=white" />
  <img alt="Bootstrap" src="https://img.shields.io/badge/Bootstrap-7952B3?style=flat&logo=bootstrap&logoColor=white" />
</p>

**Use case.** A city-wide capacity board for surge response: each hospital
publishes beds / ICU / oxygen / vaccines; patients see who can take them
*this hour*, not yesterday.

![Public availability board](https://user-images.githubusercontent.com/79561540/236925613-7d684a8d-6e6f-4a10-b0e8-951071850a47.png)

---

## Why it matters

During a health emergency, capacity changes by the hour. This app gives each
hospital a simple dashboard to keep its numbers current, and gives the public a
single paginated view of **who has what, right now** — no phone tree required.

## Features

- **Hospital accounts** — sign up and log in with passwords hashed via
  `Flask-Bcrypt` (never stored in plain text).
- **Live resource editing** — each hospital updates its beds, ICU beds, oxygen,
  and vaccine counts from one form; changes persist to the database instantly.
- **Public availability board** — a paginated directory (4 hospitals per page)
  anyone can browse to find open capacity.
- **Session-guarded pages** — the update dashboard is only reachable once a
  hospital is logged in.

## Architecture

```mermaid
flowchart LR
    V["Visitor / patient"] -->|browse| R["/resources (paginated)"]
    H["Hospital staff"] -->|sign up / log in| A["Auth<br/>Flask-Bcrypt + session"]
    A --> U["/update dashboard"]
    U -->|beds · ICU · oxygen · vaccine| M["Hospital model"]
    R --> M
    M <--> DB[("SQLite<br/>via SQLAlchemy")]
```

## How a hospital uses it

```mermaid
sequenceDiagram
    participant S as Hospital staff
    participant App as Flask app
    participant DB as SQLite
    S->>App: Sign up (name, email, contact, address, password)
    App->>DB: Store hospital (bcrypt-hashed password)
    S->>App: Log in
    App->>App: Open session for this hospital
    S->>App: Submit beds / ICU / oxygen / vaccine
    App->>DB: Update this hospital's record
    Note over App,DB: Public /resources page now shows the new numbers
```

## Tech stack

| Layer | Choice |
| ----- | ------ |
| Web framework | Flask |
| ORM / database | Flask-SQLAlchemy + SQLite |
| Auth | Flask-Bcrypt password hashing + Flask sessions |
| Templates | Jinja2 + Bootstrap |

## Data model

```
Hospital
├── name, email (unique), contact (unique), address
├── password        bcrypt hash
└── beds · icubed · oxygen · vaccine   (integers, default 0)
```

## Quickstart

```bash
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt

# the Flask app object lives in the Coronaweb package (__init__.py)
export FLASK_APP=Coronaweb
flask run
```

On first run the SQLite database (`site.db`) is created from the `Hospital`
model. Open the app, create a hospital account, then update your live resource
counts from the dashboard.

## Project structure

```
__init__.py            Flask app, SQLAlchemy + Bcrypt setup (Coronaweb package)
models.py              Hospital table definition
routes.py              home / login / sign-up / resources / update / logout
index1.html            landing page
login.html             hospital login
signup.html            hospital registration
hospitals.html         public, paginated availability board
resources_input.html   logged-in resource-update form
```

## Screenshots

![Landing page](https://user-images.githubusercontent.com/79561540/236925510-1b7415a5-97c9-421b-a459-b46381bcd03d.png)
![Hospital sign-up](https://user-images.githubusercontent.com/79561540/236925537-6ef6afd1-76cc-46e2-adbe-0fb8f201dbd7.png)
![Resource update dashboard](https://user-images.githubusercontent.com/79561540/236925574-71f2ef49-dccd-454b-858d-12b547196a8e.png)
![Paginated hospitals](https://user-images.githubusercontent.com/79561540/236925692-33b5d667-e592-43cb-bc1c-06e52983ea00.png)
