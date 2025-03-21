# ImPayQ: Privacy-Preserving Blockchain Coupon System

ImPayQ is a blockchain-based coupon system that uses zero-knowledge proofs to ensure privacy for users' purchase data while enabling merchants to issue and manage coupon programs. This system is designed for non-crypto users, using account abstraction and email-based authentication to eliminate the need for a dedicated mobile app.

## System Overview

![System Architecture](./ImPayQ.drawio)

ImPayQ consists of:

1. **User Web Portal** - Email-based authentication and coupon management
2. **Merchant App** - QR scanning and coupon program management
3. **Account Abstraction System** - Creates and manages wallets for users without crypto knowledge
4. **ZK Privacy Layer** - Generates and verifies zero-knowledge proofs
5. **Smart Contract System** - Handles on-chain operations

## Smart Contract Functions

### Coupon NFT Contract

```solidity
// Mint a new coupon token for a user
function mintCoupon(
    address user,           // Recipient's wallet address
    uint256 merchantId,     // ID of issuing merchant
    string memory metadata, // Coupon details (encrypted if needed)
    uint256 expiryDate,     // When coupon expires
    bytes memory zkProof    // ZK proof verifying purchase eligibility
) external returns (uint256 tokenId);

// Check if a coupon is valid for redemption
function isValidCoupon(uint256 tokenId) external view returns (bool);

// Redeem a coupon (mark as used)
function redeemCoupon(
    uint256 tokenId,
    bytes memory redemptionProof
) external;

// Get coupon metadata
function getCouponDetails(uint256 tokenId) external view returns (
    address merchant,
    string memory metadata,
    uint256 expiryDate,
    bool isRedeemed
);

// Get all coupons for a user
function getUserCoupons(address user) external view returns (uint256[] memory);

// Get all coupons issued by a merchant
function getMerchantCoupons(uint256 merchantId) external view returns (uint256[] memory);
```

### Merchant Registry Contract

```solidity
// Register a new merchant
function registerMerchant(
    string memory name,
    string memory details,
    address payable merchantWallet
) external returns (uint256 merchantId);

// Create a new coupon program
function createCouponProgram(
    uint256 merchantId,
    string memory programName,
    string memory programDetails,
    uint256 validityPeriod,
    uint256 maxIssuance
) external returns (uint256 programId);

// Check if merchant is registered
function isMerchantRegistered(uint256 merchantId) external view returns (bool);

// Get merchant details
function getMerchantDetails(uint256 merchantId) external view returns (
    string memory name,
    string memory details,
    address wallet,
    bool isActive
);

// Update merchant details
function updateMerchantDetails(
    uint256 merchantId,
    string memory name,
    string memory details
) external;

// Get all programs by merchant
function getMerchantPrograms(uint256 merchantId) external view returns (uint256[] memory);
```

### ZK Verifier Contract

```solidity
// Verify a ZK proof for coupon issuance
function verifyCouponIssuanceProof(
    bytes memory zkProof,
    bytes32 publicInputHash
) external view returns (bool);

// Verify a ZK proof for coupon redemption
function verifyCouponRedemptionProof(
    bytes memory zkProof,
    uint256 tokenId,
    address user
) external view returns (bool);

// Register a new verification key for a merchant's program
function registerVerificationKey(
    uint256 merchantId, 
    uint256 programId, 
    bytes memory verificationKey
) external;

// Get verification key for a program
function getVerificationKey(
    uint256 merchantId, 
    uint256 programId
) external view returns (bytes memory);
```

### Account Abstraction Factory

```solidity
// Create a new account wallet controlled by email
function createEmailWallet(
    bytes32 emailHash,         // Hash of user's email
    bytes32 recoveryHash       // Secondary recovery mechanism
) external returns (address wallet);

// Execute a transaction from the wallet after email confirmation
function executeTransaction(
    address wallet,
    address target,
    uint256 value,
    bytes memory data,
    bytes memory emailProof
) external returns (bool success);

// Get wallet address for a user by email hash
function getWalletAddress(bytes32 emailHash) external view returns (address);

// Check if a wallet exists for an email
function walletExists(bytes32 emailHash) external view returns (bool);

// Recover wallet access with new email
function recoverWallet(
    address wallet,
    bytes32 newEmailHash,
    bytes memory recoveryProof
) external returns (bool);
```

## Backend Services and APIs

### Authentication Service

```typescript
// Register a new user with email
async function registerUser(email: string): Promise<{
  userId: string;
  walletAddress: string;
  authToken: string;
}>;

// Authenticate a user
async function authenticateUser(email: string): Promise<{
  authToken: string;
  expiresIn: number;
}>;

// Verify authentication token
async function verifyToken(token: string): Promise<{
  userId: string;
  email: string;
  isValid: boolean;
}>;

// Generate email verification link
async function generateEmailVerificationLink(
  userId: string,
  action: "REGISTER" | "LOGIN" | "REDEEM",
  metadata?: any
): Promise<string>;

// Verify email confirmation
async function verifyEmailConfirmation(
  token: string
): Promise<{
  userId: string;
  action: string;
  metadata: any;
  isValid: boolean;
}>;
```

### Email Service

```typescript
// Send registration confirmation email
async function sendRegistrationEmail(
  email: string,
  verificationLink: string
): Promise<boolean>;

// Send redemption confirmation email
async function sendRedemptionEmail(
  email: string,
  couponDetails: {
    id: string;
    merchantName: string;
    description: string;
    expiryDate: Date;
  },
  confirmationLink: string
): Promise<boolean>;

// Send wallet creation confirmation
async function sendWalletCreationEmail(
  email: string,
  walletAddress: string,
  qrCodeUrl: string
): Promise<boolean>;

// Send transaction receipt
async function sendTransactionReceipt(
  email: string,
  transactionDetails: {
    type: "ISSUANCE" | "REDEMPTION";
    merchantName: string;
    couponDescription: string;
    timestamp: Date;
  }
): Promise<boolean>;
```

### Account Abstraction System

```typescript
// Create a wallet for a new user
async function createUserWallet(userId: string, email: string): Promise<{
  walletAddress: string;
  qrCodeUrl: string;
}>;

// Generate QR code for wallet address
async function generateWalletQR(walletAddress: string): Promise<{
  qrCodeUrl: string;
  qrCodeData: string;
}>;

// Execute transaction after email confirmation
async function executeTransaction(
  userId: string,
  emailConfirmationToken: string,
  transactionData: {
    target: string;
    value: string;
    data: string;
  }
): Promise<{
  transactionHash: string;
  success: boolean;
}>;

// Get user's wallet details
async function getUserWallet(userId: string): Promise<{
  walletAddress: string;
  balance: string;
  coupons: Array<{
    id: string;
    merchantName: string;
    description: string;
    issuedAt: Date;
    expiryDate: Date;
    isRedeemed: boolean;
  }>;
}>;
```

### ZK Privacy Layer

```typescript
// Generate ZK proof for purchase
async function generatePurchaseProof(
  purchaseData: {
    userId: string;
    merchantId: string;
    amount: string;
    items: Array<{
      id: string;
      quantity: number;
      price: string;
    }>;
    programId: string;
  }
): Promise<{
  proof: string;
  publicInputs: string[];
}>;

// Verify ZK proof (backend verification)
async function verifyProof(
  proof: string,
  publicInputs: string[]
): Promise<boolean>;

// Generate ZK proof for coupon redemption
async function generateRedemptionProof(
  userId: string,
  couponId: string,
  merchantId: string
): Promise<{
  proof: string;
  publicInputs: string[];
}>;

// Get ZK circuit parameters for a merchant program
async function getCircuitParameters(merchantId: string, programId: string): Promise<{
  verificationKey: string;
  programRules: any;
}>;
```

### Merchant Management

```typescript
// Register a new merchant
async function registerMerchant(
  name: string,
  email: string,
  businessDetails: any
): Promise<{
  merchantId: string;
  walletAddress: string;
}>;

// Create a new coupon program
async function createCouponProgram(
  merchantId: string,
  programDetails: {
    name: string;
    description: string;
    validity: number; // in days
    rules: any; // coupon issuance rules
    zkCircuitId: string; // which ZK circuit to use
  }
): Promise<{
  programId: string;
}>;

// Issue coupon to user
async function issueCoupon(
  merchantId: string,
  programId: string,
  userWalletAddress: string,
  purchaseData: any,
  zkProof: string
): Promise<{
  couponId: string;
  transactionHash: string;
}>;

// Initiate coupon redemption
async function initiateCouponRedemption(
  merchantId: string,
  couponId: string,
  userWalletAddress: string
): Promise<{
  redemptionId: string;
  emailSent: boolean;
}>;

// Get merchant analytics
async function getMerchantAnalytics(merchantId: string): Promise<{
  issuedCoupons: number;
  redeemedCoupons: number;
  activeCoupons: number;
  programPerformance: any[];
}>;
```

## API Endpoints

### User Routes
- `POST /api/users/register` - Register new user
- `POST /api/users/login` - User login
- `GET /api/users/wallet` - Get user wallet info
- `GET /api/users/coupons` - Get user's coupons
- `GET /api/users/qr-code` - Get user's QR code

### Merchant Routes
- `POST /api/merchants/register` - Register new merchant
- `POST /api/merchants/programs` - Create coupon program
- `GET /api/merchants/programs` - List merchant's programs
- `POST /api/merchants/issue-coupon` - Issue coupon to user
- `POST /api/merchants/redeem-coupon` - Initiate coupon redemption
- `GET /api/merchants/analytics` - Get merchant analytics

### Authentication Routes
- `POST /api/auth/verify-email` - Verify email link
- `POST /api/auth/refresh-token` - Refresh auth token

### Email Confirmation Routes
- `GET /api/confirm/redeem/:token` - Confirm redemption
- `GET /api/confirm/register/:token` - Confirm registration

### ZK Proof Routes
- `POST /api/zk/generate-proof` - Generate ZK proof
- `POST /api/zk/verify-proof` - Verify ZK proof

## User Flow

1. User registers on the ImPayQ web portal with their email
2. System creates an account abstraction wallet tied to their email
3. User receives a QR code representing their wallet address
4. Merchant scans the QR code with their ImPayQ app
5. Merchant app creates ZK proof of purchase (hiding specific purchase details)
6. Smart contract verifies proof and issues coupon token to user's wallet
7. User can view their coupons on the web portal
8. For redemption, user shows coupon QR, merchant scans and initiates redemption
9. User receives email asking to confirm redemption
10. User clicks confirmation link, and account abstraction executes the redemption transaction

## Security Considerations

- All email links are time-limited and single-use
- Rate limiting is implemented for all authentication endpoints
- ZK proofs are validated both off-chain and on-chain
- Account abstraction protects private keys
- Email confirmation provides transaction security

## Privacy Features

- Purchase details are never stored on-chain
- Only eligibility proofs are verified
- Merchants can see aggregate data but not individual user behaviors
- Private history stored locally on user's device

## Installation and Setup

[Instructions for setting up the development environment, deploying contracts, etc.]

## Contributing

[Contribution guidelines]

## License

[License information] 