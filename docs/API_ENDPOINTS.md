# API Endpoints Documentation

**Last Updated:** January 9, 2026

## Table of Contents
- [Authentication](#authentication)
- [Base URL](#base-url)
- [API Routes](#api-routes)
- [Request/Response Formats](#requestresponse-formats)
- [Error Handling](#error-handling)
- [Rate Limiting](#rate-limiting)
- [Common Response Codes](#common-response-codes)

---

## Authentication

### Authentication Methods

#### Bearer Token Authentication
All API requests require authentication using a Bearer token in the Authorization header.

```
Authorization: Bearer <access_token>
```

#### API Key Authentication
Alternative authentication method using API key headers.

```
X-API-Key: <api_key>
```

### Token Acquisition

**Endpoint:** `POST /api/v1/auth/login`

**Request:**
```json
{
  "email": "user@example.com",
  "password": "secure_password"
}
```

**Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "refresh_token_value"
}
```

### Token Refresh

**Endpoint:** `POST /api/v1/auth/refresh`

**Request:**
```json
{
  "refresh_token": "refresh_token_value"
}
```

**Response:**
```json
{
  "access_token": "new_access_token",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

---

## Base URL

```
https://api.executive-assistant.com/api/v1
```

For development:
```
http://localhost:5000/api/v1
```

---

## API Routes

### Authentication Endpoints

#### Login
- **Method:** `POST`
- **Path:** `/auth/login`
- **Authentication:** None
- **Description:** Authenticate user and receive access token
- **Request Body:**
  ```json
  {
    "email": "string (required)",
    "password": "string (required)"
  }
  ```
- **Response:** `200 OK`
  ```json
  {
    "access_token": "string",
    "token_type": "Bearer",
    "expires_in": "integer (seconds)",
    "refresh_token": "string",
    "user": {
      "id": "string",
      "email": "string",
      "name": "string",
      "role": "string"
    }
  }
  ```

#### Logout
- **Method:** `POST`
- **Path:** `/auth/logout`
- **Authentication:** Required (Bearer Token)
- **Description:** Invalidate user session
- **Response:** `200 OK`
  ```json
  {
    "message": "Successfully logged out"
  }
  ```

#### Register
- **Method:** `POST`
- **Path:** `/auth/register`
- **Authentication:** None
- **Description:** Create new user account
- **Request Body:**
  ```json
  {
    "email": "string (required)",
    "password": "string (required, min 8 chars)",
    "name": "string (required)",
    "organization": "string (optional)"
  }
  ```
- **Response:** `201 Created`
  ```json
  {
    "id": "string",
    "email": "string",
    "name": "string",
    "created_at": "ISO 8601 timestamp"
  }
  ```

---

### User Endpoints

#### Get Current User
- **Method:** `GET`
- **Path:** `/users/me`
- **Authentication:** Required (Bearer Token)
- **Description:** Retrieve current authenticated user profile
- **Response:** `200 OK`
  ```json
  {
    "id": "string",
    "email": "string",
    "name": "string",
    "role": "string",
    "organization": "string",
    "created_at": "ISO 8601 timestamp",
    "updated_at": "ISO 8601 timestamp"
  }
  ```

#### Update User Profile
- **Method:** `PUT`
- **Path:** `/users/{userId}`
- **Authentication:** Required (Bearer Token)
- **Description:** Update user profile information
- **Request Body:**
  ```json
  {
    "name": "string (optional)",
    "email": "string (optional)",
    "phone": "string (optional)",
    "preferences": {
      "timezone": "string",
      "language": "string",
      "notifications_enabled": "boolean"
    }
  }
  ```
- **Response:** `200 OK`
  ```json
  {
    "id": "string",
    "email": "string",
    "name": "string",
    "updated_at": "ISO 8601 timestamp"
  }
  ```

#### Delete User Account
- **Method:** `DELETE`
- **Path:** `/users/{userId}`
- **Authentication:** Required (Bearer Token)
- **Description:** Delete user account (irreversible)
- **Response:** `204 No Content`

#### Change Password
- **Method:** `POST`
- **Path:** `/users/{userId}/change-password`
- **Authentication:** Required (Bearer Token)
- **Description:** Update user password
- **Request Body:**
  ```json
  {
    "current_password": "string (required)",
    "new_password": "string (required, min 8 chars)"
  }
  ```
- **Response:** `200 OK`
  ```json
  {
    "message": "Password updated successfully"
  }
  ```

---

### Tasks Endpoints

#### Get All Tasks
- **Method:** `GET`
- **Path:** `/tasks`
- **Authentication:** Required (Bearer Token)
- **Query Parameters:**
  - `status` (optional): `pending`, `in_progress`, `completed`, `cancelled`
  - `priority` (optional): `low`, `medium`, `high`, `urgent`
  - `page` (optional): Page number (default: 1)
  - `limit` (optional): Items per page (default: 20, max: 100)
  - `sort_by` (optional): `created_at`, `due_date`, `priority`
  - `sort_order` (optional): `asc`, `desc`
- **Description:** List all tasks for current user
- **Response:** `200 OK`
  ```json
  {
    "data": [
      {
        "id": "string",
        "title": "string",
        "description": "string",
        "status": "string",
        "priority": "string",
        "due_date": "ISO 8601 timestamp",
        "assigned_to": "string",
        "created_at": "ISO 8601 timestamp",
        "updated_at": "ISO 8601 timestamp"
      }
    ],
    "pagination": {
      "page": "integer",
      "limit": "integer",
      "total": "integer",
      "total_pages": "integer"
    }
  }
  ```

#### Create Task
- **Method:** `POST`
- **Path:** `/tasks`
- **Authentication:** Required (Bearer Token)
- **Description:** Create new task
- **Request Body:**
  ```json
  {
    "title": "string (required, max 255 chars)",
    "description": "string (optional)",
    "priority": "string (optional, default: medium)",
    "due_date": "ISO 8601 timestamp (optional)",
    "assigned_to": "string (optional, userId)",
    "tags": ["string (optional)"]
  }
  ```
- **Response:** `201 Created`
  ```json
  {
    "id": "string",
    "title": "string",
    "description": "string",
    "status": "pending",
    "priority": "string",
    "due_date": "ISO 8601 timestamp",
    "assigned_to": "string",
    "created_at": "ISO 8601 timestamp",
    "updated_at": "ISO 8601 timestamp"
  }
  ```

#### Get Single Task
- **Method:** `GET`
- **Path:** `/tasks/{taskId}`
- **Authentication:** Required (Bearer Token)
- **Description:** Retrieve specific task details
- **Response:** `200 OK`
  ```json
  {
    "id": "string",
    "title": "string",
    "description": "string",
    "status": "string",
    "priority": "string",
    "due_date": "ISO 8601 timestamp",
    "assigned_to": "string",
    "created_at": "ISO 8601 timestamp",
    "updated_at": "ISO 8601 timestamp",
    "comments": [
      {
        "id": "string",
        "author": "string",
        "text": "string",
        "created_at": "ISO 8601 timestamp"
      }
    ]
  }
  ```

#### Update Task
- **Method:** `PUT`
- **Path:** `/tasks/{taskId}`
- **Authentication:** Required (Bearer Token)
- **Description:** Update task details
- **Request Body:**
  ```json
  {
    "title": "string (optional)",
    "description": "string (optional)",
    "status": "string (optional)",
    "priority": "string (optional)",
    "due_date": "ISO 8601 timestamp (optional)",
    "assigned_to": "string (optional)"
  }
  ```
- **Response:** `200 OK`
  ```json
  {
    "id": "string",
    "title": "string",
    "updated_at": "ISO 8601 timestamp"
  }
  ```

#### Delete Task
- **Method:** `DELETE`
- **Path:** `/tasks/{taskId}`
- **Authentication:** Required (Bearer Token)
- **Description:** Delete task
- **Response:** `204 No Content`

#### Add Comment to Task
- **Method:** `POST`
- **Path:** `/tasks/{taskId}/comments`
- **Authentication:** Required (Bearer Token)
- **Description:** Add comment to task
- **Request Body:**
  ```json
  {
    "text": "string (required, max 5000 chars)"
  }
  ```
- **Response:** `201 Created`
  ```json
  {
    "id": "string",
    "task_id": "string",
    "author": "string",
    "text": "string",
    "created_at": "ISO 8601 timestamp"
  }
  ```

---

### Calendar Endpoints

#### Get Calendar Events
- **Method:** `GET`
- **Path:** `/calendar/events`
- **Authentication:** Required (Bearer Token)
- **Query Parameters:**
  - `start_date` (optional): ISO 8601 timestamp
  - `end_date` (optional): ISO 8601 timestamp
  - `page` (optional): Page number
  - `limit` (optional): Items per page
- **Description:** List calendar events
- **Response:** `200 OK`
  ```json
  {
    "data": [
      {
        "id": "string",
        "title": "string",
        "description": "string",
        "start_time": "ISO 8601 timestamp",
        "end_time": "ISO 8601 timestamp",
        "location": "string",
        "attendees": ["string"],
        "created_at": "ISO 8601 timestamp"
      }
    ],
    "pagination": {
      "page": "integer",
      "limit": "integer",
      "total": "integer"
    }
  }
  ```

#### Create Calendar Event
- **Method:** `POST`
- **Path:** `/calendar/events`
- **Authentication:** Required (Bearer Token)
- **Description:** Create new calendar event
- **Request Body:**
  ```json
  {
    "title": "string (required)",
    "description": "string (optional)",
    "start_time": "ISO 8601 timestamp (required)",
    "end_time": "ISO 8601 timestamp (required)",
    "location": "string (optional)",
    "attendees": ["string (optional, emails)"],
    "reminder_minutes": "integer (optional, default: 15)"
  }
  ```
- **Response:** `201 Created`
  ```json
  {
    "id": "string",
    "title": "string",
    "start_time": "ISO 8601 timestamp",
    "end_time": "ISO 8601 timestamp",
    "created_at": "ISO 8601 timestamp"
  }
  ```

#### Update Calendar Event
- **Method:** `PUT`
- **Path:** `/calendar/events/{eventId}`
- **Authentication:** Required (Bearer Token)
- **Description:** Update calendar event
- **Request Body:**
  ```json
  {
    "title": "string (optional)",
    "description": "string (optional)",
    "start_time": "ISO 8601 timestamp (optional)",
    "end_time": "ISO 8601 timestamp (optional)",
    "location": "string (optional)"
  }
  ```
- **Response:** `200 OK`

#### Delete Calendar Event
- **Method:** `DELETE`
- **Path:** `/calendar/events/{eventId}`
- **Authentication:** Required (Bearer Token)
- **Description:** Delete calendar event
- **Response:** `204 No Content`

---

### Notifications Endpoints

#### Get Notifications
- **Method:** `GET`
- **Path:** `/notifications`
- **Authentication:** Required (Bearer Token)
- **Query Parameters:**
  - `read` (optional): `true`, `false`
  - `page` (optional): Page number
  - `limit` (optional): Items per page
- **Description:** List user notifications
- **Response:** `200 OK`
  ```json
  {
    "data": [
      {
        "id": "string",
        "type": "string",
        "message": "string",
        "read": "boolean",
        "created_at": "ISO 8601 timestamp"
      }
    ],
    "pagination": {
      "page": "integer",
      "limit": "integer",
      "unread_count": "integer"
    }
  }
  ```

#### Mark Notification as Read
- **Method:** `PUT`
- **Path:** `/notifications/{notificationId}/read`
- **Authentication:** Required (Bearer Token)
- **Description:** Mark single notification as read
- **Response:** `200 OK`
  ```json
  {
    "id": "string",
    "read": true
  }
  ```

#### Mark All Notifications as Read
- **Method:** `PUT`
- **Path:** `/notifications/read-all`
- **Authentication:** Required (Bearer Token)
- **Description:** Mark all notifications as read
- **Response:** `200 OK`
  ```json
  {
    "message": "All notifications marked as read"
  }
  ```

#### Delete Notification
- **Method:** `DELETE`
- **Path:** `/notifications/{notificationId}`
- **Authentication:** Required (Bearer Token)
- **Description:** Delete single notification
- **Response:** `204 No Content`

---

### Reports Endpoints

#### Get Reports List
- **Method:** `GET`
- **Path:** `/reports`
- **Authentication:** Required (Bearer Token)
- **Query Parameters:**
  - `type` (optional): Report type
  - `page` (optional): Page number
  - `limit` (optional): Items per page
- **Description:** List all reports
- **Response:** `200 OK`
  ```json
  {
    "data": [
      {
        "id": "string",
        "name": "string",
        "type": "string",
        "created_at": "ISO 8601 timestamp",
        "updated_at": "ISO 8601 timestamp"
      }
    ],
    "pagination": {
      "page": "integer",
      "limit": "integer",
      "total": "integer"
    }
  }
  ```

#### Generate Report
- **Method:** `POST`
- **Path:** `/reports/generate`
- **Authentication:** Required (Bearer Token)
- **Description:** Generate new report
- **Request Body:**
  ```json
  {
    "name": "string (required)",
    "type": "string (required)",
    "date_range": {
      "start_date": "ISO 8601 timestamp",
      "end_date": "ISO 8601 timestamp"
    },
    "filters": {
      "priority": "string (optional)",
      "status": "string (optional)"
    }
  }
  ```
- **Response:** `202 Accepted`
  ```json
  {
    "id": "string",
    "status": "processing",
    "created_at": "ISO 8601 timestamp"
  }
  ```

#### Get Report Details
- **Method:** `GET`
- **Path:** `/reports/{reportId}`
- **Authentication:** Required (Bearer Token)
- **Description:** Retrieve specific report
- **Response:** `200 OK`
  ```json
  {
    "id": "string",
    "name": "string",
    "type": "string",
    "status": "completed",
    "data": {},
    "created_at": "ISO 8601 timestamp"
  }
  ```

#### Download Report
- **Method:** `GET`
- **Path:** `/reports/{reportId}/download`
- **Authentication:** Required (Bearer Token)
- **Query Parameters:**
  - `format` (optional): `pdf`, `csv`, `json` (default: pdf)
- **Description:** Download report in specified format
- **Response:** `200 OK` (Binary file)

---

## Request/Response Formats

### Request Format

#### Headers
```
Content-Type: application/json
Authorization: Bearer <access_token>
X-Request-ID: <unique-request-id> (optional)
```

#### Body (JSON)
All request bodies must be valid JSON with proper encoding (UTF-8).

```json
{
  "key": "value",
  "nested": {
    "property": "value"
  },
  "array": [1, 2, 3]
}
```

#### Query Parameters
Query parameters should be URL-encoded and follow REST conventions.

```
GET /api/v1/tasks?status=pending&priority=high&page=1&limit=20
```

### Response Format

#### Success Response (2xx)
```json
{
  "status": "success",
  "data": {},
  "timestamp": "ISO 8601 timestamp",
  "request_id": "string"
}
```

#### List Response with Pagination
```json
{
  "status": "success",
  "data": [],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 150,
    "total_pages": 8,
    "has_next": true,
    "has_previous": false
  },
  "timestamp": "ISO 8601 timestamp"
}
```

#### Data Types
- **String:** `"value"`
- **Integer:** `123`
- **Float:** `123.45`
- **Boolean:** `true` or `false`
- **Null:** `null`
- **ISO 8601 Timestamp:** `"2026-01-09T06:33:47Z"`
- **Array:** `[1, 2, 3]`
- **Object:** `{}`

---

## Error Handling

### Error Response Format

All errors return a consistent JSON format:

```json
{
  "status": "error",
  "code": "ERROR_CODE",
  "message": "Human-readable error message",
  "details": {
    "field": "Additional error context"
  },
  "timestamp": "ISO 8601 timestamp",
  "request_id": "string"
}
```

### HTTP Status Codes

| Status Code | Name | Description |
|------------|------|-------------|
| 200 | OK | Successful GET, PUT, or PATCH request |
| 201 | Created | Successful POST request |
| 202 | Accepted | Request accepted for processing (async) |
| 204 | No Content | Successful DELETE request |
| 400 | Bad Request | Invalid request parameters or malformed JSON |
| 401 | Unauthorized | Missing or invalid authentication token |
| 403 | Forbidden | User lacks permission for this resource |
| 404 | Not Found | Resource not found |
| 409 | Conflict | Resource conflict (duplicate, etc.) |
| 422 | Unprocessable Entity | Request validation failed |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Unexpected server error |
| 502 | Bad Gateway | Service temporarily unavailable |
| 503 | Service Unavailable | Server under maintenance |

### Common Error Codes

#### Authentication Errors
- `AUTH_INVALID_CREDENTIALS`: Invalid email or password
- `AUTH_TOKEN_EXPIRED`: Access token has expired
- `AUTH_TOKEN_INVALID`: Invalid or malformed token
- `AUTH_UNAUTHORIZED`: Missing authentication header
- `AUTH_FORBIDDEN`: User lacks required permissions

#### Validation Errors
- `VALIDATION_FAILED`: Request validation failed
- `INVALID_EMAIL`: Invalid email format
- `INVALID_PASSWORD`: Password does not meet requirements
- `INVALID_DATE_FORMAT`: Date format is not ISO 8601
- `REQUIRED_FIELD_MISSING`: Required field is missing

#### Resource Errors
- `RESOURCE_NOT_FOUND`: Requested resource does not exist
- `RESOURCE_ALREADY_EXISTS`: Resource already exists
- `RESOURCE_CONFLICT`: Resource state conflict
- `RESOURCE_DELETED`: Resource has been deleted

#### Rate Limiting
- `RATE_LIMIT_EXCEEDED`: Too many requests
- `QUOTA_EXCEEDED`: User quota exceeded

#### Server Errors
- `INTERNAL_SERVER_ERROR`: Unexpected server error
- `SERVICE_UNAVAILABLE`: Service temporarily unavailable
- `DATABASE_ERROR`: Database operation failed

### Error Response Examples

**400 Bad Request - Validation Error**
```json
{
  "status": "error",
  "code": "VALIDATION_FAILED",
  "message": "Request validation failed",
  "details": {
    "title": "Title is required",
    "priority": "Invalid priority value. Must be: low, medium, high, urgent"
  },
  "timestamp": "2026-01-09T06:33:47Z",
  "request_id": "req_123456"
}
```

**401 Unauthorized**
```json
{
  "status": "error",
  "code": "AUTH_TOKEN_EXPIRED",
  "message": "Access token has expired",
  "details": {
    "hint": "Use refresh token to obtain new access token"
  },
  "timestamp": "2026-01-09T06:33:47Z",
  "request_id": "req_123456"
}
```

**404 Not Found**
```json
{
  "status": "error",
  "code": "RESOURCE_NOT_FOUND",
  "message": "Task not found",
  "details": {
    "resource": "Task",
    "resource_id": "task_123"
  },
  "timestamp": "2026-01-09T06:33:47Z",
  "request_id": "req_123456"
}
```

**429 Too Many Requests**
```json
{
  "status": "error",
  "code": "RATE_LIMIT_EXCEEDED",
  "message": "Too many requests",
  "details": {
    "limit": 100,
    "window": "1 minute",
    "retry_after": 30
  },
  "timestamp": "2026-01-09T06:33:47Z",
  "request_id": "req_123456"
}
```

---

## Rate Limiting

### Rate Limit Policy

The API implements rate limiting to ensure fair usage and system stability.

#### Standard Rate Limits
- **Anonymous Requests:** 10 requests per minute
- **Authenticated Requests:** 100 requests per minute
- **Premium Users:** 1000 requests per minute

#### Rate Limit Headers

All responses include rate limit information:

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 87
X-RateLimit-Reset: 1609972427
```

#### Handling Rate Limits

When you receive a 429 (Too Many Requests) response:

1. Check the `Retry-After` header for seconds to wait
2. Implement exponential backoff in client code
3. Consider upgrade for higher limits if applicable

### Example Rate Limit Request

```bash
curl -i -H "Authorization: Bearer token" \
  https://api.executive-assistant.com/api/v1/tasks

HTTP/1.1 200 OK
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 87
X-RateLimit-Reset: 1609972427
```

---

## Common Response Codes

### Success Codes (2xx)

#### 200 OK
- Standard response for successful HTTP requests
- Used for GET, PUT, PATCH requests
- Body contains requested resource(s)

#### 201 Created
- Response for successful resource creation (POST)
- Includes Location header with resource URL
- Body contains created resource

#### 202 Accepted
- Request accepted but processing is not complete
- Used for long-running operations
- Includes tracking ID for polling status

#### 204 No Content
- Successful request with no response body
- Used for successful DELETE requests

### Client Error Codes (4xx)

#### 400 Bad Request
- Malformed request syntax
- Invalid parameter values
- Includes error details for debugging

#### 401 Unauthorized
- Missing or invalid authentication
- Expired credentials
- Check authorization header

#### 403 Forbidden
- Valid authentication but insufficient permissions
- User lacks access to resource
- Contact administrator for access

#### 404 Not Found
- Resource does not exist
- Check resource ID/path
- Verify resource was not deleted

#### 422 Unprocessable Entity
- Request syntax valid but contains semantic errors
- Validation failures
- Review error details for corrections

#### 429 Too Many Requests
- Rate limit exceeded
- Implement backoff strategy
- Check Retry-After header

### Server Error Codes (5xx)

#### 500 Internal Server Error
- Unexpected server error
- Contact support if issue persists
- Check status page for incidents

#### 502 Bad Gateway
- Invalid response from upstream server
- Temporary service issue
- Retry after brief delay

#### 503 Service Unavailable
- Server under maintenance
- Check status page for updates
- Service should restore shortly

---

## Best Practices

1. **Always include proper authentication headers** in requests
2. **Handle errors gracefully** with appropriate retry logic
3. **Implement pagination** for list endpoints to improve performance
4. **Use request IDs** for debugging and tracing
5. **Validate inputs** before sending requests
6. **Respect rate limits** and implement backoff strategies
7. **Use ISO 8601 format** for all timestamps
8. **Keep access tokens secure** and rotate them regularly
9. **Log API interactions** for debugging and audit trails
10. **Monitor API usage** to stay within rate limits

---

## Support

For API support and documentation updates, contact:
- **Email:** support@executive-assistant.com
- **Status Page:** https://status.executive-assistant.com
- **Documentation:** https://docs.executive-assistant.com
