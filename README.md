# Centralized Calling System - Client API Integration Guide

## Overview
This guide provides instructions for integrating with the Centralized Calling System. The system handles all calling operations centrally - clients only need to add data to their queues.

## Table of Contents
1. [Getting Started](#getting-started)
2. [Authentication](#authentication)
3. [Core Operations](#core-operations)
4. [Queue Management](#queue-management)
5. [Error Handling](#error-handling)
6. [Examples](#examples)

## Getting Started

### Prerequisites
- Valid API key from the system administrator
- Access to allocated Asterisk servers
- Understanding of your account code and server configurations

### Base URL
```
https://your-domain.com/calling/api/
```

### Required Headers
All API requests must include:
```
Authorization: Bearer <your_api_key>
Content-Type: application/json
```

### 1. Obtain Credentials
- **New Clients:** Contact system administrator to create account
- **Existing Clients:** Get your API key, account code, and server configurations

### 2. Test Connection
```bash
curl -X GET "https://your-domain.com/calling/api/queues/getmasterqueues/" \
  -H "Authorization: Bearer your_api_key"
```

### 3. Make Your First Call
```bash
curl -X POST "https://your-domain.com/calling/api/queues/insertdataincallqueue/" \
  -H "Authorization: Bearer your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "batch_id": 12345,
    "data": [{
      "asterisk_server_code": "server1",
      "action": {
        "Action": "Originate",
        "Channel": "PJSIP/1234567890@endpoint1",
        "Application": "Dial",
        "Data": "PJSIP/0987654321@endpoint2,30"
      }
    }]
  }'
```

### Rate Limits
- Max batch size: 1000 calls per request
- Recommended: 100-500 calls per request

## Authentication

### API Key Management
Your API key is your primary authentication method. Keep it secure and never expose it in client-side code.

**Security Notes:**
- Store API keys securely (environment variables, secure vaults)
- Rotate API keys regularly
- Use HTTPS for all communications

### API Key Operations

#### Create New Client
**Endpoint:** `POST /calling/api/clients/`
**Auth:** Superuser API Key required

```json
{
  "name": "Client Name",
  "accountcode": "client_xyz", 
  "username": "client_username",
  "did": "+1234567890",
  "allocated_channels": 10,
  "callbackurl": "https://client.com/callback",
  "is_active": true
}
```

#### Regenerate API Key
**Endpoint:** `PUT /calling/api/clients/regenerate-api-key`
**Auth:** Client's own API Key

#### Revoke API Key  
**Endpoint:** `PUT /calling/api/clients/revoke-api-key`
**Auth:** Client's own API Key

## Core Operations

### 1. Queue Data Insertion (Bulk Calls)
**Endpoint:** `POST /calling/api/queues/insertdataincallqueue/`

For bulk calling operations. Add batches of call data that will be processed automatically.

```json
{
  "batch_id": 12345,
  "data": [
    {
      "asterisk_server_code": "server1",
      "action": {
        "Action": "Originate",
        "Channel": "PJSIP/1234567890@endpoint1",
        "Application": "Dial",
        "Data": "PJSIP/0987654321@endpoint2,30",
        "CallerID": "Your Company <1001>"
      },
      "create_recording": true,
      "project": "campaign_2024"
    }
  ]
}
```

### 2. Quick Calls (Single Call)
**Endpoint:** `POST /calling/api/queues/insertquickcall/`

For immediate, single-call operations without creating a queue.

```json
{
  "asterisk_server_code": "server1",
  "number1": "1234567890",
  "number2": "0987654321",
  "customer_name": "John Doe",
  "action": {
    "Application": "Dial",
    "Data": "PJSIP/0987654321@endpoint2,30"
  },
  "create_recording": true,
  "project": "quick_campaign"
}
```

## Queue Management

### Get All Queues
**Endpoint:** `GET /calling/api/queues/getmasterqueues/`
**Query:** `?page=1&page_size=100`

### Get Queue Details
**Endpoint:** `GET /calling/api/queues/getqueues/`
**Query:** `?master_queue_id=123`

### Update Queue Status
**Endpoint:** `PUT /calling/api/queues/updatequeuemaster/{queue_id}`

```json
{
  "queue_status": "paused|in-progress|completed",
  "retry": true,
  "retry_count": 3,
  "retry_gap": 300
}
```

### Flush Queue
**Endpoint:** `GET /calling/api/queues/flushqueue/`
**Query:** `?client_code=your_client_code`

## Error Handling

### Common Errors
- **401 Unauthorized:** Invalid API key - verify your key is correct
- **400 Bad Request:** Missing required fields or validation errors
- **Batch Size Exceeded:** Split data into batches ≤1000 calls
- **Server Not Found:** Verify asterisk_server_code is correct and allocated

### Error Response Format
```json
{
  "message": "Error description",
  "status": "error",
  "details": "Additional details (if available)"
}
```

## Examples

### Bash Examples

#### Create New Client
```bash
curl -X POST "https://your-domain.com/calling/api/clients/" \
  -H "Authorization: Bearer your_superuser_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Client Company",
    "accountcode": "client_xyz",
    "username": "client_user",
    "allocated_channels": 10,
    "is_active": true
  }'
```

#### Insert Queue Data
```bash
curl -X POST "https://your-domain.com/calling/api/queues/insertdataincallqueue/" \
  -H "Authorization: Bearer your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "batch_id": 1001,
    "data": [{
      "asterisk_server_code": "server1",
      "action": {
        "Action": "Originate",
        "Channel": "PJSIP/1234567890@endpoint1",
        "Application": "Dial",
        "Data": "PJSIP/0987654321@endpoint2,30"
      },
      "create_recording": true,
      "project": "campaign_2024"
    }]
  }'
```

#### Quick Call
```bash
curl -X POST "https://your-domain.com/calling/api/queues/insertquickcall/" \
  -H "Authorization: Bearer your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "asterisk_server_code": "server1",
    "number1": "1234567890",
    "number2": "0987654321",
    "customer_name": "Test Customer",
    "action": {
      "Application": "Dial",
      "Data": "PJSIP/0987654321@endpoint2,30"
    }
  }'
```

### Python Client
```python
import requests

class CentralizedCallingClient:
    def __init__(self, api_key: str, base_url: str):
        self.headers = {
            "Authorization": f"Bearer {api_key}",
            "Content-Type": "application/json"
        }
        self.base_url = base_url
    
    def insert_queue_data(self, batch_id: int, calls: list):
        response = requests.post(
            f"{self.base_url}/queues/insertdataincallqueue/",
            headers=self.headers,
            json={"batch_id": batch_id, "data": calls}
        )
        return response.json()
    
    def make_quick_call(self, server_code: str, number1: str, customer_name: str):
        response = requests.post(
            f"{self.base_url}/queues/insertquickcall/",
            headers=self.headers,
            json={
                "asterisk_server_code": server_code,
                "number1": number1,
                "customer_name": customer_name,
                "action": {"Application": "Dial"}
            }
        )
        return response.json()

# Usage
client = CentralizedCallingClient("your_api_key", "https://your-domain.com/calling/api")
result = client.insert_queue_data(1001, [{
    "asterisk_server_code": "server1",
    "action": {"Action": "Originate", "Channel": "PJSIP/1234567890@endpoint1"}
}])
```

---

**Last Updated:** 2024-01-01  
**Version:** 1.0
