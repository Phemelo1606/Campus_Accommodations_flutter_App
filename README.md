# Smart Student Accommodation — README

Quick reference for setting up and running the app (Firebase + IPFS + Blockchain).

## Overview
This Flutter app uses:
- Firebase (Auth, Firestore, Storage, FCM)
- IPFS (store lease PDFs)
- Blockchain (smart contract `LeaseAgreement.sol` for lease records)
- PDF generation + IPFS upload + contract interactions from chat UI

Files of interest
- `lib/main.dart` — app bootstrap (Firebase init + routes)
- `lib/core/routes.dart` — navigation
- `lib/services/*` — blockchain_service, ipfs_service, pdf_service, chatService, applicationService
- `lib/pages/chatScreen.dart` — lease creation, IPFS upload and blockchain flow
- `lease/LeaseAgreement.sol` — Solidity smart contract

---

## Prerequisites
- Flutter (stable) installed and on PATH
- Dart SDK (comes with Flutter)
- Node.js & npm (for Hardhat/Truffle deployment)
- Local blockchain (Ganache) or a testnet provider (Alchemy/Infura)
- IPFS node or IPFS pinning service account
- Firebase project (Console) access
- PowerShell (Windows) or terminal in VS Code

---

## 1) Firebase setup
1. Create a Firebase project in Firebase Console.
2. Enable:
   - Authentication (Email/Password)
   - Firestore (in production set rules)
   - Storage (for optional uploads)
   - Cloud Messaging (optional)
3. Register your app(s) and obtain config. The repo already contains `firebase_options.dart`. If regenerating:
   - Install FlutterFire CLI:
     ```
     dart pub global activate flutterfire_cli
     ```
   - Run from project root:
     ```
     flutterfire configure
     ```
   - This generates/updates `lib/firebase_options.dart`.

4. Firestore collections used by app (examples)
   - `chats/{chatId}/messages/{messageId}`: senderId, text/type, timestamp, leaseData, ipfsHash, transactionHash, leaseStatus
   - `chats/{chatId}`: participants[], lastMessageTimestamp, studentName, landlordName
   - `Accommodations/{accId}/Rooms/{roomId}`: RoomNumber, MonthlyRent, AvailabilityStatus, Images...
   - `StudentRooms/{studentId}`: RoomID, AccommodationID, LeaseDocumentId, IsActive
   - Add any additional collections used by services (applications, maintenance, users)

5. Recommended Firestore rules (development)
   - Start permissive for dev then harden before production.
   - Use Firebase Auth UID checks to restrict writes to own documents.

---

## 2) IPFS setup
Options:
- Local IPFS node: install go-ipfs (https://docs.ipfs.io/install/)
  - Start daemon:
    ```
    ipfs init
    ipfs daemon
    ```
- Hosted pinning / storage (recommended for production):
  - web3.storage, Infura IPFS, Pinata — create API key.

Configure env variables (see section Env). Example variables:
- IPFS_ENDPOINT (RPC/http endpoint)
- IPFS_API_KEY or WEB3_STORAGE_API_KEY

The app uses `lib/services/ipfs_service.dart` — ensure the service matches chosen provider API.

---

## 3) Blockchain setup (development)
1. Choose environment:
   - Local: Ganache (RPC at http://127.0.0.1:7545)
   - Testnet: Goerli/Scroll via Alchemy/Infura
2. Compile & deploy `lease/LeaseAgreement.sol` with Hardhat or Truffle:
   - Hardhat quick steps:
     ```
     npm init -y
     npm install --save-dev hardhat @nomiclabs/hardhat-ethers ethers
     npx hardhat
     # create deploy script to deploy LeaseAgreement.sol
     npx hardhat run scripts/deploy.js --network <network>
     ```
3. Note deployed contract address and ABI. Update config used by `lib/services/blockchain_service.dart` or `lib/config/app_config.dart`.

Env variables for blockchain:
- RPC_URL (e.g., http://127.0.0.1:7545 or https://eth-goerli.alchemyapi.io/v2/...)
- CONTRACT_ADDRESS (deployed LeaseAgreement address)
- LANDLORD_PRIVATE_KEY
- STUDENT_PRIVATE_KEY (for test signing flows)
- CHAIN_ID (optional for tx building)

Warning: Never commit real private keys. Use test keys in dev.

---

## 4) Environment variables (.env)
Create a `.env` or `lib/config/app_config.dart` entries. Example .env (store outside repo or in `.env.example`):
`````
