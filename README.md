# Kube Credential System

A microservice-based web application designed to issue and verify digital credentials. This project demonstrates containerization, scalability, and API-driven microservice communication using Docker and Docker Compose.

## 📋 Table of Contents

- [Overview](#overview)
- [System Architecture](#system-architecture)
- [Tech Stack](#tech-stack)
- [Design Decisions](#design-decisions)
- [Project Structure](#project-structure)
- [API Endpoints](#api-endpoints)
- [Setup & Installation](#setup--installation)
- [Configuration](#configuration)
- [Testing](#testing)
- [Assumptions & Notes](#assumptions--notes)
- [Author](#author)

## 🎯 Overview

**Kube Credential System** consists of two main components:

- **Backend Service**: Node.js + Express + TypeScript for issuing and verifying credentials
- **Frontend Client**: React + Vite + TypeScript + Tailwind CSS for user interaction

## 🏗️ System Architecture

```
┌─────────────────────────────┐
│        Frontend (React)      │
│  - Issuance & Verification   │
│  - API Calls via Axios       │
└────────────┬────────────────┘
             │
REST API (JSON over HTTP)
             │
┌────────────┴────────────────┐
│        Backend (Express)     │
│ - /issue  → Create Credential│
│ - /verify → Validate Credential│
│ - Persists data in db.json   │
│ - Returns Worker ID & Time   │
└────────────┬────────────────┘
             │
         (Docker)
             │
  Shared Network (kube-network)
```

## 🛠️ Tech Stack

| Component | Technology | Description |
|-----------|-----------|-------------|
| **Frontend** | React (Vite + TS + Tailwind) | UI to issue and verify credentials |
| **Backend** | Node.js + Express + TypeScript | Handles credential issuance and verification |
| **Storage** | JSON file (db.json) | Simple persistent data store for credentials |
| **Containerization** | Docker + Docker Compose | Isolated deployments for frontend & backend |

### Technologies Used

| Category | Tools / Libraries |
|----------|------------------|
| **Frontend** | React, Vite, Tailwind CSS, TypeScript |
| **Backend** | Node.js, Express, TypeScript |
| **Database** | JSON file (via fs module) |
| **Deployment** | Docker, Docker Compose |
| **Utility** | Axios, CORS, Body-Parser |

## 💡 Design Decisions

- **Microservice Separation**: Issuance and verification are handled via the same Express router for simplicity, but can easily be split into separate microservices or pods later.

- **Persistence**: A lightweight `db.json` file is used to persist credentials. This allows quick testing without requiring a DB instance.

- **Scalability Simulation**: Each backend instance can have a unique `WORKER_ID` (e.g., `worker-1`, `worker-2`) — mimicking Kubernetes pods handling different requests.

- **Frontend & Backend Decoupling**: Communication happens through REST APIs, making both services independently scalable and deployable.

- **Containerization**: Docker ensures consistent environment setup and enables cloud deployment (AWS ECS, Kubernetes, etc.).

- **TypeScript Safety**: Both backend and frontend use TypeScript to ensure type safety and clean code.

## 📁 Project Structure

```
KUBE-CREDENTIAL/
│
├── backend/
│   ├── Dockerfile
│   ├── package.json
│   ├── tsconfig.json
│   ├── src/
│   │   ├── db.json
│   │   ├── index.ts
│   │   ├── router.ts
│   │   └── tests/
│   │       └── routes.test.ts
│
├── frontend/
│   └── kube-credential-frontend/
│       ├── Dockerfile
│       ├── package.json
│       ├── vite.config.ts
│       └── src/
│           ├── api/
│           │   ├── issuance.ts
│           │   └── verification.ts
│           ├── pages/
│           │   ├── IssuancePage.tsx
│           │   └── VerificationPage.tsx
│           ├── App.tsx
│           ├── main.tsx
│           └── index.css
│
└── docker-compose.yml
```

## 🔌 API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/issue` | Issues a new credential if not already issued |
| POST | `/verify` | Verifies whether a credential exists |

### Example Requests & Responses

#### ✅ Issue Credential

**Request:**
```http
POST /issue
Content-Type: application/json

{
  "id": "cred-101"
}
```

**Response:**
```json
{
  "message": "Credential issued",
  "worker": "worker-1"
}
```

#### 🔍 Verify Credential

**Request:**
```http
POST /verify
Content-Type: application/json

{
  "id": "cred-101"
}
```

**Response:**
```json
{
  "valid": true,
  "worker": "worker-1",
  "issuedAt": "2025-10-07T09:42:10.001Z"
}
```

## 🚀 Setup & Installation

### Option 1: Run Locally (Without Docker)

#### 1. Clone Repository
```bash
git clone https://github.com/<your-repo>/kube-credential.git
cd kube-credential
```

#### 2. Start Backend
```bash
cd backend
npm install
npx ts-node src/index.ts
```
Server runs at: `http://localhost:3001`

#### 3. Start Frontend
```bash
cd ../frontend/kube-credential-frontend
npm install
npm run dev
```
Frontend runs at: `http://localhost:5174`

#### 4. Run Tests (Optional)
```bash
cd backend
npm test
```
This will run the test suite located in `src/tests/routes.test.ts` to verify the API endpoints.

### Option 2: 🐳 Run Using Docker Compose (Recommended)

#### 1. Build & Start Containers

From project root:
```bash
docker-compose up --build
```

#### 2. Access Services

- **Frontend**: `http://localhost:5174`
- **Backend**: `http://localhost:3001`

#### 3. Stop Containers
```bash
docker-compose down
```

## ⚙️ Configuration

### Running Locally

If you are running the application locally (without Docker), you need to update the API URLs in:

- `frontend/src/api/issuance.ts`
- `frontend/src/api/verification.ts`

Change from:
```typescript
const res = await axios.post('https://your-cloud-backend-url/issue', credential);
```

To:
```typescript
const res = await axios.post('http://localhost:3001/issue', credential);
```

## 🧪 Testing

The backend includes a comprehensive test suite to ensure API reliability.

### Running Tests

#### Run All Tests
```bash
cd backend
npm test
```

#### Test Coverage

The test suite (`src/tests/routes.test.ts`) covers:
- ✅ Credential issuance endpoint (`/issue`)
- ✅ Credential verification endpoint (`/verify`)
- ✅ Error handling and edge cases
- ✅ Worker ID assignment
- ✅ Timestamp validation

### Test Output Example
```
✓ POST /issue - should issue a new credential
✓ POST /issue - should not issue duplicate credentials
✓ POST /verify - should verify existing credentials
✓ POST /verify - should return invalid for non-existent credentials
```

## 📝 Assumptions & Notes

- Each credential must include a unique `id`.
- Backend uses `db.json` to simulate persistence (can be replaced with MongoDB).
- The app demonstrates the core Kubernetes concept of scaling pods — using the `WORKER_ID` to indicate which worker issued the credential.
- The test suite uses popular testing frameworks to ensure API reliability and correctness.
- K8s YAML manifests can be added in the `/k8s` folder for Kubernetes deployment.

## 👤 Author

**Jurair C**

- 📧 Email: jurair140@gmail.com
- 📞 Phone: +91-9656266809

---

Made with ❤️ by Jurair C
