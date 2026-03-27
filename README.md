# ProofPass

Privacy Preserving Digital Identity Verification.

## Overview

ProofPass is a privacy-preserving digital identity verification system built on zero-knowledge proofs (zk-SNARKs). It allows users to prove attributes about their identity (like age, citizenship) without revealing the underlying personal data.

## Project Structure

```
ProofPass/
├── frontend/              # Next.js 14 + React + Tailwind
├── backend/               # Node.js + Express + zkVerify
├── contracts/             # Solidity + Hardhat
├── zk-circuits/           # Circom circuits
├── scripts/               # Automation and deployment scripts
└── docs/                  # Architecture and deployment docs
```

## Quick Start

### Prerequisites
- Node.js 18+
- npm or yarn
- git

### 1. Install Dependencies

```bash
# Install all dependencies
cd contracts && npm install
cd ../backend && npm install
cd ../frontend && npm install
```

### 2. Set Up Environment Variables

```bash
# In frontend/.env
NEXT_PUBLIC_BACKEND_URL=http://localhost:3001
```

### 3. Run the Development Servers

```bash
# Terminal 1: Start the smart contract dev chain
cd contracts
npx hardhat node

# Terminal 2: Start the backend
cd backend
npm run dev

# Terminal 3: Start the frontend
cd frontend
npm run dev
```

### 4. Deploy Smart Contracts

```bash
cd contracts
npx hardhat run scripts/deploy.js --network localhost
```

### 5. Generate zk-SNARK Keys

```bash
cd zk-circuits
npx circom age_check.circom --r1cs --wasm
npx snarkjs powersoftau new bn128 12 pot12_0000.ptau
npx snarkjs powersoftau challenge contribute bn128 pot12_0000.ptau pot12_0001.ptau
npx snarkjs powersoftau prepare phase2 pot12_0001.ptau pot12_final.ptau
npx snarkjs zkey new age_check.r1cs pot12_final.ptau age_check_0000.zkey
npx snarkjs zkey contribute age_check_0000.zkey age_check_final.zkey
npx snarkjs zkey export verificationkey age_check_final.zkey verification_key.json
```

### 6. Generate a Proof

```bash
cd scripts
npm run generate-proof
```

### 7. Verify on Smart Contract

The frontend will automatically interact with the smart contract to verify the proof.

## Architecture

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│    Frontend     │────▶│     Backend     │────▶│ Smart Contract  │
│   (Next.js 14)  │     │  (Node/Express) │     │   (Solidity)    │
│                 │     │                 │     │                 │
│  - Identity Form│     │  - zkVerify SDK │     │  - Proof storage│
│  - Proof status │     │  - API routes   │     │  - Verification │
│  - User dashboard│    │  - Proof relay  │     │  - Access control│
└─────────────────┘     └─────────────────┘     └─────────────────┘
        │                        │                        │
        ▼                        ▼                        ▼
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  zk-SNARK Proof │     │  Circom Circuit │     │  Verification   │
│   Generation    │     │   (age_check)   │     │     Events      │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

## Technology Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Next.js 14, React 18, Tailwind CSS, ethers.js v6 |
| Backend | Node.js, Express, cors, zkVerify SDK |
| Smart Contracts | Solidity 0.8.x, Hardhat |
| zk-SNARKs | Circom, snarkjs |
| Testing | Mocha, Chai |

## API Endpoints

### Backend (Express)

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/identity/create` | Create identity record with proof |
| POST | `/api/identity/verify` | Verify proof against age threshold |
| GET | `/api/identity/:id` | Get identity details |
| DELETE | `/api/identity/:id` | Revoke identity |

### Smart Contract

| Function | Description |
|----------|-------------|
| `createIdentity(bytes32 proofHash, uint256 minAge)` | Store proof hash with age |
| `verifyAge(bytes32 proofHash, uint256 claimAge)` | Verify age >= threshold |
| `revokeIdentity(bytes32 proofHash)` | Revoke a proof |

## License

MIT License
