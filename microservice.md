# Microservice Architecture Specification

This document details the architectural specifications of each microservice within the AWS EC2-based infrastructure, including service responsibilities, dependencies, deployment patterns, and scaling considerations.

## Service Architecture Overview

### Core Services

| Service | Purpose | Public Access | Dependencies |
|---------|----------|---------------|---------------|
| Auth Service | User authentication and token management | Yes | MongoDB |
| Admin Panel | System configuration and monitoring | Yes | All Services |
| AI Proxy | ML model inference and rate limiting | No | MongoDB |
| MCP | Master Control Program for orchestration | No | All Services |

### Integration Services

| Service | Purpose | Public Access | Dependencies |
|---------|----------|---------------|---------------|
| Gmail Service | Email processing and management | No | MongoDB, Gmail API |
| Calendar Service | Calendar event management | No | MongoDB, Google Calendar API |
| Reminder Service | Notification scheduling | No | MongoDB, Calendar Service |
| Telegram Bot | User interaction interface | Yes | MongoDB, Telegram API |

## Service Specifications

### Auth Service

1. Core Responsibilities:
   - User authentication
   - JWT token issuance
   - Session management
   - Password reset workflows

2. Technical Implementation:
   ```javascript
   // Token Generation
   const generateToken = async (user) => {
     return jwt.sign(
       { id: user._id, role: user.role },
       process.env.JWT_SECRET,
       { expiresIn: '24h' }
     );
   };
   ```

### Admin Panel

1. Core Responsibilities:
   - System configuration management
   - Service health monitoring
   - User management
   - OAuth2 credential management

2. Access Control:
   - Role-based authentication
   - Admin-only routes
   - Audit logging

### AI Proxy Service

1. Core Responsibilities:
   - ML model inference
   - Rate limiting
   - Token management
   - Response caching

2. Scaling Configuration:
   ```json
   {
     "AutoScalingGroup": {
       "MinSize": 1,
       "MaxSize": 5,
       "DesiredCapacity": 2,
       "HealthCheckType": "ELB"
     }
   }
   ```

### MCP (Master Control Program)

1. Core Responsibilities:
   - Service orchestration
   - Task scheduling
   - System state management
   - Cross-service communication

2. Integration Pattern:
   - REST API endpoints
   - Internal service discovery
   - State synchronization

### Gmail Service

1. Core Responsibilities:
   - Email processing
   - OAuth2 token management
   - MIME parsing
   - Attachment handling

2. Security Implementation:
   - OAuth2 refresh token rotation
   - Secure credential storage
   - Rate limit compliance

### Calendar Service

1. Core Responsibilities:
   - Event management
   - Calendar synchronization
   - Recurring event handling
   - Timezone management

2. Integration Points:
   - Google Calendar API
   - Reminder Service
   - User preferences

### Reminder Service

1. Core Responsibilities:
   - Notification scheduling
   - Delivery management
   - Retry handling
   - User preferences

2. Technical Implementation:
   ```javascript
   // Notification Scheduler
   const scheduleNotification = async (event) => {
     return await Queue.add('notifications', {
       userId: event.userId,
       eventId: event._id,
       scheduledFor: event.startTime
     });
   };
   ```

### Telegram Bot Service

1. Core Responsibilities:
   - Command processing
   - User interaction
   - Message formatting
   - State management

2. Integration Pattern:
   - Webhook endpoints
   - Command routing
   - Response formatting

## Shared Infrastructure

### MongoDB Instance

1. Configuration:
   ```javascript
   const mongoConfig = {
     replicaSet: 'rs0',
     writeConcern: { w: 'majority' },
     readPreference: 'primaryPreferred'
   };
   ```

2. Access Pattern:
   - Service-specific collections
   - Indexed queries
   - Connection pooling
