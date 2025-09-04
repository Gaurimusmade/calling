# Centralized Calling System 

## Overview
This guide provides instructions for integrating with the Centralized Calling System. The system handles all calling operations centrally - clients only need to add data to their queues.

## Getting Started

### Prerequisites
- Valid API key from the system administrator
  
### Base URL
```
https://novacaller.markytics.com
```

### Required Headers
All API requests must include:
```
X-API-KEY: <your_api_key>
Content-Type: application/json
```

### 1. Obtain Credentials
- **New Clients:** Contact system administrator to create account
- **Existing Clients:** Get your API key, account code, and server configurations

### 2. Test Connection
```bash
curl -X GET "https://novacaller.markytics.com/calling/api/v1/clients/test-connection" \
  -H "X-API-KEY: your_api_key"
```

### Rate Limits
- **Each Batch:** Maximum 1000 calls per batch
- **Queue Management:** You can add multiple batches to the same queue using the same queue_id
 
## Core Operations

### 1. Queue Data Insertion (Bulk Calls)
**Endpoint:** `POST /calling/api/v1/queues/insertdataincallqueue/`

For bulk calling operations. Add batches of call data that will be processed automatically.

**Important Queue Management Workflow:**
1. **Create New Queue:** Pass `queue_id` as `null` or omit it completely to create a new queue
2. **API Response:** Returns a `queue_id` that you must use for all subsequent batches in that same queue
3. **Add More Batches:** Use the returned `queue_id` to add more batches (max 1000 calls per batch) to the same queue
4. **Create Another Queue:** To start a new queue, pass `queue_id` as `null` again

**Batch Limits:**
- Each batch can contain maximum 1000 calls
- You can add unlimited batches to the same queue
- Each batch must have a unique `batch_id`

**Request Payload Structure:**
```json
{
  "queue_id": "integer (optional - null to create new queue, use returned queue_id to add to existing queue)",
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
    "CallerID": "string (optional)",
    "Variable": "string (optional)",
    "Account": "string (optional)",
    "Async": "boolean (optional, default: true)",
    "ChannelId": "string (optional)",
    "OtherChannelId": "string (optional)",
  },
  "retry": "integer (default: 0)",
  "create_recording": "boolean (default: false)",
  "recording_path": "string (optional)",
  "mix_monitor_command": "string (optional)",
  "contact_number": "integer (optional)",
  "project": "string (optional)",
  "deadline": "datetime (optional)"
}
```

**Complete Example (Create New Queue):**
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
        "CallerID": "Your Company <1001>",
        "Account": "your_account_code",
        "Async": true,
        "ChannelId": "channel-123"
      },
      "retry": 3,
      "create_recording": true,
      "recording_path": "/recordings/call_123.wav",
      "mix_monitor_command": "MixMonitor /recordings/call_123.wav",
      "contact_number": 1234567890,
      "project": "campaign_2024",
      "deadline": "2024-12-31T23:59:59Z"
    }
  ]
}
```

**Example for Adding to Existing Queue:**
```json
{
  "queue_id": 123,
  "batch_id": 12346,
  "dids": ["+1111111111", "+2222222222"],
  "data": [
    {
      "asterisk_server_code": "server1",
      "caller_id": "Your Company <1001>",
      "action": {
        "Action": "Originate",
        "ActionID": "call-uuid-5678",
        "Channel": "PJSIP/1111111111@endpoint1",
        "Application": "Dial",
        "Data": "PJSIP/2222222222@endpoint2,30",
        "CallerID": "Your Company <1001>",
        "Account": "your_account_code",
        "Async": true,
        "ChannelId": "channel-456"
      },
      "retry": 3,
      "create_recording": true,
      "recording_path": "/recordings/call_456.wav",
      "mix_monitor_command": "MixMonitor /recordings/call_456.wav",
      "contact_number": 1111111111,
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
  "customer_name": "string (required)",
  "number1": "string (required)",
  "number2": "string (optional)",
  "did": "string (optional)",
  "action": {
    "Action": "Originate (required)",
    "Application": "Dial (required)",
    "Data": "string (optional)",
    "ChannelId": "string (optional)",
    "Async": "boolean (optional, default: true)"
  },
  "create_recording": "boolean (optional, default: true)",
}
```

**Complete Example:**
```json
{
  "asterisk_server_code": "server1",
  "customer_name": "John Doe",
  "number1": "1234567890",
  "number2": "0987654321",
  "did": "+1234567890",
  "action": {
    "Action": "Originate",
    "Application": "Dial",
    "Data": "PJSIP/0987654321@endpoint2,30",
    "ChannelId": "channel-123",
    "Async": true
  },
  "create_recording": true,
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
```

Thank You !

