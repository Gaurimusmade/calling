# Queue Data Insertion Documentation

## Overview
This documentation provides comprehensive information about adding data to queues for particular clients/account codes in the Centralized Calling System. The system supports batch insertion of call data into queues for outbound calling campaigns.

## Table of Contents
1. [API Endpoints](#api-endpoints)
2. [Data Models](#data-models)
3. [API Payload Structures](#api-payload-structures)
4. [Authentication](#authentication)
5. [Examples](#examples)
6. [Error Handling](#error-handling)
7. [Queue Management](#queue-management)
8. [Best Practices](#best-practices)

## API Endpoints

### 1. Insert Data in Call Queue
**Endpoint:** `POST /calling/api/queues/insertdataincallqueue/`

**Description:** Inserts a batch of call data into the queue for a specific client/account code.

**Authentication:** Required (Client API Key)

**Request Body:** `InsertQueueRequest`

### 2. Get Master Queues
**Endpoint:** `GET /calling/api/queues/getmasterqueues/`

**Description:** Retrieves all master queues for a client with pagination and status counts.

**Query Parameters:**
- `accountcode` (optional): Account code (required for superusers)
- `page` (optional): Page number (default: 1)
- `page_size` (optional): Items per page (default: 100)

### 3. Get Queue by ID
**Endpoint:** `GET /calling/api/queues/getqueues/`

**Description:** Retrieves specific queue items by master queue ID.

**Query Parameters:**
- `master_queue_id` (required): ID of the master queue
- `client_code` (optional): Client code (required for superusers)

### 4. Get Quick Calls
**Endpoint:** `GET /calling/api/queues/getquickcalls/`

**Description:** Retrieves quick calls (calls without queue_id).

**Query Parameters:**
- `client_code` (optional): Client code (required for superusers)

### 5. Insert Quick Call
**Endpoint:** `POST /calling/api/queues/insertquickcall/`

**Description:** Inserts a single quick call into the queue.

**Request Body:** `QuickCall`

### 6. Update Queue Master
**Endpoint:** `PUT /calling/api/queues/updatequeuemaster/{queue_id}`

**Description:** Updates queue master retry settings and status.

**Request Body:** `UpdateQueueMasterPayload`

### 7. Flush Queue
**Endpoint:** `GET /calling/api/queues/flushqueue/`

**Description:** Empties the client RabbitMQ queue to stop calling.

**Query Parameters:**
- `client_code` (required): Client code

### 8. Get Queue Statuses
**Endpoint:** `GET /calling/api/queues/getqueuestatuses/`

**Description:** Retrieves all available queue statuses.

## Data Models

### InsertQueueRequest
```json
{
  "queue_id": "integer (optional)",
  "batch_id": "integer (required)",
  "dids": ["string array (optional)"],
  "data": ["QueueItem array (required)"]
}
```

### QueueItem
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

### QuickCall
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

### UpdateQueueMasterPayload
```json
{
  "queue_status": "string (optional)",
  "retry": "boolean (optional)",
  "retry_count": "integer (optional)",
  "retry_gap": "integer (optional)"
}
```

## API Payload Structures

### Complete InsertQueueRequest Example
```json
{
  "queue_id": 123,
  "batch_id": 456,
  "dids": ["+1234567890", "+0987654321"],
  "data": [
    {
      "asterisk_server_code": "server1",
      "caller_id": "Company <1001>",
      "action": {
        "Action": "Originate",
        "ActionID": "call-uuid-1234",
        "Channel": "PJSIP/1234567890@endpoint1",
        "Application": "Dial",
        "Data": "PJSIP/0987654321@endpoint2,30",
        "Timeout": 30000,
        "CallerID": "Company <1001>",
        "Account": "client123",
        "EarlyMedia": true,
        "Async": true
      },
      "retry": 3,
      "call_tried": 0,
      "call_success": 0,
      "call_failed": 0,
      "call_status": "pending",
      "connected": false,
      "create_recording": true,
      "recording_format": "wav",
      "queue_type": "outbound",
      "queue_status": "pending",
      "call_timeout": 30,
      "recording_path": "/recordings/call_123.wav",
      "mix_monitor_command": "/home/ubuntu/asterisk_audiosocket/script.py",
      "disposition": null,
      "contact_number": 1234567890,
      "project": "campaign_2024",
      "deadline": "2024-12-31T23:59:59Z"
    }
  ]
}
```

### QuickCall Example
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
  "mix_monitor_command": "/home/ubuntu/asterisk_audiosocket/script.py",
  "call_timeout": 30,
  "project": "quick_campaign"
}
```

## Authentication

All queue endpoints require authentication using the client's API key. The API key should be included in the request headers:

```
Authorization: Bearer <your_api_key>
```

### Client Types
1. **Regular Client**: Can only access their own data
2. **Superuser**: Can access any client's data by providing `accountcode` parameter

## Examples

### Example 1: Insert Batch of Calls
```bash
curl -X POST "https://your-domain.com/calling/api/queues/insertdataincallqueue/" \
  -H "Authorization: Bearer your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "queue_id": null,
    "batch_id": 789,
    "dids": ["+1234567890"],
    "data": [
      {
        "asterisk_server_code": "server1",
        "caller_id": "Company <1001>",
        "action": {
          "Action": "Originate",
          "Channel": "PJSIP/1234567890@endpoint1",
          "Application": "Dial",
          "Data": "PJSIP/0987654321@endpoint2,30",
          "CallerID": "Company <1001>",
          "Account": "client123"
        },
        "create_recording": true,
        "project": "campaign_2024"
      }
    ]
  }'
```

### Example 2: Insert Quick Call
```bash
curl -X POST "https://your-domain.com/calling/api/queues/insertquickcall/" \
  -H "Authorization: Bearer your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
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
  }'
```

### Example 3: Get Master Queues
```bash
curl -X GET "https://your-domain.com/calling/api/queues/getmasterqueues/?page=1&page_size=10" \
  -H "Authorization: Bearer your_api_key"
```

### Example 4: Update Queue Master
```bash
curl -X PUT "https://your-domain.com/calling/api/queues/updatequeuemaster/123" \
  -H "Authorization: Bearer your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "queue_status": "paused",
    "retry": true,
    "retry_count": 5,
    "retry_gap": 300
  }'
```

## Error Handling

### Common Error Responses

#### 400 Bad Request
```json
{
  "message": "Payload is required",
  "status": "warning"
}
```

#### 400 Bad Request - Batch Size Exceeded
```json
{
  "message": "Queue data exceeds max batch size 1000",
  "status": "warning"
}
```

#### 401 Unauthorized
```json
{
  "message": "Invalid API key",
  "status": "error"
}
```

#### 403 Forbidden
```json
{
  "message": "Account code is required for superuser",
  "status": "error"
}
```

#### 404 Not Found
```json
{
  "message": "Asterisk server not found",
  "status": "warning"
}
```

#### 500 Internal Server Error
```json
{
  "message": "Internal server error",
  "status": "error"
}
```

## Queue Management

### Queue Statuses
- `pending`: Queue is waiting to be processed
- `in-progress`: Queue is currently being processed
- `paused`: Queue processing is paused
- `flushed`: Queue has been flushed/emptied
- `terminated`: Queue processing has been terminated
- `processed`: All calls in queue have been processed

### Call Statuses
- `pending`: Call is waiting to be made
- `queued`: Call is in the processing queue
- `processed`: Call has been processed
- `flushed`: Call has been flushed/expired

### Batch Size Limits
- Maximum batch size: **1000** calls per request
- If batch size is exceeded, the request will be rejected with a 400 error

### Queue Naming Convention
- Master queues: `{accountcode}_{timestamp}`
- RabbitMQ queues: `Client.Queue.{server_code}.{accountcode}`

## Best Practices

### 1. Batch Size Management
- Keep batch sizes reasonable (recommended: 100-500 calls per batch)
- Monitor system performance when inserting large batches
- Use pagination when retrieving queue data

### 2. Error Handling
- Always check response status codes
- Implement retry logic for failed requests
- Log errors for debugging purposes

### 3. Data Validation
- Validate all required fields before sending requests
- Ensure phone numbers are in correct format
- Check that asterisk server codes are valid

### 4. Queue Management
- Monitor queue statuses regularly
- Use appropriate queue statuses for different scenarios
- Implement proper retry mechanisms

### 5. Security
- Keep API keys secure and rotate them regularly
- Use HTTPS for all API communications
- Implement proper access controls

### 6. Performance
- Use connection pooling for high-volume operations
- Implement proper timeout settings
- Monitor system resources during bulk operations

## Field Descriptions

### Required Fields
- `asterisk_server_code`: The server code where the call will be processed
- `batch_id`: Unique identifier for the batch of calls
- `data`: Array of call data to be inserted

### Optional Fields
- `queue_id`: If not provided, a new queue will be created
- `dids`: Array of DID numbers for the queue
- `caller_id`: Display name for the caller
- `action`: Detailed call action configuration
- `retry`: Number of retry attempts for failed calls
- `create_recording`: Whether to record the call
- `recording_format`: Format for call recordings (default: wav)
- `project`: Project identifier for the calls
- `deadline`: Deadline for call completion

### Action Configuration
The `action` field contains detailed configuration for the Asterisk Originate action:
- `Channel`: The channel to originate the call on
- `Application`: The application to execute (Dial, AGI, etc.)
- `Data`: Data to pass to the application
- `Timeout`: Call timeout in milliseconds
- `CallerID`: Caller ID to display
- `Account`: Account code for billing
- `Async`: Whether to make the call asynchronously

## System Configuration

### Database Settings
- Queue batch size limit: 1000 calls
- Timezone: Configurable via settings
- Day end time: Configurable via settings
- Day start time: Configurable via settings

### RabbitMQ Configuration
- Queue type: Quorum queues
- Message persistence: Enabled
- Queue durability: Enabled

### Redis Configuration
- Multiple databases for different purposes
- Client cache: Database 6
- Asterisk events: Database 1
- Queue management: Database 2

This documentation provides comprehensive information for integrating with the queue system. For additional support or questions, please refer to the system administrators or technical documentation.
