# Centralized Calling System - Client API Integration Guide

## Overview
This comprehensive guide provides step-by-step instructions for clients to integrate with the Centralized Calling System. The system handles all calling operations centrally, so clients only need to add data to their queues, and the system will manage the rest automatically.

## Table of Contents
1. [Getting Started](#getting-started)
2. [Authentication](#authentication)
3. [Queue Data Insertion](#queue-data-insertion)
4. [Quick Calls](#quick-calls)
5. [Queue Management](#queue-management)
6. [Step-by-Step Integration Guide](#step-by-step-integration-guide)
7. [API Reference](#api-reference)
8. [Error Handling](#error-handling)
9. [Best Practices](#best-practices)
10. [Examples](#examples)

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

## Authentication

### API Key Management
Your API key is your primary authentication method. Keep it secure and never expose it in client-side code.

**Important Security Notes:**
- Store API keys securely (environment variables, secure vaults)
- Rotate API keys regularly
- Use HTTPS for all communications
- Never log or expose API keys in error messages

## Queue Data Insertion

### Overview
The queue data insertion system allows you to add batches of call data that will be processed automatically by the centralized system. This is the primary method for bulk calling operations.

### Endpoint
```
POST /calling/api/queues/insertdataincallqueue/
```

### Request Structure
```json
{
  "queue_id": "integer (optional)",
  "batch_id": "integer (required)",
  "dids": ["string array (optional)"],
  "data": ["QueueItem array (required)"]
}
```

### QueueItem Structure
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

### Complete Example
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

### Response
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

## Quick Calls

### Overview
Quick calls are for immediate, single-call operations without creating a queue. They are processed immediately and don't require batch management.

### Endpoint
```
POST /calling/api/queues/insertquickcall/
```

### Request Structure
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

### Quick Call Example
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

## Queue Management

### Get Master Queues
Retrieve all your queue masters with statistics.

**Endpoint:** `GET /calling/api/queues/getmasterqueues/`

**Query Parameters:**
- `page` (optional): Page number (default: 1)
- `page_size` (optional): Items per page (default: 100)

**Response:**
```json
{
  "pagination": {
    "previous_page": null,
    "next_page": 2,
    "start_index": 1,
    "end_index": 10,
    "total_entries": 25,
    "total_pages": 3,
    "page": 1
  },
  "results": [
    {
      "id": 123,
      "client_id": 1,
      "accountcode": "your_account",
      "queue_name": "campaign_2024_1234567890",
      "queue_status": "in-progress",
      "retry": true,
      "retry_count": 3,
      "retry_gap": 300,
      "created_at": "2024-01-01T10:00:00Z",
      "updated_at": "2024-01-01T10:05:00Z",
      "dids": ["+1234567890"],
      "total_count": 100,
      "pending_count": 20,
      "queued_count": 5,
      "processed_count": 70,
      "terminated_count": 5
    }
  ],
  "status": "success"
}
```

### Get Queue Details
Get specific queue items by master queue ID.

**Endpoint:** `GET /calling/api/queues/getqueues/`

**Query Parameters:**
- `master_queue_id` (required): ID of the master queue

### Update Queue Master
Update queue retry settings and status.

**Endpoint:** `PUT /calling/api/queues/updatequeuemaster/{queue_id}`

**Request Body:**
```json
{
  "queue_status": "string (optional)",
  "retry": "boolean (optional)",
  "retry_count": "integer (optional)",
  "retry_gap": "integer (optional)"
}
```

### Flush Queue
Empty the client RabbitMQ queue to stop calling.

**Endpoint:** `GET /calling/api/queues/flushqueue/`

**Query Parameters:**
- `client_code` (required): Your client code

## Step-by-Step Integration Guide

### Step 1: Obtain Your Credentials
1. Contact your system administrator to get:
   - Your API key
   - Your account code
   - Allocated server codes and configurations
   - Endpoint configurations

### Step 2: Test Your Connection
```bash
curl -X GET "https://your-domain.com/calling/api/queues/getmasterqueues/" \
  -H "Authorization: Bearer your_api_key" \
  -H "Content-Type: application/json"
```

### Step 3: Prepare Your Call Data
1. **For Bulk Operations (Queue Data Insertion):**
   - Prepare your call list with all required fields
   - Ensure phone numbers are in correct format
   - Set appropriate timeouts and retry settings
   - Group calls by batch_id for organization

2. **For Quick Calls:**
   - Prepare single call data
   - Ensure you have both numbers for Dial operations
   - Set customer name and project information

### Step 4: Insert Data into Queue
```bash
curl -X POST "https://your-domain.com/calling/api/queues/insertdataincallqueue/" \
  -H "Authorization: Bearer your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "batch_id": 12345,
    "data": [
      {
        "asterisk_server_code": "server1",
        "action": {
          "Action": "Originate",
          "Channel": "PJSIP/1234567890@endpoint1",
          "Application": "Dial",
          "Data": "PJSIP/0987654321@endpoint2,30"
        },
        "create_recording": true,
        "project": "campaign_2024"
      }
    ]
  }'
```

### Step 5: Monitor Your Queues
```bash
curl -X GET "https://your-domain.com/calling/api/queues/getmasterqueues/?page=1&page_size=10" \
  -H "Authorization: Bearer your_api_key"
```

### Step 6: Manage Queue Status
```bash
# Pause a queue
curl -X PUT "https://your-domain.com/calling/api/queues/updatequeuemaster/123" \
  -H "Authorization: Bearer your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "queue_status": "paused"
  }'

# Resume a queue
curl -X PUT "https://your-domain.com/calling/api/queues/updatequeuemaster/123" \
  -H "Authorization: Bearer your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "queue_status": "in-progress"
  }'
```

## API Reference

### Base URL
```
https://your-domain.com/calling/api/
```

### Authentication
All endpoints require Bearer token authentication:
```
Authorization: Bearer <your_api_key>
```

### Rate Limits
- Maximum batch size: 1000 calls per request
- Recommended batch size: 100-500 calls per request
- No rate limiting on API calls (handled by system capacity)

### Response Format
All responses follow this format:
```json
{
  "message": "Description of the result",
  "status": "success|warning|error",
  "data": "Response data (if applicable)"
}
```

## Error Handling

### Common HTTP Status Codes
- `200`: Success
- `201`: Created (for insertions)
- `400`: Bad Request (validation errors)
- `401`: Unauthorized (invalid API key)
- `403`: Forbidden (insufficient permissions)
- `404`: Not Found (resource doesn't exist)
- `500`: Internal Server Error

### Error Response Format
```json
{
  "message": "Error description",
  "status": "error",
  "details": "Additional error details (if available)"
}
```

### Common Error Scenarios

#### 1. Invalid API Key
```json
{
  "message": "Invalid API key",
  "status": "error"
}
```
**Solution:** Verify your API key is correct and active.

#### 2. Batch Size Exceeded
```json
{
  "message": "Queue data exceeds max batch size 1000",
  "status": "warning"
}
```
**Solution:** Split your data into smaller batches (≤1000 calls).

#### 3. Missing Required Fields
```json
{
  "message": "Queue Data is required",
  "status": "warning"
}
```
**Solution:** Ensure all required fields are provided in your request.

#### 4. Server Not Found
```json
{
  "message": "Asterisk server not found",
  "status": "warning"
}
```
**Solution:** Verify the asterisk_server_code is correct and allocated to your account.

## Best Practices

### 1. Data Preparation
- **Validate phone numbers** before sending
- **Use consistent formatting** for phone numbers
- **Set appropriate timeouts** based on your use case
- **Group related calls** using batch_id

### 2. Batch Management
- **Keep batch sizes reasonable** (100-500 calls recommended)
- **Use unique batch_ids** for tracking
- **Monitor queue status** regularly
- **Implement retry logic** for failed requests

### 3. Error Handling
- **Always check response status codes**
- **Implement exponential backoff** for retries
- **Log errors** for debugging
- **Handle network timeouts** gracefully

### 4. Performance Optimization
- **Use connection pooling** for high-volume operations
- **Send data in batches** rather than individual calls
- **Monitor system resources** during bulk operations
- **Use appropriate timeout settings**

### 5. Security
- **Store API keys securely** (environment variables, vaults)
- **Use HTTPS** for all communications
- **Rotate API keys** regularly
- **Implement proper access controls**

### 6. Monitoring
- **Track queue statuses** regularly
- **Monitor call success rates**
- **Set up alerts** for failed operations
- **Review logs** for optimization opportunities

## Examples

### Example 1: Basic Queue Insertion
```bash
#!/bin/bash

API_KEY="your_api_key_here"
BASE_URL="https://your-domain.com/calling/api"

# Insert a batch of calls
curl -X POST "${BASE_URL}/queues/insertdataincallqueue/" \
  -H "Authorization: Bearer ${API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "batch_id": 1001,
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
        "project": "test_campaign"
      }
    ]
  }'
```

### Example 2: Quick Call
```bash
#!/bin/bash

API_KEY="your_api_key_here"
BASE_URL="https://your-domain.com/calling/api"

# Make a quick call
curl -X POST "${BASE_URL}/queues/insertquickcall/" \
  -H "Authorization: Bearer ${API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "asterisk_server_code": "server1",
    "number1": "1234567890",
    "number2": "0987654321",
    "customer_name": "Test Customer",
    "action": {
      "Application": "Dial",
      "Data": "PJSIP/0987654321@endpoint2,30"
    },
    "create_recording": true,
    "project": "quick_test"
  }'
```

### Example 3: Queue Management
```bash
#!/bin/bash

API_KEY="your_api_key_here"
BASE_URL="https://your-domain.com/calling/api"

# Get all queues
echo "Getting all queues..."
curl -X GET "${BASE_URL}/queues/getmasterqueues/?page=1&page_size=10" \
  -H "Authorization: Bearer ${API_KEY}"

# Pause a specific queue
echo "Pausing queue 123..."
curl -X PUT "${BASE_URL}/queues/updatequeuemaster/123" \
  -H "Authorization: Bearer ${API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "queue_status": "paused"
  }'

# Resume the queue
echo "Resuming queue 123..."
curl -X PUT "${BASE_URL}/queues/updatequeuemaster/123" \
  -H "Authorization: Bearer ${API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "queue_status": "in-progress"
  }'
```

### Example 4: Python Integration
```python
import requests
import json
from typing import List, Dict, Any

class CentralizedCallingClient:
    def __init__(self, api_key: str, base_url: str):
        self.api_key = api_key
        self.base_url = base_url
        self.headers = {
            "Authorization": f"Bearer {api_key}",
            "Content-Type": "application/json"
        }
    
    def insert_queue_data(self, batch_id: int, calls: List[Dict[str, Any]], 
                         queue_id: int = None, dids: List[str] = None) -> Dict[str, Any]:
        """Insert batch of calls into queue"""
        payload = {
            "batch_id": batch_id,
            "data": calls
        }
        
        if queue_id:
            payload["queue_id"] = queue_id
        if dids:
            payload["dids"] = dids
        
        response = requests.post(
            f"{self.base_url}/queues/insertdataincallqueue/",
            headers=self.headers,
            json=payload
        )
        
        return response.json()
    
    def make_quick_call(self, asterisk_server_code: str, number1: str, 
                       customer_name: str, number2: str = None, 
                       project: str = None) -> Dict[str, Any]:
        """Make a quick call"""
        payload = {
            "asterisk_server_code": asterisk_server_code,
            "number1": number1,
            "customer_name": customer_name
        }
        
        if number2:
            payload["number2"] = number2
        if project:
            payload["project"] = project
        
        payload["action"] = {
            "Application": "Dial",
            "Data": f"PJSIP/{number2}@endpoint2,30" if number2 else None
        }
        
        response = requests.post(
            f"{self.base_url}/queues/insertquickcall/",
            headers=self.headers,
            json=payload
        )
        
        return response.json()
    
    def get_queues(self, page: int = 1, page_size: int = 100) -> Dict[str, Any]:
        """Get all queues with pagination"""
        response = requests.get(
            f"{self.base_url}/queues/getmasterqueues/",
            headers=self.headers,
            params={"page": page, "page_size": page_size}
        )
        
        return response.json()
    
    def update_queue_status(self, queue_id: int, status: str) -> Dict[str, Any]:
        """Update queue status"""
        payload = {"queue_status": status}
        
        response = requests.put(
            f"{self.base_url}/queues/updatequeuemaster/{queue_id}",
            headers=self.headers,
            json=payload
        )
        
        return response.json()

# Usage example
if __name__ == "__main__":
    client = CentralizedCallingClient(
        api_key="your_api_key_here",
        base_url="https://your-domain.com/calling/api"
    )
    
    # Insert batch of calls
    calls = [
        {
            "asterisk_server_code": "server1",
            "action": {
                "Action": "Originate",
                "Channel": "PJSIP/1234567890@endpoint1",
                "Application": "Dial",
                "Data": "PJSIP/0987654321@endpoint2,30"
            },
            "create_recording": True,
            "project": "test_campaign"
        }
    ]
    
    result = client.insert_queue_data(batch_id=1001, calls=calls)
    print(f"Queue insertion result: {result}")
    
    # Make a quick call
    quick_call_result = client.make_quick_call(
        asterisk_server_code="server1",
        number1="1234567890",
        customer_name="Test Customer",
        number2="0987654321",
        project="quick_test"
    )
    print(f"Quick call result: {quick_call_result}")
    
    # Get queues
    queues = client.get_queues()
    print(f"Queues: {queues}")
```

## Support and Troubleshooting

### Getting Help
1. **Check the logs** for detailed error information
2. **Verify your configuration** (API key, server codes, endpoints)
3. **Test with small batches** first
4. **Contact system administrators** for server-specific issues

### Common Issues and Solutions

#### Issue: Calls not being processed
**Possible Causes:**
- Queue status is "paused"
- Server is not allocated to your account
- Invalid server configuration

**Solutions:**
- Check queue status using GET /queues/getmasterqueues/
- Verify server allocation with administrator
- Ensure server codes are correct

#### Issue: High failure rates
**Possible Causes:**
- Invalid phone numbers
- Network connectivity issues
- Server capacity limitations

**Solutions:**
- Validate phone number formats
- Check network connectivity
- Contact administrator about server capacity

#### Issue: API authentication errors
**Possible Causes:**
- Invalid or expired API key
- Missing Authorization header
- Incorrect header format

**Solutions:**
- Verify API key is correct
- Ensure Authorization header is properly formatted
- Contact administrator for new API key if needed

## Conclusion

This guide provides comprehensive information for integrating with the Centralized Calling System. The system is designed to handle all calling operations automatically once you provide the data. Follow the step-by-step guide, implement proper error handling, and monitor your queues regularly for optimal performance.

For additional support or questions, please contact your system administrator or refer to the technical documentation.

---

**Last Updated:** 2024-01-01  
**Version:** 1.0  
**System:** Centralized Calling System
