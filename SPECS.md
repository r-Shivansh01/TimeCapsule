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
12. [Coding Standards & Conventions](#12-coding-standards--conventions)
13. [Known Constraints](#13-known-constraints)
14. [Final Checklist](#14-final-checklist)

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
| Pinata | IPFS pinning service (free tier) |

### Dev Tools
| Tool | Purpose |
|---|---|
| ESLint | JS linting |
| Prettier | Code formatting |
| Solhint | Solidity linting |

---

## 3. Repository & Directory Structure

Create exactly this structure. Do not deviate:

```
timecapsule/
├── .github/
│   └── workflows/
│       └── ci.yml                   # GitHub Actions CI pipeline
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
    "postcompile": "node scripts/update-abi.js",
    "update-abi": "node scripts/update-abi.js",
    "test": "hardhat test",
    "test:coverage": "hardhat coverage",
    "deploy:local": "hardhat run scripts/deploy.js --network localhost",
    "deploy:sepolia": "hardhat run scripts/deploy.js --network sepolia",
    "node": "hardhat node",
    "lint:sol": "solhint 'contracts/**/*.sol'",
    "lint:js": "eslint 'test/**/*.js' 'scripts/**/*.js' 'frontend/js/**/*.js'",
    "lint": "npm run lint:sol && npm run lint:js",
    "format": "prettier --write '**/*.{js,sol,json,md}'",
    "verify": "hardhat verify"
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
typechain-types/
```

> **Note:** `deployments/` is intentionally NOT gitignored. Files like `deployments/sepolia.json` contain only public contract addresses and should be committed so any team member or CI runner can reference them. The Final Checklist requires this file to be present after deployment.

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

### Step 10: `vercel.json`

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

### Step 11: `.env.example`

Create the file as shown in Section 4.

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
    string calldata title,
    string calldata metadataCID
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
- If `address(nftContract) != address(0)`, call `nftContract.mintCapsuleNFT(msg.sender, capsuleId, metadataCID)`
  - **Note:** `metadataCID` is the IPFS CID of the NFT metadata JSON (uploaded by the frontend before calling this function). It is separate from `contentCID`, which holds the encrypted message blob. This keeps NFT metadata and encrypted content cleanly separated.
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
- **Returns nothing.** The frontend must subsequently call `getCapsule(capsuleId)` after the transaction confirms to retrieve the `encryptedKey` for client-side decryption. Do not expect this function to return the key.

**`getCapsule`**
```
function getCapsule(uint256 capsuleId) external view returns (Capsule memory)
```
- Validate capsule exists: `_capsules[capsuleId].id != 0` → revert `TimeCapsule__CapsuleNotFound`
  - **Note:** Capsule IDs start at 1 (counter is incremented *before* first assignment). ID 0 is never assigned and serves as the null sentinel for this existence check. This invariant must be preserved — do not change the counter increment order.
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
```

> **Note on OZ v5:** OpenZeppelin v5 removed `Counters.sol`. Use a plain `uint256 private _nextTokenId` and increment manually. Do NOT import `Counters.sol`.

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
address public timeCapsuleContract;    // Address of TimeCapsule.sol (set in constructor). Also used by _update() for soulbound checks.
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
- **Note:** `metadataCID` is the CID of the NFT metadata JSON object (not the encrypted content blob). The frontend uploads the metadata JSON to IPFS before calling `createCapsule`, then passes `metadataCID` as a parameter.

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
  "image": "ipfs://<CAPSULE_IMAGE_CID>",
  "attributes": [
    { "trait_type": "Status", "value": "Locked" },
    { "trait_type": "Unlock Date", "display_type": "date", "value": <unix_timestamp> },
    { "trait_type": "Creator", "value": "<creator_address>" },
    { "trait_type": "Recipient", "value": "<recipient_address>" }
  ]
}
```

> **`CAPSULE_IMAGE_CID` setup (required before deployment):**
> 1. Create or export `frontend/assets/capsule-nft.png` — a 400×400px image representing a locked capsule (use the logo SVG rendered/exported as PNG, or any suitable image).
> 2. Upload it to Pinata manually via the Pinata web dashboard (or `curl`). Copy the resulting CID.
> 3. Add to `frontend/js/config.js`:
>    ```javascript
>    export const CAPSULE_IMAGE_CID = "Qm...yourCIDhere";
>    ```
> 4. The frontend's `app.js` must read `CAPSULE_IMAGE_CID` from `config.js` when building metadata objects before calling `uploadNFTMetadata`. Do **not** hardcode the CID anywhere other than `config.js`.
> 5. Add this step to the Final Checklist before Vercel deployment.

#### Refactor `TimeCapsule.sol` for NFT Integration

Add to `TimeCapsule.sol`:
- Import an interface `ITimeCapsuleNFT` with just `mintCapsuleNFT(address, uint256, string)`
- Add state variable: `ITimeCapsuleNFT public nftContract`
- Add function `setNFTContract(address _nft) external onlyOwner` — sets `nftContract`
- Update `createCapsule` signature to accept `string calldata metadataCID` as a sixth parameter (see Module 1 `createCapsule` spec, updated above)
- In `createCapsule`, after storing the capsule, if `address(nftContract) != address(0)`, call `nftContract.mintCapsuleNFT(msg.sender, capsuleId, metadataCID)`

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
 * Implementation: convert hex to Uint8Array, then call:
 *   window.crypto.subtle.importKey("raw", keyBytes, { name: "AES-GCM", length: 256 }, false, ["encrypt", "decrypt"])
 * The keyUsages array MUST include both "encrypt" and "decrypt" or subsequent operations will throw InvalidAccessError.
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
 * Upload a JSON-serialisable object to IPFS via Pinata.
 * @param {object} content - The object to upload (will be serialised internally)
 * @param {string} name - Pin name (for Pinata dashboard identification)
 * @returns {Promise<string>} - The IPFS CID (content identifier)
 */
export async function uploadToIPFS(content, name) {
  // POST to https://api.pinata.cloud/pinning/pinJSONToIPFS
  // Headers: Authorization: Bearer <PINATA_JWT>, Content-Type: application/json
  // Body: { pinataContent: content, pinataMetadata: { name } }
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
  // metadata is already an object — pass directly, do NOT JSON.stringify before passing
  return uploadToIPFS(metadata, `TimeCapsule-NFT-Metadata`);
}
```

> **Callers:** `uploadToIPFS` always takes a plain JS **object**, not a JSON string. The function serialises it internally via `pinataContent`. The return value of `packageEncryptedBlob` is a JSON string — parse it with `JSON.parse` before passing to `uploadToIPFS`.

**Error Handling:** All functions must `try/catch` and throw descriptive errors. Never let an IPFS failure silently fail.

> **⚠️ CORS Configuration Required:** Pinata blocks browser requests from unlisted origins. Before the frontend will be able to upload to IPFS in production, you **must** whitelist your Vercel deployment URL in Pinata's dashboard under **API Keys → Authorized Origins**. Without this step, all `uploadToIPFS` calls will fail with a CORS error in the browser. Add `http://localhost:*` for local development and your `https://*.vercel.app` URL for production. This step must be completed before any end-to-end testing in the browser.

---

## 9. Module 4 — Frontend

> **Goal:** Build a complete, functional single-page frontend. No frameworks. Vanilla HTML/CSS/JS. It must connect to MetaMask, allow creating and opening capsules, and display a dashboard.

---

### Design System

Before writing a single line of CSS, internalize these design tokens. Everything in the UI derives from them.

#### Color Palette

```css
:root {
  /* Backgrounds */
  --bg-base:        #0a0a0f;   /* Page background — deep space black */
  --bg-surface:     #12121a;   /* Card / panel background */
  --bg-elevated:    #1a1a2e;   /* Modal, dropdown background */
  --bg-input:       #0f0f1a;   /* Form input background */

  /* Brand */
  --indigo:         #4f46e5;   /* Primary CTA color */
  --indigo-hover:   #6366f1;   /* Hover state */
  --indigo-dim:     #312e81;   /* Muted / disabled */
  --cyan:           #06b6d4;   /* Accent, links, highlights */
  --cyan-dim:       #164e63;   /* Muted cyan background tint */

  /* Status */
  --success:        #10b981;   /* Green — opened capsule, success toasts */
  --warning:        #f59e0b;   /* Amber — expiring soon, warnings */
  --error:          #ef4444;   /* Red — error toasts, validation */
  --info:           #3b82f6;   /* Blue — info toasts */

  /* Text */
  --text-primary:   #f8fafc;   /* Headings, important labels */
  --text-secondary: #94a3b8;   /* Subtext, helper text, placeholders */
  --text-muted:     #475569;   /* Very dim text, disabled states */

  /* Borders */
  --border-subtle:  rgba(255,255,255,0.06);  /* Card borders */
  --border-active:  rgba(79,70,229,0.5);     /* Focused input borders */
  --border-glow:    rgba(6,182,212,0.3);     /* Glowing borders on hover */

  /* Effects */
  --glass-bg:       rgba(18,18,26,0.7);
  --glass-blur:     blur(16px);
  --shadow-card:    0 4px 24px rgba(0,0,0,0.4);
  --shadow-glow:    0 0 24px rgba(79,70,229,0.25);
}
```

#### Typography

```
Headings  → Space Grotesk (Google Fonts) — weights 500, 600, 700
Body text → Inter (Google Fonts) — weights 400, 500
Monospace → 'Courier New', monospace — used for wallet addresses, CIDs, keys
```

Font scale:
```
--text-xs:   0.75rem   (12px) — badges, timestamps, char counters
--text-sm:   0.875rem  (14px) — helper text, labels
--text-base: 1rem      (16px) — body copy
--text-lg:   1.125rem  (18px) — card titles
--text-xl:   1.25rem   (20px) — section subheadings
--text-2xl:  1.5rem    (24px) — page headings
--text-4xl:  2.25rem   (36px) — hero headline
--text-6xl:  3.75rem   (60px) — hero display (desktop)
```

#### Spacing & Layout

- Max content width: `1200px`, centered with `auto` margins
- Page horizontal padding: `1.5rem` mobile, `2rem` desktop
- Section vertical padding: `5rem` top/bottom
- Card border-radius: `1rem` (16px)
- Button border-radius: `0.5rem` (8px) standard, `9999px` (pill) for wallet button

---

### Visual Components

#### Glassmorphism Card

The core UI building block. Used for feature cards, capsule cards, and form panels.

```
Background:  var(--glass-bg) with backdrop-filter: var(--glass-blur)
Border:      1px solid var(--border-subtle)
Border-radius: 1rem
Box-shadow:  var(--shadow-card)
Padding:     1.5rem (24px)
Transition:  border-color 0.2s ease, box-shadow 0.2s ease

On hover:
  border-color: var(--border-glow)
  box-shadow:   var(--shadow-glow)
```

#### Buttons

**Primary button** (used for main CTA like "Create Capsule", "Open Capsule"):
```
Background:   linear-gradient(135deg, var(--indigo) 0%, var(--cyan) 100%)
Text color:   white, font-weight: 600
Padding:      0.75rem 1.75rem
Border-radius: 0.5rem
Box-shadow:   0 0 20px rgba(79,70,229,0.35)
On hover:     translateY(-1px), shadow intensifies
On active:    translateY(0)
```

**Ghost button** (secondary actions):
```
Background:   transparent
Border:       1px solid var(--border-subtle)
Text:         var(--text-secondary)
On hover:     border-color: var(--border-active), text: var(--text-primary)
```

**Wallet pill button** (navbar):
```
Shape:        pill (border-radius: 9999px)
Background:   var(--bg-elevated)
Border:       1px solid var(--border-active)
Text:         var(--cyan)
When connected: shows truncated address (0x1234...abcd)
```

#### Status Badges

```
Locked 🔒:   background: rgba(245,158,11,0.1), color: var(--warning), border: 1px solid rgba(245,158,11,0.3)
Opened 🔓:   background: rgba(16,185,129,0.1), color: var(--success), border: 1px solid rgba(16,185,129,0.3)
Font-size:   var(--text-xs), font-weight: 600, padding: 0.25rem 0.625rem, border-radius: 9999px
```

#### Toast Notifications

Appear in the bottom-right corner, slide in from the right, auto-dismiss after 4 seconds.

```
Width:        320px max
Border-radius: 0.625rem
Padding:      1rem 1.25rem
Border-left:  4px solid (color matches type)
Background:   var(--bg-elevated)
Box-shadow:   var(--shadow-card)
Animation:    slideInRight 0.3s ease → wait 3.7s → fadeOut 0.3s ease

Types:
  success  → border-left: var(--success),  icon: ✅
  error    → border-left: var(--error),    icon: ❌
  info     → border-left: var(--info),     icon: ℹ️
  warning  → border-left: var(--warning),  icon: ⚠️
```

---

### Page-by-Page Breakdown

---

#### Navbar (`<nav id="navbar">`)

Fixed to the top. Always visible. Glassmorphism background.

```
Layout:       flex, space-between, align-center
Height:       64px
Background:   var(--glass-bg), backdrop-filter: var(--glass-blur)
Border-bottom: 1px solid var(--border-subtle)
Z-index:      100

Left side:
  - Logo SVG (hourglass or capsule icon) + wordmark "TimeCapsule"
  - Color: gradient text (indigo → cyan)

Center (desktop only):
  - Nav links: Home | Create | Dashboard
  - Active link: color var(--cyan), border-bottom: 2px solid var(--cyan)
  - Inactive: var(--text-secondary), hover: var(--text-primary)

Right side:
  - Wallet pill button
  - When disconnected: "Connect Wallet" (ghost style, pulsing glow)
  - When connected:    "0x1234...abcd" (cyan text, green dot indicator)

Mobile (< 768px):
  - Hide center nav links
  - Show hamburger icon → slides down a menu with the same links
```

---

#### Landing Page (`#page-home`)

**Section 1 — Hero**

```
Layout:       centered, full viewport height (100vh), flex column
Padding-top:  80px (offset for fixed navbar)

Visual elements (top to bottom):
  1. Pill label:  "Powered by Ethereum · Sepolia Testnet"
                  Small pill badge, indigo border, space-grotesk, all caps
  2. Headline:    "Seal Your Memories
                   in Time."
                  Font: Space Grotesk, var(--text-6xl) desktop / var(--text-4xl) mobile
                  First line: var(--text-primary)
                  Second line: gradient text (indigo → cyan)
  3. Subheading:  "Lock messages into the blockchain. Encrypted. Time-locked. Forever."
                  Font: Inter, var(--text-lg), var(--text-secondary)
                  Max-width: 540px, centered
  4. CTA row:     [Create Capsule →]  (primary button, large)
                  [View Dashboard]    (ghost button, large)
                  Gap: 1rem, centered, flex-wrap on mobile
  5. Scroll hint: small animated chevron pointing down

Background:
  - Radial gradient from center: indigo at 0% opacity → transparent
  - Very subtle animated "star field" (CSS box-shadows, no JS needed)
  - Optional: a large blurred indigo circle (200px, 15% opacity) top-right
```

**Section 2 — Features Grid**

```
3-column grid (desktop) → 1-column (mobile)
Gap: 1.5rem
Max-width: 900px, centered

Card 1 — Time-Locked
  Icon:    🕐 or an hourglass SVG (48px, cyan)
  Title:   "Time-Locked by Smart Contract"
  Body:    "The blockchain enforces your unlock date.
            Nobody — not even us — can open it early."

Card 2 — Encrypted
  Icon:    🔐 or shield SVG (48px, indigo)
  Title:   "Encrypted in Your Browser"
  Body:    "Your message is encrypted client-side before it
            ever leaves your device. AES-256-GCM."

Card 3 — NFT Collectible
  Icon:    💎 or diamond SVG (48px, gradient)
  Title:   "Minted as an NFT"
  Body:    "Every capsule becomes an ERC-721 token.
            View it in MetaMask, OpenSea, or any wallet."

Each card uses the Glassmorphism Card component.
Icon sits above the title; title is var(--text-lg), font-weight 600.
```

**Section 3 — Stats Bar**

```
Full-width strip, background: var(--bg-surface)
Border-top + border-bottom: 1px solid var(--border-subtle)
Padding: 2rem 0
Layout: 3 stats, centered, flex, gap 4rem

Stat 1:  [Number fetched from contract]
         label: "Capsules Created"
Stat 2:  "Free"
         label: "Always Free (Testnet)"
Stat 3:  "AES-256"
         label: "Encryption Standard"

Number style: var(--text-4xl), Space Grotesk, gradient text (indigo→cyan)
Label style:  var(--text-sm), var(--text-secondary), uppercase, letter-spacing: 0.1em
```

---

#### Create Capsule Page (`#page-create`)

A 3-step wizard. Only one step is visible at a time. A progress bar at the top shows current position.

**Step Indicator**

```
Layout:   3 numbered circles connected by lines, full-width
Active:   filled circle (indigo), label below in var(--text-primary)
Done:     filled circle (cyan) with a ✓ checkmark
Upcoming: empty circle (border: var(--border-subtle)), label in var(--text-muted)
Line:     thin horizontal line connecting circles; fills (indigo) as steps complete
```

**Step 1 — Capsule Details**

```
Form card (Glassmorphism), max-width: 640px, centered

Fields (top to bottom):
  1. Title
     Label: "Capsule Title"
     Input: text, placeholder "e.g. Letter to Future Me"
     Helper: "Public — visible on the blockchain and your NFT"
     Max: 100 chars — show live counter "43 / 100" bottom-right

  2. Recipient Wallet Address
     Label: "Recipient Address"
     Input: text, monospace font, placeholder "0x..."
     Helper: "The only wallet that can open this capsule"
     Validation: turns border red + shows error if not valid ETH address
     Shortcut: small "[Use my address]" link fills in connected wallet

  3. Unlock Date
     Label: "Unlock Date & Time"
     Input: datetime-local, min = tomorrow 00:00
     Helper: "The capsule cannot be opened before this date"
     Below input: human-readable preview "Opens in 2 years, 3 months"

  4. Message
     Label: "Your Message"
     Textarea: min-height 160px, resizable vertically only
     Placeholder: "Write anything — a letter, a memory, a confession..."
     Helper: "Encrypted before leaving your browser. Max 10,000 characters."
     Live char counter bottom-right "842 / 10,000"

Bottom:
  [← Back] (ghost, disabled on step 1)   [Next: Preview →] (primary)
```

**Step 2 — Encryption Preview**

```
Centered content, max-width: 560px

Top:  Large shield icon (64px, gradient) with a subtle pulse animation

Title: "Ready to Encrypt"
Body:  "Your message will be encrypted using AES-256-GCM directly in your browser
        before being uploaded to IPFS. The encryption key will be stored
        alongside the capsule on-chain."

Key preview box (monospace, dark bg, bordered):
  "Key preview: 3f8a2c1d...  (stored on-chain)"
  ⚠️  small warning note: "This is an MVP — the key is publicly readable on-chain.
                           Don't use this for truly sensitive information."

Summary card:
  Title:      [capsule title]
  Recipient:  [truncated address]
  Unlocks:    [formatted date]
  Message:    [first 80 chars]...

Bottom:
  [← Back]   [Next: Confirm →]
```

**Step 3 — Confirm & Submit**

```
Centered content, max-width: 560px

Full summary card:
  Row: 📝 Title       → [value]
  Row: 👤 Recipient   → [0x...abcd]
  Row: 📅 Unlocks     → [March 15, 2027 · 10:00 AM]
  Row: 🔐 Encryption  → AES-256-GCM
  Row: 📦 Storage     → IPFS via Pinata

Gas estimate note:
  "ℹ️ This will require a MetaMask transaction (~0.001–0.005 ETH in gas on Sepolia)"

[Create Capsule 🚀] primary button, full-width

Loading state (after click, before tx confirms):
  Button disabled + spinner
  Status text below: "Encrypting message..." → "Uploading to IPFS..." → "Awaiting wallet..." → "Confirming..."
  Each step has its own sub-status message

Success state (after confirmation):
  Replaces form entirely:
  Large ✅ icon
  "Your capsule has been sealed!"
  Capsule ID: #42 (large, monospace, cyan)
  Etherscan link: "View transaction →" (external link)
  [View Dashboard] button
```

---

#### Dashboard Page (`#page-dashboard`)

```
Two-tab layout

Tab bar:
  [📤 Created by Me]   [📥 Addressed to Me]
  Active tab: bottom border in var(--cyan), text: var(--text-primary)
  Inactive:   text: var(--text-secondary)

Under each tab: a card grid
  Desktop: 3-column grid
  Tablet:  2-column grid
  Mobile:  1-column
  Gap:     1.25rem
```

**Capsule Card** (the key repeating unit)

```
Glassmorphism Card, padding: 1.25rem

Top row:
  Left:  Status badge (Locked 🔒 / Opened 🔓)
  Right: Capsule ID in small monospace text (#42)

Title: var(--text-lg), font-weight: 600, margin-top: 0.5rem

Recipient / Creator row:
  Icon + truncated address (0x1234...abcd)
  Click address to open Etherscan

Countdown or opened label:
  If locked:  Digital countdown timer:
              "03d 14h 22m 09s"
              Large monospace font, var(--cyan), updates every second
              Below: "Unlocks [date]"
  If opened:  "Opened on [date]" in var(--success)

Bottom row:
  [View Details] ghost button → opens the capsule modal
  If: user is recipient AND capsule is unlocked → shows [Open Capsule] primary button
```

**Empty State**

```
When no capsules in a tab:
  Centered illustration (inline SVG hourglass, ~120px, dim indigo)
  Title: "No capsules here yet"
  Body: "Create your first time capsule to get started."
  [Create Capsule →] primary button
```

---

#### Open Capsule Modal (`#modal-open-capsule`)

Triggered when the user clicks "View Details" or "Open Capsule" on a card.

```
Overlay: full-screen dark backdrop (rgba(0,0,0,0.7)), blur(4px) on background
Modal:   centered, max-width: 560px, max-height: 85vh, overflow-y: auto
         Glassmorphism, border: 1px solid var(--border-subtle)

Header row:
  Left:  "Capsule #42"  (title)
  Right: ✕ close button

Content rows:
  Status badge
  Title
  Creator → Recipient (with right-arrow icon between)
  Unlock date
  IPFS CID (monospace, truncated, copy-to-clipboard button)

--- DIVIDER ---

If LOCKED:
  Large 🔒 icon (64px, amber)
  "Sealed until [date]"
  Countdown:  "03 days  14 hours  22 minutes  09 seconds"
              4 boxes with labels underneath, monospace, var(--warning)
  Greyed out "Open Capsule" button with tooltip "Not unlocked yet"

If UNLOCKED & recipient & not opened:
  Large ⏰ icon (64px, cyan, subtle pulse)
  "This capsule is ready to open!"
  [Open Capsule] primary button, full-width
  Loading state: button becomes spinner + "Sending transaction..."
                 then "Fetching from IPFS..." → "Decrypting..."

If OPENED (shows decrypted message):
  Header: "Message from [creator address]"
  Message box:
    Background: var(--bg-input)
    Border: 1px solid var(--border-subtle)
    Border-radius: 0.5rem
    Padding: 1.25rem
    Font: Inter, var(--text-base), var(--text-primary)
    White-space: pre-wrap (preserves line breaks)
    Max-height: 300px, overflow-y: auto
  Below: "Decrypted locally in your browser using AES-256-GCM"
  NFT note: "Your NFT (#42) is now transferable in any compatible wallet."
```

---

### Responsive Behavior Summary

```
Breakpoint at 768px (mobile-first approach)

Mobile  (< 768px):
  - Navbar: hamburger menu, no center links
  - Hero: text-4xl headline, single-column CTA
  - Features: 1-column grid
  - Dashboard: 1-column card grid
  - Modal: full-screen (no rounded top corners, slides up from bottom)
  - Create form: full-width, no max-width constraint

Desktop (≥ 768px):
  - Navbar: full horizontal links
  - Hero: text-6xl headline, side-by-side CTA buttons
  - Features: 3-column grid
  - Dashboard: 3-column card grid
  - Modal: centered floating panel
```

---

### Animation Inventory

All animations must be subtle — this is a minimal, atmospheric UI, not a game.

```
Page transitions:    fade in (opacity 0→1, 0.25s ease) on navigateTo()
Card hover:          translateY(-2px), 0.2s ease — slight lift
CTA button hover:    translateY(-1px) + shadow intensify, 0.15s ease
Toast slide-in:      translateX(360px→0), 0.3s ease
Toast fade-out:      opacity 1→0, 0.3s ease (at 3.7s after appearing)
Step indicator fill: width 0→100%, 0.4s ease (progress bar line)
Countdown timer:     no animation — just numeric update every 1000ms
Wallet button pulse: box-shadow pulsing (keyframe), 2s loop — only when disconnected
Loading spinner:     rotate 360deg, 0.7s linear infinite
```

---

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
    <!-- Hero section -->
    <!-- Features grid -->
    <!-- Stats bar -->
  </section>

  <!-- PAGE: CREATE CAPSULE (#page-create) -->
  <section id="page-create" class="page hidden">
    <!-- Step indicator -->
    <!-- Step 1: Capsule Details form -->
    <!-- Step 2: Encryption Preview -->
    <!-- Step 3: Confirm & Submit -->
  </section>

  <!-- PAGE: DASHBOARD (#page-dashboard) -->
  <section id="page-dashboard" class="page hidden">
    <!-- Tab bar: "Created by Me" | "Addressed to Me" -->
    <!-- Capsule card grid (rendered by ui.js) -->
    <!-- Empty state (shown when no capsules) -->
  </section>

  <!-- MODAL: OPEN CAPSULE -->
  <div id="modal-overlay" class="modal-overlay hidden">
    <div id="modal-open-capsule" class="modal">
      <!-- Capsule details, countdown or decrypted message -->
    </div>
  </div>

  <!-- TOAST CONTAINER -->
  <div id="toast-container"></div>

  <!-- Scripts (type="module") -->
  <script type="module" src="js/app.js"></script>
</body>
</html>
```

### `frontend/js/config.js`

```javascript
// This file is auto-updated by the deploy script and scripts/update-abi.js.
// Do not edit manually.

export const NETWORK_CHAIN_ID = 11155111; // Sepolia
export const NETWORK_NAME = "Sepolia";

export const TIMECAPSULE_ADDRESS = "REPLACE_AFTER_DEPLOY";
export const TIMECAPSULE_NFT_ADDRESS = "REPLACE_AFTER_DEPLOY";

// Pinata JWT — NOTE: In production, this should be a read-only gateway key.
// For MVP/testnet this is acceptable. You MUST whitelist your Vercel domain
// in Pinata's "Authorized Origins" settings, or all uploads will fail with CORS errors.
export const PINATA_JWT = "REPLACE_WITH_PINATA_JWT";

// These arrays are populated automatically by `npm run compile` (via postcompile → update-abi.js).
// NEVER deploy with empty arrays — the frontend will fail to call any contract function.
// If empty, run `npm run compile` or `npm run update-abi` first.
export const TIMECAPSULE_ABI = []; // auto-filled by scripts/update-abi.js
export const TIMECAPSULE_NFT_ABI = []; // auto-filled by scripts/update-abi.js
```

> **AI Instruction:** After `npm run compile`, the `postcompile` hook will automatically call `scripts/update-abi.js`, which reads ABIs from `artifacts/` and writes them into `config.js`. Verify the arrays are non-empty before proceeding to frontend deployment. If they remain empty, run `npm run update-abi` manually.

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
 * Implementation:
 *   1. Call wallet_switchEthereumChain with chainId "0xaa36a7"
 *   2. If MetaMask throws error code 4902 (chain not added), call wallet_addEthereumChain with:
 *      { chainId: "0xaa36a7", chainName: "Sepolia", rpcUrls: ["https://rpc.ankr.com/eth_sepolia"],
 *        nativeCurrency: { name: "ETH", symbol: "ETH", decimals: 18 },
 *        blockExplorerUrls: ["https://sepolia.etherscan.io"] }
 *   3. Re-throw any other errors.
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
 * Update the status text below the submit button during multi-phase operations.
 * @param {string} message - Status message to display, e.g. "Encrypting message..."
 *                           Pass null or "" to hide the status line.
 */
export function setStatus(message) { ... }

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
import { showToast, navigateTo, updateWalletButton, renderCapsuleCard, renderCountdown, updateStepIndicator, setButtonLoading, setStatus } from "./ui.js";

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
  // 1. setStatus("Encrypting message...") → Generate AES key → Encrypt message
  // 2. Package encrypted blob (ciphertext + iv → JSON object)
  // 3. setStatus("Uploading to IPFS...") → Upload encrypted blob to IPFS → get contentCID
  // 4. Build NFT metadata object using CAPSULE_IMAGE_CID from config.js → upload to IPFS → get metadataCID
  // 5. setStatus("Awaiting wallet...") → Call createCapsule() on contract (passes contentCID, encryptedKeyHex, metadataCID)
  // 6. setStatus("Confirming...") → Wait for receipt; check receipt.status === 1
  // 7. setStatus(null) → Show success state with capsule ID + Etherscan link
  // On any error: setStatus(null), setButtonLoading(btn, false), showToast(error.message, "error")
}

// === Event: Open Capsule (from modal) ===
async function handleOpenCapsule(capsuleId) {
  // 1. Call openCapsule() on contract → mark as opened
  // 2. Check receipt.status === 1; if not, show error toast and STOP — do not proceed to decrypt
  // 3. Fetch encryptedKey from capsule data (call getCapsule() after confirmed tx)
  // 4. Reconstruct CryptoKey from hex
  // 5. Fetch blob from IPFS
  // 6. Unpackage blob → decrypt
  // 7. Display plaintext message in modal
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

Implement the full stylesheet using the design tokens from the Design System section above. The file must cover:

- CSS custom properties (all variables from the `:root` block above)
- CSS reset / base styles (`*, box-sizing: border-box`, `body` background, font)
- Gradient text utility class (`.gradient-text`)
- Navbar — fixed, glassmorphism, hamburger on mobile
- Page system — `.page { display: none }`, `.page.active { display: block }`, fade-in animation
- Hero section — centered flex column, star-field background, pill badge, headline, CTA row
- Features grid — 3-column → 1-column
- Stats bar — 3-column flex strip
- Glassmorphism card — `.card` class used everywhere
- Button variants — `.btn-primary`, `.btn-ghost`
- Status badges — `.badge-locked`, `.badge-opened`
- Create form — step indicator, input styles, char counter, validation error states
- Dashboard — tab bar, capsule card grid, empty state
- Modal — overlay, modal panel, countdown boxes, message display box
- Toast notifications — slide-in/fade-out keyframes, 4 type variants
- Loading spinner — `.spinner` with rotate keyframe
- Responsive breakpoints — mobile-first, single breakpoint at `768px`

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
  // Regex matches any quoted string value — safe on first deploy and on re-deploys
  configContent = configContent
    .replace(/TIMECAPSULE_ADDRESS\s*=\s*"[^"]*"/, `TIMECAPSULE_ADDRESS = "${timeCapsuleAddress}"`)
    .replace(/TIMECAPSULE_NFT_ADDRESS\s*=\s*"[^"]*"/, `TIMECAPSULE_NFT_ADDRESS = "${timeCapsuleNFTAddress}"`);
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

### ABI Auto-Population Script (`scripts/update-abi.js`)

This script is run automatically as part of `deploy.js` (step 6 below) and can also be run standalone via `npm run update-abi`. It reads compiled ABIs from Hardhat artifacts and writes them into `frontend/js/config.js`.

```javascript
const fs = require("fs");
const path = require("path");

function updateABIs() {
  const configPath = path.join(__dirname, "../frontend/js/config.js");

  const timeCapsuleArtifact = require("../artifacts/contracts/TimeCapsule.sol/TimeCapsule.json");
  const nftArtifact = require("../artifacts/contracts/TimeCapsuleNFT.sol/TimeCapsuleNFT.json");

  let configContent = fs.readFileSync(configPath, "utf8");

  // Replace the ABI arrays — matches from `= [` to the closing `];` on the same export line
  configContent = configContent.replace(
    /export const TIMECAPSULE_ABI\s*=\s*\[[\s\S]*?\];/,
    `export const TIMECAPSULE_ABI = ${JSON.stringify(timeCapsuleArtifact.abi, null, 2)};`
  );
  configContent = configContent.replace(
    /export const TIMECAPSULE_NFT_ABI\s*=\s*\[[\s\S]*?\];/,
    `export const TIMECAPSULE_NFT_ABI = ${JSON.stringify(nftArtifact.abi, null, 2)};`
  );

  fs.writeFileSync(configPath, configContent);
  console.log("✅ ABIs written to frontend/js/config.js");
}

module.exports = { updateABIs };

if (require.main === module) {
  updateABIs();
}
```

Add to `package.json` scripts:
```json
"update-abi": "node scripts/update-abi.js",
"postcompile": "node scripts/update-abi.js"
```

> **Note:** `postcompile` runs `update-abi.js` automatically after every `npm run compile`. The deploy script must also call `updateABIs()` (imported from this file) after compilation, before updating addresses.

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
    needs: [test]   # Avoids a duplicate cold npm ci install on every run

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

## 12. Coding Standards & Conventions

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
- Never commit: `.env`, `artifacts/`, `cache/`, `node_modules/`

---

## 13. Known Constraints

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

## 14. Final Checklist

Before marking the project complete, verify every item:

### Contracts
- [ ] `TimeCapsule.sol` compiles with 0 errors, 0 warnings
- [ ] `TimeCapsuleNFT.sol` compiles with 0 errors, 0 warnings
- [ ] All custom errors defined and used
- [ ] All events emitted correctly
- [ ] `npm test` — all tests pass
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
- [ ] `frontend/js/config.js` has correct addresses and ABIs after deploy (verify ABI arrays are non-empty)
- [ ] `frontend/js/config.js` has `CAPSULE_IMAGE_CID` set to a valid pinned Pinata CID
- [ ] Pinata "Authorized Origins" includes your Vercel URL and `http://localhost:*` (required for CORS)
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

*End of SPEC.md — Total Modules: 7 (0–6). Build in order. Test before proceeding. Ship it.*

