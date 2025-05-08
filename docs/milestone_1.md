# Milestone 1: Development and Blockchain Integration Plan

## **Project Overview**

**Project Name:** Kanine Klans  
**Description:** Mars Terrain Exploration and Racing Game  
**Category:** Web3 Gaming Platform  
**Milestone:** Pre-Alpha (Milestone 1)  
**Blockchain:** [Diamante](https://www.diamante.io/)  
**SDK Used:** [Diamante SDK](https://developers.diamante.io/)

## **Objective**

Milestone 1 focuses on developing the core game mechanics, integrating blockchain for secure transactions and asset management. To enhance user experience, blockchain interactions will be abstracted, ensuring seamless gameplay without requiring players to manually sign transactions.

---

## **Key Components**

### **1. Architecture Finalization**

We aim to create a hybrid system where the game operates traditionally while leveraging Diamante blockchain for asset management and transactions. The finalized architecture will include:

- **Game Client:** Built using Unity Game Engine.
- **Backend Server:** Node.js and Express for API handling.
- **Database:** PostgreSQL to store user details, progress and assets.
- **Blockchain Integration Layer:** Server-side integration of Diamante SDK to handle transactions securely.

### **2. Blockchain Integration Approach**

Since Diamante does not support smart contracts, we will:

- Use **Diamante SDK** to facilitate secure transactions.
- Implement a **server-managed wallet system** to handle in-game assets, reducing friction for players.
- Store game assets off-chain while periodically verifying balances and ownership on-chain.
- Develop **API endpoints** to abstract blockchain operations, so gamers interact seamlessly without direct blockchain knowledge.

### **3. Encryption & Access Control**

To ensure secure asset management, we will implement:

- **Token-Gated Features:** Access control mechanisms to validate ownership of in-game assets before allowing interactions. Entry to every new terrain, usage of new props and collectiable will be authenticated with the ownership of the nft. NFT acts as the pass.
- **Private Key Encryption:** Server-side wallets will be securely encrypted. Users will be encouraged to not use the wallet for personal use apart from the gaming. Users will be made aware that our server store the private key(to reduce game play friction) in encrptied form.
- **Assets as NFT:** Assets like Rover skins, props, wheels, etc can be tradable as nft.
- **Token Utility:** kkToken(KK) will be used for any sort of monetary and rewarding mechanism within the game as well as community engagement.

## **Technical Implementation**

### **Backend API Design**

| Endpoint                         | Method | Description                  |
| -------------------------------- | ------ | ---------------------------- |
| `/api/account/create`            | POST   | Authenticate users           |
| `/api/account/balances`          | GET    | Fetch user’s assets balances |
| `/api/asset/transfer_from_admin` | POST   | Mint in-game assets          |
| `/api/asset/get_metadata`        | POST   | Fetch Klan NFT metadata      |

### **API Details**

All the APIs are authenticated with jwt token. The token is passed as a Bearer token in Authentication header. The payload is as follows:

```json
{
  "playerEmail": "user_email@gmail.com"
}
```

Game client gets the email of the user after google oAuth authentication. New token is generated with the user's email. That token is passed with the request.

#### **1. `/api/account/create`**

- **Method:** POST
- **Description:** Creates a new account if one does not exist. A new diamante account is createad from random key pair if the given email does not exist in the database. The account is funded with 500 testnet diam tokens and 100 `kkToken`. Every new user is rewarded with kkToken as a early bird. Returns the user object from database if the user already exist for the corresponding email. User interacts with the game client. Game client interacts with diamante blockchain server.
- **Request Body:** No request body.
- **Response:**
  ```json
  {
    "success": true,
    "message": "Account already exists.",
    "data": {
      "player": {
        "id": 62,
        "publicKey": "GD742NKGLMW3EPQPQV3YFGUI2WSNFWJSA3KZ5DGIW7ZAJXV4U4XORJ2L",
        "privateKey": "SDIVGNBTLPBM6JHZTEZ6QSLGXY5YVPCSWUL3MJHFN4SOWAQB5TVWMCJZ",
        "playerEmail": "bharat@gmail.com",
        "createdAt": "2025-05-03T18:34:50.903Z"
      }
    }
  }
  ```
- **Usage:** To create new dimante account or retrive existing account for authenticated user.

---

#### **2. `/api/account/balances`**

- **Method:** GET
- **Description:** Retrieves the balance of in-game assets and tokens for the authenticated user.
- **Request:** No request body.
- **Response:**

  ```json
  {
    "success": true,
    "message": "Fetched all asset balances",
    "data": {
      "balances": [
        {
          "balance": "1.0000000",
          "limit": "922337203685.4775807",
          "buying_liabilities": "0.0000000",
          "selling_liabilities": "0.0000000",
          "last_modified_ledger": 3964055,
          "is_authorized": true,
          "is_authorized_to_maintain_liabilities": true,
          "asset_type": "credit_alphanum12",
          "asset_code": "testAsset1",
          "asset_issuer": "GDBAWRQ3XLRRX5DKQBDDWPXZH654LX7RYW7ENU5DFTCZHC55S5FUM6N6"
        },
        {
          "balance": "489.9999600",
          "buying_liabilities": "0.0000000",
          "selling_liabilities": "0.0000000",
          "asset_type": "native"
        }
      ],
      "wallet": "GAK3P3KCMXSBA2AANJZKPBJASFRCVUTW46BXPRKCDMIE2WGHSS4KDZ35"
    }
  }
  ```

- **Usages:** To show the list of assets and their balances while user tries to deposit or withdraw in and from the game account.

---

#### **3. `/api/asset/transfer_from_admin`**

- **Method:** POST
- **Description:** Transfers `amount` no. of assets to the authenticated user from admin account. Full supply of the assets are already minted to admin account from issuer account.
- **Request Body:**
  ```json
  {
    "assetName": "testAsset1",
    "amount": "1"
  }
  ```
- **Response:**
  ```json
  {
    "success": true,
    "message": "Transfer successful.",
    "data": {
      "assetName": "testAsset1",
      "from": "GD5YY3ZP7YSMECJHJPUPO2TICRAHDYSFJ6536BN4YJE76TBF3BZT2YA3",
      "to": "GAK3P3KCMXSBA2AANJZKPBJASFRCVUTW46BXPRKCDMIE2WGHSS4KDZ35",
      "amount": "1",
      "hash": "f6ac0fef41c19f02eb3d59b119855f1f70c8777e97d32f68784fbb9505966d44"
    }
  }
  ```
- **Usage:** To mint/transfer any asset to user's account. For example; transferring klan nft when they select any klan, transfer kk token as a reward, transfer skin nft, etc.

---

#### **4. `/api/asset/get_metadata`**

- **Method:** GET
- **Description:** Fetches metadata of all assets in admin account with valid metadata.
- **Request Body:** No request body.
- **Response:**

  ```json
  {
    "success": true,
    "message": "Metadata retrieved successfully.",
    "data": {
      "testAsset1": "Link to the actual metadata of testAsset1",
      "testAsset2": "Link to the actual metadata of testAsset2"
    }
  }
  ```

- **Usage:** To get the list of assets with metadata(nft) while klan selection, buying skins, powerups, etc in garage.

---

### **Game Flow**

1. User logs in via game client.
2. Server assigns or retrieves user’s wallet.
3. User chooses the klan they want to represent.
4. User goes to main hub.
5. User explorer the terrain.
6. User enters the race track.
7. User competes with the bots.

---

## **Expected Outcomes for Milestone 1**

✅ A functional **terrain exploration and racing game** with blockchain-integrated asset management.  
✅ **Secure, abstracted blockchain transactions** ensuring a smooth gaming experience.  
✅ **Seamless backend integration** with Diamante SDK for secure transactions.

_*For detailed API documentation [click here](https://documenter.getpostman.com/view/28209780/2sB2j1hCT4)!*_
