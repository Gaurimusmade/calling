# Centralized Calling System - Client Onboarding Documentation

## Table of Contents
1. [System Overview](#system-overview)
2. [Prerequisites](#prerequisites)
3. [Client Registration Process](#client-registration-process)
4. [Authentication Methods](#authentication-methods)
5. [Core API Endpoints](#core-api-endpoints)
6. [Server Allocation & Configuration](#server-allocation--configuration)
7. [Call Queue Management](#call-queue-management)
8. [Monitoring & Reporting](#monitoring--reporting)
9. [WebSocket Integration](#websocket-integration)
10. [Error Handling](#error-handling)
11. [Integration Examples](#integration-examples)
12. [Troubleshooting Guide](#troubleshooting-guide)

---

## System Overview

The Centralized Calling System is a comprehensive platform designed to manage and monitor Asterisk telephony servers. It provides a FastAPI-based backend API for interacting with various components of a calling system, including Asterisk servers, clients, call queues, and batch operations.

### Key Features:
- **Real-time Call Management**: Process and monitor calls across multiple Asterisk servers
- **Queue Management**: Handle call queues with batch operations and real-time processing
- **Server Allocation**: Dynamic allocation of Asterisk servers and channels to clients
- **Monitoring & Reporting**: Comprehensive call analytics and real-time monitoring
- **WebSocket Support**: Real-time event streaming for live call monitoring
- **Rate Limiting**: Configurable rate limits per client and server
- **Multi-tenant Architecture**: Isolated client environments with secure access

---

## Prerequisites

Before integrating with the Centralized Calling System, ensure you have:

### Technical Requirements:
- **HTTP/HTTPS Client**: For making API requests
- **WebSocket Client**: For real-time monitoring (optional but recommended)
- **Database Access**: PostgreSQL database for storing call data (if using client-specific databases)
- **Network Access**: Ensure your systems can reach the Centralized Calling API endpoints

### Business Requirements:
- **Account Code**: Unique identifier for your organization
- **DID Numbers**: Direct Inward Dialing numbers for outbound calls
- **Call Volume Estimates**: Expected call volume for proper server allocation
- **Rate Limits**: Desired call rate limits per hour
- **Active Time Windows**: Business hours when your system should be active

---

## Client Registration Process

### Step 1: Initial Client Creation

Clients are created by system administrators. Contact your system administrator to request client registration with the following information:

#### Required Information:
```json
{
  "name": "Your Company Name",
  "accountcode": "UNIQUE_ACCOUNT_CODE",
  "username": "your_username",
  "password": "secure_password",
  "did": "your_did_number",
  "phone_number": "your_contact_number",
  "allocated_channels": 100,
  "rate_limit": 1000,
  "callbackurl": "https://your-domain.com/callback",
  "active_time_start": "2024-01-01T09:00:00",
  "active_time_end": "2024-01-01T18:00:00",
  "is_active": true,
  "status": "active"
}
```

#### Optional Information:
```json
{
  "asyncdatabaseurl": "postgresql://user:pass@host:port/db",
  "syncdatabaseurl": "postgresql://user:pass@host:port/db"
}
```

### Step 2: API Key Generation

Upon successful client creation, you will receive:
- **API Key**: For authenticating API requests
- **Client ID**: Unique identifier for your client account
- **Account Code**: Your unique account identifier

**Important**: Store your API key securely and never expose it in client-side code.

---

## Authentication Methods

The system supports two authentication methods:

### 1. API Key Authentication (Recommended)

#### Header-based Authentication:
```http
X-API-KEY: your_api_key_here
```

#### Query Parameter Authentication:
```http
GET /api/v1/endpoint?x-api-key=your_api_key_here
```

### 2. JWT Bearer Token Authentication

#### Login to get JWT token:
```http
POST /api/v1/auth/login
Content-Type: application/json

{
  "username": "your_username",
  "password": "your_password"
}
```

#### Use JWT token:
```http
Authorization: Bearer your_jwt_token_here
```

### Authentication Examples:

#### Python (requests):
```python
import requests

# API Key method
headers = {
    'X-API-KEY': 'your_api_key_here',
    'Content-Type': 'application/json'
}

# JWT method
headers = {
    'Authorization': 'Bearer your_jwt_token_here',
    'Content-Type': 'application/json'
}

response = requests.get('https://api.example.com/api/v1/monitoring/health', headers=headers)
```

#### JavaScript (fetch):
```javascript
// API Key method
const headers = {
    'X-API-KEY': 'your_api_key_here',
    'Content-Type': 'application/json'
};

// JWT method
const headers = {
    'Authorization': 'Bearer your_jwt_token_here',
    'Content-Type': 'application/json'
};

fetch('https://api.example.com/api/v1/monitoring/health', { headers })
    .then(response => response.json())
    .then(data => console.log(data));
```

---

## Core API Endpoints

### Base URL
```
https://your-centralized-calling-domain.com/api/v1
```

### 1. Health & Status Endpoints

#### Check API Health
```http
GET /monitoring/health
```
**Response:**
```json
{
  "status": "healthy"
}
```

#### Get API Status
```http
GET /monitoring/status
```
**Response:**
```json
{
  "status": "running",
  "version": "1.0.0",
  "environment": "production"
}
```

### 2. Client Management Endpoints

#### Get Client Information
```http
GET /clients/me
```
**Response:**
```json
{
  "id": 1,
  "name": "Your Company",
  "accountcode": "YOUR_ACCOUNT_CODE",
  "did": "1234567890",
  "phone_number": "9876543210",
  "allocated_channels": 100,
  "rate_limit": 1000,
  "callbackurl": "https://your-domain.com/callback",
  "is_active": true,
  "status": "active",
  "created_at": "2024-01-01T00:00:00Z",
  "updated_at": "2024-01-01T00:00:00Z"
}
```

#### Regenerate API Key
```http
POST /clients/regenerate-api-key
```
**Response:**
```json
{
  "id": 1,
  "api_key": "new_generated_api_key_here"
}
```

### 3. Server Allocation Endpoints

#### Get Allocated Servers
```http
GET /clients/allocated-servers
```
**Response:**
```json
{
  "allocated_servers": {
    "server1": {
      "allowed_active_calls": 50,
      "rate_limit": 500,
      "status": "active"
    },
    "server2": {
      "allowed_active_calls": 30,
      "rate_limit": 300,
      "status": "active"
    }
  }
}
```

---

## Server Allocation & Configuration

### Understanding Server Allocation

The system dynamically allocates Asterisk servers to clients based on:
- **Available server capacity**
- **Client's allocated channels**
- **Rate limits per server**
- **Server health status**

### Server Configuration Parameters

Each allocated server includes:

```json
{
  "server_code": "server1",
  "allowed_active_calls": 50,
  "rate_limit": 500,
  "default_directory": "/var/spool/asterisk/monitor",
  "recording_directory": "/var/spool/asterisk/recordings",
  "provided_prefix": ["0", "1"],
  "context": ["default", "internal"],
  "allocated_dids": ["1234567890", "0987654321"],
  "provided_endpoints": ["1001", "1002"],
  "status": "active"
}
```

### Channel Allocation

Channels are allocated based on:
- **Client's allocated_channels setting**
- **Server capacity**
- **Current active calls**
- **Rate limiting rules**

---

## Call Queue Management

### 1. Insert Call Queue Data

#### Endpoint:
```http
POST /queues/insertdataincallqueue/
```

#### Request Body:
```json
{
  "batch_id": 12345,
  "data": [
    {
      "phone_number": "9876543210",
      "custom_data": {
        "customer_id": "CUST001",
        "campaign_id": "CAMP001"
      }
    },
    {
      "phone_number": "9876543211",
      "custom_data": {
        "customer_id": "CUST002",
        "campaign_id": "CAMP001"
      }
    }
  ]
}
```

#### Response:
```json
{
  "message": "Queue data inserted successfully",
  "status": "success",
  "batch_id": 12345,
  "inserted_count": 2
}
```

### 2. Quick Call (Single Call)

#### Endpoint:
```http
POST /queues/quickcall/
```

#### Request Body:
```json
{
  "phone_number": "9876543210",
  "custom_data": {
    "customer_id": "CUST001",
    "campaign_id": "CAMP001"
  }
}
```

### 3. Queue Management

#### Get Queue Masters
```http
GET /clients/queue-masters/{client_id}?page=1&size=10
```

#### Get Queue Entries
```http
GET /clients/calling-queue-entries/{client_id}/{queue_master_id}?page=1&size=10
```

### 4. Batch Operations

#### Dispatch Calls
```http
POST /controls/dispatch-calls?accountcode=YOUR_ACCOUNT_CODE
```

#### Setup Channels
```http
POST /controls/setup-channels/?accountcode=YOUR_ACCOUNT_CODE
```

---

## Monitoring & Reporting

### 1. Call Summary

#### Get Call Summary
```http
GET /monitoring/callingsummary?accountcode=YOUR_ACCOUNT_CODE&server_code=server1
```

**Response:**
```json
{
  "total_calls": 1500,
  "successful_calls": 1200,
  "failed_calls": 300,
  "success_rate": 80.0,
  "average_call_duration": 120.5,
  "total_duration": 180750.0
}
```

### 2. Client Call Details

#### Get Call Details
```http
GET /client-call-details/details?accountcode=YOUR_ACCOUNT_CODE&start_date=2024-01-01&end_date=2024-01-31
```

#### Get Call Summary with Time Periods
```http
GET /client-call-details/summary?accountcode=YOUR_ACCOUNT_CODE&start_date=2024-01-01&end_date=2024-01-31
```

**Response:**
```json
{
  "summary": {
    "Before 09AM": {"calls": 50, "duration": 6000},
    "09AM-11AM": {"calls": 200, "duration": 24000},
    "11AM-01PM": {"calls": 300, "duration": 36000},
    "01PM-03PM": {"calls": 250, "duration": 30000},
    "03PM-05PM": {"calls": 200, "duration": 24000},
    "05PM-07PM": {"calls": 150, "duration": 18000},
    "After 7PM": {"calls": 100, "duration": 12000},
    "Grand Total": {"calls": 1250, "duration": 150000}
  }
}
```

### 3. Real-time Monitoring

#### Get Active Calls
```http
GET /monitoring/activecalls?accountcode=YOUR_ACCOUNT_CODE
```

#### Get Server Status
```http
GET /monitoring/serverstatus?server_code=server1
```

---

## WebSocket Integration

### Real-time Monitoring WebSocket

#### Connection URL:
```
wss://your-centralized-calling-domain.com/api/ws/v1/monitoring/ws?x-api-key=your_api_key_here
```

#### JavaScript Example:
```javascript
const ws = new WebSocket('wss://your-domain.com/api/ws/v1/monitoring/ws?x-api-key=your_api_key');

ws.onopen = function(event) {
    console.log('WebSocket connected');
    
    // Subscribe to call events
    ws.send(JSON.stringify({
        type: 'subscribe',
        events: ['call_started', 'call_ended', 'call_failed']
    }));
};

ws.onmessage = function(event) {
    const data = JSON.parse(event.data);
    console.log('Received event:', data);
    
    switch(data.type) {
        case 'call_started':
            handleCallStarted(data.payload);
            break;
        case 'call_ended':
            handleCallEnded(data.payload);
            break;
        case 'call_failed':
            handleCallFailed(data.payload);
            break;
    }
};

ws.onclose = function(event) {
    console.log('WebSocket disconnected');
    // Implement reconnection logic
};

ws.onerror = function(error) {
    console.error('WebSocket error:', error);
};
```

#### Python Example:
```python
import asyncio
import websockets
import json

async def monitor_calls():
    uri = "wss://your-domain.com/api/ws/v1/monitoring/ws?x-api-key=your_api_key"
    
    async with websockets.connect(uri) as websocket:
        # Subscribe to events
        await websocket.send(json.dumps({
            "type": "subscribe",
            "events": ["call_started", "call_ended", "call_failed"]
        }))
        
        async for message in websocket:
            data = json.loads(message)
            print(f"Received event: {data}")
            
            if data["type"] == "call_started":
                handle_call_started(data["payload"])
            elif data["type"] == "call_ended":
                handle_call_ended(data["payload"])
            elif data["type"] == "call_failed":
                handle_call_failed(data["payload"])

def handle_call_started(payload):
    print(f"Call started: {payload['phone_number']}")

def handle_call_ended(payload):
    print(f"Call ended: {payload['phone_number']}, Duration: {payload['duration']}")

def handle_call_failed(payload):
    print(f"Call failed: {payload['phone_number']}, Reason: {payload['reason']}")

# Run the monitoring
asyncio.run(monitor_calls())
```

---

## Error Handling

### Common HTTP Status Codes

| Status Code | Description | Action Required |
|-------------|-------------|-----------------|
| 200 | Success | Continue processing |
| 400 | Bad Request | Check request format and parameters |
| 401 | Unauthorized | Verify API key or JWT token |
| 403 | Forbidden | Check client permissions and status |
| 404 | Not Found | Verify endpoint URL and resource ID |
| 429 | Too Many Requests | Implement rate limiting and retry logic |
| 500 | Internal Server Error | Contact support team |

### Error Response Format

```json
{
  "detail": "Error description",
  "status_code": 400,
  "timestamp": "2024-01-01T12:00:00Z"
}
```

### Rate Limiting

The system implements rate limiting based on:
- **Client-level rate limits**
- **Server-level rate limits**
- **API endpoint rate limits**

#### Rate Limit Headers:
```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1640995200
```

#### Handling Rate Limits:
```python
import time
import requests

def make_request_with_retry(url, headers, max_retries=3):
    for attempt in range(max_retries):
        response = requests.get(url, headers=headers)
        
        if response.status_code == 200:
            return response.json()
        elif response.status_code == 429:
            # Rate limited, wait and retry
            retry_after = int(response.headers.get('Retry-After', 60))
            print(f"Rate limited. Waiting {retry_after} seconds...")
            time.sleep(retry_after)
        else:
            raise Exception(f"Request failed: {response.status_code}")
    
    raise Exception("Max retries exceeded")
```

---

## Integration Examples

### Complete Python Integration Example

```python
import requests
import json
import time
from datetime import datetime, timedelta

class CentralizedCallingClient:
    def __init__(self, base_url, api_key):
        self.base_url = base_url.rstrip('/')
        self.api_key = api_key
        self.headers = {
            'X-API-KEY': api_key,
            'Content-Type': 'application/json'
        }
    
    def health_check(self):
        """Check API health"""
        response = requests.get(f"{self.base_url}/api/v1/monitoring/health", headers=self.headers)
        return response.json()
    
    def get_client_info(self):
        """Get client information"""
        response = requests.get(f"{self.base_url}/api/v1/clients/me", headers=self.headers)
        return response.json()
    
    def insert_call_queue(self, batch_id, phone_numbers, custom_data=None):
        """Insert multiple phone numbers into call queue"""
        data = []
        for phone in phone_numbers:
            entry = {"phone_number": phone}
            if custom_data:
                entry["custom_data"] = custom_data
            data.append(entry)
        
        payload = {
            "batch_id": batch_id,
            "data": data
        }
        
        response = requests.post(
            f"{self.base_url}/api/v1/queues/insertdataincallqueue/",
            headers=self.headers,
            json=payload
        )
        return response.json()
    
    def make_quick_call(self, phone_number, custom_data=None):
        """Make a single quick call"""
        payload = {"phone_number": phone_number}
        if custom_data:
            payload["custom_data"] = custom_data
        
        response = requests.post(
            f"{self.base_url}/api/v1/queues/quickcall/",
            headers=self.headers,
            json=payload
        )
        return response.json()
    
    def get_call_summary(self, start_date, end_date, server_code=None):
        """Get call summary for date range"""
        params = {
            "accountcode": self.get_client_info()["accountcode"],
            "start_date": start_date,
            "end_date": end_date
        }
        if server_code:
            params["server_code"] = server_code
        
        response = requests.get(
            f"{self.base_url}/api/v1/client-call-details/summary",
            headers=self.headers,
            params=params
        )
        return response.json()
    
    def dispatch_calls(self):
        """Dispatch queued calls"""
        accountcode = self.get_client_info()["accountcode"]
        response = requests.post(
            f"{self.base_url}/api/v1/controls/dispatch-calls",
            headers=self.headers,
            params={"accountcode": accountcode}
        )
        return response.json()

# Usage Example
def main():
    # Initialize client
    client = CentralizedCallingClient(
        base_url="https://your-domain.com",
        api_key="your_api_key_here"
    )
    
    # Check health
    health = client.health_check()
    print(f"API Health: {health}")
    
    # Get client info
    info = client.get_client_info()
    print(f"Client: {info['name']} ({info['accountcode']})")
    
    # Insert call queue
    phone_numbers = ["9876543210", "9876543211", "9876543212"]
    batch_result = client.insert_call_queue(
        batch_id=12345,
        phone_numbers=phone_numbers,
        custom_data={"campaign_id": "CAMP001"}
    )
    print(f"Queue Insert Result: {batch_result}")
    
    # Dispatch calls
    dispatch_result = client.dispatch_calls()
    print(f"Dispatch Result: {dispatch_result}")
    
    # Get call summary for today
    today = datetime.now().strftime("%Y-%m-%d")
    summary = client.get_call_summary(today, today)
    print(f"Today's Call Summary: {summary}")

if __name__ == "__main__":
    main()
```

### JavaScript/Node.js Integration Example

```javascript
class CentralizedCallingClient {
    constructor(baseUrl, apiKey) {
        this.baseUrl = baseUrl.replace(/\/$/, '');
        this.apiKey = apiKey;
        this.headers = {
            'X-API-KEY': apiKey,
            'Content-Type': 'application/json'
        };
    }
    
    async makeRequest(endpoint, options = {}) {
        const url = `${this.baseUrl}${endpoint}`;
        const config = {
            headers: this.headers,
            ...options
        };
        
        try {
            const response = await fetch(url, config);
            const data = await response.json();
            
            if (!response.ok) {
                throw new Error(`HTTP ${response.status}: ${data.detail || 'Unknown error'}`);
            }
            
            return data;
        } catch (error) {
            console.error('API Request failed:', error);
            throw error;
        }
    }
    
    async healthCheck() {
        return this.makeRequest('/api/v1/monitoring/health');
    }
    
    async getClientInfo() {
        return this.makeRequest('/api/v1/clients/me');
    }
    
    async insertCallQueue(batchId, phoneNumbers, customData = null) {
        const data = phoneNumbers.map(phone => {
            const entry = { phone_number: phone };
            if (customData) {
                entry.custom_data = customData;
            }
            return entry;
        });
        
        return this.makeRequest('/api/v1/queues/insertdataincallqueue/', {
            method: 'POST',
            body: JSON.stringify({
                batch_id: batchId,
                data: data
            })
        });
    }
    
    async makeQuickCall(phoneNumber, customData = null) {
        const payload = { phone_number: phoneNumber };
        if (customData) {
            payload.custom_data = customData;
        }
        
        return this.makeRequest('/api/v1/queues/quickcall/', {
            method: 'POST',
            body: JSON.stringify(payload)
        });
    }
    
    async getCallSummary(startDate, endDate, serverCode = null) {
        const clientInfo = await this.getClientInfo();
        const params = new URLSearchParams({
            accountcode: clientInfo.accountcode,
            start_date: startDate,
            end_date: endDate
        });
        
        if (serverCode) {
            params.append('server_code', serverCode);
        }
        
        return this.makeRequest(`/api/v1/client-call-details/summary?${params}`);
    }
    
    async dispatchCalls() {
        const clientInfo = await this.getClientInfo();
        return this.makeRequest('/api/v1/controls/dispatch-calls', {
            method: 'POST',
            body: JSON.stringify({ accountcode: clientInfo.accountcode })
        });
    }
}

// Usage Example
async function main() {
    const client = new CentralizedCallingClient(
        'https://your-domain.com',
        'your_api_key_here'
    );
    
    try {
        // Check health
        const health = await client.healthCheck();
        console.log('API Health:', health);
        
        // Get client info
        const info = await client.getClientInfo();
        console.log(`Client: ${info.name} (${info.accountcode})`);
        
        // Insert call queue
        const phoneNumbers = ['9876543210', '9876543211', '9876543212'];
        const batchResult = await client.insertCallQueue(
            12345,
            phoneNumbers,
            { campaign_id: 'CAMP001' }
        );
        console.log('Queue Insert Result:', batchResult);
        
        // Dispatch calls
        const dispatchResult = await client.dispatchCalls();
        console.log('Dispatch Result:', dispatchResult);
        
        // Get call summary for today
        const today = new Date().toISOString().split('T')[0];
        const summary = await client.getCallSummary(today, today);
        console.log("Today's Call Summary:", summary);
        
    } catch (error) {
        console.error('Error:', error.message);
    }
}

// Run the example
main();
```

---

## Troubleshooting Guide

### Common Issues and Solutions

#### 1. Authentication Errors

**Problem**: `401 Unauthorized` errors
**Solutions**:
- Verify API key is correct and not expired
- Check if API key is properly included in headers
- Ensure client account is active
- Verify JWT token is not expired (if using JWT authentication)

#### 2. Rate Limiting Issues

**Problem**: `429 Too Many Requests` errors
**Solutions**:
- Implement exponential backoff retry logic
- Reduce request frequency
- Check rate limit headers for reset time
- Contact administrator to increase rate limits

#### 3. Server Allocation Issues

**Problem**: No servers allocated or calls not going through
**Solutions**:
- Check client's allocated_servers configuration
- Verify server status and health
- Ensure sufficient channel allocation
- Contact administrator for server allocation

#### 4. Call Queue Issues

**Problem**: Calls not being processed from queue
**Solutions**:
- Verify batch_id is unique and valid
- Check phone number format
- Ensure client is within active time window
- Verify server capacity and rate limits

#### 5. WebSocket Connection Issues

**Problem**: WebSocket connection fails or disconnects
**Solutions**:
- Check network connectivity and firewall settings
- Verify WebSocket URL and API key
- Implement reconnection logic
- Check for proxy or load balancer issues

### Debugging Tips

#### 1. Enable Detailed Logging
```python
import logging
logging.basicConfig(level=logging.DEBUG)
```

#### 2. Check API Response Headers
```python
response = requests.get(url, headers=headers)
print("Status Code:", response.status_code)
print("Headers:", dict(response.headers))
print("Response:", response.text)
```

#### 3. Validate Request Format
```python
import json
print("Request JSON:", json.dumps(payload, indent=2))
```

#### 4. Monitor Rate Limits
```python
def check_rate_limits(response):
    if 'X-RateLimit-Limit' in response.headers:
        print(f"Rate Limit: {response.headers['X-RateLimit-Remaining']}/{response.headers['X-RateLimit-Limit']}")
        print(f"Reset Time: {response.headers['X-RateLimit-Reset']}")
```

### Support Contacts

For technical support and assistance:
- **Email**: support@your-domain.com
- **Documentation**: https://docs.your-domain.com
- **API Status**: https://status.your-domain.com

### Best Practices

1. **Always implement error handling and retry logic**
2. **Use appropriate rate limiting in your application**
3. **Store API keys securely and never expose them**
4. **Monitor API responses and handle edge cases**
5. **Implement proper logging for debugging**
6. **Test with small batches before processing large volumes**
7. **Keep your integration updated with API changes**

---

## Conclusion

This documentation provides comprehensive guidance for integrating with the Centralized Calling System. Follow the step-by-step processes outlined above to ensure a smooth onboarding experience. For additional support or questions not covered in this documentation, please contact the support team.

**Remember**: Always test your integration in a development environment before deploying to production.
