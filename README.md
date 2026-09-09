# F-Pay (Farmer Pay)

**Digital Payment & Traceability Solution for Smallholder Coffee Farmers**

F-Pay is a digital platform designed to help smallholder coffee farmers keep reliable records of their coffee deliveries and payments.

The project is being developed around a simple principle:

> Build a reliable source of truth for farmer deliveries and payments first. Add USSD, mobile money, SMS, and other integrations after the core system is stable.

## Current Stack

| Layer | Technology |
|---|---|
| Frontend | Next.js |
| Backend | Node.js |
| Database | MySQL |
| Authentication | JWT |
| API | REST |
| Future access channel | USSD |

## Project Status

F-Pay is currently in the **core web application development phase**.

### Current focus

- Farmer registration
- Farmer login
- JWT authentication
- Farmer profiles
- Coffee delivery records
- Payment records
- Farmer dashboard
- Admin management
- Audit trail

### Not implemented yet

- USSD
- M-Pesa/mobile money integration
- SMS notifications
- Blockchain/distributed ledger
- Native mobile application

These features are planned for later phases.

---

# Problem

Smallholder coffee farmers can face problems such as:

- Paper-based delivery records
- Payment discrepancies
- Delayed payments
- Poor visibility into payment calculations
- Difficulty maintaining records across multiple seasons
- Risks associated with collecting large amounts of cash

F-Pay aims to provide farmers with a digital record of their deliveries and payments that they can access through the web application.

---

# Core MVP

The first version focuses on two main actors.

## Farmer

A farmer can:

- Register
- Log in
- View their profile
- View coffee deliveries
- View payment history
- View total coffee delivered
- View total amount paid
- View pending amounts
- View historical records

Farmers cannot create or modify delivery or payment records.

## Admin

An admin can:

- Log in
- Manage farmer records
- Record coffee deliveries
- Record payments
- View farmer history
- Search farmers
- View system statistics
- Review audit history

---

# Architecture

```text
                    F-Pay
                      |
          +-----------+-----------+
          |                       |
      Next.js                 Node.js API
      Frontend                   Backend
          |                       |
          |                 JWT Authentication
          |                       |
          |                  Business Logic
          |                       |
          +------- REST API ------+
                                  |
                                MySQL
```

Future architecture:

```text
                    F-Pay
                      |
        +-------------+-------------+
        |             |             |
     Next.js        USSD       Other Clients
        |             |             |
        +-------------+-------------+
                      |
                 Node.js API
                      |
                     MySQL
                      |
        +-------------+-------------+
        |             |             |
    Payments         SMS       Future Ledger
```

USSD should use the same backend services as the web application rather than becoming a separate application.

---

# Repository Structure

```text
f-pay/
│
├── backend/
│   ├── src/
│   │   ├── config/
│   │   │   ├── database.js
│   │   │   └── env.js
│   │   │
│   │   ├── controllers/
│   │   │   ├── auth.controller.js
│   │   │   ├── farmer.controller.js
│   │   │   ├── delivery.controller.js
│   │   │   └── payment.controller.js
│   │   │
│   │   ├── middleware/
│   │   │   ├── auth.middleware.js
│   │   │   ├── role.middleware.js
│   │   │   └── error.middleware.js
│   │   │
│   │   ├── models/
│   │   │   ├── user.model.js
│   │   │   ├── farmer.model.js
│   │   │   ├── delivery.model.js
│   │   │   └── payment.model.js
│   │   │
│   │   ├── routes/
│   │   │   ├── auth.routes.js
│   │   │   ├── farmer.routes.js
│   │   │   ├── delivery.routes.js
│   │   │   └── payment.routes.js
│   │   │
│   │   ├── services/
│   │   │   ├── auth.service.js
│   │   │   ├── farmer.service.js
│   │   │   ├── delivery.service.js
│   │   │   └── payment.service.js
│   │   │
│   │   ├── utils/
│   │   │   ├── jwt.js
│   │   │   ├── password.js
│   │   │   └── validators.js
│   │   │
│   │   └── app.js
│   │
│   ├── tests/
│   ├── .env.example
│   ├── package.json
│   └── README.md
│
├── frontend/
│   ├── app/
│   │   ├── login/
│   │   ├── register/
│   │   ├── dashboard/
│   │   ├── deliveries/
│   │   ├── payments/
│   │   └── profile/
│   │
│   ├── components/
│   ├── lib/
│   │   ├── api.js
│   │   └── auth.js
│   ├── types/
│   ├── public/
│   ├── .env.example
│   ├── package.json
│   └── README.md
│
├── database/
│   ├── migrations/
│   └── seed/
│
├── docs/
│   ├── API.md
│   ├── DATABASE.md
│   └── ARCHITECTURE.md
│
├── .gitignore
└── README.md
```

The exact implementation can use JavaScript or TypeScript on the backend. Express is sufficient for the initial API.

---

# Database

The initial database contains the following core entities:

```text
users
farmers
deliveries
payments
audit_logs
```

## Users

Authentication information:

```text
id
phone
email
password_hash
role
is_active
created_at
updated_at
```

Roles:

```text
FARMER
ADMIN
```

## Farmers

Farmer information:

```text
id
user_id
farmer_number
first_name
last_name
national_id
county
sub_county
cooperative_name
created_at
updated_at
```

## Deliveries

Coffee delivery records:

```text
id
farmer_id
delivery_reference
delivery_date
coffee_type
quantity_kg
quality_grade
factory_name
recorded_by
created_at
updated_at
```

Every delivery receives a unique reference such as:

```text
DEL-2026-000001
```

## Payments

Payment records:

```text
id
farmer_id
payment_reference
delivery_id
amount
payment_status
payment_method
payment_date
transaction_reference
recorded_by
created_at
updated_at
```

Example:

```text
PAY-2026-000001
```

Possible statuses:

```text
PENDING
COMPLETED
FAILED
CANCELLED
```

## Audit Logs

Important system actions are recorded:

```text
id
user_id
action
entity_type
entity_id
old_value
new_value
created_at
```

This provides an application-level audit trail before any blockchain implementation is considered.

---

# Authentication

F-Pay uses JWT authentication.

## Registration

```http
POST /api/auth/register
```

Example request:

```json
{
  "phone": "0712345678",
  "password": "password",
  "firstName": "John",
  "lastName": "Doe"
}
```

Passwords are hashed before being stored.

## Login

```http
POST /api/auth/login
```

Example:

```json
{
  "phone": "0712345678",
  "password": "password"
}
```

Successful authentication returns a JWT.

Protected requests use:

```text
Authorization: Bearer <JWT>
```

The backend remains responsible for authentication and authorization. Frontend route protection alone is not considered security.

---

# API Overview

Base URL during development:

```text
http://localhost:5000/api
```

## Authentication

```text
POST /auth/register
POST /auth/login
GET  /auth/me
POST /auth/logout
```

## Farmer

```text
GET /farmers/me
PUT /farmers/me
GET /farmers/me/deliveries
GET /farmers/me/payments
GET /farmers/me/summary
```

## Admin Farmers

```text
GET  /admin/farmers
GET  /admin/farmers/:id
POST /admin/farmers
PUT  /admin/farmers/:id
```

## Admin Deliveries

```text
POST /admin/deliveries
GET  /admin/deliveries
GET  /admin/deliveries/:id
PUT  /admin/deliveries/:id
```

## Admin Payments

```text
POST /admin/payments
GET  /admin/payments
GET  /admin/payments/:id
PUT  /admin/payments/:id
```

---

# Local Development

## Requirements

Install:

- Node.js
- npm
- MySQL
- Git

Check Node.js:

```bash
node --version
```

Check npm:

```bash
npm --version
```

Check MySQL:

```bash
mysql --version
```

---

# Clone the Repository

```bash
git clone <YOUR_REPOSITORY_URL>
cd f-pay
```

Replace `<YOUR_REPOSITORY_URL>` with the actual Git repository URL.

---

# Backend Setup

Enter the backend directory:

```bash
cd backend
```

Install dependencies:

```bash
npm install
```

Create the environment file:

```bash
cp .env.example .env
```

On Windows PowerShell, if `cp` is unavailable:

```powershell
Copy-Item .env.example .env
```

Configure `.env`:

```env
PORT=5000

DATABASE_URL=mysql://username:password@localhost:3306/fpay

JWT_SECRET=replace_with_a_strong_secret
JWT_EXPIRES_IN=1d

NODE_ENV=development
```

Start the backend:

```bash
npm run dev
```

The API should be available at:

```text
http://localhost:5000
```

Health check:

```http
GET /api/health
```

Expected response:

```json
{
  "success": true,
  "message": "F-Pay API is running"
}
```

---

# Frontend Setup

Open another terminal:

```bash
cd frontend
```

Install dependencies:

```bash
npm install
```

Create:

```text
.env.local
```

Add:

```env
NEXT_PUBLIC_API_URL=http://localhost:5000/api
```

Start Next.js:

```bash
npm run dev
```

The frontend will normally be available at:

```text
http://localhost:3000
```

---

# MySQL Setup

Create the database:

```sql
CREATE DATABASE fpay;
```

Then configure the backend database connection.

Example:

```env
DATABASE_URL=mysql://root:password@localhost:3306/fpay
```

Database migrations should be stored under:

```text
database/migrations/
```

Seed data should be stored under:

```text
database/seed/
```

A development admin account should be created through a seed script rather than manually inserting passwords into the database.

---

# Development Phases

## Phase 1: Foundation

- Repository setup
- Node.js backend
- Next.js frontend
- MySQL connection
- Environment configuration
- API health endpoint
- CORS configuration

**Done when:** frontend, backend, and database communicate successfully.

## Phase 2: Database

- Users
- Farmers
- Deliveries
- Payments
- Audit logs
- Relationships
- Constraints
- Indexes
- Seed data

**Done when:** the backend can reliably read and write the core data.

## Phase 3: Authentication

- Farmer registration
- Login
- Password hashing
- JWT generation
- JWT verification
- Role middleware
- Protected routes

**Done when:** a farmer can register, log in, and access protected resources.

## Phase 4: Farmer Profile

- Profile API
- Profile page
- Farmer number
- Personal information
- Cooperative information

**Done when:** a farmer can securely view their own profile.

## Phase 5: Deliveries

- Admin delivery creation
- Delivery references
- Quantity validation
- Delivery history
- Farmer delivery view
- Admin search/filter

**Done when:** a farmer can verify their recorded coffee deliveries.

## Phase 6: Payments

- Admin payment recording
- Payment references
- Payment status
- Payment history
- Delivery/payment relationship
- Farmer payment view

**Done when:** a farmer can see exactly what has been paid and the status of each payment.

## Phase 7: Dashboard

- Total delivered quantity
- Total paid
- Pending amount
- Recent deliveries
- Recent payments
- Season summary

**Done when:** the farmer can understand their financial position from one page.

## Phase 8: Audit Trail

- Record important actions
- Record responsible user
- Record affected entity
- Record timestamps
- Record relevant changes

**Done when:** administrators can determine who changed what and when.

## Phase 9: Testing

Test:

- Authentication
- Authorization
- Farmer isolation
- Delivery validation
- Payment validation
- Database constraints
- API errors
- Security cases

**Done when:** core business rules are covered by tests.

## Phase 10: Deployment

Prepare:

- Production database
- Backend deployment
- Frontend deployment
- HTTPS
- Environment variables
- Database backups
- Logging
- Rate limiting
- Secure headers

**Done when:** the production system works without development credentials or configuration.

---

# Future Development

## USSD

USSD is planned after the core web system is stable.

Future flow:

```text
Farmer Phone
     |
    USSD
     |
USSD Provider
     |
Node.js API
     |
   MySQL
```

Possible menu:

```text
F-Pay

1. My Deliveries
2. My Payments
3. My Balance
4. Payment History
```

USSD should reuse existing backend business logic.

## Mobile Money

Mobile money integration will be added later.

Future flow:

```text
Payment Request
      |
Mobile Money Provider
      |
Webhook/Callback
      |
F-Pay Backend
      |
Verify Transaction
      |
Update Payment
      |
Audit Log
      |
Farmer Notification
```

A payment should only become `COMPLETED` after successful transaction verification.

## SMS

SMS can later notify farmers when:

- A delivery is recorded
- A payment is completed
- A payment is pending
- Other important account events occur

## Blockchain

Blockchain is not part of the MVP.

The initial system will use:

- Database constraints
- Unique transaction references
- Timestamps
- Access control
- Audit logs
- Controlled transaction updates

Blockchain should only be introduced if a genuine business or regulatory requirement justifies it.

---

# Security Principles

F-Pay handles financial and farmer information, so security is part of the core implementation.

The project must:

- Never store plain-text passwords
- Use strong JWT secrets
- Validate all input
- Use parameterized database queries
- Enforce authorization on the backend
- Prevent farmers from accessing other farmers' records
- Keep secrets out of Git
- Use HTTPS in production
- Add rate limiting
- Log important administrative actions
- Avoid silently deleting financial records

---

# MVP Definition of Done

The MVP is considered functional when:

```text
[ ] MySQL works
[ ] Node.js connects to MySQL
[ ] Next.js connects to Node.js
[ ] Farmer registration works
[ ] Farmer login works
[ ] JWT authentication works
[ ] Role authorization works
[ ] Farmer profile works
[ ] Admin management works
[ ] Delivery records work
[ ] Payment records work
[ ] Farmer dashboard works
[ ] Delivery references are unique
[ ] Payment references are unique
[ ] Audit logs work
[ ] Validation works
[ ] Error handling is consistent
[ ] Tests pass
[ ] Production deployment works
```

---

# Development Principle

F-Pay should not be built as a collection of impressive features.

The priority is:

```text
Reliable Database
       ↓
Correct Backend Logic
       ↓
Secure Authentication
       ↓
Accurate Delivery Records
       ↓
Accurate Payment Records
       ↓
Useful Farmer Interface
       ↓
Testing
       ↓
Deployment
       ↓
USSD
       ↓
Mobile Money
       ↓
SMS
       ↓
Advanced Traceability
```

The core system must work before additional channels and integrations are introduced.

---

# License

License to be determined.
