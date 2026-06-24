# 🖊️ CollabEditor — Real-Time Collaborative Code Editor

A full-stack web application where two or more users can write and edit code together in a shared editor. Every keystroke is instantly synced across all connected sessions using WebSockets — no refresh needed.

The project uses a **multi-stage Docker build** that compiles the React frontend into a `dist/` bundle and serves it as static files directly from the Node.js backend, producing a single container image.

---

## ✨ Features

- **Real-time sync** — Code changes broadcast instantly to all connected users via Socket.IO
- **Monaco Editor** — The same editor engine that powers VS Code, with syntax highlighting
- **Room-based sessions** — Users share a room ID to collaborate in the same editor instance
- **Single-container deployment** — Multi-stage Dockerfile bundles frontend + backend into one image
- **No database required** — All session state is held in-memory on the server

---

## 🛠️ Tech Stack

### Frontend
| Technology | Role |
|---|---|
| React | UI framework |
| Monaco Editor (`@monaco-editor/react`) | Code editing surface |
| Socket.IO Client | Real-time event communication |
| Vite | Dev server & production bundler |

### Backend
| Technology | Role |
|---|---|
| Node.js (v20) | Runtime |
| Express | HTTP server — serves the built React app from `public/` |
| Socket.IO | WebSocket server for broadcasting editor changes |

### DevOps
| Technology | Role |
|---|---|
| Docker (multi-stage) | Builds frontend, copies `dist/` into backend, runs as one container |
| `node:20-alpine` | Lightweight base image for both build stages |
| `.dockerignore` | Keeps `node_modules` and dev files out of the image |

---

## 📁 Project Structure

```
Docker-Aws/
├── Frontend/               # React + Monaco Editor (Vite)
│   ├── src/
│   └── package.json
├── backend/                # Node.js + Express + Socket.IO
│   ├── server.js
│   └── package.json
├── dockerfile              # Multi-stage build
├── .dockerignore
└── README.md
```

---

## 🐳 Dockerfile — How It Works

```dockerfile
# Stage 1: Build React app
FROM node:20-alpine AS frontend-builder
COPY ./Frontend /app
WORKDIR /app
RUN npm install
RUN npm run build          # outputs to /app/dist

# Stage 2: Run backend + serve frontend
FROM node:20-alpine
COPY ./backend /app
WORKDIR /app
RUN npm install
COPY --from=frontend-builder /app/dist /app/public   # frontend served as static files
CMD ["node", "server.js"]
```

The backend's Express server serves the compiled frontend from `public/` and the Socket.IO server handles all real-time collaboration on the same port.

---

## 🚀 Getting Started

### Prerequisites
- [Node.js](https://nodejs.org/) v18+ (for running locally without Docker)
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (for the containerized setup)

---

### Option 1 — Run Locally (No Docker)

**1. Clone the repo**
```bash
git clone https://github.com/JoyPatel13/Docker-Aws.git
cd Docker-Aws
```

**2. Start the backend**
```bash
cd backend
npm install
node server.js
# Runs on http://localhost:5000
```

**3. Start the frontend** (in a separate terminal)
```bash
cd Frontend
npm install
npm run dev
# Runs on http://localhost:5173
```

**4. Test collaboration** — open two browser tabs at `http://localhost:5173`, join the same room, and start typing.

---

### Option 2 — Run with Docker

**1. Build the image**
```bash
docker build -t collab-editor .
```

**2. Run the container**
```bash
docker run -p 5000:5000 collab-editor
```

**3. Open** `http://localhost:5000` in two browser tabs and start collaborating.

> **Windows users:** Make sure Docker Desktop is running and WSL2 is enabled before running the build command.

---

## ⚙️ How Real-Time Sync Works

```
User A types → Monaco onChange fires
                    ↓
         Socket.IO emits `code-change` to server
                    ↓
         Server broadcasts to all others in the room
                    ↓
         User B's Monaco editor receives `code-update`
         and applies the new value
```

All state is ephemeral — if the server restarts, the session resets. There is no persistence layer by design.

---

## 🔌 Socket.IO Events

| Event | Direction | Description |
|---|---|---|
| `join-room` | Client → Server | User joins a named room |
| `code-change` | Client → Server | Sends the latest editor content |
| `code-update` | Server → Client | Broadcasts content to all other users in the room |
| `disconnect` | Auto | Removes user from the room on tab close |

---
