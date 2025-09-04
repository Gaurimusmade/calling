# Centralized Calling System - Client API Integration Guide

## Overview
This guide provides instructions for integrating with the Centralized Calling System. The system handles all calling operations centrally - clients only need to add data to their queues.

## Getting Started

### Prerequisites
- Valid API key from the system administrator
- 
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

## Core Operations

### 1. Queue Data Insertion (Bulk Calls)
**Endpoint:** `POST /calling/api/queues/insertdataincallqueue/`

For bulk calling operations. Add batches of call data that will be processed automatically.

**Request Payload Structure:**
```json
{
  "queue_id": "integer (optional)",
  "batch_id": "integer (required)",
  "dids": ["string array (optional)"],
  "data": ["QueueItem array (required)"]
}
```

**QueueItem Structure:**
```json
{
  "asterisk_server_code": "string (required)",
  "caller_id": "string (optional)",
  "action": {
    "Action": "Originate (required)",
    "ActionID": "string (optional)",
    "Channel": "string (required)",
    "Application": "string (optional)",
    "Data": "string (optional)",
    "Timeout": "integer (optional, default: 30000)",
    "CallerID": "string (optional)",
    "Variable": "string (optional)",
    "Account": "string (optional)",
    "EarlyMedia": "boolean (optional, default: true)",
    "Async": "boolean (optional, default: true)",
    "Codecs": "string (optional)",
    "ChannelId": "string (optional)",
    "OtherChannelId": "string (optional)",
    "PreDialGoSub": "string (optional)"
  },
  "retry": "integer (default: 0)",
  "call_tried": "integer (default: 0)",
  "call_success": "integer (default: 0)",
  "call_failed": "integer (default: 0)",
  "call_status": "string (default: 'pending')",
  "connected": "boolean (default: false)",
  "create_recording": "boolean (default: false)",
  "recording_format": "string (default: 'wav')",
  "queue_type": "string (default: 'outbound')",
  "queue_status": "string (default: 'pending')",
  "call_timeout": "integer (default: 30)",
  "recording_path": "string (optional)",
  "mix_monitor_command": "string (optional)",
  "disposition": "string (optional)",
  "contact_number": "integer (optional)",
  "project": "string (optional)",
  "deadline": "datetime (optional)"
}
```

**Complete Example:**
```json
{
  "queue_id": null,
  "batch_id": 12345,
  "dids": ["+1234567890", "+0987654321"],
  "data": [
    {
      "asterisk_server_code": "server1",
      "caller_id": "Your Company <1001>",
      "action": {
        "Action": "Originate",
        "ActionID": "call-uuid-1234",
        "Channel": "PJSIP/1234567890@endpoint1",
        "Application": "Dial",
        "Data": "PJSIP/0987654321@endpoint2,30",
        "Timeout": 30000,
        "CallerID": "Your Company <1001>",
        "Account": "your_account_code",
        "EarlyMedia": true,
        "Async": true
      },
      "retry": 3,
      "create_recording": true,
      "recording_format": "wav",
      "queue_type": "outbound",
      "call_timeout": 30,
      "contact_number": 1234567890,
      "project": "campaign_2024",
      "deadline": "2024-12-31T23:59:59Z"
    }
  ]
}
```

**Response:**
```json
{
  "message": "Queue data inserted successfully",
  "status": "success",
  "data": {
    "queue_id": 123,
    "inserted_count": 1,
    "batch_id": 12345
  }
}
```

### 2. Quick Calls (Single Call)
**Endpoint:** `POST /calling/api/queues/insertquickcall/`

For immediate, single-call operations without creating a queue.

**Request Payload Structure:**
```json
{
  "asterisk_server_code": "string (required)",
  "number1": "string (required)",
  "number2": "string (optional)",
  "customer_name": "string (required)",
  "prefix": "string (optional)",
  "endpoint": "string (optional)",
  "action": {
    "Application": "string (required)",
    "Data": "string (optional)",
    "Async": "boolean (optional)",
    "ChannelId": "string (optional)"
  },
  "create_recording": "boolean (optional)",
  "recording_format": "string (optional)",
  "mix_monitor_command": "string (optional)",
  "call_timeout": "integer (optional)",
  "project": "string (optional)"
}
```

**Complete Example:**
```json
{
  "asterisk_server_code": "server1",
  "number1": "1234567890",
  "number2": "0987654321",
  "customer_name": "John Doe",
  "prefix": "1",
  "endpoint": "provider1",
  "action": {
    "Application": "Dial",
    "Data": "PJSIP/0987654321@endpoint2,30",
    "Async": true,
    "ChannelId": "channel-123"
  },
  "create_recording": true,
  "recording_format": "wav",
  "call_timeout": 30,
  "project": "quick_campaign"
}
```

**Response:**
```json
{
  "message": "Quick call inserted successfully",
  "status": "success",
  "data": {
    "call_id": "call-uuid-1234",
    "queue_id": 124
  }
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



Thank You !
```

