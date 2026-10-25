<div style="text-align: center; margin-bottom: 2rem; border: 1px solid #e1e4e6; padding: 1.5rem; border-radius: 8px; background: #ffffff;">
  <a href="https://zibo-informational-website.vercel.app/" target="_blank" title="Visit the live public informational website">
    <img src="../assets/brand/get.png" alt="Mbegu Link Logo" style="max-height: 396px; width: auto; cursor: pointer;" />
    <p style="margin-top: 0.5rem; font-size: 0.9rem; color: #004F2E; font-weight: bold; text-decoration: underline;">
      Click here to view the Public Informational Website
    </p>
  </a>
</div>

# Getting Started

This guide gets the full Mbegu Link system running locally: the backend API, the mobile app and the web PWA. 

Getting Started is the "how do I run this project on my machine". A new teammate should be able to clone, install, and run the backend + mobile + frontend by following it. Here it's how it can be done locally :

## Prerequisites

| Tool | Version | Used for |
|---|---|---|
| Python | 3.11+ | Backend API (FastAPI) |
| PostgreSQL | 14+ | Production database (SQLite works for dev) |
| DBeaver | any | Viewing the database |
| Flutter | 3.x | Mobile app |
| Node.js + pnpm | LTS | Frontend PWA |
| Git | any | Source control |

## 1. Get the code

The project has three repositories under the akirachix org:

```bash
git clone https://github.com
git clone https://github.com
git clone https://github.com
```

## 2. Run the backend

```bash
cd zibo-backend
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Create a `.env` file (see `.env.example` for structure):

```text
DATABASE_URL=sqlite:///./mbegu_dev.db
SECRET_KEY=change-me
```

Start the local service loop:

```bash
uvicorn main:app --reload
```

*   **API runs at:** `http://127.0.0.1:8000`
*   **Interactive Swagger UI documentation:** `http://127.0.0`

On startup, the application populates database rows and seeds core mock data directly so testing can occur without manually building structural profiles.

### Developer Authentication Mock Bypass
The current MVP utilizes explicit identity mock headers rather than validation login keys:
*   `X-User-Id`: A seeded user account UUID
*   `X-Org-Id`: The corresponding organization UUID

To retrieve matching testing parameters, execute a `GET /knowledge/entries` query inside Swagger or review local seed terminal execution printouts.

To view tables directly, map your favorite database driver tool (e.g., DBeaver) to the file matching your environmental string tracking.

## 3. Run the mobile app

```bash
cd zibo-mobile/mbegu_link
flutter pub get
flutter run
```

To configure endpoints correctly, adjust the target environmental host paths inside the main entry points (located within `lib/view_models/` — defaults point to `http://127.0.0.1:8000`). When deploying to physical Android terminals, swap out the localhost loopback with your local area network (LAN) host IP footprint.

To run verification validation scripts:
```bash
flutter test
```

## 4. Run the frontend PWA

```bash
cd zibo-frontend
pnpm install
pnpm dev
```

The PWA launches on the port configuration outputted by your local package execution console (typically defaults to `http://localhost:5173`). Confirm your backend container instance is up and running before connecting.

## 5. Verify everything works

*   **Backend Validation:** Open `http://127.0.0` and confirm the active route registry outputs cleanly.
*   **Transaction Verification:** Push a placeholder entry via `POST /knowledge/entries` utilizing test mock parameters to confirm a clean `201` execution return.
*   **Mobile Review:** Ensure your Flutter layout surfaces records reflecting your test payload.

### Project conventions
*   **Backend Architecture Layering:** Core modules organize across sequential folders: `routers/` → `services/` → `repositories/` → `models/` → `schemas/`.
*   **Syntax Code Formatting:** Clean tracking hooks using `ruff format . && ruff check . --fix` prior to version commits.
*   **Git Strategy Rules:** Branch names isolate down to standard development trackers (`feature/<name>`) and open pull requests against your develop line.

## Troubleshooting

| Problem | Fix |
|---|---|
| **422 Field required:** `X-User-Id` | Ensure you have included your active mock parameters inside the request headers profile. |
| **Mobile connection timeouts** | Swap out the default `127.0.0.1` routing with your machine's actual LAN IP configuration. |
| **No such table errors within DBeaver** | Confirm your management driver connects to the exact directory target mapped within your configuration. |
| **Data vanishes following a service reboot** | The development file builds completely fresh on restart parameters; recreate files within the active loop context. |
