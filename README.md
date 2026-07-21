# OpsRoom

OpsRoom is a real-time MERN-stack event operations and coordination platform. It gives organizers, team leads, and volunteers one live command center for managing dynamic zones, teams, task dispatch, incidents, volunteer availability, and event activity.

It is designed for college fests, hackathons, conferences, sports events, weddings, and corporate events. OpsRoom is an operational coordination platform, not a ticketing or event-booking application.

## Contents

- [Features](#features)
- [Technology stack](#technology-stack)
- [Project structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Environment variables](#environment-variables)
- [First-time setup](#first-time-setup)
- [Running the application](#running-the-application)
- [Demo accounts](#demo-accounts)
- [User roles](#user-roles)
- [Real-time updates](#real-time-updates)
- [AI incident assistant](#ai-incident-assistant)
- [GitHub Codespaces](#github-codespaces)
- [Available commands](#available-commands)
- [Troubleshooting](#troubleshooting)
- [Production notes](#production-notes)

## Features

- JWT authentication with protected backend routes and persistent frontend login.
- Platform and event-level role-based access control.
- Event creation with public/private visibility, status, venue, dates, and join codes.
- Fully dynamic, event-specific zones with optional hierarchy, status, colour, capacity, teams, and managers.
- Event-specific team management with leads, members, and assigned zones.
- Live task workflow: open, assigned, accepted, in progress, blocked, completed, rejected, and cancelled.
- Safe task acceptance rules: only the assigned volunteer can accept an assigned task.
- Incident reporting with severity, category, zone, assigned team, responder, and status tracking.
- Volunteer presence and availability updates: Available, Busy, On Break, and Offline.
- Event schedules, announcements, notifications, audit logs, and analytics API endpoints.
- Socket.IO event rooms and browser notifications for confirmed operational changes.
- Optional LangGraph workflow to structure an unstructured incident report for human review.
- Seed script with a complete TechNova 2026 demonstration event.

## Technology stack

| Layer | Technology |
| --- | --- |
| Frontend | React, Vite, React Router, Axios |
| User interface | CSS dashboard design, Lucide React, React Hot Toast |
| Backend | Node.js, Express |
| Database | MongoDB Atlas, Mongoose |
| Real time | Socket.IO |
| Authentication | JSON Web Tokens, bcrypt |
| Validation | Zod |
| Security | Helmet, CORS, express-rate-limit |
| AI helper | LangGraph.js with optional Groq provider |

## Project structure

```text
opsroom/
├── .devcontainer/
│   └── devcontainer.json
├── client/
│   ├── src/
│   │   ├── api/
│   │   ├── context/
│   │   ├── pages/
│   │   ├── socket/
│   │   ├── App.jsx
│   │   ├── main.jsx
│   │   └── styles.css
│   ├── .env.example
│   └── package.json
├── server/
│   ├── src/
│   │   ├── ai/
│   │   │   └── incidentGraph.js
│   │   ├── config/
│   │   ├── middleware/
│   │   ├── models/
│   │   ├── routes/
│   │   ├── services/
│   │   ├── app.js
│   │   ├── seed.js
│   │   └── server.js
│   ├── .env.example
│   └── package.json
├── package.json
└── README.md
```

## Prerequisites

- Node.js **22 or later**
- npm
- A MongoDB Atlas database, or a reachable MongoDB deployment
- Git

Check your installed versions:

```bash
node -v
npm -v
```

## Installation

Clone the repository:

```bash
git clone https://github.com/YOUR_GITHUB_USERNAME/YOUR_REPOSITORY_NAME.git
cd YOUR_REPOSITORY_NAME
```

Install root, backend, and frontend dependencies:

```bash
npm install
npm run install:all
```

## Environment variables

Environment files are not committed. Create them from the example files:

```bash
cp server/.env.example server/.env
cp client/.env.example client/.env
```

On Windows PowerShell:

```powershell
Copy-Item server/.env.example server/.env
Copy-Item client/.env.example client/.env
```

### `server/.env`

```env
PORT=5000
MONGODB_URI=mongodb+srv://USERNAME:PASSWORD@YOUR_CLUSTER.mongodb.net/opsroom?retryWrites=true&w=majority
JWT_SECRET=replace_with_a_long_random_secret_at_least_32_chars
CLIENT_URL=http://localhost:5173
AI_PROVIDER=groq
AI_MODEL=llama-3.3-70b-versatile
AI_API_KEY=
```

`AI_API_KEY` is optional. When it is absent or the provider cannot be reached, the incident report remains available for manual creation and the workflow returns a safe local suggestion.

### `client/.env`

```env
VITE_API_URL=http://localhost:5000/api
VITE_SOCKET_URL=http://localhost:5000
```

Generate a secure JWT secret if needed:

```bash
node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"
```

Never commit `.env` files, MongoDB credentials, JWT secrets, or AI API keys.

## First-time setup

### 1. Configure MongoDB

Create a MongoDB Atlas cluster, create a database user, allow your Codespaces or development network to connect, and put the full connection string in `server/.env` as `MONGODB_URI`.

### 2. Add demo data

Run the seed command from the repository root:

```bash
npm run seed
```

This creates the TechNova 2026 demo event, zones, an Operations team, members, one task, and one incident.

### 3. Start the application

```bash
npm run dev
```

Open the frontend at:

```text
http://localhost:5173
```

The backend API runs at:

```text
http://localhost:5000/api
```

## Running the application

Start both applications from the project root:

```bash
npm run dev
```

Or run them separately:

```bash
cd server
npm run dev
```

```bash
cd client
npm run dev
```

## Demo accounts

After running `npm run seed`, use any of these accounts:

| Role | Email | Password |
| --- | --- | --- |
| Organizer | `organizer@opsroom.dev` | `Password123!` |
| Team lead | `lead@opsroom.dev` | `Password123!` |
| Volunteer | `volunteer@opsroom.dev` | `Password123!` |

Change or remove demo accounts before using the system with real event data.

## User roles

### Platform admin

- View users and events.
- Disable inappropriate accounts or events.
- Access platform-level statistics when implemented in the administrator workspace.

### Event organizer

- Create and manage events.
- Create zones, teams, member invitations, tasks, schedules, and announcements.
- Monitor live operations, incidents, analytics, and audit logs.

### Team lead

- View team tasks and incident information.
- Create and update operational work within the event scope.
- Coordinate volunteers assigned to the team.

### Volunteer

- Set availability status.
- View assigned tasks.
- Accept, start, block, or complete assigned tasks.
- Receive live task and notification updates.

### Attendee

- Intended for public event schedules, announcements, and issue reporting when those public views are enabled.

## Real-time updates

Socket.IO uses authenticated event rooms. The application broadcasts confirmed database changes, including:

- `event:status-updated`
- `zone:created`, `zone:updated`, `zone:disabled`
- `team:updated`
- `volunteer:availability-updated`
- `task:created`, `task:status-updated`, `task:blocked`, `task:completed`
- `incident:reported`, `incident:updated`, `incident:resolved`
- `notification:new`

REST API requests perform the mutation first. Socket.IO then broadcasts the confirmed result, avoiding duplicate business logic in browser socket handlers.

## AI incident assistant

The AI assistant is deliberately limited to incident structuring. It never creates, assigns, escalates, or closes incidents automatically.

The LangGraph workflow:

1. Receives raw incident text.
2. Loads the event's active zones and teams.
3. Calls Groq when configured, or uses a safe fallback suggestion.
4. Validates suggested zone and team names against actual event data.
5. Returns a suggestion for a human to review and approve manually.

## GitHub Codespaces

The repository includes `.devcontainer/devcontainer.json`, which installs Node.js 22 and forwards ports `5173` and `5000`.

After opening the repository in Codespaces:

```bash
npm install
npm run install:all
cp server/.env.example server/.env
cp client/.env.example client/.env
```

Set Codespaces URLs in your environment files.

### `server/.env`

```env
MONGODB_URI=your_mongodb_atlas_connection_string
JWT_SECRET=your_long_random_secret
CLIENT_URL=https://YOUR-CODESPACE-NAME-5173.app.github.dev
PORT=5000
```

### `client/.env`

```env
VITE_API_URL=https://YOUR-CODESPACE-NAME-5000.app.github.dev/api
VITE_SOCKET_URL=https://YOUR-CODESPACE-NAME-5000.app.github.dev
```

Start the project:

```bash
npm run seed
npm run dev
```

In the **Ports** tab, open port `5173`. Ensure port `5000` is browser-accessible if you use the separate Codespaces API URL.

## Available commands

| Command | Purpose |
| --- | --- |
| `npm install` | Install root development dependencies. |
| `npm run install:all` | Install backend and frontend dependencies. |
| `npm run dev` | Start Express and Vite together. |
| `npm run seed` | Add the demo users and TechNova 2026 sample event. |
| `npm run dev --prefix server` | Start only the backend development server. |
| `npm run dev --prefix client` | Start only the frontend development server. |
| `npm run build --prefix client` | Create a production frontend build. |

## Troubleshooting

### `Failed to fetch` or login does not work

1. Confirm the backend is running on port `5000`.
2. Confirm `VITE_API_URL` ends with `/api`.
3. Confirm `CLIENT_URL` is the frontend origin without a trailing slash.
4. Restart `npm run dev` after changing any environment variable.
5. In Codespaces, verify that port `5000` is accessible to the browser.

### MongoDB connection error

1. Check that `MONGODB_URI` is complete and contains the correct username and password.
2. In MongoDB Atlas, allow the network your Codespace uses.
3. Make sure the database user has read/write access.

### `401 Authentication required`

Sign in again. The frontend clears an expired or invalid JWT automatically. Check that the client is using the same backend URL that issued the token.

### Port is already in use

Stop the existing development process with `Ctrl + C`, then restart `npm run dev`.

## Production notes

- Use a unique, long `JWT_SECRET` for every deployed environment.
- Store secrets in the hosting provider's environment-variable manager, never in Git.
- Set `CLIENT_URL` to the exact deployed frontend origin.
- Restrict MongoDB Atlas network access and grant least-privilege database access.
- Use HTTPS for both frontend and backend deployments.
- Review rate limits, CORS origins, logging, monitoring, backup policy, and file storage before operating a public production event.
- Keep AI assistance review-only; organizers must approve every operational action.
