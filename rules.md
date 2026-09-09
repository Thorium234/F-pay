# F-Pay Engineering Specification

**Document:** `specs.md`  
**Status:** Mandatory engineering contract  
**Project:** F-Pay (Farmer Pay)  
**Frontend:** Next.js  
**Backend:** Node.js  
**Database:** MySQL  
**Authentication:** JWT  
**API:** REST

---

# 0. Purpose

This document is a strict engineering contract for all human developers and coding agents working on F-Pay.

The agent MUST treat this document as higher priority than convenience, speed, assumptions, or aesthetic improvements.

The objective is not to produce the largest amount of code.

The objective is to produce a **small, correct, testable, maintainable system**.

A coding agent MUST NOT:

- Invent requirements.
- Invent APIs.
- Invent database fields without justification.
- Implement unfinished future features.
- Modify unrelated parts of the application.
- Hide errors to make tests appear successful.
- Disable tests to make the build pass.
- Remove validation to simplify development.
- Replace real business logic with mock logic in production code.
- Claim a feature is complete when it is only partially implemented.
- Continue generating code when the repository is in a broken state.
- Create duplicate implementations of the same business rule.

When requirements are unclear, the agent MUST stop and identify the ambiguity rather than silently making a large architectural decision.

---

# 1. Non-Negotiable Rules

## 1.1 Build incrementally

Never implement the entire system in one operation.

Every feature MUST follow:

```text
Understand
   ↓
Inspect existing code
   ↓
Plan minimal change
   ↓
Implement
   ↓
Format/lint
   ↓
Test
   ↓
Verify integration
   ↓
Review changed files
   ↓
Continue
```

An agent MUST NOT proceed to the next major feature if the current feature is broken.

---

## 1.2 Never hide failures

Forbidden:

```text
try {
   ...
} catch {
   return []
}
```

when the real operation failed.

Forbidden:

```text
catch (error) {
   console.log(error)
}
```

without proper error handling.

Forbidden:

- Ignoring database errors.
- Returning fake success responses.
- Returning empty data after a failed database query.
- Suppressing TypeScript/compiler errors.
- Disabling lint rules merely to pass lint.
- Disabling tests.
- Commenting out broken code and declaring the feature complete.

Errors must be handled intentionally.

---

# 2. Scope Control

## 2.1 MVP scope

The current implementation is limited to:

1. Farmer registration
2. Farmer login
3. JWT authentication
4. Farmer profile
5. Admin authentication
6. Farmer management
7. Delivery management
8. Payment management
9. Farmer dashboard
10. Audit logging
11. Automated tests
12. Deployment readiness

## 2.2 Explicitly forbidden for the MVP

Do NOT implement:

- USSD
- M-Pesa integration
- Payment-provider webhooks
- SMS gateway
- Blockchain
- AI
- Native mobile application
- Complex notification infrastructure
- Cryptocurrency
- Advanced analytics
- Microservices
- Event-driven architecture
- Kubernetes
- Redis unless a concrete requirement appears
- Message queues unless a concrete requirement appears

A future feature MUST NOT be implemented merely because its architecture is already known.

---

# 3. Architecture Rules

The system consists of:

```text
Next.js
   |
   | REST/HTTPS
   ↓
Node.js API
   |
   ↓
MySQL
```

The frontend MUST NOT connect directly to MySQL.

The frontend MUST NOT contain database credentials.

The frontend MUST NOT implement authoritative business rules.

The backend MUST be the authority for:

- Authentication
- Authorization
- Validation
- Financial calculations
- Ownership checks
- Delivery rules
- Payment rules
- Audit logging

---

# 4. Backend Architecture

Use clear separation:

```text
routes
   ↓
controllers
   ↓
services
   ↓
models/repositories
   ↓
database
```

## Routes

Routes define HTTP endpoints.

Routes MUST NOT contain substantial business logic.

## Controllers

Controllers:

- Receive HTTP input.
- Validate/request parsing.
- Call services.
- Return HTTP responses.

Controllers MUST NOT contain large database queries or complex business calculations.

## Services

Services contain business logic.

Examples:

```text
auth.service
farmer.service
delivery.service
payment.service
```

A business rule MUST have one authoritative implementation.

Do not duplicate payment calculations in:

```text
controller
service
frontend
database query
```

---

# 5. Database Rules

MySQL is the source of truth.

All important relationships MUST be enforced at the database level where practical.

Required entities:

```text
users
farmers
deliveries
payments
audit_logs
```

## 5.1 users

Required conceptual fields:

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

Requirements:

- `id` is the primary key.
- `phone` is unique.
- `password_hash` never contains a plain password.
- `role` is restricted to supported roles.
- `is_active` controls whether authentication is allowed.

---

# 6. Farmer Rules

A farmer account MUST have:

```text
user
farmer profile
unique farmer number
```

The relationship MUST be enforced.

A farmer MUST NOT be able to modify:

- Delivery quantity
- Delivery date
- Delivery reference
- Payment amount
- Payment status
- Payment transaction reference
- Audit records

through farmer endpoints.

---

# 7. Delivery Rules

Every delivery MUST have:

```text
id
farmer_id
delivery_reference
delivery_date
quantity_kg
recorded_by
created_at
updated_at
```

Optional business fields may include:

```text
coffee_type
quality_grade
factory_name
```

## Validation

`quantity_kg` MUST:

- Be numeric.
- Be greater than zero.
- Have a sensible maximum validation.
- Never accept negative values.
- Never accept `NaN`.
- Never accept infinity.
- Never silently convert invalid input to zero.

Delivery reference MUST be unique.

A delivery MUST belong to an existing farmer.

The API MUST reject an unknown farmer.

---

# 8. Payment Rules

Every payment MUST have a unique payment reference.

A payment MUST have:

```text
farmer_id
amount
status
payment reference
created timestamp
```

Amount MUST:

- Be numeric.
- Be greater than zero.
- Never be negative.
- Never be `NaN`.
- Never be infinity.
- Never be silently rounded incorrectly.

Financial values MUST NOT be stored using JavaScript floating-point arithmetic when precision matters.

Use MySQL `DECIMAL` for monetary amounts.

Example:

```text
DECIMAL(15,2)
```

Do NOT use:

```text
FLOAT
```

for money.

---

# 9. Payment Status

Only supported statuses may be stored:

```text
PENDING
COMPLETED
FAILED
CANCELLED
```

The backend MUST reject arbitrary status values.

A payment status transition MUST be validated.

Do not allow nonsensical transitions without an explicit business rule.

For example, do not automatically change:

```text
COMPLETED → PENDING
```

just because an admin edited another field.

---

# 10. Financial Data Protection

Financial records are sensitive.

The following operations require extra care:

- Creating payments
- Updating payments
- Changing payment status
- Deleting financial records
- Modifying delivery quantities

Hard deletion of financial records SHOULD NOT be used for normal business corrections.

Prefer:

```text
audit trail
status changes
correction records
```

rather than destroying history.

If deletion is ever introduced, it MUST be explicitly authorized and audited.

---

# 11. Authentication

Use JWT.

Authentication flow:

```text
Login
  ↓
Validate credentials
  ↓
Verify password hash
  ↓
Create JWT
  ↓
Return authentication result
```

Passwords MUST be hashed with a suitable password hashing algorithm such as bcrypt or Argon2.

Never:

```text
password = database.password
```

Never store:

```text
password: "123456"
```

---

# 12. JWT Rules

JWT payload MUST contain only information required for authentication/authorization.

Example:

```json
{
  "sub": "123",
  "role": "FARMER"
}
```

Do NOT store:

- Passwords
- Password hashes
- Full farmer profiles
- Sensitive financial data
- Large database objects

JWT secrets MUST come from environment configuration.

Never hard-code:

```text
JWT_SECRET
```

into source code.

Tokens MUST have an expiration.

---

# 13. Authorization

Authentication answers:

> Who are you?

Authorization answers:

> What are you allowed to do?

Both MUST be implemented.

Example:

```text
Farmer → /api/farmers/me
Allowed

Farmer → /api/admin/payments
Forbidden

Admin → /api/admin/payments
Allowed
```

Frontend hiding a button does NOT count as authorization.

The backend MUST enforce authorization.

---

# 14. IDOR Prevention

The agent MUST actively prevent insecure direct object references.

Bad:

```text
GET /api/farmers/25
```

where any authenticated farmer can change `25` and view another farmer.

For farmer-owned resources, ownership MUST be checked server-side.

Example:

```text
Authenticated user
       ↓
Resolve farmer
       ↓
Query resource
       ↓
WHERE farmer_id = authenticatedFarmerId
```

Never trust a farmer-supplied `farmer_id`.

---

# 15. Input Validation

Every external input is untrusted.

Validate:

- JSON bodies
- Query parameters
- Route parameters
- Headers where relevant
- IDs
- Dates
- Amounts
- Quantities
- Phone numbers
- Enum values

Validation MUST happen before business logic.

Do not rely on frontend validation.

Frontend validation improves user experience.

Backend validation provides security and correctness.

---

# 16. SQL Security

Use parameterized queries or a trusted ORM/query builder.

Never construct SQL using string interpolation from user input.

Forbidden:

```js
const sql = `SELECT * FROM farmers WHERE phone = '${phone}'`;
```

Required pattern:

```text
Parameterized query
```

or a properly configured ORM.

---

# 17. API Response Contract

Use a consistent response format.

Success:

```json
{
  "success": true,
  "data": {}
}
```

Error:

```json
{
  "success": false,
  "message": "Invalid credentials"
}
```

Validation error:

```json
{
  "success": false,
  "message": "Validation failed",
  "errors": {}
}
```

Do not return database stack traces to clients.

---

# 18. HTTP Status Codes

Use appropriate status codes.

```text
200 OK
201 Created
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
422 Unprocessable Entity
500 Internal Server Error
```

Do not return `200 OK` for every operation.

Examples:

Duplicate phone:

```text
409 Conflict
```

Invalid credentials:

```text
401 Unauthorized
```

Authenticated farmer attempting admin operation:

```text
403 Forbidden
```

Missing resource:

```text
404 Not Found
```

---

# 19. Error Handling

Use centralized error handling.

Controllers SHOULD NOT each invent their own error response format.

Production responses MUST NOT expose:

- SQL queries
- Stack traces
- Environment variables
- JWT secrets
- Database passwords
- Internal filesystem paths

Detailed errors belong in server-side logs.

---

# 20. Logging

Logs MUST be useful.

Log:

- Server startup
- Database connection status
- Authentication failures where appropriate
- Important administrative operations
- Unexpected errors

Do NOT log:

- Passwords
- JWT secrets
- Full authentication tokens
- Database passwords
- Sensitive personal data unnecessarily

---

# 21. Audit Logging

Important mutations MUST create audit records.

At minimum:

```text
FARMER_CREATED
FARMER_UPDATED
DELIVERY_CREATED
DELIVERY_UPDATED
PAYMENT_CREATED
PAYMENT_UPDATED
PAYMENT_STATUS_CHANGED
```

An audit entry should identify:

```text
who
what
which record
when
```

Audit logs MUST NOT be editable by normal farmers.

---

# 22. Transactions

Use database transactions when multiple related writes must succeed together.

Example farmer registration:

```text
BEGIN
  Create user
  Create farmer
COMMIT
```

If farmer creation fails:

```text
ROLLBACK
```

Do not leave:

```text
user exists
farmer does not exist
```

when both are logically required.

Payment operations that update multiple financial records MUST be reviewed for transaction requirements.

---

# 23. Concurrency

Do not assume only one request will happen at a time.

The backend MUST consider:

```text
two admins
same farmer
same delivery
same payment
```

Unique database constraints MUST protect against duplicate records.

Do not rely only on:

```js
if (!exists) {
    create()
}
```

because concurrent requests can both pass the check.

The database constraint remains the final protection.

---

# 24. Frontend Rules

The Next.js frontend MUST:

- Use the backend API.
- Never contain database credentials.
- Never trust frontend authorization.
- Handle loading states.
- Handle error states.
- Handle empty states.
- Handle expired authentication.
- Prevent accidental duplicate submissions.
- Validate user input for usability.

Every data-dependent page MUST have:

```text
Loading
Success
Empty
Error
```

states where applicable.

---

# 25. No Fake Data

Mock data is allowed only during isolated UI development.

It MUST NOT remain connected to production functionality.

Forbidden:

```js
const payments = [
  { amount: 50000 }
];
```

when the real API is supposed to provide payments.

If an endpoint is not implemented, clearly mark it as incomplete.

Never fake successful API responses.

---

# 26. Dashboard Rules

Dashboard numbers MUST come from the backend.

The frontend MUST NOT independently calculate authoritative financial totals from partial records.

Example:

```text
GET /api/farmers/me/summary
```

The backend calculates:

```text
total delivered
total paid
pending amount
```

The frontend displays the result.

This prevents different parts of the application from producing different numbers.

---

# 27. Data Ownership

A farmer can only access their own:

```text
profile
deliveries
payments
summary
```

Admin access is explicitly role-controlled.

Every resource query MUST answer:

```text
Who is requesting this?
What role do they have?
Which records are they allowed to access?
```

---

# 28. API Design Rules

Endpoints MUST be predictable.

Use:

```text
/api/auth/...
/api/farmers/...
/api/admin/...
```

Do not create random endpoint naming.

Avoid:

```text
/getFarmerData
/getPaymentsNow
/doPaymentThing
```

Prefer resource-oriented endpoints:

```text
GET /api/farmers/me/payments
GET /api/farmers/me/deliveries
POST /api/admin/payments
```

---

# 29. Database Migration Rules

Database changes MUST be represented as migrations.

Never depend on manually changing the production database.

Every schema change must be reproducible.

Example:

```text
migration 001
migration 002
migration 003
```

A new developer must be able to create the database from the repository.

---

# 30. Environment Configuration

Secrets MUST be environment variables.

Required categories:

```text
DATABASE
JWT
APPLICATION
```

Commit:

```text
.env.example
```

Never commit:

```text
.env
.env.local
real credentials
production secrets
```

---

# 31. Dependency Rules

Before adding a dependency, the agent MUST ask:

1. Is it actually required?
2. Can the existing stack solve the problem?
3. Is it maintained?
4. Does it introduce unnecessary complexity?
5. Does it create security or licensing concerns?

Do not install packages simply because they are popular.

Do not introduce multiple libraries that solve the same problem.

---

# 32. Type Safety

If TypeScript is used, avoid:

```text
any
```

unless there is a documented reason.

API request and response structures SHOULD have explicit types.

Database entities SHOULD have defined types.

Do not silence compiler errors with:

```text
@ts-ignore
```

unless absolutely necessary and documented.

---

# 33. Testing Contract

A feature is NOT complete because the page renders.

A feature is complete only when:

```text
Code
+
Validation
+
Authorization
+
Database behavior
+
Error handling
+
Tests
```

are working.

---

# 34. Required Authentication Tests

At minimum:

```text
Register valid farmer
Register duplicate phone
Register invalid input
Login valid credentials
Login invalid password
Login unknown phone
Protected route without token
Protected route with invalid token
Protected route with expired token
Farmer accessing admin endpoint
Admin accessing authorized endpoint
```

---

# 35. Required Farmer Security Tests

Test:

```text
Farmer A requests Farmer B's profile
Farmer A requests Farmer B's delivery
Farmer A requests Farmer B's payment
Farmer A modifies another farmer's data
Farmer changes farmer_id in request body
```

All unauthorized operations MUST fail.

---

# 36. Required Delivery Tests

Test:

```text
Valid delivery
Zero quantity
Negative quantity
Invalid quantity
Unknown farmer
Duplicate delivery reference
Unauthorized creation
Unauthorized update
Farmer reading own delivery
Farmer reading another farmer's delivery
```

---

# 37. Required Payment Tests

Test:

```text
Valid payment
Zero amount
Negative amount
Invalid amount
Unknown farmer
Duplicate payment reference
Invalid status
Unauthorized creation
Unauthorized update
Farmer reading own payment
Farmer reading another farmer's payment
```

---

# 38. Test Before and After Changes

Before changing a feature:

```text
Run existing tests.
```

After changing it:

```text
Run targeted tests.
Run full test suite.
Run lint/type checks.
```

An agent MUST NOT assume an unrelated change is safe.

---

# 39. Definition of Done

A task is DONE only if:

- [ ] Requirement is understood.
- [ ] Existing implementation was inspected.
- [ ] Minimal required files were changed.
- [ ] Input validation exists.
- [ ] Authorization exists where required.
- [ ] Database constraints exist where required.
- [ ] Error handling exists.
- [ ] Tests were added or updated.
- [ ] Existing tests still pass.
- [ ] Lint passes.
- [ ] Type checks pass if applicable.
- [ ] No secrets were added.
- [ ] No unrelated code was changed.
- [ ] No TODO was used to hide incomplete functionality.
- [ ] The implementation was manually reviewed for obvious security problems.

---

# 40. Agent Workflow

Every coding task MUST follow this workflow.

## Step 1: Inspect

Before editing:

```text
Inspect repository
Inspect package.json
Inspect database configuration
Inspect existing routes
Inspect existing services
Inspect existing tests
Inspect environment configuration
```

Never assume the repository is empty.

---

## Step 2: State the change

Internally determine:

```text
What is being changed?
Why?
Which files are required?
What existing behavior could break?
What tests are required?
```

Do not modify unrelated files.

---

## Step 3: Implement the smallest correct change

Prefer:

```text
one feature
one business rule
one testable change
```

over large rewrites.

---

## Step 4: Verify immediately

Run:

```text
tests
lint
typecheck
build
```

as applicable to the project.

Do not accumulate multiple unverified features.

---

## Step 5: Investigate failures

If a test fails:

```text
Read the actual error
Find the root cause
Fix the root cause
Run the failing test again
Run the full suite
```

Do not randomly modify multiple files until the error disappears.

---

# 41. Bug Fixing Rules

When fixing a bug:

## Forbidden

```text
Change random code
Run application
Hope it works
```

## Required

```text
Reproduce
   ↓
Identify root cause
   ↓
Write/identify regression test
   ↓
Fix root cause
   ↓
Run regression test
   ↓
Run full test suite
```

A bug fix SHOULD include a regression test whenever practical.

---

# 42. No Infinite Debugging Loops

A coding agent MUST NOT repeatedly make speculative changes.

If the same failure persists after reasonable attempts:

```text
STOP
```

Then produce:

```text
Problem
Observed error
What was tested
What changed
Current hypothesis
What information is missing
```

The agent MUST NOT generate dozens of increasingly unrelated changes.

---

# 43. No Architecture Thrashing

Do not repeatedly switch:

```text
Express → Fastify → NestJS → Express
```

or:

```text
ORM A → ORM B → raw SQL → ORM A
```

without a concrete requirement.

Choose a simple architecture and stabilize it.

---

# 44. No Premature Optimization

Do not introduce:

```text
Redis
queues
caching layers
microservices
WebSockets
distributed locks
```

unless actual requirements demonstrate the need.

Correctness comes before performance optimization.

---

# 45. No Premature Blockchain

Do not implement blockchain to satisfy the word "traceability".

The MVP traceability mechanism is:

```text
Unique reference
+
Database record
+
Timestamp
+
User attribution
+
Audit log
+
Access control
```

Blockchain may be evaluated later.

---

# 46. No Premature USSD

USSD must not be implemented inside the current web MVP.

When eventually implemented, USSD MUST call the same backend business services.

Bad architecture:

```text
Web → MySQL
USSD → MySQL
```

Preferred:

```text
Web → API → Services → MySQL
USSD → API/Services → MySQL
```

---

# 47. Security Review Before Merge

Before considering a feature complete, check:

```text
[ ] Authentication
[ ] Authorization
[ ] IDOR
[ ] Input validation
[ ] SQL injection
[ ] Sensitive data exposure
[ ] Password handling
[ ] JWT handling
[ ] Error leakage
[ ] Rate limiting where appropriate
[ ] Audit logging
```

---

# 48. Performance Baseline

Do not optimize blindly.

The first goal is correct database access.

Avoid:

```text
N+1 queries
```

where practical.

Use indexes for frequently queried fields such as:

```text
users.phone
farmers.user_id
farmers.farmer_number
deliveries.farmer_id
deliveries.delivery_reference
payments.farmer_id
payments.payment_reference
```

Only add additional indexes based on actual query requirements.

---

# 49. API Pagination

Administrative lists MUST eventually support pagination.

Do not return an unlimited number of database records.

For example:

```text
GET /api/admin/farmers?page=1&limit=20
```

The backend MUST enforce a maximum page size.

Never allow:

```text
limit=999999999
```

to cause an uncontrolled database query.

---

# 50. Database Query Safety

Every query MUST have a clear purpose.

Avoid:

```text
SELECT *
```

when only a few fields are required, especially for sensitive entities.

Do not return:

```text
password_hash
```

through normal API responses.

---

# 51. Secrets and Sensitive Data

Never expose:

```text
password_hash
JWT_SECRET
DATABASE_PASSWORD
API_SECRET
private keys
provider credentials
```

in:

- API responses
- frontend bundles
- logs
- Git history
- screenshots
- test fixtures

---

# 52. Git Rules

Commits SHOULD be small and meaningful.

Examples:

```text
feat(auth): add farmer registration
feat(auth): add JWT login
feat(deliveries): add delivery creation
feat(payments): add payment recording
test(auth): add login authorization tests
fix(farmers): prevent cross-account access
```

Do not commit:

```text
.env
node_modules
build artifacts
debug dumps
credentials
```

---

# 53. Pull Request / Merge Gate

No feature should be merged if:

```text
tests fail
lint fails
build fails
security behavior is broken
database migration is missing
required environment configuration is undocumented
```

"Works on my machine" is not acceptance criteria.

---

# 54. Frontend UX Minimum Standard

The interface does NOT need fancy animation.

It MUST be:

- Clear
- Responsive
- Understandable
- Accessible
- Functional

Every important operation needs clear feedback.

Examples:

```text
Saving...
Saved successfully
Failed to save
```

Do not use animations to conceal slow or broken functionality.

---

# 55. Loading and Error States

Every API-driven component must handle:

```text
Loading
Success
Empty
Error
```

Example:

```text
Loading deliveries...

No deliveries found.

Unable to load deliveries. Try again.
```

Never leave the user staring at a blank screen.

---

# 56. Form Rules

Forms MUST:

- Validate required fields.
- Show useful validation errors.
- Prevent duplicate submission.
- Disable submission while processing where appropriate.
- Display server-side validation errors.
- Never expose raw backend stack traces.

---

# 57. Authentication State

The frontend MUST correctly handle:

```text
authenticated
unauthenticated
expired session
logout
```

When a JWT becomes invalid:

```text
Clear authentication state
Redirect to login
```

Do not repeatedly retry an expired token indefinitely.

---

# 58. Infinite Request Protection

The frontend MUST NOT create accidental infinite API requests.

Be especially careful with:

```text
useEffect
setState
router.push
authentication redirects
token refresh
```

Any request retry mechanism MUST have a bounded retry strategy.

Never implement:

```text
retry forever
```

---

# 59. Infinite Render Protection

Avoid state effects that create:

```text
render
→ state update
→ render
→ state update
→ ...
```

Before adding an effect, identify:

```text
What triggers it?
What does it update?
Can that update trigger the effect again?
```

---

# 60. No Silent Fallbacks

Bad:

```text
API failed
↓
show fake dashboard
```

Correct:

```text
API failed
↓
show error state
↓
allow retry
```

The system must never create the illusion that financial information is correct when the database request failed.

---

# 61. Financial Calculation Rules

Financial calculations MUST be deterministic.

The same input MUST produce the same result.

Never rely on frontend formatting for financial calculations.

Avoid:

```js
0.1 + 0.2
```

as the basis for financial accounting.

Use decimal-safe calculations and database `DECIMAL` values.

---

# 62. Time and Date Rules

Store timestamps consistently.

Do not rely on the user's browser clock for authoritative transaction timestamps.

Important record creation times MUST originate from the backend/database.

Date display can be localized in the frontend.

---

# 63. Referential Integrity

Do not create orphan records.

Examples:

```text
delivery.farmer_id
```

must refer to an existing farmer.

```text
payment.farmer_id
```

must refer to an existing farmer.

Where a payment references a delivery:

```text
payment.delivery_id
```

must refer to a valid delivery.

Use foreign keys where appropriate.

---

# 64. No Destructive Database Changes Without Explicit Approval

An agent MUST NOT casually execute:

```text
DROP DATABASE
DROP TABLE
TRUNCATE
DELETE FROM
```

against a real environment.

Destructive migrations require explicit review.

Development seed/reset operations must clearly identify that they are destructive.

---

# 65. Production Safety

Production configuration MUST NOT use:

```text
DEBUG=true
weak JWT secrets
default passwords
development credentials
wildcard CORS without justification
```

Production database backups MUST be considered before deployment.

---

# 66. CORS

CORS MUST be explicitly configured.

Do not blindly use:

```text
Access-Control-Allow-Origin: *
```

for an authenticated production application unless there is a deliberate reason.

Allow only trusted frontend origins.

---

# 67. Rate Limiting

Authentication endpoints SHOULD be rate limited.

At minimum, protect:

```text
/login
/register
```

from obvious abuse.

Do not build a complex distributed rate limiting system for the MVP.

A simple appropriate implementation is sufficient.

---

# 68. API Versioning

Do not add versioning complexity unless required.

If versioning becomes necessary, use a predictable structure such as:

```text
/api/v1/...
```

Do not mix:

```text
/api/login
/api/v1/farmers
/api/v2/payments
```

without a deliberate migration strategy.

---

# 69. Documentation

Every major API feature MUST be documented.

Documentation should identify:

```text
Endpoint
Method
Authentication requirement
Request
Response
Validation
Possible errors
```

Do not allow undocumented APIs to become permanent dependencies.

---

# 70. Agent Stop Conditions

The coding agent MUST stop instead of guessing when:

1. A required business rule is contradictory.
2. A database migration could destroy existing data.
3. Authentication behavior is ambiguous.
4. Payment semantics are unclear.
5. A requested change requires a major architectural decision not defined by the project.
6. Existing code conflicts with the specification.
7. Tests fail for an unrelated pre-existing reason and the agent cannot safely determine whether its change caused the failure.
8. A dependency or external integration requires credentials that are not available.
9. The requested implementation would compromise security.
10. The agent is making repeated speculative changes without identifying the root cause.

When stopping, report the exact blocker.

---

# 71. Agent Completion Report

After completing a task, the agent MUST provide:

```text
IMPLEMENTED
- What changed

FILES CHANGED
- file 1
- file 2

TESTS
- Tests run
- Result

VALIDATION
- Lint result
- Typecheck result
- Build result

DATABASE
- Migration added/changed

SECURITY
- Authorization checked
- Input validation checked
- Sensitive data checked

KNOWN LIMITATIONS
- Anything genuinely incomplete
```

Never claim:

```text
All done
```

if tests or builds are failing.

---

# 72. Final Engineering Rule

The most important rule in this specification is:

> **Never add complexity to hide uncertainty.**

When something is broken:

```text
Do not patch randomly.
Do not fake success.
Do not disable the test.
Do not rewrite the whole project.
Do not add five new libraries.
Do not continue blindly.
```

Instead:

```text
Reproduce
   ↓
Understand
   ↓
Identify root cause
   ↓
Make the smallest correct change
   ↓
Test
   ↓
Verify
```

F-Pay is a financial and traceability system.

Correctness, security, data integrity, and auditability take priority over speed, visual effects, feature count, and architectural fashion.
