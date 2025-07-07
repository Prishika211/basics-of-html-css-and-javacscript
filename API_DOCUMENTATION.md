# InsiderDepth Backend API Documentation

A comprehensive API for managing trading journals and cryptocurrency trades with authentication and user management.

## Base URL
```
https://api.insiderdepth.com/api/v1
```

## Authentication

All protected endpoints require JWT authentication via Bearer token in the Authorization header:

```
Authorization: Bearer <your_jwt_token>
```

## Response Format

All API responses follow a consistent format using the ApiResponse utility:

```json
{
  "status": "success" | "error",
  "message": "Human readable message",
  "data": {} // Response data (success only)
}
```

---

## Authentication Endpoints

### Register User
```http
POST /users/register
```

Register a new user account.

**Request Body:**
```json
{
  "email": "string",
  "username": "string", 
  "password": "string",
  "walletAddress": "string", // optional, Ethereum wallet format
  "confirmPassword": "string" // optional
}
```

**Validation Rules:**
- Email: Valid email format, max 254 characters
- Username: 3-30 characters, alphanumeric, dots, underscores, hyphens only
- Password: Min 8 chars, must contain uppercase, lowercase, number, special character
- Wallet Address: Valid Ethereum format (0x + 40 hex chars) or empty

**Response:**
```json
{
  "status": "success",
  "message": "User registered successfully. Please check your email for verification.",
  "data": {
    "user": {
      "id": "string",
      "username": "string",
      "email": "string",
      "isEmailVerified": false,
      "role": "user",
      "walletAddress": "string"
    },
    "accessToken": "string"
  }
}
```

**Status Codes:**
- `201` - Created
- `400` - Bad Request (validation errors)

---

### Login User
```http
POST /users/login
```

Authenticate user and receive access tokens.

**Request Body:**
```json
{
  "email": "string",
  "password": "string"
}
```

**Response:**
```json
{
  "status": "success",
  "message": "Login successful",
  "data": {
    "user": {
      "id": "string",
      "username": "string",
      "email": "string",
      "isEmailVerified": boolean,
      "role": "user"
    },
    "accessToken": "string"
  }
}
```

**Status Codes:**
- `200` - OK
- `401` - Invalid credentials
- `423` - Account locked (too many failed attempts)

---

### Refresh Token
```http
POST /users/refresh-token
```

Get new access token using refresh token (from HTTP-only cookie or request body).

**Request Body (optional if using cookies):**
```json
{
  "refreshToken": "string"
}
```

**Response:**
```json
{
  "status": "success",
  "message": "Token refreshed successfully",
  "data": {
    "accessToken": "string"
  }
}
```

---

### Verify Email
```http
POST /users/verify-email
```

Verify user email with verification token.

**Request Body:**
```json
{
  "token": "string"
}
```

---

### Forgot Password
```http
POST /users/forgot-password
```

Send password reset email.

**Request Body:**
```json
{
  "email": "string"
}
```

---

### Reset Password
```http
POST /users/reset-password
```

Reset password with token.

**Request Body:**
```json
{
  "token": "string",
  "newPassword": "string",
  "confirmPassword": "string" // optional
}
```

---

### Logout
```http
POST /users/logout
```
🔒 **Protected Route**

Logout user and invalidate current refresh token.

---

### Logout All Devices
```http
POST /users/logout-all
```
🔒 **Protected Route**

Logout from all devices (invalidate all refresh tokens).

---

### Get All Sessions
```http
GET /users/sessions
```
🔒 **Protected Route**

Get all active sessions/devices for the current user.

**Response:**
```json
{
  "status": "success",
  "data": {
    "totalSessions": number,
    "sessions": [
      {
        "sessionId": "string",
        "device": {
          "name": "string",
          "type": "mobile" | "tablet" | "desktop",
          "browser": "string",
          "os": "string"
        },
        "location": {
          "ipAddress": "string"
        },
        "activity": {
          "createdAt": "string",
          "lastUsed": "string",
          "isCurrentSession": boolean,
          "expiresAt": "string"
        }
      }
    ]
  }
}
```

---

## User Profile Management

### Get User Profile
```http
GET /users/profile
```
🔒 **Protected Route**

Get current user's profile information.

---

### Update User Profile
```http
PUT /users/profile
```
🔒 **Protected Route**

Update current user's profile.

**Request Body:**
```json
{
  "username": "string", // optional
  "walletAddress": "string" // optional
}
```

---

### Change Password
```http
POST /users/change-password
```
🔒 **Protected Route**

Change user password.

**Request Body:**
```json
{
  "currentPassword": "string",
  "newPassword": "string",
  "confirmNewPassword": "string" // optional
}
```

---

## Admin Routes

### Get Admin Data
```http
GET /users/admin
```
🔒 **Protected Route (Admin Only)**

Get admin dashboard statistics.

---

### Manage User
```http
POST /users/admin/manage-user
```
🔒 **Protected Route (Admin Only)**

Manage user accounts.

**Request Body:**
```json
{
  "userId": "string",
  "action": "activate" | "deactivate" | "changeRole",
  "role": "user" | "admin" | "moderator" // required for changeRole action
}
```

---

## Journal Management

All journal routes are prefixed with the authenticated journal router.

### Get All Journals
```http
GET /journals
```
🔒 **Protected Route**

Get all journals for the authenticated user.

**Response:**
```json
{
  "status": "success",
  "message": "Journals retrieved successfully",
  "data": [
    {
      "id": "string",
      "name": "string", 
      "description": "string",
      "totalTrades": number,
      "totalPnL": number,
      "totalReturnPercentage": number,
      "winRate": number,
      "isActive": boolean,
      "createdAt": "string",
      "updatedAt": "string"
    }
  ]
}
```

---

### Get Journal by ID
```http
GET /journals/{id}
```
🔒 **Protected Route**

Get specific journal with recent trades.

**Response includes recent trades:**
```json
{
  "status": "success",
  "data": {
    "id": "string",
    "name": "string", 
    "description": "string",
    "totalTrades": number,
    "totalPnL": number,
    "totalReturnPercentage": number,
    "winRate": number,
    "isActive": boolean,
    "createdAt": "string",
    "updatedAt": "string",
    "recentTrades": [
      {
        "id": "string",
        "cryptoPair": "string",
        "tradeType": "spot" | "futures",
        "pnl": number,
        "returnPercentage": number,
        "status": "open" | "closed" | "cancelled",
        "createdAt": "string"
      }
    ]
  }
}
```

---

### Create Journal
```http
POST /journals
```
🔒 **Protected Route**

Create a new trading journal.

**Request Body:**
```json
{
  "name": "string", // 1-100 characters, required
  "description": "string" // max 500 characters, optional
}
```

**Validation:**
- Journal names must be unique per user
- Handles duplicate name conflicts

---

### Update Journal  
```http
PUT /journals/{id}
```
🔒 **Protected Route**

Update existing journal.

**Request Body:**
```json
{
  "name": "string", // optional, 1-100 characters
  "description": "string" // optional, max 500 characters  
}
```

---

### Delete Journal
```http
DELETE /journals/{id}
```
🔒 **Protected Route**

Soft delete a journal (sets isActive to false).

**Business Rules:**
- Soft delete only (preserves data)

---

## Trade Management

### Get All Trades
```http
GET /trades
```
🔒 **Protected Route**

Get all trades for user with filtering and pagination.

**Query Parameters:**
- `tradeType` (optional): "spot" | "futures"
- `status` (optional): "open" | "closed" | "cancelled" 
- `cryptoPair` (optional): Filter by crypto pair (regex search)
- `dateFrom` (optional): ISO datetime string
- `dateTo` (optional): ISO datetime string
- `page` (optional): Page number (default: 1)
- `limit` (optional): Items per page (default: 20, max: 100)

**Response:**
```json
{
  "status": "success", 
  "data": {
    "trades": [
      {
        "id": "string",
        "journalId": {
          "name": "string"
        },
        "tradeType": "spot" | "futures",
        "cryptoPair": "string",
        "status": "open" | "closed" | "cancelled",
        "pnl": number,
        "returnPercentage": number,
        "entryDate": "string",
        "exitDate": "string",
        "notes": "string",
        "tags": ["string"],
        "spotData": {
          "buyPrice": number,
          "sellPrice": number,
          "quantity": number
        },
        "futuresData": {
          "entryPrice": number,
          "exitPrice": number,
          "capital": number,
          "leverage": number,
          "position": "long" | "short"
        }
      }
    ],
    "pagination": {
      "page": number,
      "limit": number, 
      "totalPages": number,
      "totalCount": number,
      "hasNext": boolean,
      "hasPrev": boolean
    }
  }
}
```

---

### Get Journal Trades
```http
GET /journals/{journalId}/trades
```
🔒 **Protected Route**

Get trades for a specific journal with same filtering options as `/trades`.

**Response includes journal summary:**
```json
{
  "status": "success",
  "data": {
    "journal": {
      "id": "string",
      "name": "string", 
      "totalTrades": number,
      "totalPnL": number,
      "winRate": number
    },
    "trades": [], // Same format as /trades
    "pagination": {} // Same pagination format
  }
}
```

---

### Create Spot Trade
```http
POST /trades/spot
```
🔒 **Protected Route**

Create a new spot trade.

**Request Body:**
```json
{
  "journalId": "string",
  "cryptoPair": "string", // Format: "BTC/USDT"
  "buyPrice": number, // positive number
  "quantity": number, // positive number
  "entryDate": "string", // optional, ISO datetime
  "notes": "string", // optional, max 1000 characters
  "tags": ["string"] // optional, max 10 tags, 50 chars each
}
```

**Validation:**
- Crypto pair format: `/^[A-Z0-9]+\/[A-Z0-9]+$/`
- Verifies journal ownership
- Creates trade with status "open"

---

### Create Futures Trade
```http
POST /trades/futures
```
🔒 **Protected Route**

Create a new futures trade.

**Request Body:**
```json
{
  "journalId": "string",
  "cryptoPair": "string",
  "entryPrice": number, // positive number
  "capital": number, // positive number
  "leverage": number, // 1-125
  "position": "long" | "short",
  "exitPrice": number, // optional, for pre-closed trades
  "entryDate": "string", // optional, ISO datetime
  "exitDate": "string", // optional, ISO datetime
  "notes": "string", // optional, max 1000 characters
  "tags": ["string"] // optional
}
```

**Business Logic:**
- If exitPrice provided, trade is created as "closed"
- P&L automatically calculated for closed trades
- Updates journal statistics

---

### Close Trade
```http
PUT /trades/{id}/close
```
🔒 **Protected Route**

Close an open trade (generic endpoint).

**Request Body:**
```json
{
  "exitPrice": number, // positive number required
  "exitDate": "string", // optional, ISO datetime
  "quantity": number // optional, for partial spot closes
}
```

---

### Close Spot Trade (with Partial Support)
```http
PUT /trades/{id}/close-spot
```
🔒 **Protected Route**

Close spot trade with partial selling support.

**Request Body:**
```json
{
  "sellPrice": number, // positive number required
  "quantity": number, // optional, defaults to full quantity
  "exitDate": "string" // optional, ISO datetime
}
```

**Business Logic:**
- If quantity < total quantity: Creates new closed trade for sold portion, updates original
- If quantity equals total (or omitted): Closes original trade completely
- Validates quantity doesn't exceed available

**Response for Partial Sale:**
```json
{
  "status": "success",
  "data": {
    "soldTrade": {}, // New closed trade object
    "remainingTrade": {} // Updated original trade
  }
}
```

---

### Update Trade
```http
PUT /trades/{id}
```
🔒 **Protected Route**

Update trade details (notes, tags only).

**Request Body:**
```json
{
  "notes": "string", // optional
  "tags": ["string"] // optional
}
```

---

### Delete Trade
```http
DELETE /trades/{id}
```
🔒 **Protected Route**

Soft delete a trade and update journal statistics.

---

### Get Trading Dashboard
```http
GET /trades/dashboard
```
🔒 **Protected Route**

Get comprehensive dashboard data.

**Response:**
```json
{
  "status": "success",
  "data": {
    "overallStats": {
      "totalJournals": number,
      "totalTrades": number,
      "openPositions": number,
      "totalPnL": number,
      "winRate": number,
      "totalReturnPercentage": number
    },
    "journals": [], // All user journals
    "recentTrades": [], // Last 10 trades with journal names
    "openTrades": [] // All open trades with journal names
  }
}
```

---

### Get Portfolio
```http
GET /trades/portfolio
```
🔒 **Protected Route**

Get current spot positions grouped by crypto pair.

**Response:**
```json
{
  "status": "success",
  "data": {
    "portfolio": [
      {
        "cryptoPair": "string",
        "totalQuantity": number,
        "totalInvested": number,
        "averageBuyPrice": number,
        "trades": [
          {
            "id": "string",
            "buyPrice": number,
            "quantity": number,
            "entryDate": "string",
            "journalName": "string",
            "notes": "string",
            "tags": ["string"]
          }
        ]
      }
    ],
    "totalPositions": number,
    "totalOpenTrades": number
  }
}
```

---

## Trading Calculations

### Spot Trading P&L
```typescript
// P&L = (sellPrice - buyPrice) × quantity
// Return % = P&L / (buyPrice × quantity) × 100
```

### Futures Trading P&L  
```typescript
// Long Position:
// P&L = (exitPrice - entryPrice) × (capital × leverage / entryPrice)

// Short Position: 
// P&L = (entryPrice - exitPrice) × (capital × leverage / entryPrice)

// Return % = P&L / capital × 100
```

---

## Validation Rules

### Journal Validation
- Name: 1-100 characters, unique per user
- Description: max 500 characters

### Trade Validation  
- Crypto pairs: Format `/^[A-Z0-9]+\/[A-Z0-9]+$/` (e.g., "BTC/USDT")
- Prices: positive numbers only
- Leverage: 1x to 125x range
- Notes: max 1000 characters
- Tags: max 10 tags, 50 characters each

### User Validation
- Email: valid email format, max 254 characters
- Password: min 8 chars, complexity requirements
- Username: 3-30 characters, alphanumeric + dots/underscores/hyphens
- Wallet Address: Ethereum format or empty

---

## Error Handling

### Status Codes
- `200` - OK
- `201` - Created  
- `400` - Bad Request / Validation Error
- `401` - Unauthorized
- `403` - Forbidden / Insufficient Permissions
- `404` - Not Found
- `409` - Conflict (duplicate data)
- `423` - Locked (account locked)
- `429` - Too Many Requests
- `500` - Internal Server Error

### Error Response Format
```json
{
  "status": "error",
  "message": "Human readable error message",
  "code": "ERROR_CODE", // optional
  "details": [], // validation errors array, optional
  "timestamp": "ISO datetime string"
}
```

---

## Security Features

- **JWT Authentication**: Stateless access tokens
- **Refresh Token Rotation**: HTTP-only cookie based
- **Session Management**: Multi-device support with detailed tracking
- **Account Locking**: Protection against brute force attacks  
- **Input Validation**: Comprehensive Zod-based validation
- **Data Sanitization**: XSS and injection prevention
- **Rate Limiting**: Protection against abuse
- **Audit Logging**: Comprehensive security event logging

---

## Rate Limiting

- Authentication endpoints: 5 requests per minute
- General endpoints: 100 requests per minute per user
- Password reset: 15-minute cooldown between requests
- Email verification: 5-minute cooldown between requests
