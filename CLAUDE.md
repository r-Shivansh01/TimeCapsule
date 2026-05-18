# CLAUDE.md — TimeCapsule Project

> This file is the canonical context document for any AI agent (Claude Code or equivalent) working on this codebase.
> Read this entire file before writing a single line of code.

---

## Project Overview

**TimeCapsule** is a decentralized application (dApp) on Ethereum that lets users lock encrypted messages into a smart contract until a future unlock date. Each capsule is minted as an ERC-721 NFT. The stack is entirely free-tier: Sepolia testnet, Vercel, Pinata, GitHub Actions.

**Core flow:**
1. User connects MetaMask → writes message + sets unlock date + recipient address
2. Message encrypted client-side (AES-GCM via Web Crypto API)
3. Encrypted blob uploaded to IPFS via Pinata
4. CID + timestamp + recipient sent to smart contract → NFT minted
5. On/after unlock date, recipient calls `openCapsule()` → frontend fetches + decrypts

---

## Environment Setup (Codespace / Local)

This project is developed inside a **GitHub Codespace** with Ollama routing Claude Code to a local `qwen2.5-coder:3b` model (free, no Anthropic billing).

### Required env vars (set in shell before running Claude Code):
```bash
export ANTHROPIC_BASE_URL=http://localhost:11434/v1
export ANTHROPIC_API_KEY=ollama   # dummy value; Ollama ignores it
```

### Required `.env` file (never commit):
```
PRIVATE_KEY=your_wallet_private_key
SEPOLIA_RPC_URL=https://eth-sepolia.g.alchemy.com/v2/YOUR_KEY
ETHERSCAN_API_KEY=your_etherscan_key
PINATA_API_KEY=your_pinata_api_key
PINATA_SECRET_API_KEY=your_pinata_secret
PINATA_JWT=your_pinata_jwt
TIMECAPSULE_ADDRESS=        # populated after deploy
TIMECAPSULE_NFT_ADDRESS=    # populated after deploy
```

---

## Tech Stack

| Layer | Tool | Version |
|---|---|---|
| Contracts | Solidity | ^0.8.20 |
| Dev framework | Hardhat | ^2.22.x |
| Contract libs | OpenZeppelin | ^5.x |
| JS ↔ contract | Ethers.js | v6.x |
| Frontend | Vanilla HTML5 / CSS3 / ES6 | — |
| Wallet | MetaMask (`window.ethereum`) | — |
| Encryption | Web Crypto API (AES-GCM) | browser-native |
| IPFS | Pinata SDK / REST | free 1GB tier |
| Testnet | Sepolia | — |
| RPC | Alchemy or Infura | free tier |
| Hosting | Vercel | free tier |
| CI | GitHub Actions | free tier |

---

## Directory Structure

```
timecapsule/
├── .github/workflows/ci.yml
├── contracts/
│   ├── TimeCapsule.sol          # Core contract
│   └── TimeCapsuleNFT.sol       # ERC-721 extension
├── scripts/
│   ├── deploy.js
│   ├── seed.js
│   └── update-abi.js            # Auto-populates ABIs into config.js
├── test/
│   ├── TimeCapsule.test.js
│   └── TimeCapsuleNFT.test.js
├── frontend/
│   ├── index.html
│   ├── css/styles.css
│   └── js/
│       ├── config.js            # Addresses + ABIs (auto-populated)
│       ├── wallet.js
│       ├── encrypt.js
│       ├── ipfs.js
│       ├── contract.js
│       ├── ui.js
│       └── app.js
├── deployments/sepolia.json     # Auto-generated after deploy
├── .env                         # Secret — NEVER commit
├── .env.example
├── hardhat.config.js
├── package.json
└── vercel.json
```

---

## Build Order — Follow Strictly

Work through modules in this order. **Run tests and confirm they pass before moving to the next module.**

| Module | Goal | Done? |
|---|---|---|
| Module 0 | Repo init, all deps installed, `npx hardhat compile` succeeds | [ ] |
| Module 1 | `TimeCapsule.sol` — core contract, unit tests pass | [ ] |
| Module 2 | `TimeCapsuleNFT.sol` — ERC-721 extension, unit tests pass | [ ] |
| Module 3 | Encryption (`encrypt.js`) + IPFS (`ipfs.js`) layer | [ ] |
| Module 4 | Frontend — full UI wired up | [ ] |
| Module 5 | Deploy scripts + `deployments/sepolia.json` generated | [ ] |
| Module 6 | CI/CD — GitHub Actions pipeline green | [ ] |

---

## Key Constraints

| Constraint | Rule |
|---|---|
| Free tools only | No paid APIs, RPC, or hosting at any point |
| No backend | Fully client-side + smart contract. Zero Node.js server |
| Sepolia only | Mainnet deployment is out of scope |
| Vanilla JS | No React, Vue, or any JS framework in frontend |
| OZ v5 | `Counters.sol` removed — use manual `uint256` counter |
| Ethers.js v6 | `ethers.utils` is gone. Use top-level `ethers.*`. BigNumber → native `BigInt` |
| AES key storage | MVP: encryption key stored on-chain in plaintext. Known limitation — do not silently "fix" this |
| Text only | No binary file uploads in MVP; text messages only |

---

## Coding Standards

### Solidity
- Every `public`/`external` function needs a NatSpec `/// @notice` comment
- Use **custom errors** everywhere — no `require(condition, "string")`
- Emit an **event** for every state change
- State variables: `private` by default, expose via `view` functions
- No `payable` functions (no ETH in this version)
- Follow **Checks-Effects-Interactions** pattern strictly

### JavaScript
- ES6+ only: `const`, `let`, arrow functions, `async/await`, template literals
- No `var` anywhere
- All async operations wrapped in `try/catch`
- All user-facing errors surface via `showToast(error.message, "error")`
- No hardcoded addresses in JS — all imported from `config.js`
- No `console.log` in production code (scripts/tests are fine)

### Git
- Commit format: `type(scope): description`
  - Types: `feat`, `fix`, `test`, `chore`, `docs`, `refactor`
  - Example: `feat(contract): add soulbound transfer restriction to NFT`
- Branch naming: `feat/module-1-core-contract`, `fix/nft-transfer-bug`
- **Never commit**: `.env`, `artifacts/`, `cache/`, `node_modules/`

---

## npm Scripts Reference

```bash
npm run compile         # Hardhat compile (also runs update-abi.js via postcompile)
npm test                # Run all Hardhat tests
npm run test:coverage   # Solidity coverage report
npm run lint:sol        # Solhint
npm run lint:js         # ESLint
npm run deploy:sepolia  # Deploy to Sepolia testnet
npm run update-abi      # Manually sync ABIs into frontend/js/config.js
```

---

## Post-Deploy Checklist

After running the deploy script, verify:
- [ ] `deployments/sepolia.json` exists with contract addresses
- [ ] `frontend/js/config.js` has non-empty ABI arrays and correct addresses
- [ ] `CAPSULE_IMAGE_CID` in `config.js` points to a valid Pinata-pinned CID
- [ ] Pinata "Authorized Origins" includes your Vercel URL + `http://localhost:*`
- [ ] Both contracts verified on Sepolia Etherscan
- [ ] Frontend deployed to Vercel and accessible

---

## Known Limitations (do not silently work around these)

1. **AES key on-chain**: The encryption key is stored in the smart contract in plaintext. This is an MVP tradeoff — anyone who can read contract storage can decrypt the message. Do not attempt to fix this without explicit instruction; the SPEC acknowledges it.
2. **Text-only**: Binary file uploads are not in scope for MVP.
3. **Sepolia only**: No mainnet deployment logic should be added.

---

*Generated from SPECS.md — last updated May 2026*
