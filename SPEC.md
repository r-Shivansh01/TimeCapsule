# TimeCapsule — Complete Project Specification

> **For Agentic AI (Claude Code or equivalent):**
> Read this entire file before writing a single line of code.
> Follow every module in order. Do not skip setup steps.
> All tools used must be free-tier compatible.
> After each module, run the relevant tests and confirm they pass before proceeding.

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Tech Stack](#2-tech-stack)
3. [Repository & Directory Structure](#3-repository--directory-structure)
4. [Environment Variables](#4-environment-variables)
5. [Module 0 — Pre-Flight Setup](#5-module-0--pre-flight-setup)
6. [Module 1 — Core Smart Contract](#6-module-1--core-smart-contract)
7. [Module 2 — NFT Extension](#7-module-2--nft-extension)
8. [Module 3 — Encryption & IPFS Layer](#8-module-3--encryption--ipfs-layer)
9. [Module 4 — Frontend](#9-module-4--frontend)
10. [Module 5 — Deployment Scripts](#10-module-5--deployment-scripts)
11. [Module 6 — CI/CD & GitHub Actions](#11-module-6--cicd--github-actions)
12. [Module 7 — End-to-End Testing](#12-module-7--end-to-end-testing)
13. [Coding Standards & Conventions](#13-coding-standards--conventions)
14. [Known Constraints](#14-known-constraints)
15. [Final Checklist](#15-final-checklist)

---

## 1. Project Overview

### What is TimeCapsule?

**TimeCapsule** is a decentralized application (dApp) on the Ethereum blockchain that allows users to lock encrypted messages, files, or memories into a "time capsule" smart contract. The capsule cannot be opened until a user-specified future date. Each capsule is minted as an ERC-721 NFT, making it a unique, transferable digital artifact.

### Core User Flow

```
User visits dApp
  → Connects MetaMask wallet
  → Writes a message & sets an unlock date + recipient address
  → Message is encrypted CLIENT-SIDE using AES-GCM (Web Crypto API)
  → Encrypted blob is uploaded to IPFS (via Pinata free tier)
  → IPFS content hash (CID) + unlock timestamp + recipient are sent to smart contract
  → Smart contract stores the capsule, mints an NFT to the creator
  → On/after unlock date, recipient connects wallet and calls openCapsule()
  → Frontend fetches encrypted blob from IPFS, decrypts using stored key
  → Decrypted message is displayed to recipient
```

### Key Features

| Feature | Description |
|---|---|
| Time-locked messages | Smart contract enforces unlock date; no one can read before it |
| Client-side encryption | Message never travels the network unencrypted |
| IPFS storage | Encrypted content stored on decentralized storage, not on-chain |
| NFT capsules | Each capsule is ERC-721; can be viewed in wallets like MetaMask/OpenSea |
| Recipient system | Creator specifies who can open the capsule |
| Dashboard | View all capsules created by or addressed to connected wallet |
| Free deployment | Sepolia testnet + Vercel + Pinata free tier + GitHub Actions free tier |

---

## 2. Tech Stack

### Smart Contracts
| Tool | Version | Purpose |
|---|---|---|
| Solidity | ^0.8.20 | Contract language |
| Hardhat | ^2.22.x | Dev framework, local node, testing |
| OpenZeppelin Contracts | ^5.x | ERC-721, Ownable, ReentrancyGuard |
| Ethers.js | v6.x | JS ↔ contract interaction |
| hardhat-gas-reporter | latest | Gas usage reports during tests |
| solidity-coverage | latest | Code coverage |
| dotenv | latest | Environment variable management |

### Frontend
| Tool | Purpose |
|---|---|
| Vanilla HTML5 + CSS3 + ES6 JS | No frameworks; single self-contained `index.html` + linked JS/CSS files |
| MetaMask (injected `window.ethereum`) | Wallet connection |
| Web Crypto API (browser-native) | AES-GCM client-side encryption/decryption |
| Pinata SDK / REST API | IPFS upload/fetch (free 1GB tier) |
| Ethers.js v6 (CDN) | Contract calls from browser |

### Infrastructure
| Tool | Purpose |
|---|---|
| Sepolia Testnet | Ethereum test network for deployment |
| Alchemy or Infura (free tier) | RPC provider for Sepolia |
| Vercel | Frontend hosting (free tier, no build step needed for vanilla JS) |
| GitHub Actions | CI — run tests on every push/PR |
| CodeRabbit | AI PR review (configured via `.coderabbit.yaml`) |
| Pinata | IPFS pinning service (free tier) |

### Dev Tools
| Tool | Purpose |
|---|---|
| ESLint | JS linting |
| Prettier | Code formatting |
| Solhint | Solidity linting |
| Husky + lint-staged | Pre-commit hooks |

---

## 3. Repository & Directory Structure

Create exactly this structure. Do not deviate:

```
timecapsule/
├── .github/
│   ├── workflows/
│   │   └── ci.yml                   # GitHub Actions CI pipeline
│   └── PULL_REQUEST_TEMPLATE.md
├── .husky/
│   └── pre-commit
├── contracts/
│   ├── TimeCapsule.sol              # Core contract (Module 1)
│   └── TimeCapsuleNFT.sol           # NFT extension (Module 2)
├── scripts/
│   ├── deploy.js                    # Deploy both contracts to network
│   └── seed.js                      # Seed a few test capsules (optional)
├── test/
│   ├── TimeCapsule.test.js          # Unit tests for core contract
│   └── TimeCapsuleNFT.test.js       # Unit tests for NFT contract
├── frontend/
│   ├── index.html                   # Main HTML shell
│   ├── css/
│   │   └── styles.css
│   ├── js/
│   │   ├── config.js                # Contract addresses & ABIs (auto-populated post-deploy)
│   │   ├── wallet.js                # MetaMask connection logic
│   │   ├── encrypt.js               # AES-GCM encryption/decryption
│   │   ├── ipfs.js                  # Pinata upload/download
│   │   ├── contract.js              # All ethers.js contract calls
│   │   ├── ui.js                    # UI state management, rendering
│   │   └── app.js                   # Main entry point, event wiring
│   └── assets/
│       └── logo.svg
├── deployments/
│   └── sepolia.json                 # Auto-generated after deploy; stores contract addresses
├── coverage/                        # Auto-generated by solidity-coverage
├── artifacts/                       # Auto-generated by Hardhat
├── cache/                           # Auto-generated by Hardhat
├── .env                             # Secret keys (NEVER commit)
├── .env.example                     # Template (commit this)
├── .eslintrc.json
├── .prettierrc
├── .solhint.json
├── .gitignore
├── .coderabbit.yaml
├── hardhat.config.js
├── package.json
├── vercel.json
└── README.md
```

---

## 4. Environment Variables

### `.env.example` (commit this file)

```
# Wallet private key (used only for deployment scripts — never exposed to frontend)
PRIVATE_KEY=your_wallet_private_key_here

# Sepolia RPC endpoint (get free from https://alchemy.com or https://infura.io)
SEPOLIA_RPC_URL=https://eth-sepolia.g.alchemy.com/v2/YOUR_KEY

# Etherscan API key for contract verification (free at https://etherscan.io)
ETHERSCAN_API_KEY=your_etherscan_api_key

# Pinata API keys (free at https://pinata.cloud)
PINATA_API_KEY=your_pinata_api_key
PINATA_SECRET_API_KEY=your_pinata_secret_key
PINATA_JWT=your_pinata_jwt_token

# Deployed contract addresses (populated automatically by deploy script)
TIMECAPSULE_ADDRESS=
TIMECAPSULE_NFT_ADDRESS=
```

### `.env` (never commit)

Populate `.env` with real values. The `.gitignore` must exclude `.env`.

---

## 5. Module 0 — Pre-Flight Setup

> **Goal:** Initialize the repository, install all dependencies, configure all tooling. After this module, `npx hardhat compile` must succeed and `npx hardhat test` must run (even with 0 tests).

### Step 1: Initialize Node Project

```bash
mkdir timecapsule && cd timecapsule
git init
npm init -y
```

### Step 2: Install All Dependencies

```bash
# Hardhat and core plugins
npm install --save-dev hardhat @nomicfoundation/hardhat-toolbox dotenv

# OpenZeppelin contracts
npm install @openzeppelin/contracts

# Linting and formatting
npm install --save-dev eslint prettier solhint eslint-config-prettier

# Pre-commit hooks
npm install --save-dev husky lint-staged
npx husky init
```

### Step 3: Initialize Hardhat

```bash
npx hardhat init
# Choose: "Create a JavaScript project"
# Accept all defaults
```

### Step 4: `hardhat.config.js`

Replace the generated `hardhat.config.js` with exactly this:

```javascript
require("@nomicfoundation/hardhat-toolbox");
require("dotenv").config();

const PRIVATE_KEY = process.env.PRIVATE_KEY || "0x" + "0".repeat(64);
const SEPOLIA_RPC_URL = process.env.SEPOLIA_RPC_URL || "https://rpc.ankr.com/eth_sepolia";

/** @type import('hardhat/config').HardhatUserConfig */
module.exports = {
  solidity: {
    version: "0.8.20",
    settings: {
      optimizer: {
        enabled: true,
        runs: 200,
      },
    },
  },
  networks: {
    hardhat: {
      chainId: 31337,
    },
    localhost: {
      url: "http://127.0.0.1:8545",
      chainId: 31337,
    },
    sepolia: {
      url: SEPOLIA_RPC_URL,
      accounts: [PRIVATE_KEY],
      chainId: 11155111,
    },
  },
  etherscan: {
    apiKey: {
      sepolia: process.env.ETHERSCAN_API_KEY || "",
    },
  },
  gasReporter: {
    enabled: process.env.REPORT_GAS === "true",
    currency: "USD",
  },
  paths: {
    sources: "./contracts",
    tests: "./test",
    cache: "./cache",
    artifacts: "./artifacts",
  },
};
```

### Step 5: `package.json` Scripts

Add these scripts to `package.json`:

```json
{
  "scripts": {
    "compile": "hardhat compile",
    "test": "hardhat test",
    "test:coverage": "hardhat coverage",
    "test:gas": "REPORT_GAS=true hardhat test",
    "deploy:local": "hardhat run scripts/deploy.js --network localhost",
    "deploy:sepolia": "hardhat run scripts/deploy.js --network sepolia",
    "node": "hardhat node",
    "lint:sol": "solhint 'contracts/**/*.sol'",
    "lint:js": "eslint 'test/**/*.js' 'scripts/**/*.js' 'frontend/js/**/*.js'",
    "lint": "npm run lint:sol && npm run lint:js",
    "format": "prettier --write '**/*.{js,sol,json,md}'",
    "verify": "hardhat verify"
  },
  "lint-staged": {
    "*.sol": ["solhint", "prettier --write"],
    "*.{js,json,md}": ["eslint --fix", "prettier --write"]
  }
}
```

### Step 6: `.gitignore`

```
node_modules/
artifacts/
cache/
coverage/
coverage.json
.env
deployments/
typechain-types/
```

### Step 7: `.eslintrc.json`

```json
{
  "env": {
    "browser": true,
    "es2021": true,
    "node": true,
    "mocha": true
  },
  "extends": ["eslint:recommended", "prettier"],
  "parserOptions": {
    "ecmaVersion": "latest",
    "sourceType": "module"
  },
  "rules": {
    "no-console": "warn",
    "no-unused-vars": "warn"
  }
}
```

### Step 8: `.prettierrc`

```json
{
  "semi": true,
  "singleQuote": false,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 100,
  "overrides": [
    {
      "files": "*.sol",
      "options": {
        "tabWidth": 4,
        "singleQuote": false
      }
    }
  ]
}
```

### Step 9: `.solhint.json`

```json
{
  "extends": "solhint:recommended",
  "rules": {
    "compiler-version": ["error", "^0.8.20"],
    "func-visibility": ["warn", { "ignoreConstructors": true }],
    "no-empty-blocks": "warn",
    "reason-string": "warn"
  }
}
```

### Step 10: `.coderabbit.yaml`

```yaml
language: "en-US"
early_access: true
reviews:
  profile: "chill"
  request_changes_workflow: false
  high_level_summary: true
  poem: false
  review_status: true
  collapse_walkthrough: false
  path_filters:
    - "!artifacts/**"
    - "!cache/**"
    - "!coverage/**"
    - "!node_modules/**"
  path_instructions:
    - path: "contracts/**/*.sol"
      instructions: |
        Focus on:
        - Reentrancy vulnerabilities
        - Access control correctness
        - Integer overflow/underflow (even with 0.8.x checked math)
        - Gas optimization opportunities
        - Event emission completeness
    - path: "test/**/*.js"
      instructions: |
        Ensure edge cases are tested: zero values, boundary timestamps, unauthorized access.
```

### Step 11: Husky Pre-commit Hook

Edit `.husky/pre-commit`:

```bash
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

npx lint-staged
```

### Step 12: `vercel.json`

```json
{
  "version": 2,
  "builds": [
    {
      "src": "frontend/**",
      "use": "@vercel/static"
    }
  ],
  "routes": [
    {
      "src": "/(.*)",
      "dest": "/frontend/$1"
    }
  ]
}
```

### Step 13: `.env.example`

Create the file as shown in Section 4.

### Step 14: `PULL_REQUEST_TEMPLATE.md`

Create `.github/PULL_REQUEST_TEMPLATE.md`:

```markdown
## Summary
<!-- What does this PR do? -->

## Type of Change
- [ ] Smart contract change
- [ ] Frontend change
- [ ] Test addition/fix
- [ ] Config/tooling change

## Testing
- [ ] Tests pass locally (`npm test`)
- [ ] New tests added for new functionality
- [ ] Coverage maintained or improved

## Checklist
- [ ] `npm run lint` passes
- [ ] No `.env` or secrets committed
- [ ] `deployments/` not committed (if local only)
```

### Verification (Module 0)

```bash
npm run compile   # Should succeed with 0 contracts (no contracts written yet)
npm test          # Should output "0 passing"
npm run lint:sol  # Should output nothing (no .sol files yet)
```

---

## 6. Module 1 — Core Smart Contract

> **Goal:** Write, test, and verify `TimeCapsule.sol`. This contract manages capsule creation, storage, and unlocking WITHOUT NFT functionality (that comes in Module 2).

### `contracts/TimeCapsule.sol`

Write this contract with the following complete specification:

```
SPDX-License-Identifier: MIT
Pragma: 0.8.20
Imports: @openzeppelin/contracts/utils/ReentrancyGuard.sol
         @openzeppelin/contracts/access/Ownable.sol
```

#### Structs

```
struct Capsule {
    uint256 id;               // Unique capsule ID (auto-incremented)
    address creator;          // Wallet that created the capsule
    address recipient;        // Wallet allowed to open the capsule
    uint256 unlockTime;       // Unix timestamp after which capsule can be opened
    string contentCID;        // IPFS CID of the encrypted content blob
    string encryptedKey;      // AES key encrypted with recipient's public key (stored as hex string)
    bool isOpened;            // Whether the capsule has been opened
    string title;             // Public title of the capsule (not encrypted)
    uint256 createdAt;        // block.timestamp at creation
}
```

#### State Variables

```
uint256 private _capsuleCounter;                          // starts at 0, incremented before each creation
mapping(uint256 => Capsule) private _capsules;            // id => Capsule
mapping(address => uint256[]) private _creatorCapsules;   // creator => list of capsule IDs
mapping(address => uint256[]) private _recipientCapsules; // recipient => list of capsule IDs
```

#### Events

```
event CapsuleCreated(
    uint256 indexed capsuleId,
    address indexed creator,
    address indexed recipient,
    uint256 unlockTime,
    string title
);

event CapsuleOpened(
    uint256 indexed capsuleId,
    address indexed opener,
    uint256 openedAt
);
```

#### Custom Errors (use instead of require strings for gas efficiency)

```
error TimeCapsule__UnlockTimeInPast();
error TimeCapsule__NotYetUnlocked(uint256 unlockTime, uint256 currentTime);
error TimeCapsule__NotRecipient(address caller, address recipient);
error TimeCapsule__AlreadyOpened(uint256 capsuleId);
error TimeCapsule__CapsuleNotFound(uint256 capsuleId);
error TimeCapsule__EmptyContentCID();
error TimeCapsule__InvalidRecipient();
error TimeCapsule__TitleTooLong();
```

#### Functions

**`createCapsule`**
```
function createCapsule(
    address recipient,
    uint256 unlockTime,
    string calldata contentCID,
    string calldata encryptedKey,
    string calldata title
) external returns (uint256 capsuleId)
```
- Validate: `unlockTime > block.timestamp` → revert `TimeCapsule__UnlockTimeInPast`
- Validate: `recipient != address(0)` → revert `TimeCapsule__InvalidRecipient`
- Validate: `bytes(contentCID).length > 0` → revert `TimeCapsule__EmptyContentCID`
- Validate: `bytes(title).length <= 100` → revert `TimeCapsule__TitleTooLong`
- Increment `_capsuleCounter`
- Assign `capsuleId = _capsuleCounter`
- Create and store Capsule struct
- Push id to `_creatorCapsules[msg.sender]` and `_recipientCapsules[recipient]`
- Emit `CapsuleCreated`
- Return `capsuleId`

**`openCapsule`**
```
function openCapsule(uint256 capsuleId) external nonReentrant
```
- Validate capsule exists: `_capsules[capsuleId].id != 0` → revert `TimeCapsule__CapsuleNotFound`
- Validate caller is recipient: `msg.sender == capsule.recipient` → revert `TimeCapsule__NotRecipient`
- Validate not already opened: `!capsule.isOpened` → revert `TimeCapsule__AlreadyOpened`
- Validate unlock time passed: `block.timestamp >= capsule.unlockTime` → revert `TimeCapsule__NotYetUnlocked`
- Set `_capsules[capsuleId].isOpened = true`
- Emit `CapsuleOpened`

**`getCapsule`**
```
function getCapsule(uint256 capsuleId) external view returns (Capsule memory)
```
- Validate capsule exists
- Return capsule struct

**`getCapsulesByCreator`**
```
function getCapsulesByCreator(address creator) external view returns (uint256[] memory)
```
- Return `_creatorCapsules[creator]`

**`getCapsulesByRecipient`**
```
function getCapsulesByRecipient(address recipient) external view returns (uint256[] memory)
```
- Return `_recipientCapsules[recipient]`

**`getTotalCapsules`**
```
function getTotalCapsules() external view returns (uint256)
```
- Return `_capsuleCounter`

### `test/TimeCapsule.test.js`

Write comprehensive Mocha/Chai tests covering ALL of the following cases:

#### Deployment Tests
- Contract deploys successfully
- `getTotalCapsules()` returns 0 on fresh deploy

#### `createCapsule` Tests
- ✅ Creates a capsule with valid inputs; returns correct ID
- ✅ Emits `CapsuleCreated` event with correct args
- ✅ Increments total capsules counter
- ✅ Stores capsule data correctly (verify via `getCapsule`)
- ✅ Creator can also be recipient (self-addressed capsule)
- ❌ Reverts if `unlockTime` is in the past
- ❌ Reverts if `unlockTime` equals `block.timestamp`
- ❌ Reverts if `recipient` is zero address
- ❌ Reverts if `contentCID` is empty string
- ❌ Reverts if `title` exceeds 100 characters
- ✅ Multiple capsules can be created; each gets unique ID
- ✅ `getCapsulesByCreator` returns correct IDs
- ✅ `getCapsulesByRecipient` returns correct IDs

#### `openCapsule` Tests
- ✅ Recipient opens capsule after unlock time → `isOpened` becomes true
- ✅ Emits `CapsuleOpened` event with correct args
- ❌ Reverts if `capsuleId` does not exist
- ❌ Reverts if caller is not the recipient
- ❌ Reverts if capsule is already opened
- ❌ Reverts if called before `unlockTime`
- Use `time.increaseTo()` from `@nomicfoundation/hardhat-network-helpers` to manipulate time

#### Test Helper Pattern

```javascript
const { time } = require("@nomicfoundation/hardhat-network-helpers");
const { expect } = require("chai");
const { ethers } = require("hardhat");

describe("TimeCapsule", function () {
  let timeCapsule;
  let owner, creator, recipient, other;
  const ONE_DAY = 86400;

  async function getUnlockTime(secondsFromNow = ONE_DAY) {
    const latest = await time.latest();
    return latest + secondsFromNow;
  }

  beforeEach(async function () {
    [owner, creator, recipient, other] = await ethers.getSigners();
    const TimeCapsule = await ethers.getContractFactory("TimeCapsule");
    timeCapsule = await TimeCapsule.deploy();
    await timeCapsule.waitForDeployment();
  });

  // ... tests here
});
```

### Verification (Module 1)

```bash
npm run compile    # Must succeed, 0 warnings ideally
npm test           # All tests must pass
npm run test:gas   # Review gas report; createCapsule should be < 200k gas
npm run lint:sol   # 0 errors
```

---

## 7. Module 2 — NFT Extension

> **Goal:** Create `TimeCapsuleNFT.sol`, an ERC-721 contract that mints an NFT for each capsule created. The NFT is soulbound (non-transferable) until the capsule is opened.

### `contracts/TimeCapsuleNFT.sol`

```
SPDX-License-Identifier: MIT
Pragma: 0.8.20
Imports:
  @openzeppelin/contracts/token/ERC721/ERC721.sol
  @openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol
  @openzeppelin/contracts/access/Ownable.sol
  @openzeppelin/contracts/utils/Counters.sol  (or manual counter for OZ v5)
```

> **Note on OZ v5:** OpenZeppelin v5 removed `Counters.sol`. Use a plain `uint256 private _nextTokenId` and increment manually.

#### Key Design

- Inherits `ERC721URIStorage`, `Ownable`
- The `TimeCapsule.sol` contract address is set as an authorized minter (set in constructor)
- On every `createCapsule()` call in `TimeCapsule.sol`, it calls `TimeCapsuleNFT.mintCapsuleNFT(to, tokenURI)`
- NFT token ID == capsule ID for 1:1 mapping
- **Soulbound mechanism:** Override `_update()` (OZ v5) to block transfers unless `isOpened == true`
  - The NFT contract must have a reference to the `TimeCapsule` contract to check `isOpened`
- `tokenURI` points to a JSON metadata object pinned on IPFS

#### State Variables

```
address public timeCapsuleContract;    // Address of TimeCapsule.sol (set in constructor)
address public timeCapsuleAddress;     // Same reference for capsule status lookup
uint256 private _nextTokenId;
```

#### Functions

**Constructor**
```
constructor(address _timeCapsuleContract) ERC721("TimeCapsule", "TCAP") Ownable(msg.sender)
```
- Set `timeCapsuleContract = _timeCapsuleContract`

**`mintCapsuleNFT`**
```
function mintCapsuleNFT(
    address to,
    uint256 tokenId,
    string calldata metadataCID
) external
```
- Only callable by `timeCapsuleContract` address
- Mint token with `_safeMint(to, tokenId)`
- Set token URI: `_setTokenURI(tokenId, string.concat("ipfs://", metadataCID))`

**Override `_update` (Soulbound logic)**
```
function _update(
    address to,
    uint256 tokenId,
    address auth
) internal override returns (address)
```
- Allow minting (from == address(0)) always
- Allow burning (to == address(0)) always
- For transfers: check if capsule `tokenId` is opened via `ITimeCapsule(timeCapsuleContract).getCapsule(tokenId).isOpened`
- If NOT opened → revert with `TimeCapsuleNFT__CapsuleLocked(tokenId)`
- If opened → allow transfer (call `super._update`)

**`tokenURI`**
```
function tokenURI(uint256 tokenId) public view override(ERC721, ERC721URIStorage) returns (string memory)
```
- Override both parents

#### NFT Metadata Format (for IPFS)

Each capsule NFT's metadata JSON must follow this schema:

```json
{
  "name": "TimeCapsule #<id>",
  "description": "A cryptographic time capsule locked until <date>. Created by <creator_short_address>.",
  "image": "ipfs://<static_image_CID>",
  "attributes": [
    { "trait_type": "Status", "value": "Locked" },
    { "trait_type": "Unlock Date", "display_type": "date", "value": <unix_timestamp> },
    { "trait_type": "Creator", "value": "<creator_address>" },
    { "trait_type": "Recipient", "value": "<recipient_address>" }
  ]
}
```

#### Refactor `TimeCapsule.sol` for NFT Integration

Add to `TimeCapsule.sol`:
- Import an interface `ITimeCapsuleNFT` with just `mintCapsuleNFT(address, uint256, string)`
- Add state variable: `ITimeCapsuleNFT public nftContract`
- Add function `setNFTContract(address _nft) external onlyOwner` — sets `nftContract`
- In `createCapsule`, after storing the capsule, if `address(nftContract) != address(0)`, call `nftContract.mintCapsuleNFT(msg.sender, capsuleId, contentCID)`

### `test/TimeCapsuleNFT.test.js`

Cover:
- ✅ NFT minted when capsule created (check `ownerOf(1) == creator`)
- ✅ Token URI set correctly
- ✅ Transfer BLOCKED before capsule opened
- ✅ Transfer ALLOWED after capsule opened
- ✅ Only `timeCapsuleContract` can call `mintCapsuleNFT`
- ❌ External wallet cannot call `mintCapsuleNFT`

### Verification (Module 2)

```bash
npm test           # All existing + new NFT tests pass
npm run test:gas   # Review NFT minting gas cost
```

---

## 8. Module 3 — Encryption & IPFS Layer

> **Goal:** Build the browser-native encryption utilities and Pinata IPFS integration. No server-side code. Everything runs in the user's browser.

### Encryption Design

**Algorithm:** AES-GCM with 256-bit key  
**Why:** Browser-native via Web Crypto API, authenticated encryption (prevents tampering), no library needed.

**Key Storage Strategy:**
- An AES-GCM key is generated per capsule
- The raw key bytes are exported and stored as a hex string in the contract's `encryptedKey` field
- **Important note:** In this MVP, the key is stored on-chain in plaintext hex (accessible to anyone who knows the capsule ID). This is intentional for the MVP to avoid asymmetric encryption complexity. The spec for a future version would encrypt the key with the recipient's public key (e.g., via ECIES). Document this limitation prominently in README.

### `frontend/js/encrypt.js`

Implement the following exported functions:

```javascript
/**
 * Generate a new AES-GCM 256-bit key.
 * @returns {Promise<{key: CryptoKey, keyHex: string}>}
 * keyHex is the raw exported key as a lowercase hex string.
 */
export async function generateKey() { ... }

/**
 * Encrypt a plaintext string.
 * @param {string} plaintext - The message to encrypt
 * @param {CryptoKey} key - The AES-GCM key
 * @returns {Promise<{ciphertext: ArrayBuffer, iv: Uint8Array}>}
 * iv is 12 random bytes (96 bits), required for decryption.
 */
export async function encryptMessage(plaintext, key) { ... }

/**
 * Decrypt ciphertext.
 * @param {ArrayBuffer} ciphertext
 * @param {CryptoKey} key
 * @param {Uint8Array} iv
 * @returns {Promise<string>} - The original plaintext
 */
export async function decryptMessage(ciphertext, key, iv) { ... }

/**
 * Convert a hex string back to a CryptoKey.
 * @param {string} keyHex
 * @returns {Promise<CryptoKey>}
 */
export async function hexToKey(keyHex) { ... }

/**
 * Package ciphertext + iv into a single JSON blob for IPFS storage.
 * @param {ArrayBuffer} ciphertext
 * @param {Uint8Array} iv
 * @returns {string} - JSON string: { "iv": "<base64>", "data": "<base64>" }
 */
export function packageEncryptedBlob(ciphertext, iv) { ... }

/**
 * Unpackage the JSON blob from IPFS back into ciphertext + iv.
 * @param {string} jsonStr
 * @returns {{ ciphertext: ArrayBuffer, iv: Uint8Array }}
 */
export function unpackageEncryptedBlob(jsonStr) { ... }
```

**Implementation Notes:**
- Use `window.crypto.subtle` for all operations
- IV must be random per encryption (`window.crypto.getRandomValues`)
- Export/import key using `"raw"` format
- All ArrayBuffer ↔ base64 conversions via `btoa`/`atob` on `Uint8Array`

### `frontend/js/ipfs.js`

Implement Pinata integration using their REST API (no SDK needed — plain `fetch`):

```javascript
// Read from config.js
import { PINATA_JWT } from "./config.js";

/**
 * Upload a JSON string to IPFS via Pinata.
 * @param {string} jsonContent - The JSON string to upload
 * @param {string} name - Pin name (for Pinata dashboard identification)
 * @returns {Promise<string>} - The IPFS CID (content identifier)
 */
export async function uploadToIPFS(jsonContent, name) {
  // POST to https://api.pinata.cloud/pinning/pinJSONToIPFS
  // Headers: Authorization: Bearer <PINATA_JWT>, Content-Type: application/json
  // Body: { pinataContent: JSON.parse(jsonContent), pinataMetadata: { name } }
  // Returns: response.IpfsHash (the CID)
}

/**
 * Fetch content from IPFS via Pinata gateway.
 * @param {string} cid - The IPFS CID
 * @returns {Promise<string>} - The raw content string
 */
export async function fetchFromIPFS(cid) {
  // GET https://gateway.pinata.cloud/ipfs/<cid>
  // Returns response.text()
}

/**
 * Upload NFT metadata JSON to IPFS.
 * @param {object} metadata - The NFT metadata object
 * @returns {Promise<string>} - CID of the metadata
 */
export async function uploadNFTMetadata(metadata) {
  return uploadToIPFS(JSON.stringify(metadata), `TimeCapsule-NFT-Metadata`);
}
```

**Error Handling:** All functions must `try/catch` and throw descriptive errors. Never let an IPFS failure silently fail.

---

## 9. Module 4 — Frontend

> **Goal:** Build a complete, functional single-page frontend. No frameworks. Vanilla HTML/CSS/JS. It must connect to MetaMask, allow creating and opening capsules, and display a dashboard.

### Design Aesthetic

- **Theme:** Dark, mysterious, futuristic — like a digital time vault
- **Color palette:** Deep space black (`#0a0a0f`), electric indigo (`#4f46e5`), neon cyan (`#06b6d4`), soft white (`#f8fafc`)
- **Font:** Use Google Fonts CDN — `Space Grotesk` for headings, `Inter` for body
- **UI Feel:** Glassmorphism cards, subtle gradient borders, glowing CTA buttons
- **Fully responsive:** Works on mobile and desktop

### `frontend/index.html`

Structure:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <!-- Meta tags, title, Google Fonts CDN, Ethers.js v6 CDN, stylesheet -->
</head>
<body>
  <!-- NAVBAR -->
  <nav id="navbar">
    <!-- Logo | Nav Links: Home, Create, Dashboard | Connect Wallet Button -->
  </nav>

  <!-- PAGE: LANDING (#page-home) -->
  <section id="page-home" class="page active">
    <!-- Hero: "Seal Your Memories in Time" + CTA "Create Capsule" -->
    <!-- Features grid: 3 cards (Time-Locked, Encrypted, NFT Collectible) -->
    <!-- Stats bar: Total Capsules Created (fetched from contract) -->
  </section>

  <!-- PAGE: CREATE CAPSULE (#page-create) -->
  <section id="page-create" class="page hidden">
    <!-- Step indicator: Step 1 → Step 2 → Step 3 -->
    <!-- Step 1: Capsule Details
           - Title (text input, max 100 chars, char counter)
           - Recipient address (text input, validate eth address format)
           - Unlock Date (datetime-local input, min = tomorrow)
           - Message (textarea, up to 10,000 chars)
    -->
    <!-- Step 2: Encryption Preview
           - Show "Your message will be encrypted" notice
           - Show generated key preview (first 8 chars + "...")
           - Warn: "Save your capsule ID after creation"
    -->
    <!-- Step 3: Confirm & Submit
           - Summary card showing all details
           - Gas estimate notice
           - "Create Capsule" button → triggers wallet transaction
           - Loading spinner during tx
           - Success card with capsule ID + TX hash link (Sepolia Etherscan)
    -->
  </section>

  <!-- PAGE: DASHBOARD (#page-dashboard) -->
  <section id="page-dashboard" class="page hidden">
    <!-- Tab bar: "Created by Me" | "Addressed to Me" -->
    <!-- Capsule card grid: renders dynamically -->
    <!-- Empty state illustration if no capsules -->
  </section>

  <!-- MODAL: OPEN CAPSULE -->
  <div id="modal-open-capsule" class="modal hidden">
    <!-- Shows capsule details -->
    <!-- If locked: countdown timer to unlock time -->
    <!-- If unlocked: "Open Capsule" button → triggers tx → decrypts → shows message -->
  </div>

  <!-- TOAST NOTIFICATIONS -->
  <div id="toast-container"></div>

  <!-- Scripts (type="module") -->
  <script type="module" src="js/app.js"></script>
</body>
</html>
```

### `frontend/js/config.js`

```javascript
// This file is auto-updated by the deploy script.
// Do not edit manually.

export const NETWORK_CHAIN_ID = 11155111; // Sepolia
export const NETWORK_NAME = "Sepolia";

export const TIMECAPSULE_ADDRESS = "REPLACE_AFTER_DEPLOY";
export const TIMECAPSULE_NFT_ADDRESS = "REPLACE_AFTER_DEPLOY";

// Pinata JWT — NOTE: In production, this should be a read-only gateway key.
// For MVP/testnet this is acceptable.
export const PINATA_JWT = "REPLACE_WITH_PINATA_JWT";

// Paste compiled ABI arrays here after running `npm run compile`
export const TIMECAPSULE_ABI = []; // paste from artifacts/contracts/TimeCapsule.sol/TimeCapsule.json
export const TIMECAPSULE_NFT_ABI = []; // paste from artifacts/contracts/TimeCapsuleNFT.sol/TimeCapsuleNFT.json
```

> **AI Instruction:** After `npm run compile`, read the ABI from `artifacts/contracts/TimeCapsule.sol/TimeCapsule.json` and `artifacts/contracts/TimeCapsuleNFT.sol/TimeCapsuleNFT.json` and paste the `abi` arrays into `config.js`.

### `frontend/js/wallet.js`

```javascript
/**
 * Connect to MetaMask and return the provider + signer.
 * Prompt network switch to Sepolia if on wrong network.
 * @returns {Promise<{provider, signer, address}>}
 */
export async function connectWallet() { ... }

/**
 * Get current connected account without prompting.
 * Returns null if not connected.
 */
export async function getCurrentAccount() { ... }

/**
 * Listen for account changes and chain changes.
 * Calls provided callbacks on change.
 */
export function setupWalletListeners(onAccountChange, onChainChange) { ... }

/**
 * Request Sepolia network switch.
 */
export async function switchToSepolia() { ... }
```

### `frontend/js/contract.js`

```javascript
import { ethers } from "ethers"; // from CDN: window.ethers
import { TIMECAPSULE_ADDRESS, TIMECAPSULE_ABI, TIMECAPSULE_NFT_ADDRESS, TIMECAPSULE_NFT_ABI } from "./config.js";

/**
 * Get a read-only contract instance (no signer needed).
 */
export function getReadContract(provider) { ... }

/**
 * Get a write contract instance (signer required).
 */
export function getWriteContract(signer) { ... }

/**
 * Create a new time capsule.
 * @param {object} signer
 * @param {string} recipient
 * @param {number} unlockTimestamp
 * @param {string} contentCID - IPFS CID
 * @param {string} encryptedKeyHex
 * @param {string} title
 * @returns {Promise<{tx, receipt, capsuleId}>}
 */
export async function createCapsule(signer, recipient, unlockTimestamp, contentCID, encryptedKeyHex, title) { ... }

/**
 * Open a capsule (must be recipient, must be after unlockTime).
 * @returns {Promise<{tx, receipt}>}
 */
export async function openCapsule(signer, capsuleId) { ... }

/**
 * Fetch a single capsule's data.
 */
export async function getCapsule(provider, capsuleId) { ... }

/**
 * Fetch all capsule IDs created by an address.
 */
export async function getCapsulesByCreator(provider, address) { ... }

/**
 * Fetch all capsule IDs addressed to an address.
 */
export async function getCapsulesByRecipient(provider, address) { ... }

/**
 * Get total capsules ever created.
 */
export async function getTotalCapsules(provider) { ... }
```

### `frontend/js/ui.js`

Implement all UI rendering:

```javascript
/**
 * Render a capsule card.
 * @param {object} capsule - From getCapsule()
 * @param {string} role - "creator" | "recipient"
 * @returns {HTMLElement}
 */
export function renderCapsuleCard(capsule, role) { ... }

/**
 * Show a toast notification.
 * @param {string} message
 * @param {string} type - "success" | "error" | "info" | "warning"
 */
export function showToast(message, type = "info") { ... }

/**
 * Show/hide pages by ID.
 */
export function navigateTo(pageId) { ... }

/**
 * Update wallet button UI (show address or "Connect Wallet").
 */
export function updateWalletButton(address) { ... }

/**
 * Render a countdown timer for a locked capsule.
 * @param {number} unlockTime - Unix timestamp
 * @param {HTMLElement} targetEl - Element to update
 */
export function renderCountdown(unlockTime, targetEl) { ... }

/**
 * Show the create capsule step indicator.
 * @param {number} step - 1, 2, or 3
 */
export function updateStepIndicator(step) { ... }

/**
 * Show loading spinner on a button.
 */
export function setButtonLoading(btn, loading) { ... }
```

### `frontend/js/app.js` — Main Entry Point

Wire everything together:

```javascript
import { connectWallet, getCurrentAccount, setupWalletListeners } from "./wallet.js";
import { createCapsule, openCapsule, getCapsule, getCapsulesByCreator, getCapsulesByRecipient, getTotalCapsules } from "./contract.js";
import { generateKey, encryptMessage, packageEncryptedBlob, hexToKey, unpackageEncryptedBlob, decryptMessage } from "./encrypt.js";
import { uploadToIPFS, fetchFromIPFS, uploadNFTMetadata } from "./ipfs.js";
import { showToast, navigateTo, updateWalletButton, renderCapsuleCard, renderCountdown, updateStepIndicator, setButtonLoading } from "./ui.js";

// === App State ===
const state = {
  provider: null,
  signer: null,
  address: null,
  currentPage: "home",
  createStep: 1,
  pendingCapsule: {},   // Holds form data between steps
};

// === Event: Connect Wallet Button ===
document.getElementById("btn-connect").addEventListener("click", async () => { ... });

// === Event: Nav Links ===
// Navigate between pages

// === Event: Create Capsule Form (Step 1 → 2 → 3) ===
// Handle step transitions, validate inputs, show previews

// === Event: Submit Create Capsule (Step 3) ===
async function handleCreateCapsule() {
  // 1. Generate AES key
  // 2. Encrypt message
  // 3. Package blob
  // 4. Upload blob to IPFS → get contentCID
  // 5. Build NFT metadata → upload to IPFS → get metadataCID
  // 6. Call createCapsule() on contract
  // 7. Show success with capsule ID + etherscan link
}

// === Event: Open Capsule (from modal) ===
async function handleOpenCapsule(capsuleId) {
  // 1. Call openCapsule() on contract → mark as opened
  // 2. Fetch encryptedKey from capsule data
  // 3. Reconstruct CryptoKey from hex
  // 4. Fetch blob from IPFS
  // 5. Unpackage blob → decrypt
  // 6. Display plaintext message in modal
}

// === Dashboard: Load capsules ===
async function loadDashboard() {
  // Fetch created + received capsule IDs
  // For each ID, fetch capsule data
  // Render capsule cards in correct tabs
}

// === Init ===
async function init() {
  const account = await getCurrentAccount();
  if (account) {
    // restore connected state
  }
  // Fetch total capsules for landing page stats
  setupWalletListeners(onAccountChange, onChainChange);
}

init();
```

### `frontend/css/styles.css`

Write complete CSS with:
- CSS custom properties (variables) for the color palette
- Reset / base styles
- Navbar (fixed, glassmorphism)
- Page layout system (`.page`, `.page.active`, `.page.hidden`)
- Hero section with gradient text
- Feature cards (glassmorphism, hover glow)
- Create capsule form (step wizard, step indicator progress bar)
- Capsule cards (status badge: Locked 🔒 / Opened 🔓, countdown)
- Modal styles
- Toast notification animations (slide in from right, auto-dismiss after 4s)
- Loading spinner
- Responsive breakpoints (mobile-first, breakpoint at 768px)
- Etherscan link button

---

## 10. Module 5 — Deployment Scripts

### `scripts/deploy.js`

```javascript
const { ethers } = require("hardhat");
const fs = require("fs");
const path = require("path");

async function main() {
  console.log("🚀 Starting deployment...\n");

  const [deployer] = await ethers.getSigners();
  console.log(`Deploying with account: ${deployer.address}`);
  console.log(`Account balance: ${ethers.formatEther(await deployer.provider.getBalance(deployer.address))} ETH\n`);

  // 1. Deploy TimeCapsule.sol
  console.log("Deploying TimeCapsule...");
  const TimeCapsule = await ethers.getContractFactory("TimeCapsule");
  const timeCapsule = await TimeCapsule.deploy();
  await timeCapsule.waitForDeployment();
  const timeCapsuleAddress = await timeCapsule.getAddress();
  console.log(`✅ TimeCapsule deployed to: ${timeCapsuleAddress}`);

  // 2. Deploy TimeCapsuleNFT.sol (passing TimeCapsule address)
  console.log("\nDeploying TimeCapsuleNFT...");
  const TimeCapsuleNFT = await ethers.getContractFactory("TimeCapsuleNFT");
  const timeCapsuleNFT = await TimeCapsuleNFT.deploy(timeCapsuleAddress);
  await timeCapsuleNFT.waitForDeployment();
  const timeCapsuleNFTAddress = await timeCapsuleNFT.getAddress();
  console.log(`✅ TimeCapsuleNFT deployed to: ${timeCapsuleNFTAddress}`);

  // 3. Link NFT contract to TimeCapsule
  console.log("\nLinking NFT contract...");
  const tx = await timeCapsule.setNFTContract(timeCapsuleNFTAddress);
  await tx.wait();
  console.log("✅ NFT contract linked");

  // 4. Save deployment info
  const deploymentInfo = {
    network: hre.network.name,
    chainId: (await ethers.provider.getNetwork()).chainId.toString(),
    deployedAt: new Date().toISOString(),
    deployer: deployer.address,
    contracts: {
      TimeCapsule: timeCapsuleAddress,
      TimeCapsuleNFT: timeCapsuleNFTAddress,
    },
  };

  const deploymentsDir = path.join(__dirname, "../deployments");
  if (!fs.existsSync(deploymentsDir)) fs.mkdirSync(deploymentsDir);
  const filename = path.join(deploymentsDir, `${hre.network.name}.json`);
  fs.writeFileSync(filename, JSON.stringify(deploymentInfo, null, 2));
  console.log(`\n📄 Deployment info saved to deployments/${hre.network.name}.json`);

  // 5. Update frontend/js/config.js with new addresses
  const configPath = path.join(__dirname, "../frontend/js/config.js");
  let configContent = fs.readFileSync(configPath, "utf8");
  configContent = configContent
    .replace(/TIMECAPSULE_ADDRESS = ".*?"/, `TIMECAPSULE_ADDRESS = "${timeCapsuleAddress}"`)
    .replace(/TIMECAPSULE_NFT_ADDRESS = ".*?"/, `TIMECAPSULE_NFT_ADDRESS = "${timeCapsuleNFTAddress}"`);
  fs.writeFileSync(configPath, configContent);
  console.log("✅ frontend/js/config.js updated with contract addresses");

  console.log("\n🎉 Deployment complete!\n");
  console.log("Next steps:");
  console.log(`  1. Verify on Etherscan: npx hardhat verify --network sepolia ${timeCapsuleAddress}`);
  console.log(`  2. Verify NFT: npx hardhat verify --network sepolia ${timeCapsuleNFTAddress} "${timeCapsuleAddress}"`);
  console.log("  3. Update PINATA_JWT in frontend/js/config.js");
  console.log("  4. Paste ABIs into frontend/js/config.js");
  console.log("  5. Deploy frontend to Vercel");
}

main().catch((err) => {
  console.error(err);
  process.exit(1);
});
```

### ABI Auto-Population Script

After `npm run compile`, also run this snippet (can be part of `deploy.js` or a separate `scripts/update-abi.js`):

```javascript
// Read ABI from artifacts and write to config.js
const timeCapsuleArtifact = require("../artifacts/contracts/TimeCapsule.sol/TimeCapsule.json");
const nftArtifact = require("../artifacts/contracts/TimeCapsuleNFT.sol/TimeCapsuleNFT.json");

// Replace TIMECAPSULE_ABI = [] and TIMECAPSULE_NFT_ABI = [] in config.js
// with the actual ABI arrays from the artifacts
```

---

## 11. Module 6 — CI/CD & GitHub Actions

### `.github/workflows/ci.yml`

```yaml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test:
    name: Compile & Test Contracts
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Create .env for CI
        run: |
          echo "PRIVATE_KEY=0x0000000000000000000000000000000000000000000000000000000000000001" >> .env
          echo "SEPOLIA_RPC_URL=https://rpc.ankr.com/eth_sepolia" >> .env
          echo "ETHERSCAN_API_KEY=dummy" >> .env

      - name: Compile contracts
        run: npm run compile

      - name: Run Solidity linter
        run: npm run lint:sol

      - name: Run tests
        run: npm test

      - name: Run coverage
        run: npm run test:coverage

      - name: Upload coverage report
        uses: actions/upload-artifact@v4
        with:
          name: coverage-report
          path: coverage/

  lint-js:
    name: Lint JavaScript
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"
      - run: npm ci
      - run: npm run lint:js
```

---

## 12. Module 7 — End-to-End Testing

> **Goal:** Write integration tests that simulate a complete user flow against a local Hardhat node.

### Test Scenario: Full Happy Path

File: `test/integration.test.js`

```
Scenario: Alice creates a capsule addressed to Bob.
Bob cannot open it before unlock time.
Time is advanced past unlock time.
Bob opens the capsule.
NFT transfer is blocked before opening, allowed after.
```

Cover all these assertions in sequence:
1. Deploy both contracts & link
2. Alice calls `createCapsule` with Bob as recipient, unlock = now + 7 days
3. Contract emits `CapsuleCreated` with correct args
4. NFT minted to Alice (ownerOf(1) == Alice)
5. Bob calls `openCapsule` → REVERTS with `TimeCapsule__NotYetUnlocked`
6. Alice tries to transfer NFT to Charlie → REVERTS with `TimeCapsuleNFT__CapsuleLocked`
7. `time.increaseTo(unlockTime + 1)` — advance time
8. Bob calls `openCapsule` → SUCCEEDS, `isOpened == true`
9. `CapsuleOpened` event emitted
10. Alice transfers NFT to Charlie → SUCCEEDS (capsule now opened)

### Coverage Requirements

Run `npm run test:coverage` and ensure:
- `TimeCapsule.sol` → **≥ 95% line coverage**
- `TimeCapsuleNFT.sol` → **≥ 90% line coverage**

If below threshold, write additional edge-case tests until met.

---

## 13. Coding Standards & Conventions

### Solidity
- All `public`/`external` functions must have NatSpec `/// @notice` comments
- Use `custom errors` instead of `require(condition, "string")` everywhere
- Events must be emitted for every state change
- State variables: `private` by default, expose via `view` functions
- No `payable` functions (no ETH involved in this version)
- Follow Checks-Effects-Interactions pattern in all functions

### JavaScript
- ES6+ syntax only: `const`, `let`, arrow functions, async/await, template literals
- No `var`
- All async operations wrapped in `try/catch`
- All user-facing errors surfaced via `showToast(error.message, "error")`
- No hardcoded addresses in JS files — all from `config.js`
- No `console.log` in production code — use only in dev/scripts

### Git
- Commit message format: `type(scope): description`
  - Types: `feat`, `fix`, `test`, `chore`, `docs`, `refactor`
  - Example: `feat(contract): add soulbound transfer restriction to NFT`
- Branch naming: `feat/module-1-core-contract`, `fix/nft-transfer-bug`
- Never commit: `.env`, `artifacts/`, `cache/`, `node_modules/`, `deployments/`

---

## 14. Known Constraints

| Constraint | Detail |
|---|---|
| Free tools only | No paid APIs, no paid RPC, no paid hosting |
| No backend | Fully client-side + smart contract. No Node.js server |
| Sepolia testnet only | Mainnet deployment is out of scope |
| MVP encryption | AES key stored on-chain in plaintext. Noted as a known limitation |
| Vanilla JS frontend | No React, Vue, or any framework |
| Single HTML file for Vercel | All frontend in `frontend/` dir; `vercel.json` serves it as static |
| OZ v5 compatibility | `Counters.sol` removed in v5; use manual `uint256` counter |
| Ethers.js v6 breaking changes | `ethers.utils` removed; use top-level `ethers.*`. BigNumber replaced with native JS BigInt |
| No file uploads | MVP supports text messages only; no binary file encryption |

---

## 15. Final Checklist

Before marking the project complete, verify every item:

### Contracts
- [ ] `TimeCapsule.sol` compiles with 0 errors, 0 warnings
- [ ] `TimeCapsuleNFT.sol` compiles with 0 errors, 0 warnings
- [ ] All custom errors defined and used
- [ ] All events emitted correctly
- [ ] `npm test` — 100% of tests pass
- [ ] `npm run test:coverage` — ≥ 95% coverage on core contract
- [ ] `npm run test:gas` — `createCapsule` gas < 200k
- [ ] `npm run lint:sol` — 0 errors

### Frontend
- [ ] MetaMask connection works
- [ ] Wrong network prompts Sepolia switch
- [ ] Create capsule form validates all inputs (empty fields, invalid address, past date)
- [ ] Step wizard transitions correctly
- [ ] Message encrypts → uploads to IPFS → transaction goes through → success shown
- [ ] Dashboard loads capsules for connected wallet
- [ ] Locked capsule shows countdown timer
- [ ] Opened capsule allows decryption and message display
- [ ] Toast notifications appear for success, error, info states
- [ ] Mobile responsive layout

### Infrastructure
- [ ] GitHub Actions CI passes on push to main
- [ ] `.env` not committed (check `git log --all -- .env`)
- [ ] `deployments/sepolia.json` present after deploy
- [ ] `frontend/js/config.js` has correct addresses and ABIs after deploy
- [ ] Contracts verified on Sepolia Etherscan
- [ ] Frontend deployed to Vercel and accessible via URL
- [ ] README.md complete with: project description, live demo link, setup instructions, architecture diagram (ASCII is fine), known limitations

### README.md must include:
1. Project title + one-liner description
2. Live demo URL (Vercel)
3. Contract addresses (Sepolia Etherscan links)
4. Tech stack badges
5. Architecture diagram (ASCII or Mermaid)
6. Local setup steps (`git clone` → `npm install` → set `.env` → `npx hardhat node` → deploy local → open frontend)
7. How to create a capsule (step-by-step)
8. How to open a capsule
9. Known limitations (encryption MVP warning)
10. License (MIT)

---

*End of SPEC.md — Total Modules: 8 (0–7). Build in order. Test before proceeding. Ship it.*
