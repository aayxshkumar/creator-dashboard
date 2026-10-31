# Creator Dashboard

![CI](https://github.com/aayxshkumar/creator-dashboard/actions/workflows/ci.yml/badge.svg)

> Built as a take-home assignment for Vertx in May 2025.

A full-stack app where content creators register, earn credit points, and browse a content feed they can save, share or report. A Create React App client talks to an Express + MongoDB API that issues JWTs, tracks credits and activity, caches the feed in Redis and pushes notifications over Socket.io. The feed currently serves built-in sample Twitter and Reddit items rather than calling those APIs. No live demo.

## Project status

This is an unfinished take-home and does not build or start as committed:

- Client: `npm run build` fails because `src/pages/admin/AdminDashboard.js` imports `components/admin/UsersTable` and `components/admin/ReportedContentTable`, which were never committed. `react-toastify` is also imported but not listed in `client/package.json`.
- Server: `axios`, `bcryptjs`, `express-validator`, `redis` and `socket.io` are required by the code but not listed in `server/package.json`, so `npm start` fails until they are installed. `utils/cacheManager.js` uses the callback-style Redis v3 API.

CI installs both packages from their lockfiles and syntax-checks the server.

## Features

- Registration and login with JWT, and `user` / `admin` roles
- Credits for the daily login, completing a profile, and saving or sharing feed items
- Feed with save, share and report actions; saved-content page
- User dashboard with credits and activity
- Admin dashboard route with user, credit and interaction stats
- Feed responses cached in Redis for 10 minutes
- Notifications listed and marked read over REST, with a Socket.io connection for live updates

## Tech stack

- Client: React 19, React Router 7, Create React App (`react-scripts` 5), Tailwind CSS, Chart.js, Axios, socket.io-client
- Server: Node.js, Express 5, Mongoose 8, jsonwebtoken, bcrypt, Redis, Socket.io
- Deployment config: `server/Dockerfile` (Cloud Run style, port 8080) and `server/firebase.json` (Firebase Hosting)

## Getting started

Requires Node.js 20 or newer (CI runs on Node 22), MongoDB, and Redis.

```bash
git clone https://github.com/aayxshkumar/creator-dashboard.git
cd creator-dashboard
```

Server:

```bash
cd server
npm ci
cp .env.example .env   # then fill in the values
npm run dev            # nodemon server.js, http://localhost:5000
```

Client, in a second terminal:

```bash
cd client
npm ci
cp .env.example .env
npm start              # http://localhost:3000
```

See the status section above for what has to be fixed before these run.

### Environment variables

Server (`server/.env.example`):

| Name | Purpose |
| --- | --- |
| `PORT`, `NODE_ENV` | API port (default 5000) and mode |
| `MONGO_URI` | MongoDB connection string |
| `MONGO_URI_TEST` | MongoDB used by `__tests__/auth.test.js` |
| `JWT_SECRET`, `JWT_EXPIRE` | Token signing key and lifetime (default `30d`) |
| `REDIS_URL` | Redis for the feed cache (default `redis://localhost:6379`) |
| `CLIENT_URL` | Allowed Socket.io origin (default `http://localhost:3000`) |
| `TWITTER_*`, `REDDIT_*` | Read by `utils/apiClients.js`, which the feed does not call yet |

Client (`client/.env.example`):

| Name | Purpose |
| --- | --- |
| `REACT_APP_API_URL` | API base URL (default `http://localhost:5000/api`) |
| `REACT_APP_VAPID_PUBLIC_KEY` | Web push public key for the notification permission prompt |

## API

All routes are under `/api`. Everything except register, login and logout needs `Authorization: Bearer <token>`.

| Method | Path | Notes |
| --- | --- | --- |
| POST | `/auth/register` | |
| POST | `/auth/login` | |
| GET | `/auth/logout` | |
| GET | `/auth/me` | |
| PUT | `/auth/profile` | |
| GET | `/feed` | |
| POST | `/feed/save`, `/feed/share`, `/feed/report` | |
| GET | `/dashboard` | |
| GET | `/dashboard/saved` | |
| GET | `/dashboard/admin` | admin only |
| PUT | `/dashboard/admin/credits/:userId` | admin only |
| GET | `/notifications` | |
| PUT | `/notifications/read` | |

## Project structure

```
client/                React app (Create React App)
  src/pages/           Dashboard, Feed, Profile, SavedContent, admin/AdminDashboard
  src/components/      auth, feed, layout, notifications, routing, admin
  src/context/         AuthContext
server/                Express API
  routes/, controllers/, models/, middleware/
  utils/               Redis cache, Socket.io, notifications, Twitter/Reddit clients
  __tests__/, tests/   Jest + supertest tests (not wired to npm test)
API.md                 development notes
```
