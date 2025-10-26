# Airbnb Clone - Backend Requirements Specification

## Table of Contents
1. [Overview](#overview)
2. [User Authentication](#user-authentication)
3. [Property Management](#property-management)
4. [Booking System](#booking-system)
5. [General Requirements](#general-requirements)

---

## Overview

This document outlines the technical and functional requirements for the Airbnb Clone backend system. The system provides a platform for property rental management with user authentication, property listing, and booking capabilities.

### System Architecture
- **Backend Framework**: Node.js with Express.js
- **Database**: PostgreSQL with Prisma ORM
- **Authentication**: JWT (JSON Web Tokens)
- **API Design**: RESTful API
- **File Storage**: AWS S3 or similar cloud storage for property images

---

## User Authentication

### Functional Requirements

#### 1.1 User Registration
- **Description**: Allow new users to create accounts on the platform
- **User Roles**: Guest, Host, Admin
- **Input Validation**: Email format, password strength, required fields

#### 1.2 User Login
- **Description**: Authenticate existing users and provide access tokens
- **Security**: Password hashing, JWT token generation
- **Session Management**: Token expiration and refresh mechanisms

#### 1.3 Profile Management
- **Description**: Allow users to view and update their profile information
- **Data**: Personal information, contact details, preferences

### API Endpoints

#### POST /api/auth/register
**Purpose**: Register a new user account

**Request Body**:
```json
{
  "firstName": "string (required, 2-50 chars)",
  "lastName": "string (required, 2-50 chars)",
  "email": "string (required, valid email format)",
  "password": "string (required, min 8 chars, 1 uppercase, 1 lowercase, 1 number)",
  "role": "enum (guest|host|admin, default: guest)"
}
```

**Response (201 Created)**:
```json
{
  "success": true,
  "message": "User registered successfully",
  "data": {
    "id": "uuid",
    "firstName": "string",
    "lastName": "string",
    "email": "string",
    "role": "string",
    "createdAt": "ISO 8601 timestamp"
  }
}
```

**Validation Rules**:
- Email must be unique across the system
- Password must meet complexity requirements
- All required fields must be provided
- Role must be one of: guest, host, admin

#### POST /api/auth/login
**Purpose**: Authenticate user and return access token

**Request Body**:
```json
{
  "email": "string (required, valid email format)",
  "password": "string (required)"
}
```

**Response (200 OK)**:
```json
{
  "success": true,
  "message": "Login successful",
  "data": {
    "user": {
      "id": "uuid",
      "firstName": "string",
      "lastName": "string",
      "email": "string",
      "role": "string"
    },
    "accessToken": "jwt_token",
    "refreshToken": "jwt_token",
    "expiresIn": "3600"
  }
}
```

#### GET /api/auth/profile
**Purpose**: Get current user profile information

**Headers**: `Authorization: Bearer <access_token>`

**Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "firstName": "string",
    "lastName": "string",
    "email": "string",
    "role": "string",
    "createdAt": "ISO 8601 timestamp",
    "updatedAt": "ISO 8601 timestamp"
  }
}
```

#### PUT /api/auth/profile
**Purpose**: Update user profile information

**Headers**: `Authorization: Bearer <access_token>`

**Request Body**:
```json
{
  "firstName": "string (optional, 2-50 chars)",
  "lastName": "string (optional, 2-50 chars)",
  "email": "string (optional, valid email format)"
}
```

### Performance Criteria
- Registration response time: < 2 seconds
- Login response time: < 1 second
- Profile retrieval: < 500ms
- Support concurrent users: 1000+
- Password hashing: bcrypt with salt rounds 12

---

## Property Management

### Functional Requirements

#### 2.1 Property Creation
- **Description**: Allow hosts to create new property listings
- **Required Information**: Title, description, location, pricing, amenities
- **Media**: Support for multiple property images

#### 2.2 Property Updates
- **Description**: Allow hosts to modify existing property information
- **Restrictions**: Only property owner can update
- **Validation**: Ensure data integrity and business rules

#### 2.3 Property Retrieval
- **Description**: Allow users to search and view property listings
- **Filtering**: By location, price range, availability, amenities
- **Pagination**: Support for large result sets

#### 2.4 Property Deletion
- **Description**: Allow hosts to remove property listings
- **Business Rules**: Cannot delete properties with active bookings

### API Endpoints

#### POST /api/properties
**Purpose**: Create a new property listing

**Headers**: `Authorization: Bearer <access_token>`

**Request Body**:
```json
{
  "title": "string (required, 5-100 chars)",
  "description": "string (required, 10-1000 chars)",
  "location": {
    "address": "string (required)",
    "city": "string (required)",
    "state": "string (required)",
    "country": "string (required)",
    "coordinates": {
      "latitude": "number (required, -90 to 90)",
      "longitude": "number (required, -180 to 180)"
    }
  },
  "pricePerNight": "number (required, > 0)",
  "maxGuests": "number (required, 1-20)",
  "bedrooms": "number (required, 1-10)",
  "bathrooms": "number (required, 1-10)",
  "amenities": "array of strings (optional)",
  "propertyType": "enum (apartment|house|condo|villa|other)",
  "images": "array of file uploads (max 10, max 5MB each)"
}
```

**Response (201 Created)**:
```json
{
  "success": true,
  "message": "Property created successfully",
  "data": {
    "id": "uuid",
    "title": "string",
    "description": "string",
    "location": "object",
    "pricePerNight": "number",
    "maxGuests": "number",
    "bedrooms": "number",
    "bathrooms": "number",
    "amenities": "array",
    "propertyType": "string",
    "images": "array of URLs",
    "hostId": "uuid",
    "createdAt": "ISO 8601 timestamp"
  }
}
```

#### GET /api/properties
**Purpose**: Retrieve property listings with filtering and pagination

**Query Parameters**:
- `page`: number (default: 1)
- `limit`: number (default: 10, max: 50)
- `city`: string (optional)
- `minPrice`: number (optional)
- `maxPrice`: number (optional)
- `maxGuests`: number (optional)
- `propertyType`: string (optional)
- `amenities`: string (comma-separated, optional)
- `checkIn`: date (ISO 8601, optional)
- `checkOut`: date (ISO 8601, optional)

**Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "properties": [
      {
        "id": "uuid",
        "title": "string",
        "description": "string",
        "location": "object",
        "pricePerNight": "number",
        "maxGuests": "number",
        "bedrooms": "number",
        "bathrooms": "number",
        "amenities": "array",
        "propertyType": "string",
        "images": "array of URLs",
        "host": {
          "id": "uuid",
          "firstName": "string",
          "lastName": "string"
        },
        "averageRating": "number",
        "reviewCount": "number"
      }
    ],
    "pagination": {
      "currentPage": "number",
      "totalPages": "number",
      "totalItems": "number",
      "hasNext": "boolean",
      "hasPrev": "boolean"
    }
  }
}
```

#### GET /api/properties/:id
**Purpose**: Get detailed information about a specific property

**Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "title": "string",
    "description": "string",
    "location": "object",
    "pricePerNight": "number",
    "maxGuests": "number",
    "bedrooms": "number",
    "bathrooms": "number",
    "amenities": "array",
    "propertyType": "string",
    "images": "array of URLs",
    "host": "object",
    "reviews": "array",
    "availability": "array of date ranges",
    "createdAt": "ISO 8601 timestamp"
  }
}
```

#### PUT /api/properties/:id
**Purpose**: Update property information

**Headers**: `Authorization: Bearer <access_token>`

**Request Body**: Same as POST /api/properties (all fields optional)

**Response (200 OK)**:
```json
{
  "success": true,
  "message": "Property updated successfully",
  "data": "updated property object"
}
```

#### DELETE /api/properties/:id
**Purpose**: Delete a property listing

**Headers**: `Authorization: Bearer <access_token>`

**Response (200 OK)**:
```json
{
  "success": true,
  "message": "Property deleted successfully"
}
```

### Validation Rules
- Title must be unique per host
- Price per night must be positive
- Coordinates must be valid latitude/longitude
- Images must be valid image files (JPEG, PNG, WebP)
- Cannot delete properties with active bookings
- Only property owner can update/delete

### Performance Criteria
- Property creation: < 3 seconds (including image upload)
- Property search: < 1 second for first 50 results
- Property details: < 500ms
- Support concurrent property operations: 500+
- Image upload: < 5 seconds per image

---

## Booking System

### Functional Requirements

#### 3.1 Booking Creation
- **Description**: Allow guests to create bookings for available properties
- **Validation**: Check property availability, date conflicts
- **Business Rules**: Minimum/maximum stay duration, advance booking limits

#### 3.2 Booking Management
- **Description**: Allow users to view, modify, and cancel bookings
- **Status Tracking**: Pending, confirmed, cancelled, completed
- **Modification Rules**: Based on cancellation policy

#### 3.3 Availability Checking
- **Description**: Real-time availability checking for properties
- **Integration**: Calendar system for date management
- **Conflict Prevention**: Prevent double bookings

#### 3.4 Booking Notifications
- **Description**: Notify hosts and guests about booking status changes
- **Channels**: Email, in-app notifications

### API Endpoints

#### POST /api/bookings
**Purpose**: Create a new booking

**Headers**: `Authorization: Bearer <access_token>`

**Request Body**:
```json
{
  "propertyId": "uuid (required)",
  "checkInDate": "date (required, ISO 8601, future date)",
  "checkOutDate": "date (required, ISO 8601, after checkInDate)",
  "numberOfGuests": "number (required, 1-20)",
  "specialRequests": "string (optional, max 500 chars)"
}
```

**Response (201 Created)**:
```json
{
  "success": true,
  "message": "Booking created successfully",
  "data": {
    "id": "uuid",
    "propertyId": "uuid",
    "guestId": "uuid",
    "checkInDate": "ISO 8601 date",
    "checkOutDate": "ISO 8601 date",
    "numberOfGuests": "number",
    "totalPrice": "number",
    "status": "pending",
    "specialRequests": "string",
    "createdAt": "ISO 8601 timestamp"
  }
}
```

#### GET /api/bookings
**Purpose**: Get user's bookings

**Headers**: `Authorization: Bearer <access_token>`

**Query Parameters**:
- `page`: number (default: 1)
- `limit`: number (default: 10, max: 50)
- `status`: string (pending|confirmed|cancelled|completed, optional)
- `type`: string (guest|host, default: guest)

**Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "bookings": [
      {
        "id": "uuid",
        "property": {
          "id": "uuid",
          "title": "string",
          "images": "array of URLs",
          "location": "object"
        },
        "guest": "object (if type=host)",
        "host": "object (if type=guest)",
        "checkInDate": "ISO 8601 date",
        "checkOutDate": "ISO 8601 date",
        "numberOfGuests": "number",
        "totalPrice": "number",
        "status": "string",
        "specialRequests": "string",
        "createdAt": "ISO 8601 timestamp"
      }
    ],
    "pagination": "object"
  }
}
```

#### GET /api/bookings/:id
**Purpose**: Get detailed booking information

**Headers**: `Authorization: Bearer <access_token>`

**Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "property": "detailed property object",
    "guest": "user object",
    "host": "user object",
    "checkInDate": "ISO 8601 date",
    "checkOutDate": "ISO 8601 date",
    "numberOfGuests": "number",
    "totalPrice": "number",
    "status": "string",
    "specialRequests": "string",
    "payment": "payment object",
    "createdAt": "ISO 8601 timestamp",
    "updatedAt": "ISO 8601 timestamp"
  }
}
```

#### PUT /api/bookings/:id/status
**Purpose**: Update booking status (host confirmation/cancellation)

**Headers**: `Authorization: Bearer <access_token>`

**Request Body**:
```json
{
  "status": "enum (confirmed|cancelled, required)",
  "reason": "string (optional, required for cancellation)"
}
```

#### DELETE /api/bookings/:id
**Purpose**: Cancel a booking (guest only)

**Headers**: `Authorization: Bearer <access_token>`

**Response (200 OK)**:
```json
{
  "success": true,
  "message": "Booking cancelled successfully",
  "data": {
    "refundAmount": "number",
    "refundMethod": "string"
  }
}
```

#### GET /api/properties/:id/availability
**Purpose**: Check property availability for date range

**Query Parameters**:
- `checkIn`: date (required, ISO 8601)
- `checkOut`: date (required, ISO 8601)

**Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "available": "boolean",
    "price": "number",
    "conflicts": "array of conflicting bookings (if not available)"
  }
}
```

### Validation Rules
- Check-in date must be in the future
- Check-out date must be after check-in date
- Minimum stay: 1 night
- Maximum stay: 30 nights
- Number of guests cannot exceed property capacity
- Cannot book already booked dates
- Cancellation policy applies based on timing

### Business Rules
- Booking confirmation required within 24 hours
- Cancellation allowed up to 24 hours before check-in
- Refund policy: 100% if cancelled > 7 days, 50% if 1-7 days, 0% if < 24 hours
- Automatic confirmation after 24 hours if no host response

### Performance Criteria
- Booking creation: < 2 seconds
- Availability check: < 500ms
- Booking retrieval: < 1 second
- Support concurrent bookings: 100+
- Real-time availability updates

---

## General Requirements

### Security Requirements
- All API endpoints (except public property search) require authentication
- JWT tokens expire after 1 hour, refresh tokens after 7 days
- Password hashing using bcrypt with salt rounds 12
- Input validation and sanitization for all user inputs
- Rate limiting: 100 requests per minute per IP
- CORS configuration for cross-origin requests

### Error Handling
- Consistent error response format across all endpoints
- HTTP status codes following REST conventions
- Detailed error messages for development, generic for production
- Logging of all errors for monitoring and debugging

### Database Requirements
- PostgreSQL 13+ with proper indexing
- Database migrations for schema changes
- Connection pooling for performance
- Backup and recovery procedures
- Data encryption at rest

### Monitoring and Logging
- Application performance monitoring
- Error tracking and alerting
- User activity logging
- API usage analytics
- Database performance monitoring

### Scalability Requirements
- Horizontal scaling capability
- Load balancing support
- Caching strategy for frequently accessed data
- CDN for static assets (images)
- Microservices architecture consideration for future growth

### Compliance
- GDPR compliance for user data
- PCI DSS compliance for payment processing
- Data retention policies
- User consent management
- Privacy policy adherence

---

*This document serves as the technical specification for the Airbnb Clone backend system. All implementations should follow these requirements to ensure consistency, security, and performance.*
