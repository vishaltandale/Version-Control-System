# GitHub Clone (ApnaGit)

A MERN-based GitHub replica with a custom version-control CLI (ApnaGit) implemented from scratch. Users can sign up, log in, browse repositories on a dashboard, and view a profile page with a contribution heat map. The backend also exposes REST APIs for users, repositories, and issues, plus CLI commands for init, add, commit, push, pull, and revert backed by local filesystem storage and AWS S3.

> **Repository origin:** [apna-college/Github](https://github.com/apna-college/Github.git)  
> **Author (backend package.json):** Vishal Tandale

---

## Features

- **User authentication** — Sign up and log in with email/password; JWT tokens issued on success
- **Repository dashboard** — View your repositories and browse all repositories with client-side search
- **User profile** — Display username, mock follower counts, and a contribution heat map
- **REST APIs** — CRUD endpoints for users, repositories, and issues
- **Custom Git CLI (ApnaGit)** — `init`, `add`, `commit`, `push`, `pull`, `revert` via `yargs` in the backend entry point
- **Real-time scaffolding** — Socket.IO server with room join support (not wired to the frontend)
- **GitHub-inspired UI** — Primer React components and dark-themed auth screens

---

## Screenshots

| Dashboard | Login |
|-----------|-------|
| ![Dashboard placeholder](./docs/screenshots/dashboard.png) | ![Login placeholder](./docs/screenshots/login.png) |

| Profile | Signup |
|---------|--------|
| ![Profile placeholder](./docs/screenshots/profile.png) | ![Signup placeholder](./docs/screenshots/signup.png) |

> Screenshot paths are placeholders. Capture running app screenshots and place them under `docs/screenshots/`.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     Frontend (React + Vite)                      │
│  Routes.jsx → Login / Signup / Dashboard / Profile              │
│  authContext.jsx → localStorage (token, userId)                 │
└────────────────────────────┬────────────────────────────────────┘
                             │ HTTP (axios / fetch)
                             │ localhost:3002 (hardcoded in frontend)
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                  Backend (Express + Node.js)                     │
│  index.js → yargs CLI + startServer()                           │
│  routes/ → user, repo, issue routers                            │
│  controllers/ → business logic                                  │
└──────────────┬──────────────────────────────┬───────────────────┘
               │                              │
               ▼                              ▼
┌──────────────────────────┐    ┌─────────────────────────────────┐
│ MongoDB                   │    │ Local .apnaGit + AWS S3          │
│ • Native driver (users)   │    │ init / add / commit / push / pull│
│ • Mongoose (repos/issues) │    │ revert                           │
└──────────────────────────┘    └─────────────────────────────────┘
```

---

## Tech Stack

| Layer | Technologies |
|-------|--------------|
| Frontend | React 18, Vite 5, React Router 6, Axios, Primer React, @uiw/react-heat-map |
| Backend | Node.js, Express 4, Mongoose 8, MongoDB native driver, bcryptjs, jsonwebtoken |
| CLI / VCS | yargs, uuid, fs (local `.apnaGit` directory), aws-sdk (S3) |
| Real-time | Socket.IO 4 (server only) |
| Tooling | ESLint, Vitest (declared, no test files present) |

---

## Folder Structure

```
Github/
├── README.md
├── backend-main/
│   ├── index.js              # CLI entry + Express server
│   ├── config/
│   │   └── aws-config.js     # AWS S3 client
│   ├── controllers/          # Route handlers + Git CLI logic
│   ├── models/               # Mongoose schemas (User, Repository, Issue)
│   ├── routes/               # Express routers
│   ├── commit.json           # Sample commit metadata
│   └── package.json
└── frontend-main/
    ├── index.html
    ├── src/
    │   ├── main.jsx          # App bootstrap
    │   ├── Routes.jsx        # Route definitions + auth guard
    │   ├── authContext.jsx   # Auth state provider
    │   ├── components/
    │   │   ├── auth/         # Login, Signup
    │   │   ├── dashboard/    # Dashboard
    │   │   └── user/         # Profile, HeatMap
    │   └── assets/
    └── package.json
```

---

## Installation

### Prerequisites

- Node.js (LTS recommended)
- MongoDB instance
- AWS account with S3 bucket (for CLI push/pull only)

### Backend

```bash
cd backend-main
npm install
```

### Frontend

```bash
cd frontend-main
npm install
```

---

## Environment Variables

Create a `.env` file in `backend-main/` (no `.env.example` is provided in the repository):

| Variable | Used in | Description |
|----------|---------|-------------|
| `MONGODB_URI` | `index.js`, `userController.js` | MongoDB connection string |
| `JWT_SECRET_KEY` | `userController.js` | Secret for signing JWT tokens |
| `PORT` | `index.js` | Server port (default: `3000`) |
| `S3_BUCKET` | `controllers/init.js` | Bucket name written to `.apnaGit/config.json` |

AWS SDK credentials (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, etc.) are required for S3 push/pull via the standard AWS credential chain. The file `config/aws-config.js` hardcodes `S3_BUCKET = "insert_bucket_name"` and region `ap-south-1`.

---

## Local Setup

1. Start MongoDB and set `MONGODB_URI` in `backend-main/.env`.
2. Set `JWT_SECRET_KEY` in the same `.env` file.
3. Start the backend on port **3002** (frontend hardcodes this port):

   ```bash
   cd backend-main
   # Default script uses PORT from env or 3000 — set PORT=3002 to match frontend
   set PORT=3002   # Windows CMD
   # $env:PORT=3002  # PowerShell
   npm start
   ```

4. Start the frontend dev server:

   ```bash
   cd frontend-main
   npm run dev
   ```

5. Open the Vite dev URL (typically `http://localhost:5173`).

---

## Production Build

### Frontend

```bash
cd frontend-main
npm run build    # Output: dist/
npm run preview  # Preview production build locally
```

### Backend

The backend `package.json` only defines `"start": "node index.js start"`. No separate production build step exists. Deploy as a Node.js process with environment variables configured.

---

## Deployment

The repository does **not** include Docker, CI/CD pipelines (`.github/workflows`), or deployment manifests. A typical deployment would:

1. Host MongoDB (Atlas or self-managed)
2. Deploy `backend-main` to a Node host (Render, Railway, EC2, etc.) with env vars set
3. Build `frontend-main` with `npm run build` and serve `dist/` via static hosting or reverse proxy
4. Update hardcoded `http://localhost:3002` URLs in frontend components to the production API URL

---

## API Documentation

Base URL (as used by frontend): `http://localhost:3002`

### Users

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/allUsers` | List all users |
| POST | `/signup` | Register (`username`, `email`, `password`) → `{ token, userId }` |
| POST | `/login` | Login (`email`, `password`) → `{ token, userId }` |
| GET | `/userProfile/:id` | Get user by ID |
| PUT | `/updateProfile/:id` | Update email/password |
| DELETE | `/deleteProfile/:id` | Delete user |

### Repositories

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/repo/create` | Create repository |
| GET | `/repo/all` | List all repositories (populated) |
| GET | `/repo/:id` | Get by ID |
| GET | `/repo/name/:name` | Get by name |
| GET | `/repo/user/:userID` | Get repositories for user |
| PUT | `/repo/update/:id` | Update content/description |
| PATCH | `/repo/toggle/:id` | Toggle visibility |
| DELETE | `/repo/delete/:id` | Delete repository |

### Issues

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/issue/create` | Create issue |
| GET | `/issue/all` | List issues |
| GET | `/issue/:id` | Get issue by ID |
| PUT | `/issue/update/:id` | Update issue |
| DELETE | `/issue/delete/:id` | Delete issue |

### Root

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/` | Returns `"Welcome!"` |

> **Note:** No authentication middleware protects these routes. JWT tokens are issued but not validated on API requests.

---

## Authentication

- **Signup/Login:** Passwords hashed with bcrypt (salt rounds: 10). JWT signed with `JWT_SECRET_KEY`, expiry `1h`.
- **Frontend:** Token and `userId` stored in `localStorage`. Route guard in `Routes.jsx` redirects unauthenticated users to `/auth`.
- **Gap:** API routes do not verify JWT. Frontend does not send `Authorization` headers on subsequent requests.

---

## Database

- **Database name (native driver):** `githubclone`, collection `users`
- **Mongoose models:** `User`, `Repository`, `Issue` in `backend-main/models/`
- User signup uses the **MongoDB native driver**; repository and issue operations use **Mongoose**. These are separate access patterns to the same MongoDB deployment.

### Schemas (Mongoose)

- **User:** username, email, password, repositories[], followedUsers[], starRepos[]
- **Repository:** name, description, content[], visibility, owner (ref User), issues[] (ref Issue)
- **Issue:** title, description, status (`open`/`closed`), repository (ref Repository)

---

## ApnaGit CLI

Run from `backend-main/` (or any directory when using Git commands):

```bash
node index.js init
node index.js add <file>
node index.js commit <message>
node index.js push
node index.js pull
node index.js revert <commitID>
node index.js start    # Start Express + Socket.IO server
```

Local state is stored in `.apnaGit/` (staging, commits, config.json).


---

## Future Improvements

- Add auth middleware and send JWT on all frontend API calls
- Unify MongoDB access through Mongoose or a single data layer
- Environment-based API base URL (e.g. `VITE_API_URL`)
- Implement missing routes: create repository, starred repositories
- Wire Socket.IO to frontend for real-time updates
- Add repository CRUD UI and issue management UI
- Fix `issueController` async bugs and signup `insertedId`
- Add tests, `.env.example`, Docker, and CI/CD
- Replace mock heat map and follower data with real commit/activity data
- Add `vite.config.js` and centralize frontend configuration

---

## License

ISC (as declared in `backend-main/package.json`). Frontend `package.json` does not declare a license.

---

## Author

**Vishal Tandale** — as credited in `backend-main/package.json`.
