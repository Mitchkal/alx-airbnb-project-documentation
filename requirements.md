# Requirements Specifications

This document outlines technical and functional requirements for three key backend features of the Airbnb Clone system: User Authentication, Property Management, and Booking System.

## 1. User Authentication

**Functional Requirements**:

- Enable users (Guests/Hosts) to register, log in, and manage profiles securely.
- Support email/password and OAuth (e.g., Google) login.
- Issue JWT tokens for session management.

**Technical Requirements**:

- **API Endpoints**:
  - `POST /auth/register`: Register a new user.
    - **Input**: JSON `{ "email": string, "password": string, "role": "guest|host", "name": string }`
    - **Output**: JSON `{ "userId": string, "message": "Registration successful" }` (201) or `{ "error": string }` (400/409).
  - `POST /auth/login`: Authenticate a user.
    - **Input**: JSON `{ "email": string, "password": string }` or `{ "oauthToken": string }`
    - **Output**: JSON `{ "userId": string, "jwt": string }` (200) or `{ "error": string }` (401).
  - `PATCH /auth/profile`: Update user profile (authenticated).
    - **Input**: JSON `{ "name": string, "photo": string }`, Header: `Authorization: Bearer <JWT>`
    - **Output**: JSON `{ "message": "Profile updated" }` (200) or `{ "error": string }` (401/400).
- **Validation Rules**:
  - Email: Valid format, unique in `Users` table.
  - Password: 8+ characters, 1 uppercase, 1 number, 1 special character.
  - Role: Must be “guest” or “host”.
  - JWT: Validated on protected endpoints, expires in 24 hours.
- **Performance Criteria**:
  - Response time: <200ms for login/register (95th percentile).
  - Scalability: Handle 10,000 concurrent authentications.
  - Security: Store passwords with bcrypt, use HTTPS, validate OAuth tokens.

## 2. Property Management

**Functional Requirements**:

- Allow Hosts to create, update, and delete property listings.
- Display property details (title, price, amenities) to Guests.

**Technical Requirements**:

- **API Endpoints**:
  - `POST /properties`: Create a property (Host only).
    - **Input**: JSON `{ "title": string, "description": string, "price": number, "location": string, "amenities": string[] }`, Header: `Authorization: Bearer <JWT>`
    - **Output**: JSON `{ "propertyId": string, "message": "Property created" }` (201) or `{ "error": string }` (401/400).
  - `PATCH /properties/:id`: Update a property.
    - **Input**: JSON `{ "title": string, "price": number }`, Header: `Authorization: Bearer <JWT>`
    - **Output**: JSON `{ "message": "Property updated" }` (200) or `{ "error": string }` (401/404).
  - `DELETE /properties/:id`: Delete a property.
    - **Input**: Header: `Authorization: Bearer <JWT>`
    - **Output**: JSON `{ "message": "Property deleted" }` (200) or `{ "error": string }` (401/404).
- **Validation Rules**:
  - Title: 5-100 characters, non-empty.
  - Price: Positive number, max 10,000.
  - Location: Valid string, max 200 characters.
  - Host: Must own the property for updates/deletes.
- **Performance Criteria**:
  - Response time: <300ms for create/update (95th percentile).
  - Database: Index `Properties` table on `location` for fast queries.

## 3. Booking System

**Functional Requirements**:

- Enable Guests to search properties and create/cancel bookings.
- Validate availability and process payments.

**Technical Requirements**:

- **API Endpoints**:
  - `GET /properties/search`: Search available properties.
    - **Input**: Query `{ location: string, checkIn: date, checkOut: date, guests: number }`
    - **Output**: JSON `[ { "propertyId": string, "title": string, "price": number } ]` (200) or `{ "error": string }` (400).
  - `POST /bookings`: Create a booking.
    - **Input**: JSON `{ "propertyId": string, "checkIn": date, "checkOut": date, "guests": number, "paymentDetails": object }`, Header: `Authorization: Bearer <JWT>`
    - **Output**: JSON `{ "bookingId": string, "status": "confirmed" }` (201) or `{ "error": string }` (401/400/409).
- **Validation Rules**:
  - Dates: Check-in < check-out, future dates only.
  - Guests: Positive integer, max property capacity.
  - Property: Must be available (no overlapping bookings in `Bookings` table).
  - Payment: Valid via Stripe/PayPal API.
- **Performance Criteria**:
  - Response time: <500ms for search, <1s for booking creation (95th percentile).
  - Concurrency: Handle 1,000 simultaneous booking requests.
