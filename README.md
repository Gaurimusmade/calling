# Client-Side Changes Required for Centralized Calling Integration

## Table of Contents
1. [Overview](#overview)
2. [Database Setup Requirements](#database-setup-requirements)
3. [Callback URL Implementation](#callback-url-implementation)
4. [Client Database Tables](#client-database-tables)
5. [API Integration Changes](#api-integration-changes)
6. [Configuration Changes](#configuration-changes)
7. [Monitoring and Logging Setup](#monitoring-and-logging-setup)
8. [Security Considerations](#security-considerations)
9. [Testing and Validation](#testing-and-validation)
10. [Migration Checklist](#migration-checklist)

---

## Overview

This document outlines the specific changes that clients need to make on their side to integrate with the Centralized Calling System. These changes include database modifications, callback implementations, configuration updates, and API integration adjustments.

---

## Database Setup Requirements

### 1. PostgreSQL Database Configuration

#### Required Database Setup:
```sql
-- Create database for client (if using separate database)
CREATE DATABASE client_calling_db;

-- Create user with appropriate permissions
CREATE USER client_user WITH PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE client_calling_db TO client_user;
```

#### Database Connection URLs:
You need to provide these URLs during client registration:
- **Async Database URL**: `postgresql+asyncpg://user:pass@host:port/db`
- **Sync Database URL**: `postgresql+psycopg://user:pass@host:port/db`

---

## Callback URL Implementation

### 1. Callback Endpoint Setup

You must implement a callback endpoint to receive call status updates from the centralized system.

#### Required Callback Endpoint:
```http
POST https://your-domain.com/callback
Content-Type: application/json
```

#### Callback Payload Structure:
```json
{
  "event_type": "call_status_update",
  "accountcode": "YOUR_ACCOUNT_CODE",
  "batch_id": 12345,
  "call_id": "unique_call_id",
  "phone_number": "9876543210",
  "call_status": "completed",
  "disposition": "answered",
  "duration": 120,
  "start_time": "2024-01-01T10:00:00Z",
  "end_time": "2024-01-01T10:02:00Z",
  "recording_url": "https://recordings.example.com/call_123.wav",
  "custom_data": {
    "customer_id": "CUST001",
    "campaign_id": "CAMP001"
  },
  "server_code": "server1",
  "error_message": null
}
```

#### Callback Event Types:
- `call_started`: Call initiation
- `call_answered`: Call answered by recipient
- `call_completed`: Call finished successfully
- `call_failed`: Call failed with error
- `call_busy`: Recipient line busy
- `call_no_answer`: No answer from recipient
- `call_timeout`: Call timed out

### 2. Callback Implementation Examples

#### Python Flask Implementation:
```python
from flask import Flask, request, jsonify
import logging

app = Flask(__name__)
logging.basicConfig(level=logging.INFO)

@app.route('/callback', methods=['POST'])
def handle_callback():
    try:
        data = request.get_json()
        
        # Validate callback data
        if not data or 'event_type' not in data:
            return jsonify({'error': 'Invalid callback data'}), 400
        
        # Process callback based on event type
        event_type = data['event_type']
        
        if event_type == 'call_started':
            handle_call_started(data)
        elif event_type == 'call_completed':
            handle_call_completed(data)
        elif event_type == 'call_failed':
            handle_call_failed(data)
        else:
            handle_other_event(data)
        
        return jsonify({'status': 'success'}), 200
        
    except Exception as e:
        logging.error(f"Callback processing error: {e}")
        return jsonify({'error': 'Internal server error'}), 500

def handle_call_started(data):
    """Handle call started event"""
    logging.info(f"Call started: {data['phone_number']}")
    # Update your database with call start information
    # Example: update_call_status(data['call_id'], 'started')

def handle_call_completed(data):
    """Handle call completed event"""
    logging.info(f"Call completed: {data['phone_number']}, Duration: {data['duration']}")
    # Update your database with call completion information
    # Example: update_call_status(data['call_id'], 'completed', data['duration'])

def handle_call_failed(data):
    """Handle call failed event"""
    logging.error(f"Call failed: {data['phone_number']}, Error: {data.get('error_message')}")
    # Update your database with call failure information
    # Example: update_call_status(data['call_id'], 'failed', error=data.get('error_message'))

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, ssl_context='adhoc')
```

#### Node.js Express Implementation:
```javascript
const express = require('express');
const app = express();

app.use(express.json());

app.post('/callback', (req, res) => {
    try {
        const data = req.body;
        
        // Validate callback data
        if (!data || !data.event_type) {
            return res.status(400).json({ error: 'Invalid callback data' });
        }
        
        // Process callback based on event type
        switch (data.event_type) {
            case 'call_started':
                handleCallStarted(data);
                break;
            case 'call_completed':
                handleCallCompleted(data);
                break;
            case 'call_failed':
                handleCallFailed(data);
                break;
            default:
                handleOtherEvent(data);
        }
        
        res.json({ status: 'success' });
        
    } catch (error) {
        console.error('Callback processing error:', error);
        res.status(500).json({ error: 'Internal server error' });
    }
});

function handleCallStarted(data) {
    console.log(`Call started: ${data.phone_number}`);
    // Update your database with call start information
}

function handleCallCompleted(data) {
    console.log(`Call completed: ${data.phone_number}, Duration: ${data.duration}`);
    // Update your database with call completion information
}

function handleCallFailed(data) {
    console.error(`Call failed: ${data.phone_number}, Error: ${data.error_message}`);
    // Update your database with call failure information
}

app.listen(5000, () => {
    console.log('Callback server running on port 5000');
});
```

---

## Client Database Tables

### 1. Required Table Structures

You need to create these tables in your database to store call-related data:

#### Call Logs Table:
```sql
CREATE TABLE call_logs (
    id SERIAL PRIMARY KEY,
    call_id VARCHAR(255) UNIQUE NOT NULL,
    accountcode VARCHAR(100) NOT NULL,
    batch_id INTEGER,
    phone_number VARCHAR(20) NOT NULL,
    call_status VARCHAR(50) NOT NULL,
    disposition VARCHAR(50),
    duration INTEGER DEFAULT 0,
    start_time TIMESTAMP,
    end_time TIMESTAMP,
    recording_url TEXT,
    server_code VARCHAR(100),
    error_message TEXT,
    custom_data JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Indexes for better performance
CREATE INDEX idx_call_logs_accountcode ON call_logs(accountcode);
CREATE INDEX idx_call_logs_batch_id ON call_logs(batch_id);
CREATE INDEX idx_call_logs_phone_number ON call_logs(phone_number);
CREATE INDEX idx_call_logs_call_status ON call_logs(call_status);
CREATE INDEX idx_call_logs_created_at ON call_logs(created_at);
```

#### Call Batches Table:
```sql
CREATE TABLE call_batches (
    id SERIAL PRIMARY KEY,
    batch_id INTEGER UNIQUE NOT NULL,
    accountcode VARCHAR(100) NOT NULL,
    batch_name VARCHAR(255),
    total_calls INTEGER DEFAULT 0,
    completed_calls INTEGER DEFAULT 0,
    failed_calls INTEGER DEFAULT 0,
    batch_status VARCHAR(50) DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Indexes
CREATE INDEX idx_call_batches_accountcode ON call_batches(accountcode);
CREATE INDEX idx_call_batches_batch_status ON call_batches(batch_status);
```

#### Call Queue Table (Optional - for local queue management):
```sql
CREATE TABLE local_call_queue (
    id SERIAL PRIMARY KEY,
    phone_number VARCHAR(20) NOT NULL,
    batch_id INTEGER,
    custom_data JSONB,
    queue_status VARCHAR(50) DEFAULT 'pending',
    retry_count INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Indexes
CREATE INDEX idx_local_queue_batch_id ON local_call_queue(batch_id);
CREATE INDEX idx_local_queue_status ON local_call_queue(queue_status);
```

### 2. Database Migration Scripts

#### Python Alembic Migration:
```python
# migrations/versions/001_add_calling_tables.py
"""Add calling system tables

Revision ID: 001
Revises: 
Create Date: 2024-01-01 10:00:00.000000

"""
from alembic import op
import sqlalchemy as sa

# revision identifiers
revision = '001'
down_revision = None
branch_labels = None
depends_on = None

def upgrade():
    # Create call_logs table
    op.create_table('call_logs',
        sa.Column('id', sa.Integer(), nullable=False),
        sa.Column('call_id', sa.String(255), nullable=False),
        sa.Column('accountcode', sa.String(100), nullable=False),
        sa.Column('batch_id', sa.Integer(), nullable=True),
        sa.Column('phone_number', sa.String(20), nullable=False),
        sa.Column('call_status', sa.String(50), nullable=False),
        sa.Column('disposition', sa.String(50), nullable=True),
        sa.Column('duration', sa.Integer(), nullable=True),
        sa.Column('start_time', sa.DateTime(), nullable=True),
        sa.Column('end_time', sa.DateTime(), nullable=True),
        sa.Column('recording_url', sa.Text(), nullable=True),
        sa.Column('server_code', sa.String(100), nullable=True),
        sa.Column('error_message', sa.Text(), nullable=True),
        sa.Column('custom_data', sa.JSON(), nullable=True),
        sa.Column('created_at', sa.DateTime(), nullable=True),
        sa.Column('updated_at', sa.DateTime(), nullable=True),
        sa.PrimaryKeyConstraint('id'),
        sa.UniqueConstraint('call_id')
    )
    
    # Create indexes
    op.create_index('idx_call_logs_accountcode', 'call_logs', ['accountcode'])
    op.create_index('idx_call_logs_batch_id', 'call_logs', ['batch_id'])
    op.create_index('idx_call_logs_phone_number', 'call_logs', ['phone_number'])
    op.create_index('idx_call_logs_call_status', 'call_logs', ['call_status'])
    op.create_index('idx_call_logs_created_at', 'call_logs', ['created_at'])

def downgrade():
    op.drop_table('call_logs')
```

---

## API Integration Changes

### 1. Update Your API Client

#### Python Integration:
```python
import requests
import json
from datetime import datetime

class CentralizedCallingClient:
    def __init__(self, base_url, api_key):
        self.base_url = base_url.rstrip('/')
        self.api_key = api_key
        self.headers = {
            'X-API-KEY': api_key,
            'Content-Type': 'application/json'
        }
    
    def submit_call_batch(self, batch_id, phone_numbers, custom_data=None):
        """Submit a batch of phone numbers for calling"""
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
    
    def get_call_status(self, batch_id):
        """Get status of calls in a batch"""
        response = requests.get(
            f"{self.base_url}/api/v1/queues/batch-status/{batch_id}",
            headers=self.headers
        )
        return response.json()
    
    def dispatch_calls(self):
        """Dispatch queued calls"""
        response = requests.post(
            f"{self.base_url}/api/v1/controls/dispatch-calls",
            headers=self.headers
        )
        return response.json()
```

### 2. Update Your Application Logic

#### Call Processing Workflow:
```python
def process_calling_campaign(campaign_data):
    """Process a calling campaign using centralized system"""
    
    # Initialize centralized calling client
    calling_client = CentralizedCallingClient(
        base_url="https://centralized-calling.example.com",
        api_key="your_api_key_here"
    )
    
    # Generate unique batch ID
    batch_id = generate_batch_id()
    
    # Prepare phone numbers
    phone_numbers = campaign_data['phone_numbers']
    custom_data = {
        'campaign_id': campaign_data['campaign_id'],
        'customer_segment': campaign_data['segment']
    }
    
    # Submit batch to centralized system
    result = calling_client.submit_call_batch(
        batch_id=batch_id,
        phone_numbers=phone_numbers,
        custom_data=custom_data
    )
    
    if result.get('status') == 'success':
        # Dispatch calls
        dispatch_result = calling_client.dispatch_calls()
        
        # Update local database
        update_local_batch_status(batch_id, 'submitted')
        
        return {
            'batch_id': batch_id,
            'status': 'submitted',
            'total_calls': len(phone_numbers)
        }
    else:
        raise Exception(f"Failed to submit batch: {result.get('message')}")

def handle_callback_update(callback_data):
    """Handle callback updates from centralized system"""
    
    call_id = callback_data['call_id']
    phone_number = callback_data['phone_number']
    call_status = callback_data['call_status']
    
    # Update local database
    update_call_log(
        call_id=call_id,
        phone_number=phone_number,
        call_status=call_status,
        duration=callback_data.get('duration'),
        disposition=callback_data.get('disposition'),
        recording_url=callback_data.get('recording_url'),
        error_message=callback_data.get('error_message')
    )
    
    # Update batch statistics
    update_batch_statistics(callback_data['batch_id'], call_status)
    
    # Trigger any business logic based on call status
    if call_status == 'completed':
        handle_successful_call(callback_data)
    elif call_status == 'failed':
        handle_failed_call(callback_data)
```

---

## Configuration Changes

### 1. Environment Variables

Add these environment variables to your configuration:

```bash
# Centralized Calling System Configuration
CENTRALIZED_CALLING_BASE_URL=https://centralized-calling.example.com
CENTRALIZED_CALLING_API_KEY=your_api_key_here
CENTRALIZED_CALLING_CALLBACK_URL=https://your-domain.com/callback

# Database Configuration (if using separate database)
CLIENT_CALLING_DB_URL=postgresql://user:pass@host:port/client_calling_db
CLIENT_CALLING_ASYNC_DB_URL=postgresql+asyncpg://user:pass@host:port/client_calling_db

# Callback Security
CALLBACK_SECRET_KEY=your_callback_secret_key
CALLBACK_VERIFICATION_ENABLED=true
```

### 2. Application Configuration

#### Python Configuration:
```python
# config.py
import os
from dataclasses import dataclass

@dataclass
class CentralizedCallingConfig:
    base_url: str = os.getenv('CENTRALIZED_CALLING_BASE_URL')
    api_key: str = os.getenv('CENTRALIZED_CALLING_API_KEY')
    callback_url: str = os.getenv('CENTRALIZED_CALLING_CALLBACK_URL')
    callback_secret: str = os.getenv('CALLBACK_SECRET_KEY')
    verification_enabled: bool = os.getenv('CALLBACK_VERIFICATION_ENABLED', 'true').lower() == 'true'
    
    def __post_init__(self):
        if not all([self.base_url, self.api_key, self.callback_url]):
            raise ValueError("Missing required centralized calling configuration")
```

#### Node.js Configuration:
```javascript
// config.js
module.exports = {
    centralizedCalling: {
        baseUrl: process.env.CENTRALIZED_CALLING_BASE_URL,
        apiKey: process.env.CENTRALIZED_CALLING_API_KEY,
        callbackUrl: process.env.CENTRALIZED_CALLING_CALLBACK_URL,
        callbackSecret: process.env.CALLBACK_SECRET_KEY,
        verificationEnabled: process.env.CALLBACK_VERIFICATION_ENABLED === 'true'
    }
};
```

---

## Monitoring and Logging Setup

### 1. Logging Configuration

#### Python Logging Setup:
```python
import logging
import logging.handlers
from datetime import datetime

def setup_calling_logger():
    """Setup dedicated logger for calling system"""
    
    logger = logging.getLogger('calling_system')
    logger.setLevel(logging.INFO)
    
    # Create file handler with rotation
    file_handler = logging.handlers.RotatingFileHandler(
        'logs/calling_system.log',
        maxBytes=10*1024*1024,  # 10MB
        backupCount=5
    )
    
    # Create console handler
    console_handler = logging.StreamHandler()
    
    # Create formatter
    formatter = logging.Formatter(
        '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    )
    
    file_handler.setFormatter(formatter)
    console_handler.setFormatter(formatter)
    
    logger.addHandler(file_handler)
    logger.addHandler(console_handler)
    
    return logger

# Usage
calling_logger = setup_calling_logger()
```

### 2. Monitoring Metrics

#### Key Metrics to Track:
```python
from prometheus_client import Counter, Histogram, Gauge
import time

# Metrics
call_submissions_total = Counter('call_submissions_total', 'Total call submissions', ['status'])
call_duration_seconds = Histogram('call_duration_seconds', 'Call duration in seconds', ['status'])
active_batches = Gauge('active_batches', 'Number of active batches')
callback_processing_time = Histogram('callback_processing_time_seconds', 'Callback processing time')

def track_call_submission(status):
    """Track call submission metrics"""
    call_submissions_total.labels(status=status).inc()

def track_call_duration(duration, status):
    """Track call duration metrics"""
    call_duration_seconds.labels(status=status).observe(duration)

def track_callback_processing(func):
    """Decorator to track callback processing time"""
    def wrapper(*args, **kwargs):
        start_time = time.time()
        try:
            result = func(*args, **kwargs)
            callback_processing_time.observe(time.time() - start_time)
            return result
        except Exception as e:
            callback_processing_time.observe(time.time() - start_time)
            raise
    return wrapper
```

---

## Security Considerations

### 1. Callback Security

#### Implement Callback Verification:
```python
import hmac
import hashlib
from flask import request, abort

def verify_callback_signature(payload, signature, secret):
    """Verify callback signature"""
    expected_signature = hmac.new(
        secret.encode('utf-8'),
        payload.encode('utf-8'),
        hashlib.sha256
    ).hexdigest()
    
    return hmac.compare_digest(signature, expected_signature)

@app.route('/callback', methods=['POST'])
def secure_callback():
    # Get signature from headers
    signature = request.headers.get('X-Signature')
    if not signature:
        abort(401, 'Missing signature')
    
    # Verify signature
    payload = request.get_data(as_text=True)
    if not verify_callback_signature(payload, signature, app.config['CALLBACK_SECRET']):
        abort(401, 'Invalid signature')
    
    # Process callback
    data = request.get_json()
    # ... rest of callback processing
```

### 2. API Key Security

#### Secure API Key Storage:
```python
import os
from cryptography.fernet import Fernet

class SecureConfig:
    def __init__(self):
        self.encryption_key = os.getenv('ENCRYPTION_KEY')
        self.cipher = Fernet(self.encryption_key.encode())
    
    def encrypt_api_key(self, api_key):
        """Encrypt API key for storage"""
        return self.cipher.encrypt(api_key.encode()).decode()
    
    def decrypt_api_key(self, encrypted_key):
        """Decrypt API key for use"""
        return self.cipher.decrypt(encrypted_key.encode()).decode()
```

---

## Testing and Validation

### 1. Unit Tests

#### Python Test Examples:
```python
import unittest
from unittest.mock import patch, Mock
import requests

class TestCentralizedCallingIntegration(unittest.TestCase):
    
    def setUp(self):
        self.client = CentralizedCallingClient(
            base_url="https://test.example.com",
            api_key="test_api_key"
        )
    
    @patch('requests.post')
    def test_submit_call_batch(self, mock_post):
        """Test call batch submission"""
        mock_response = Mock()
        mock_response.json.return_value = {'status': 'success', 'batch_id': 123}
        mock_response.status_code = 200
        mock_post.return_value = mock_response
        
        result = self.client.submit_call_batch(
            batch_id=123,
            phone_numbers=['9876543210', '9876543211']
        )
        
        self.assertEqual(result['status'], 'success')
        self.assertEqual(result['batch_id'], 123)
    
    def test_callback_processing(self):
        """Test callback processing"""
        callback_data = {
            'event_type': 'call_completed',
            'call_id': 'test_call_123',
            'phone_number': '9876543210',
            'duration': 120,
            'disposition': 'answered'
        }
        
        # Test callback processing logic
        result = handle_callback_update(callback_data)
        self.assertIsNotNone(result)
```

### 2. Integration Tests

#### End-to-End Test:
```python
def test_end_to_end_calling_flow():
    """Test complete calling flow"""
    
    # 1. Submit batch
    batch_id = 12345
    phone_numbers = ['9876543210', '9876543211']
    
    result = calling_client.submit_call_batch(
        batch_id=batch_id,
        phone_numbers=phone_numbers
    )
    
    assert result['status'] == 'success'
    
    # 2. Dispatch calls
    dispatch_result = calling_client.dispatch_calls()
    assert dispatch_result['status'] == 'success'
    
    # 3. Simulate callback
    callback_data = {
        'event_type': 'call_completed',
        'batch_id': batch_id,
        'phone_number': phone_numbers[0],
        'call_status': 'completed',
        'duration': 120
    }
    
    # Process callback
    handle_callback_update(callback_data)
    
    # 4. Verify database update
    call_log = get_call_log(batch_id, phone_numbers[0])
    assert call_log['call_status'] == 'completed'
    assert call_log['duration'] == 120
```

---

## Migration Checklist

### Pre-Migration Checklist:
- [ ] **Database Setup**
  - [ ] Create PostgreSQL database
  - [ ] Create required tables (call_logs, call_batches, local_call_queue)
  - [ ] Set up database indexes
  - [ ] Configure database connections

- [ ] **Callback Implementation**
  - [ ] Implement callback endpoint
  - [ ] Set up SSL/TLS for callback URL
  - [ ] Implement callback signature verification
  - [ ] Test callback endpoint accessibility

- [ ] **API Integration**
  - [ ] Update API client code
  - [ ] Implement batch submission logic
  - [ ] Add call status monitoring
  - [ ] Update error handling

- [ ] **Configuration**
  - [ ] Add environment variables
  - [ ] Update application configuration
  - [ ] Set up logging and monitoring
  - [ ] Configure security settings

### Migration Steps:
1. [ ] **Backup Current System**
   - [ ] Backup existing call data
   - [ ] Document current call flows
   - [ ] Create rollback plan

2. [ ] **Deploy Changes**
   - [ ] Deploy database changes
   - [ ] Deploy application updates
   - [ ] Update configuration
   - [ ] Test in staging environment

3. [ ] **Register with Centralized System**
   - [ ] Submit client registration request
   - [ ] Provide callback URL
   - [ ] Receive API key
   - [ ] Test API connectivity

4. [ ] **Validation**
   - [ ] Run integration tests
   - [ ] Test callback processing
   - [ ] Validate call flow
   - [ ] Monitor system performance

### Post-Migration Checklist:
- [ ] **Monitoring**
  - [ ] Set up monitoring dashboards
  - [ ] Configure alerting
  - [ ] Monitor call success rates
  - [ ] Track system performance

- [ ] **Documentation**
  - [ ] Update internal documentation
  - [ ] Document new call flows
  - [ ] Create troubleshooting guides
  - [ ] Train support team

- [ ] **Optimization**
  - [ ] Analyze call patterns
  - [ ] Optimize batch sizes
  - [ ] Fine-tune rate limits
  - [ ] Improve error handling

---

## Support and Troubleshooting

### Common Issues and Solutions:

#### 1. Callback Not Receiving Updates
**Problem**: Callbacks not being received
**Solutions**:
- Verify callback URL is accessible from internet
- Check SSL certificate validity
- Ensure callback endpoint returns 200 status
- Verify signature verification logic

#### 2. Database Connection Issues
**Problem**: Cannot connect to database
**Solutions**:
- Check database connection strings
- Verify database user permissions
- Ensure database server is accessible
- Check firewall settings

#### 3. API Authentication Failures
**Problem**: API requests failing with 401 errors
**Solutions**:
- Verify API key is correct
- Check API key format
- Ensure API key is not expired
- Verify request headers

### Support Contacts:
- **Technical Support**: support@centralized-calling.com
- **Documentation**: https://docs.centralized-calling.com
- **Status Page**: https://status.centralized-calling.com

---

## Conclusion

This documentation provides a comprehensive guide for implementing the necessary client-side changes to integrate with the Centralized Calling System. Follow the checklist and implementation examples to ensure a smooth migration and successful integration.

**Important Notes**:
- Test all changes in a staging environment before production deployment
- Implement proper error handling and monitoring
- Keep your API keys secure and never expose them in client-side code
- Monitor system performance and call success rates after migration
- Have a rollback plan ready in case of issues

For additional support or questions not covered in this documentation, please contact the technical support team.
