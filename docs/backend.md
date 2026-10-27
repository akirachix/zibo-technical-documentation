# Mbegu Link Backend

## 1. Overview
The backend is the shared part of Mbegu Link. The mobile app and the PWA both send their requests to it. It checks who the user is, applies the business rules, saves data in PostgreSQL, and creates notifications.

The backend is written in Python with FastAPI. The database is PostgreSQL. The mobile app and the PWA work online only, so every action goes to the backend in real time.

## 2. Users and Roles
Three roles use the system:
* **Seed custodian:** a farmer who keeps seed. Uses the mobile app.
* **Community Seed Bank (CSB) Coordinator:** manages the seed bank. Uses the PWA.
* **GeRRI Seed Conservation Specialist:** works at the national genebank. Uses the PWA.

The database also has an admin role. The backend decides what each role can do. The apps do not make these decisions.

## 3. Modules
* **Seed registry and conservation records:** This is the core module. It keeps one record for each seed batch. The record holds local and scientific names, the Mbegu Link lot ID, source, quantity, storage information, germination history, regeneration history, and duplication status.
* **Seed movement, custody and support:** It handles transfers, loans, deposits, and storage requests. It also handles requests for conservation support from a CSB to GeRRI.
* **Knowledge and community sharing:** It stores practical seed conservation knowledge with text, photos, and videos. Each entry has its own visibility setting.
* **Conservation attention and coordination:** It shows records that need attention, such as overdue tests, falling germination, missing safety duplication, pending transfers, and open support requests.
* **Notifications:** This is a shared service used by all modules. The MVP sends in-app notifications only.

## 4. How a Request Is Handled
1. The user signs in through the mobile app or the PWA and gets access based on the role.
2. The client sends the request to the backend right away.
3. The backend checks that the user is allowed to do the action and that the input is valid.
4. The backend applies the business rules and saves the record.
5. The backend writes an audit record.
6. If needed, the backend creates a notification.

## 5. Business Rules
* Every germination test, regeneration, and safety duplication is saved as a new dated record. Old results are never overwritten.
* The germination percent is calculated from the seeds germinated and the seeds tested.
* A request has a type (loan, transfer, deposit, or support) and a status (pending, approved, declined, in progress, completed, or returned). Every status change must be linked to a user and recorded.
* When a transfer request is completed, the system creates a new seed record for the receiving organisation. It copies the seed details and sets the sending CSB as the source.
* A GeRRI specialist only sees seed records that are linked to requests sent to GeRRI.
* Knowledge entries are private by default. Public sharing needs a clear confirmation from the author. Publishing an entry does not expose a seed record.

## 6. On boarding the user in our system

### Authentication
* Custodians sign in with a phone number and an OTP, using Firebase Authentication.
* Coordinators, specialists, and admins sign in with an email and a password. Passwords are hashed with bcrypt.
* The backend checks the password length before hashing, because bcrypt ignores input after 72 bytes.

### Tokens
* Access tokens are short-lived JWTs that expire after five minutes.
* On the PWA, refresh tokens are stored in HttpOnly, Secure, SameSite Strict cookies.
* On mobile, refresh tokens are stored in the Android Keystore.
* The backend validates the JWT on every protected route.

### Authorization
* Access is controlled by role and is enforced on the backend.
* Only authorized CSB roles can approve and manage seed loans.
* Only authorized GeRRI roles can handle safety duplication and support responses.

### Data protection
* Traffic uses TLS 1.3, with HSTS turned on.
* Database storage is encrypted at rest with AES-256.
* Sensitive details such as names, GPS location, phone numbers, and emails are masked on public endpoints.
* Private seed entries are left out of public searches.

### Audit
* Status changes, seed movement, and security actions are written to an append-only audit trail. Normal users cannot edit or delete it.

## 7. Developer Getting Started Guide

Follow these steps to clone the repository and set up your local development environment.

### Step 1: Clone the Repository
Open your terminal and run the following commands to pull down the project:
```bash
git clone https://github.com
cd zibo_Backend
```

### Step 2: Set Up Your Virtual Environment
We use Python 3.11+ for this project. Create and activate a virtual environment:
```bash
# Create the environment
python3 -m venv venv
source venv/bin/activate

### Step 3: Install Dependencies
Install all required modules and development packages:
```bash
pip install --upgrade pip
pip install -r requirements.txt
```

### Step 4: Configure Environment Variables
Copy the example environment template and fill in your local secrets (PostgreSQL connection strings, JWT keys, Firebase configuration):
```bash
cp .env.example .env
```
*Open the `.env` file in your text editor and update the fields.*

### Step 5: Run Database Migrations
We use Alembic to manage database schemas. Upgrade your local database to the latest state:
```bash
alembic upgrade head
```

### Step 6: Start the Development Server
Launch the FastAPI development application:
```bash
uvicorn app.main:app --reload
```
You can now access the interactive Swagger API documentation at: `http://127.0.0`

```text
zibo_Backend/
├── app/
│   ├── core/           # System configuration, security, and errors
│   ├── models/         # SQLAlchemy database models (Schema layout)
│   ├── schemas/        # Pydantic validation rules (Data in/out)
│   ├── repositories/   # Direct SQL / Database querying logic
│   ├── services/       # Core business logic processing
│   ├── routers/        # HTTP API endpoints and route configurations
│   └── main.py         # App initialization & middleware binding
└── tests/              # Automated pytest files
```

### Directory Content Responsibility Matrix

#### `app/core/`
* **Purpose:** Holds global constants, configuration definitions, custom exception handlers, and security definitions.
* **What goes here:** JWT token generation, Firebase validator logic, password hashing helper utilities, and global dependency functions like `get_current_user`.

#### `app/models/`
* **Purpose:** Maps database tables directly to Python objects using SQLAlchemy.
* **What goes here:** Structural definition of tables (e.g., `SeedBatch`, `User`, `Request`, `AuditLog`, `Notification`). No business calculations happen here; it only defines fields, keys, and foreign relationships.

#### `app/schemas/`
* **Purpose:** Contains Pydantic models that manage data validation and serialization.
* **What goes here:** DTOs (Data Transfer Objects). For instance, `SeedCreateSchema` validates what a client sends when adding seeds, while `SeedResponseSchema` filters and masks sensitive data (like removing GPS details) before returning the object to the client.

#### `app/repositories/`
* **Purpose:** Handles all direct database access. 
* **What goes here:** Raw database operations. If you need to perform a `.filter()`, `.add()`, or a specialized SQL join to get seed records, write it inside a repository class. This isolates data layer access completely from the business logic.

#### `app/services/`
* **Purpose:** Orchestrates business processes and applies validation rules. This is the brain of the backend.
* **What goes here:** Multi-step business conditions. For example, the `TransferService` will coordinate: checking if the current user has the correct role, triggering a repository call to update the seed count, generating a new seed record for the destination bank, and building an automated tracking log for the append-only audit trail.

#### `app/routers/`
* **Purpose:** Declares the API routes, parameters, and binds HTTP actions (GET, POST, PUT, DELETE).
* **What goes here:** Route definitions. They intercept incoming requests, call a dependency to verify authorization tokens, call a designated *Service* to run the business logic, and hand the response output back to the client using a *Schema*. Routers must remain thin.

## 9. Testing
* Backend logic is tested with `pytest`.
* Postman is used to test the API workflows manually.
* Tests check that private records stay private.
* Tests check network loss during a write, and that a retry does not create a duplicate record.
* Tests check that two users editing the same record at the same time is reported and not silently overwritten.
* The clients are tested separately: Cypress and Playwright for the PWA, Appium for the mobile app.

### Core Core Architecture Test Suite (`tests/test_core_models.py`)
## 9. Testing
* Backend logic is tested with `pytest`.
* Postman is used to test the API workflows manually.
* Tests check that private records stay private.
* Tests check network loss during a write, and that a retry does not create a duplicate record.
* Tests check that two users editing the same record at the same time is reported and not silently overwritten.
* The clients are tested separately: Cypress and Playwright for the PWA, Appium for the mobile app.

### Core Core Architecture Test Suite (`tests/test_core_rules.py`)
Below is the example of pytest implementation verifying models for all entities we have, database layer enforcement, integrity constraints, and user roles:

```python
import uuid
from datetime import date, datetime, timedelta, timezone
from decimal import Decimal

import pytest
from sqlalchemy import create_engine
from sqlalchemy.exc import IntegrityError
from database import Base
from models.events import Event, EventType

pytestmark = pytest.mark.filterwarnings("ignore::sqlalchemy.exc.SAWarning")


def test_defaults_are_applied(db, seed_and_user):
    seed, user = seed_and_user
    db.commit()
    assert user.is_active is False
    assert user.otp_attempts == 0
    assert seed.quantity_unit == "kg"


def test_germination_percent_is_calculated(db, seed_and_user):
    seed, user = seed_and_user
    test_event = Event(
        seed_registry_id=seed.seed_registry_id,
        recorded_by=user.user_id,
        event_type=EventType.germination_test,
        event_date=date(2026, 1, 15),
        seeds_tested=100,
        seeds_germinated=82,
    )
    db.add(test_event)
    db.commit()
    assert test_event.germination_percent == Decimal("82.00")



def test_custodian_defaults(db, organisation):
    user = User(
        organisation_id=organisation.organisation_id,
        first_name="Mwende",
        last_name="Kioko",
        phone_number="+254700000001",
        role=UserRole.custodian,
    )
    db.add(user)
    db.commit()
    assert user.is_active is False
    assert user.otp_attempts == 0
    assert user.must_change_password is False
    assert user.otp_hash is None

def test_non_admin_requires_organisation(db):
    assert_rejected(
        db,
        User(
            first_name="No",
            last_name="Org",
            phone_number="+254700000002",
            role=UserRole.custodian,
        ),
    )


def test_custodian_requires_phone_number(db, organisation):
    assert_rejected(
        db,
        User(
            organisation_id=organisation.organisation_id,
            first_name="No",
            last_name="Phone",
            role=UserRole.custodian,
        ),
    )


@pytest.mark.parametrize("role", [UserRole.csb_coordinator, UserRole.gerri_specialist])
def test_staff_requires_temp_password_expiry(db, organisation, role):
    assert_rejected(
        db,
        User(
            organisation_id=organisation.organisation_id,
            first_name="No",
            role=role,
        ),
    )


def test_email_must_be_unique(db, organisation):
    db.add(
        User(
            organisation_id=organisation.organisation_id,
            first_name="First",
            last_name="Owner",
            email="same@csb.test",
            temp_password_expires_at=temp_expiry(),
        )
    )
    db.commit()
    assert_rejected(
        db,
        User(
            organisation_id=organisation.organisation_id,
            first_name="Second",
            last_name="Owner",
        ),
    )
```
## API Response Enforcements & HTTP Status Codes

To ensure consistent integration between the FastAPI backend, the mobile app, and the PWA client, all endpoint responses adhere to a strict set of standardized HTTP status codes.

### 1. Success Status Codes (2xx)
* **200 OK:** The request succeeded. The backend returns this for successful data lookups (`GET`), seed record updates (`PUT`), or deletions (`DELETE`).
* **201 Created:** The request succeeded and a new database resource was saved. The backend returns this when registering a new seed lot, logging a germination test, or creating a support request (`POST`).

### 2. Client Error Status Codes (4xx)
* **400 Bad Request:** A core business logic rule was violated. For example, if a coordinator attempts to save a germination test where `seeds_germinated` is greater than `seeds_tested`, the backend blocks the operation.
* **401 Unauthorized:** The user is not authenticated. This means the short-lived 5-minute JWT access token is missing, expired, or failed signature verification. The client must invoke the refresh token pipeline.
* **403 Forbidden:** The user is authenticated but lacks required role privileges. For example, if a Farmer Custodian attempts to hit a GeRRI specialist endpoint to process safety duplications, the backend drops the request.
* **404 Not Found:** The requested resource does not exist in the database. This occurs when looking up a non-existent Mbegu Link lot ID, a missing organization, or an invalid user account.
* **422 Unprocessable Entity:** This is FastAPI’s built-in validation error managed via Pydantic schemas. It means the inbound data structure was incorrect (e.g., passing text into an integer field or missing mandatory validation properties).

### 3. Server Error Status Codes (5xx)
* **500 Internal Server Error:** An unhandled exception occurred within the Python runtime code, or the PostgreSQL database connection dropped unexpectedly.

### Standard API Response Codes Reference

| HTTP Status | Title | Mbegu Link Context / Meaning | Client Action Required |
| :--- | :--- | :--- | :--- |
| **200** | OK | Data retrieved, updated, or deleted successfully. | Render or process the returned payload. |
| **201** | Created | Seed batch, event record, or request created successfully. | Redirect or update local tracking state. |
| **400** | Bad Request | Business rule broken (e.g., germinated seeds > tested seeds). | Display the business error message to the user. |
| **401** | Unauthorized | Token expired or invalid signature credentials. | Attempt token refresh via cookie / Android Keystore. |
| **403** | Forbidden | Role restriction (e.g., Custodian accessing GeRRI features). | Block access and display a permissions error. |
| **404** | Not Found | Seed lot code, organization ID, or user does not exist. | Route to a "Not Found" state or generic error. |
| **422** | Unprocessable Entity | Pydantic validation failure (bad data formatting or missing fields). | Correct schema types before resubmitting request. |
| **500** | Internal Server Error | Unhandled FastAPI exception or broken database state. | Alert developer team; prompt user to retry later. |


## 10. Not Part of the MVP
* Offline-first use
* WhatsApp integration or chatbot
* Audio knowledge sharing
* Automatic seed identification
* Map and GIS features
* Integration with GGCE or Genesys
* Automatic transfer of records into genebank systems
* Seed network coordinator interface

### Project Layer Summary Reference

| Layer / Step | Main Tool / Technology | Core Responsibility | Key Dependency Example |
| :--- | :--- | :--- | :--- |
| **Setup Pipeline** | Git, Venv, Pip, Alembic | Environment synchronization & versioning | Cloning repo -> installing requirements -> migrations |
| **Routers** | FastAPI Routing | Receives requests, maps URL endpoints | Dependent on Schemas & Services |
| **Schemas** | Pydantic v2 | Inbound validation, outbound parsing / masking | Sanitizing sensitive user context |
| **Services** | Native Python Logic | Enforcing multi-entity business rules & safety audits | Calculating germination percentages |
| **Repositories**| SQLAlchemy Async Session| Raw database communication and reads/writes | Execution of SELECT / INSERT queries |
| **Models** | SQLAlchemy ORM | Maps strict PostgreSQL schema data shapes | Defining primary keys and foreign constraints |
