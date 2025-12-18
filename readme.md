# NPChain

On-chain traits + verifiable memory for AI NPCs.

NPChain is a demo project that combines:

- **A 3D web UI** (Three.js + GLTF models) where you chat with different NPC characters.
- **An Express backend** that drives the NPC “brain” and orchestrates storage.
- **BNB Greenfield** for storing interaction/memory JSON objects.
- **BSC smart contracts** for storing/updating NPC traits and anchoring memory hashes on-chain.
---

## Table of Contents

- [Features](#features)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Quick Start (Local)](#quick-start-local)
- [Configuration (Environment Variables)](#configuration-environment-variables)
- [Using the App](#using-the-app)
- [Backend API](#backend-api)
- [Smart Contracts](#smart-contracts)
- [Troubleshooting](#troubleshooting)
- [Security Notes](#security-notes)
- [Contributing](#contributing)
- [License](#license)

---

## Features

- **Multi-NPC chat**
  - Switch between multiple NPCs with distinct prompts and personalities.
- **Trait synchronization**
  - NPC traits can be read from / updated to a BSC contract.
- **Memory persistence**
  - Interactions can be stored as objects in a Greenfield bucket.
- **Verifiable memory anchoring**
  - A SHA-256 hash of stored memory can be anchored on-chain.
- **Health & diagnostics endpoints**
  - Endpoints to test contract calls and Greenfield connectivity.

---

## Tech Stack

- **Frontend**
  - Vanilla HTML/CSS/JS
  - Three.js (GLTFLoader, OrbitControls)
- **Backend**
  - Node.js + Express
  - Web3.js
  - Groq (OpenAI-compatible chat completions)
- **Storage**
  - BNB Greenfield (object storage)
  - BSC (on-chain traits + memory hash anchoring)

---

## Architecture

```
Browser (index.html + script.js)
  |  POST /chat
  v
Express API (backend/index.js)
  |  calls LLM (Groq OpenAI-compatible endpoint)
  |  reads/writes NPC local memory JSON
  |  uploads objects to BNB Greenfield
  |  reads/writes traits + memory hashes on BSC
  v
Greenfield Bucket + BSC Contracts
```

---

## Project Structure

```
NPChain/
├── backend/                       # Express API + NPC logic
│   ├── abis/                      # ABI files used by backend
│   │   ├── NPCTraitsABI.json
│   │   └── NPCMemorySystem.json
│   ├── npc_memory/                # Local JSON snapshots per NPC
│   ├── index.js                   # Express app + endpoints
│   ├── llmclient.js               # Groq chat + classification helpers
│   ├── npc.js                     # NPC memory/traits logic
│   └── server.js                  # Minimal server entry
├── contracts/                     # Solidity contracts
│   ├── MemoryProof.sol
│   └── NPCTraitStorage.sol
├── build/contracts/               # Truffle build artifacts (ABI + bytecode)
│   ├── MemoryProof.json
│   └── NPCTraitStorage.json
├── migrations/                    # Truffle migration scripts
├── models/                        # 3D/AI related assets
├── index.html                     # Frontend UI
├── script.js                      # Frontend logic (Three.js + chat calls)
├── style.css                      # Frontend styling
└── truffle-config.js              # Truffle configuration (BSC testnet)
```

---

## Prerequisites

- **Node.js** 18+ recommended
- **npm** (bundled with Node)
- **BNB Chain accounts** for:
  - BSC transactions (gas)
  - Greenfield interactions (bucket access)
- Optional (for contracts): **Truffle CLI** (`truffle`) or use `npx truffle`

---

## Quick Start (Local)

### 1) Install backend dependencies

Run from `backend/`:

```bash
npm install
```

### 2) Configure environment

Create `backend/.env` (see [Configuration](#configuration-environment-variables)).

### 3) Start the backend

Run from `backend/`:

```bash
node index.js
```

Alternative:

```bash
node server.js
```

The server runs on `http://localhost:3000` by default.

### 4) Open the UI

You have two options:

- **Option A (recommended)**: open `http://localhost:3000/`
  - The backend serves the repository root as static assets.
- **Option B**: use a static server (e.g. VS Code Live Server)
  - The frontend calls `http://localhost:3000/chat`.
  - CORS is configured to allow common dev ports like `3000` and `5500`.

---

## Configuration (Environment Variables)

NPChain uses **two separate dotenv locations**:

- `backend/.env` for the Express backend.
- `.env` (repo root) for Truffle network deployment (uses `MNEMONIC`).

### `backend/.env`

Create a file at `backend/.env`:

```dotenv
# LLM (Groq - OpenAI compatible)
GROQ_API_KEY="YOUR_GROQ_KEY"

# BSC / Web3
BSC_RPC_URL="https://bsc-dataseed.bnbchain.org"  # or your preferred RPC
BSC_CONTRACT_ADDRESS="0xYOUR_CONTRACT_ADDRESS"
BSC_WALLET_PRIVATE_KEY="0xYOUR_PRIVATE_KEY"

# Greenfield
GREENFIELD_BUCKET="your-bucket-name"

# Server
PORT=3000
NODE_ENV=development
```

Notes:

- **`BSC_CONTRACT_ADDRESS`** should point to the deployed contract that matches the ABI used by the backend.
- **`GREENFIELD_BUCKET`** must already exist and be accessible by the configured account.

### Root `.env` (for Truffle)

Create (or edit) the repo root `.env`:

```dotenv
MNEMONIC="your twelve or twenty-four word seed phrase"
```

---

## Using the App

1. Start the backend.
2. Open the UI.
3. Select an NPC and send a message.

The frontend sends chat requests to:

- `POST http://localhost:3000/chat`

---

## Backend API

Base URL (local): `http://localhost:3000`

### `GET /`

Returns a small JSON payload with available endpoints.

### `POST /chat`

Chat with an NPC.

Request body:

```json
{
  "npcId": "Bionex",
  "messages": [
    { "role": "user", "content": "Hello" }
  ]
}
```

Response:

```json
{ "reply": "..." }
```

### `GET /traits/:npcId`

Fetch NPC traits from the configured BSC contract.

### `POST /storeMemory`

Stores a memory object to Greenfield and attempts to anchor a hash on BSC.

Request body:

```json
{
  "npcId": "Bionex",
  "memory": {
    "summary": "User asked about ancient technology",
    "tags": ["tech", "lore"]
  }
}
```

### `GET /fetchMemory/:npcId`

Fetches the latest stored memory JSON from Greenfield for the given NPC.

### `GET /test-greenfield`

Creates and uploads a small test object to the configured bucket, then downloads it to verify.

### `GET /test-contract/:npcId`

Attempts to call a traits method on the configured contract, returning the raw result.

### `GET /health`

Basic health check including Greenfield bucket validation.

---

## Scripts

### Backend (`backend/`)

- `npm test`
  - Runs Jest tests.

---

## Smart Contracts

This repository includes Solidity contracts under `contracts/` and compiled artifacts under `build/contracts/`.

### Contracts included

- **`NPCTraitStorage.sol`**
  - Stores a struct of NPC identity fields + mutable traits.
  - Exposes `createNPC(...)`, `updateTraits(...)`, `getTraits(...)`.
- **`MemoryProof.sol`**
  - Stores memory hash records per NPC.

### Deploying (BSC Testnet)

This repo includes a Truffle config at `truffle-config.js` with a `bsctestnet` network.

1) Install tooling (if needed)

```bash
npm install
```

If `npx truffle` is not available on your machine, install Truffle:

```bash
npm install -g truffle
```

2) Compile

```bash
npx truffle compile
```

3) Migrate/deploy

```bash
npx truffle migrate --network bsctestnet
```

After deployment:

- Copy the deployed contract address into `backend/.env` as `BSC_CONTRACT_ADDRESS`.

---

## Troubleshooting

- **Backend starts but UI can’t chat**
  - Confirm the UI is calling `http://localhost:3000/chat`.
  - Confirm the backend is running on `PORT=3000`.

- **`Loaded MNEMONIC: [MISSING or EMPTY]`**
  - Set `MNEMONIC` in the repo root `.env` before running Truffle.

- **Contract calls fail / “Contract does not exist at address”**
  - Verify `BSC_CONTRACT_ADDRESS` matches the deployed contract.
  - Ensure `BSC_RPC_URL` is correct.

- **Greenfield errors (bucket not found / permission denied)**
  - Ensure `GREENFIELD_BUCKET` exists.
  - Ensure the configured account has access to the bucket.

- **LLM errors**
  - Ensure `GROQ_API_KEY` is set.
