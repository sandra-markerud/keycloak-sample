# Keycloak Token Exchange Test Project

This project demonstrates token exchange functionality between two Keycloak realms. It's designed to
help troubleshoot and showcase token exchange configuration issues.

## Overview

This test setup includes:

- Two Keycloak realms: `internal` and `external`
- Token exchange from external realm tokens to internal realm tokens
- HTTP requests for testing the complete flow
- Docker Compose setup for easy deployment
- Pre-configured test users for each realm

## Problem Statement

This project addresses issues with configuring Keycloak token exchange between two realms. The goal
is to exchange an access token from the `external` realm for a token valid in the `internal` realm.

**Current Issue**: The token exchange is failing with the following HTTP 400 error:

```json
{
  "error": "invalid_request",
  "error_description": "Invalid token"
}
```

**Keycloak Log**:

```
2025-06-26 07:05:44,006 WARN  [org.keycloak.events] (executor-thread-1) type="TOKEN_EXCHANGE_ERROR", realmId="d0418d1d-0102-4f6f-9add-f3469e079937", realmName="internal", clientId="internal-cli", userId="null", ipAddress="192.168.143.2", error="invalid_token", reason="subject_token validation failure", auth_method="token_exchange", grant_type="urn:ietf:params:oauth:grant-type:token-exchange", client_auth_method="client-secret"
```

The error indicates a "subject_token validation failure", suggesting that the internal realm cannot
validate the token issued by the external realm.

## Setup

### Prerequisites

- Docker and Docker Compose installed
- Access to the Keycloak Docker image
- HTTP client that supports `.http` files (e.g., IntelliJ IDEA, VS Code with REST Client extension)

### Quick Start

1. **Start Keycloak using Docker Compose**:
   Navigate to the docker directory and run `docker-compose up -d`

2. **Wait for Keycloak to start**:
   Keycloak will be available at `http://localhost:8080` once fully started.

3. **Test the Token Exchange Flow**:
   Open the `requests/requests.http` file in your HTTP client and execute the requests in order.

## Testing the Token Exchange

The `requests/requests.http` file contains three main requests that demonstrate the complete token
exchange flow:

### 1. Get External Realm Token

Endpoint: `POST http://localhost:8080/realms/external/protocol/openid-connect/token`

- **Purpose**: Authenticate user `emmaextern` in the external realm
- **Credentials**:
    - Username: `emmaextern`
    - Password: `test1234`
    - Client: `external-cli`
- **Result**: Returns an access token for the external realm

### 2. Get Internal Realm Token (Optional)

Endpoint: `POST http://localhost:8080/realms/internal/protocol/openid-connect/token`

- **Purpose**: Authenticate user `idaintern` in the internal realm (for comparison)
- **Credentials**:
    - Username: `idaintern`
    - Password: `test1234`
    - Client: `internal-cli`
- **Result**: Returns an access token for the internal realm

### 3. Token Exchange

Endpoint: `POST http://localhost:8080/realms/internal/protocol/openid-connect/token`

- **Purpose**: Exchange the external realm token for an internal realm token
- **Grant Type**: `urn:ietf:params:oauth:grant-type:token-exchange`
- **Input**: Uses the `access_token` from the external realm request
- **Result**: Returns a new token valid for the internal realm

## Pre-configured Test Users

### External Realm (`external`)

- **Username**: `emmaextern`
- **Password**: `test1234`
- **Client ID**: `external-cli`
- **Client Secret**: `P3mWZacixWo5cmSLIXo8gZmYAm26JLzl`

### Internal Realm (`internal`)

- **Username**: `idaintern`
- **Password**: `test1234`
- **Client ID**: `internal-cli`
- **Client Secret**: `ttaLQppzpcYj016gqELBrj1RcQSDt9pW`

## How to Execute Requests

### Using IntelliJ IDEA or VS Code with REST Client

- Open `requests/requests.http`
- Click the "Run" button next to each request
- Execute requests in order (external token → token exchange)

### Using curl (alternative)

For users who prefer command line tools, you can use curl commands to perform the same operations.
First get an external token, then use that token in the exchange request.

## Request Flow Details

The HTTP requests file includes automatic token handling:

- The external realm request stores the received access token in a variable
- The token exchange request automatically uses this stored token
- This eliminates the need to manually copy tokens between requests
