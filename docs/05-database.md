# SpareLink | Database & Data Model

> **The missing part is just one message away.**

Part 5 defines the database architecture and data model for the SpareLink MVP. It follows Parts 1–4 and is intentionally relational, secure, understandable, and small enough for a student hackathon team to implement.

The database must support the complete workflow from a shop account and inventory through AI-assisted requests, matching, supplier confirmation, transactions, and notifications.

---

# 1. Database Objective

The database supports:

~~~text
User
  ↓
Shop
  ↓
Inventory
  ↓
Spare-Part Request
  ↓
AI-processed Requirement
  ↓
Matching
  ↓
Supplier Request
  ↓
Transaction
  ↓
Notification
~~~

## Database Role

The database is the system of record for:

- User/application identity
- Shop ownership and membership
- Canonical spare-part information
- Shop-specific inventory
- Spare-part requests
- AI analysis attempts
- Potential matches
- Supplier requests
- Confirmed transactions
- Notifications
- File metadata

The database should not contain secrets such as passwords or raw API keys, and large uploaded images should remain in object storage.

---

# 2. Database Technology

## Recommended Technology

**PostgreSQL through Supabase**

This is consistent with Part 4.

| Capability | Role in SpareLink |
|---|---|
| PostgreSQL | Primary relational database |
| Supabase Auth | User authentication |
| Supabase Storage | Images and uploaded media |
| Row Level Security | Defense-in-depth access control |
| FastAPI | Main application/business API |
| PostGIS | Radius and distance queries |

## Why It Fits

PostgreSQL matches the relational nature of SpareLink:

~~~text
Users
  ↓
Shops
  ↓
Inventory
  ↓
Requests
  ↓
Transactions
~~~

Supabase reduces infrastructure work for a hackathon team while allowing the backend architecture from Part 4 to remain in control of business rules.

## Authentication Integration

Supabase Auth owns authentication identities.

The application database stores a users record linked to the external Auth identity.

Passwords are **not duplicated** in the application database.

## Storage Integration

The database stores metadata and storage references. The actual image bytes live in Supabase Storage.

## Row Level Security

RLS provides an additional protection layer for data accessed through Supabase.

The FastAPI backend remains responsible for explicit application authorization.

## API Interaction

Preferred flow:

~~~text
Frontend
   ↓
Supabase Auth
   ↓
Access Token
   ↓
FastAPI
   ↓
Authorization + Business Rules
   ↓
PostgreSQL
~~~

The frontend should not directly perform privileged business mutations.

---

# 3. Data Model Principles

## Normalization

Separate reusable part information from shop-specific inventory.

For example:

~~~text
PART
Samsung Galaxy M14 Display

INVENTORY
Shop A → Part → Quantity 1 → ₹1850
Shop B → Part → Quantity 2 → ₹1750
~~~

This avoids storing the same canonical part repeatedly.

## Data Integrity

Use:

- Primary keys
- Foreign keys
- CHECK constraints
- Unique constraints
- Controlled status values

## Clear Ownership

Every private resource should have a clear owner or owning shop.

## Minimal Duplication

Store a canonical relationship where possible, but keep historical snapshots such as the original user input and confirmed request fields where auditability requires them.

## Foreign Keys

Use foreign keys for important relationships and avoid destructive cascades for historical records.

## Indexing

Index the queries used most often:

- Nearby shop lookup
- Inventory by shop
- Inventory by part
- Request status
- Supplier requests
- Notifications
- Transaction history

## Auditability

Keep:

- Original request text
- AI analysis output
- Important timestamps
- Supplier response
- Transaction record

## Security

Enforce authorization before allowing shop-owned data to be modified.

## Extensibility

Use flexible categories, normalized part fields, and independent modules so more repair industries can be added later.

## Reliable Status Management

Keep the lifecycle explicit and validate state transitions in the backend.

---

# 4. Core Entities

The recommended MVP uses these tables:

| Entity | Final Table? | Purpose |
|---|---|---|
| Users | Yes | Application-level user record linked to Auth |
| Shops | Yes | Repair business profile |
| Shop Members | Yes | User-to-shop ownership/membership |
| Parts | Yes | Canonical spare-part identity |
| Inventory | Yes | Shop-specific stock |
| Spare Requests | Yes | A shop's requirement |
| AI Analyses | Yes | Auditable AI processing attempts |
| Request Matches | Yes | Potential request-to-inventory matches |
| Supplier Requests | Yes | Actual request sent to a supplier |
| Transactions | Yes | Confirmed exchange/fulfillment |
| Notifications | Yes | In-app events for users |
| Uploaded Files | Yes | Metadata for files stored externally |

No separate rating, delivery, payment, analytics, or verification tables are required for the MVP.

---

# 5. Entity Relationship Diagram

~~~mermaid
erDiagram
    USERS ||--o{ SHOP_MEMBERS : belongs_to
    SHOPS ||--o{ SHOP_MEMBERS : has
    SHOPS ||--o{ INVENTORY : owns
    PARTS ||--o{ INVENTORY : stocked_as

    SHOPS ||--o{ SPARE_REQUESTS : creates
    PARTS o|--o{ SPARE_REQUESTS : normalizes_to
    SPARE_REQUESTS ||--o{ AI_ANALYSES : has

    SPARE_REQUESTS ||--o{ REQUEST_MATCHES : produces
    INVENTORY ||--o{ REQUEST_MATCHES : candidate

    SPARE_REQUESTS ||--o{ SUPPLIER_REQUESTS : sends
    SHOPS ||--o{ SUPPLIER_REQUESTS : receives
    INVENTORY ||--o{ SUPPLIER_REQUESTS : fulfills_from

    SUPPLIER_REQUESTS ||--o| TRANSACTIONS : becomes
    SHOPS ||--o{ TRANSACTIONS : supplies
    SHOPS ||--o{ TRANSACTIONS : receives
    PARTS ||--o{ TRANSACTIONS : records
    INVENTORY ||--o{ TRANSACTIONS : fulfilled_from

    USERS ||--o{ NOTIFICATIONS : receives
    USERS ||--o{ UPLOADED_FILES : owns
    SHOPS ||--o{ UPLOADED_FILES : owns
    SPARE_REQUESTS ||--o{ UPLOADED_FILES : attaches
    INVENTORY ||--o{ UPLOADED_FILES : attaches
~~~

### Relationship Summary

~~~text
User
 └── Shop Membership
       └── Shop
             ├── Inventory ── Part
             ├── Spare Request
             │      ├── AI Analysis
             │      ├── Matches ── Inventory
             │      └── Supplier Requests ── Supplier Shop
             │                                  ↓
             │                               Transaction
             └── Notifications / Files
~~~

---

# 6. Users Table

## Purpose

Represents the application-level user linked to a Supabase Auth identity.

## Recommended Fields

| Field | Type | Notes |
|---|---|---|
| id | UUID | Primary key |
| auth_user_id | UUID | References Auth identity |
| name | TEXT | Display name |
| email | TEXT | Optional application copy |
| phone | TEXT | Optional application copy |
| global_role | TEXT | user/admin |
| account_status | TEXT | active/inactive/suspended |
| created_at | TIMESTAMPTZ | UTC |
| updated_at | TIMESTAMPTZ | UTC |

## Role Decision

Do **not** duplicate shop membership roles such as owner, manager, or staff inside users.

Use:

- users.global_role for application-wide privileges
- shop_members.role for shop-specific permissions

This prevents inconsistent role information.

## Important Rule

Do not store:

- Password hashes created by the application
- Authentication secrets
- Session tokens

Supabase Auth handles authentication credentials.

---

# 7. Shops Table

## Purpose

Represents a repair business.

## Recommended Fields

| Field | Type | Notes |
|---|---|---|
| id | UUID | Primary key |
| name | TEXT | Shop name |
| category | TEXT | Mobile Repair initially; extensible |
| phone | TEXT | Business contact |
| address | TEXT | Display address/area |
| location | GEOGRAPHY(Point, 4326) | PostGIS coordinate |
| status | TEXT | active/inactive |
| created_at | TIMESTAMPTZ | UTC |
| updated_at | TIMESTAMPTZ | UTC |

## Ownership

Do not store owner_id directly on shops.

Use shop_members so that one shop can support multiple users.

## Future Categories

The category should remain a text/controlled application value rather than a hard-coded table relationship in the MVP.

Possible categories:

- mobile_repair
- bike_repair
- car_repair
- appliance_repair
- electrical
- electronics
- machinery

Only mobile_repair needs to be active for the first demo.

## Location

Store one shop coordinate as a PostGIS geography point.

This allows:

- Radius search
- Distance calculation
- Local match ranking

Precise coordinates should not be exposed publicly unless necessary.

---

# 8. Shop Membership / Ownership

## Final Decision

Yes, SpareLink should support more than one user per shop.

Use a shop_members table.

## Why

A shop may eventually have:

- Owner
- Manager
- Staff member

Even in the MVP, this avoids designing the database around one login per business.

## Fields

| Field | Type |
|---|---|
| id | UUID |
| shop_id | UUID |
| user_id | UUID |
| role | TEXT |
| created_at | TIMESTAMPTZ |
| updated_at | TIMESTAMPTZ |

## Roles

~~~text
owner
manager
staff
~~~

## Permission Model

| Action | Owner | Manager | Staff |
|---|---:|---:|---:|
| Edit shop profile | Yes | Yes, if enabled | No |
| Add inventory | Yes | Yes | Yes |
| Edit inventory | Yes | Yes | Yes |
| Remove inventory | Yes | Yes | Restricted/Yes |
| Create request | Yes | Yes | Yes |
| Accept supplier request | Yes | Yes | Yes |
| View transactions | Yes | Yes | Yes |
| Change ownership | Yes | No | No |

The backend remains the final authorization layer.

## Ownership Constraint

A shop should have exactly one active owner.

Use a partial unique index to prevent two owner rows from existing at the same time.

---

# 9. Parts Table

## Purpose

Represents the canonical spare part being searched for.

A part is **not** a shop's stock record.

## Recommended Fields

| Field | Type | Purpose |
|---|---|---|
| id | UUID | Primary key |
| name | TEXT | Canonical part name |
| normalized_name | TEXT | Search-normalized form |
| category | TEXT | Part category |
| brand | TEXT | Manufacturer/brand |
| model | TEXT | Relevant device/model |
| manufacturer_part_number | TEXT | Optional exact identifier |
| aliases | TEXT[] | Alternate names |
| description | TEXT | Optional explanation |
| created_at | TIMESTAMPTZ | UTC |
| updated_at | TIMESTAMPTZ | UTC |

## Part vs Inventory Item

### Part

A canonical concept:

~~~text
Samsung
Galaxy M14
Display
~~~

### Inventory Item

A shop-specific stock record:

~~~text
Shop B
Samsung Galaxy M14 Display
Quantity: 2
Condition: New
Price: ₹1850
Availability: Available
~~~

The same part can therefore exist in many shops.

## Field Ownership

### Belongs in Parts

- Canonical name
- Normalized name
- Brand
- Model
- Manufacturer part number
- General description
- Common aliases

### Belongs in Inventory

- Quantity
- Price
- Condition
- Availability
- Shop
- Last updated

Complex compatibility graphs are outside the MVP.

---

# 10. Inventory Table

## Purpose

Connects:

**Shop ↔ Part**

## Recommended Fields

| Field | Type | Notes |
|---|---|---|
| id | UUID | Primary key |
| shop_id | UUID | Owning shop |
| part_id | UUID | Canonical part |
| quantity | INTEGER | Physical quantity |
| reserved_quantity | INTEGER | Quantity reserved by accepted requests |
| price | NUMERIC(12,2) | Optional |
| condition | TEXT | new/used/refurbished/unknown |
| availability_status | TEXT | available/low_stock/out_of_stock/inactive |
| last_quantity_changed_at | TIMESTAMPTZ | Inventory change |
| last_updated_at | TIMESTAMPTZ | Latest update |
| created_at | TIMESTAMPTZ | UTC |
| updated_at | TIMESTAMPTZ | UTC |

## Why reserved_quantity Helps

It provides a simple MVP mechanism to avoid accepting multiple requests for the same physical stock.

Conceptually:

~~~text
Available to promise
=
quantity - reserved_quantity
~~~

An accepted supplier request can reserve stock. The reservation is released on decline/cancel/expiry or converted into a completed transaction.

The exact reservation transaction should use a database transaction and row lock.

## Multiple Shops

Allowed:

~~~text
Shop A → Part X
Shop B → Part X
Shop C → Part X
~~~

## Quantity Updates

Inventory should support:

- Add stock
- Reduce stock
- Increase quantity
- Mark out of stock
- Mark inactive

## Price Differences

Price belongs to inventory, not parts, because different shops can set different prices.

## Condition Differences

Condition belongs to inventory because the same canonical part can be:

- New at Shop A
- Used at Shop B
- Refurbished at Shop C

---

# 11. Inventory Data Freshness

Inventory freshness is critical because database availability is not guaranteed to equal physical availability.

## Required Timestamps

- created_at
- updated_at
- last_quantity_changed_at
- last_updated_at

## Availability Statuses

~~~text
available
low_stock
out_of_stock
inactive
~~~

## Stale Inventory

Do not make inventory automatically disappear merely because it is old.

Instead:

1. Calculate a stale flag using last_updated_at.
2. Show the user that the data may be stale.
3. Require supplier confirmation before accepting.
4. Encourage inventory updates.

The stale threshold should be configurable in application settings rather than hard-coded into the business meaning.

Example behavior:

~~~text
Listed Availability
       ↓
Potential Match
       ↓
Supplier Checks Actual Stock
       ↓
Confirmed Availability
~~~

The MVP must not claim real-time inventory.

---

# 12. Spare Requests Table

## Purpose

Represents:

> **“I need this part.”**

## Recommended Fields

| Field | Type | Purpose |
|---|---|---|
| id | UUID | Primary key |
| requesting_shop_id | UUID | Shop making request |
| part_id | UUID nullable | Canonical matched part after confirmation |
| input_type | TEXT | text/image/voice |
| raw_input | TEXT | Original user input |
| brand | TEXT nullable | Confirmed snapshot |
| model | TEXT nullable | Confirmed snapshot |
| part_name | TEXT | Confirmed part text |
| quantity | INTEGER | Requested quantity |
| urgency | TEXT | low/medium/high |
| request_location | GEOGRAPHY(Point, 4326) | Location snapshot |
| address_snapshot | TEXT nullable | Optional display snapshot |
| search_radius_km | NUMERIC(6,2) | Search radius |
| notes | TEXT nullable | Optional notes |
| status | TEXT | Lifecycle state |
| confirmed_at | TIMESTAMPTZ nullable | User confirmation |
| created_at | TIMESTAMPTZ | UTC |
| updated_at | TIMESTAMPTZ | UTC |

## Why Preserve raw_input?

Raw input supports:

- Auditability
- Debugging
- AI improvement
- Understanding what the user originally typed
- Reproducing incorrect AI interpretations

Example:

~~~text
raw_input:
"Samsung M14 display kavali urgent ga."
~~~

The normalized fields preserve the structured interpretation separately.

## Why Keep Both part_id and Snapshot Fields?

part_id points to the canonical entity.

The snapshot fields preserve what the user confirmed at request time even if canonical part metadata later changes.

This supports historical interpretation without relying entirely on mutable reference data.

---

# 13. AI Analysis Data

## Final Decision

Use a dedicated **ai_analyses** table.

## Why

A request may have:

- Multiple analysis attempts
- A retry after failure
- A corrected interpretation
- A model/version change
- An uncertain first result

Keeping AI attempts separate makes debugging and auditability much cleaner.

## Recommended Fields

| Field | Type | Purpose |
|---|---|---|
| id | UUID | Primary key |
| request_id | UUID | Related request |
| attempt_number | INTEGER | Retry/order |
| ai_provider | TEXT | Provider |
| ai_model | TEXT | Model identifier |
| input_type | TEXT | text/image/voice-transcript |
| raw_ai_output | JSONB nullable | Raw provider result |
| structured_output | JSONB nullable | Parsed schema |
| confidence | NUMERIC(4,3) nullable | 0 to 1 |
| analysis_status | TEXT | completed/uncertain/failed |
| error_code | TEXT nullable | Safe application error |
| created_at | TIMESTAMPTZ | UTC |

## Trust Rule

The database may store an AI result, but that result is not automatically the final request.

The transition remains:

~~~text
AI Output
   ↓
Validation
   ↓
User Confirmation
   ↓
Confirmed Request
~~~

---

# 14. Request Status Model

## Main Request Statuses

~~~text
draft
analyzing
confirmed
searching
matched
request_sent
accepted
handover_pending
completed
declined
cancelled
expired
no_match
~~~

## Status Ownership

### Spare Requests Own

- draft
- analyzing
- confirmed
- searching
- matched
- request_sent
- accepted
- handover_pending
- completed
- declined
- cancelled
- expired
- no_match

### Supplier Requests Own

- pending
- accepted
- declined
- expired
- cancelled

### Transactions Own

- pending
- handover_pending
- completed
- cancelled

Do not duplicate all possible states into every table.

---

# 15. Match Table

## Purpose

Records a **potential match** between a spare request and a shop inventory item.

## Recommended Fields

| Field | Type |
|---|---|
| id | UUID |
| request_id | UUID |
| inventory_id | UUID |
| distance_m | INTEGER |
| match_type | TEXT |
| rank_position | INTEGER |
| created_at | TIMESTAMPTZ |

## Match Types

~~~text
exact
possible
~~~

## Match Score Decision

Do **not** persist match_score in the MVP as a user-facing fact.

Calculate ranking internally during the search.

Store:

- distance snapshot
- match type
- rank position

This keeps the database independent of a specific ranking formula.

If future analytics requires score history, an algorithm_version and score can be added later.

## Important Rule

A match means:

> **Potentially suitable inventory**

It does not mean:

> **Confirmed available part**

Supplier confirmation remains separate.

---

# 16. Supplier Request Table

## Purpose

Separates:

**Potential Match**

from:

**Actual Request Sent to Supplier**

## Recommended Fields

| Field | Type |
|---|---|
| id | UUID |
| request_id | UUID |
| supplier_shop_id | UUID |
| inventory_id | UUID |
| quantity_requested | INTEGER |
| confirmed_quantity | INTEGER nullable |
| confirmed_price | NUMERIC(12,2) nullable |
| handover_method | TEXT nullable |
| supplier_response | TEXT nullable |
| status | TEXT |
| created_at | TIMESTAMPTZ |
| updated_at | TIMESTAMPTZ |
| responded_at | TIMESTAMPTZ nullable |
| accepted_at | TIMESTAMPTZ nullable |
| cancelled_at | TIMESTAMPTZ nullable |

## Statuses

~~~text
pending
accepted
declined
expired
cancelled
~~~

## Why Separate It from Matches?

A request can produce multiple candidate matches:

~~~text
Request
 ├── Match A
 ├── Match B
 └── Match C
~~~

The shop may then send requests selectively:

~~~text
Request
 ├── Supplier Request A
 └── Supplier Request B
~~~

A supplier request contains an actual business action and response.

## One Accepted Supplier

The MVP should allow multiple pending supplier requests but prevent more than one accepted supplier request for the same parent request at the same time.

Use a partial unique index for accepted supplier requests.

---

# 17. Transactions Table

## Purpose

Represents a confirmed exchange/fulfillment event.

A transaction should not be created merely because a match exists.

## Recommended Fields

| Field | Type |
|---|---|
| id | UUID |
| request_id | UUID |
| supplier_request_id | UUID |
| from_shop_id | UUID |
| to_shop_id | UUID |
| inventory_id | UUID |
| part_id | UUID |
| quantity | INTEGER |
| agreed_price | NUMERIC(12,2) nullable |
| handover_method | TEXT |
| status | TEXT |
| completed_at | TIMESTAMPTZ nullable |
| created_at | TIMESTAMPTZ |
| updated_at | TIMESTAMPTZ |

## Direction

For consistency:

~~~text
from_shop_id = supplying shop
to_shop_id   = requesting shop
~~~

## Transaction Status

~~~text
pending
handover_pending
completed
cancelled
~~~

## No Payment System

agreed_price records the agreed amount when provided.

It does not imply that SpareLink processes payments.

---

# 18. Transaction Integrity

The backend/database workflow must ensure:

- A transaction references an existing supplier request.
- The supplier request is accepted before fulfillment.
- Quantity is greater than zero.
- Confirmed quantity does not exceed requested quantity.
- A transaction cannot use a cancelled request.
- A supplier request cannot be completed twice.
- Participating shops must exist.
- from_shop_id and to_shop_id must differ.
- Inventory cannot be over-reserved.

## Cross-Table Rules

Some rules cannot be represented with a simple CHECK constraint.

For example:

> transaction.status = completed only when supplier_request.status = accepted

These rules should be enforced with an application transaction and, where appropriate, database triggers.

## Recommended Completion Transaction

Conceptually:

~~~text
BEGIN
  Lock supplier request
  Verify accepted status

  Lock inventory row
  Verify enough reserved/available quantity

  Update inventory
  Create/finalize transaction
  Update request status
  Create notifications
COMMIT
~~~

This keeps concurrent updates consistent.

---

# 19. Notifications Table

## Purpose

Stores useful in-app notifications.

## Recommended Fields

| Field | Type |
|---|---|
| id | UUID |
| user_id | UUID |
| shop_id | UUID nullable |
| type | TEXT |
| title | TEXT |
| message | TEXT |
| reference_type | TEXT nullable |
| reference_id | UUID nullable |
| is_read | BOOLEAN |
| created_at | TIMESTAMPTZ |

## Notification Types

~~~text
match_found
new_supplier_request
request_accepted
request_declined
request_cancelled
handover_confirmed
transaction_completed
request_expired
~~~

## Relationship to Other Records

reference_type/reference_id can point to the related request, supplier request, or transaction.

Because this is polymorphic, the application layer must validate the reference.

The notification remains an event, not the source of truth. The request/transaction tables remain authoritative.

---

# 20. Files / Images Table

## Purpose

Stores metadata about uploaded files.

The actual image remains in object storage.

## Recommended Fields

| Field | Type |
|---|---|
| id | UUID |
| owner_id | UUID |
| shop_id | UUID nullable |
| request_id | UUID nullable |
| inventory_id | UUID nullable |
| storage_path | TEXT |
| file_type | TEXT |
| mime_type | TEXT |
| file_size | INTEGER |
| created_at | TIMESTAMPTZ |

## Database Record ≠ Image

The database contains:

~~~text
File ID
Storage Path
Owner
Related Entity
File Type
Size
~~~

Storage contains the actual bytes.

## Security

Use:

- Private storage by default
- Generated storage identifiers
- MIME/type validation
- Maximum upload size
- Authorized access
- Short-lived signed URLs where required

## Ownership Rule

Each uploaded file should belong to exactly one main target:

- shop image
- request image
- inventory image

Enforce this with a CHECK constraint requiring exactly one target ID.

---

# 21. Location Data

## Recommended Technology

**PostGIS**

## Stored Data

Shops:

- location
- address

Requests:

- request_location snapshot
- address_snapshot where useful

## Why PostGIS?

It supports the core operations:

- Radius search
- Distance calculation
- Nearby shop filtering

## Core Query Concept

~~~text
Requester location
      ↓
Configured radius
      ↓
Candidate inventory
      ↓
Distance calculation
      ↓
Ranked results
~~~

Use the shop's geospatial index rather than calculating distances in application code for every row.

## Privacy

Do not expose raw latitude/longitude to every user.

The API can return:

- approximate area
- distance
- map-safe location information where needed

---

# 22. Indexing Strategy

Indexes should follow actual MVP query patterns.

## Shop Location

Use a GiST index on shops.location.

Purpose:

- Radius queries
- Nearby search

## Inventory by Shop

B-tree:

~~~text
inventory(shop_id)
~~~

Purpose:

- Inventory page
- Shop-owned inventory CRUD

## Inventory by Part

B-tree:

~~~text
inventory(part_id)
~~~

Purpose:

- Candidate lookup

Also consider a composite index that starts with part_id and availability_status.

## Inventory Availability

Useful composite index:

~~~text
inventory(part_id, availability_status, shop_id)
~~~

## Requests

Indexes on:

~~~text
spare_requests(requesting_shop_id)
spare_requests(status, created_at)
~~~

## Supplier Requests

Indexes on:

~~~text
supplier_requests(supplier_shop_id, status)
supplier_requests(request_id)
~~~

## Matches

Indexes on:

~~~text
request_matches(request_id, rank_position)
request_matches(inventory_id)
~~~

## Notifications

Useful index:

~~~text
notifications(user_id, is_read, created_at DESC)
~~~

## Transactions

Useful indexes:

~~~text
transactions(from_shop_id, created_at DESC)
transactions(to_shop_id, created_at DESC)
transactions(supplier_request_id)
~~~

## Avoid Excessive Indexes

Every index adds write/storage cost. Add only indexes serving real queries.

---

# 23. Unique Constraints

## Recommended

### Shop Membership

One user should have at most one membership record per shop:

~~~text
UNIQUE(shop_id, user_id)
~~~

### Active Owner

Only one owner per shop using a partial unique index.

### Inventory

One active inventory row for a shop and canonical part:

~~~text
UNIQUE(shop_id, part_id)
~~~

### AI Attempts

Unique request + attempt number.

### Matches

A request/inventory pair should not be duplicated:

~~~text
UNIQUE(request_id, inventory_id)
~~~

### Supplier Requests

Prevent repeated requests to the same inventory record for one request:

~~~text
UNIQUE(request_id, inventory_id)
~~~

### Transaction

One supplier request can produce at most one transaction:

~~~text
UNIQUE(supplier_request_id)
~~~

Do not create uniqueness rules that prevent legitimate records such as separate requests on different dates.

---

# 24. Foreign Key Strategy

Major relationships:

~~~text
shop_members.user_id
    ↓
users.id

shop_members.shop_id
    ↓
shops.id

inventory.shop_id
    ↓
shops.id

inventory.part_id
    ↓
parts.id

spare_requests.requesting_shop_id
    ↓
shops.id

spare_requests.part_id
    ↓
parts.id

request_matches.request_id
    ↓
spare_requests.id

request_matches.inventory_id
    ↓
inventory.id

supplier_requests.request_id
    ↓
spare_requests.id

supplier_requests.supplier_shop_id
    ↓
shops.id

supplier_requests.inventory_id
    ↓
inventory.id

transactions.supplier_request_id
    ↓
supplier_requests.id
~~~

## Delete Strategy

Prefer:

- RESTRICT for historical business entities
- Soft/inactivation for shops and inventory
- RESTRICT for parts referenced by transactions
- RESTRICT for requests referenced by transactions

Avoid destructive cascading through:

~~~text
Request → Transaction
Shop → Transaction
Part → Transaction
~~~

Historical transactions should survive later inventory or profile changes.

---

# 25. Soft Delete vs Hard Delete

## Shops

**Soft delete/inactivate**

Use status such as inactive.

Reason: historical transactions must remain understandable.

## Inventory

**Soft delete/inactivate**

Set availability to inactive rather than physically deleting immediately.

## Parts

**Prefer retain**

A part referenced by inventory or history should remain available as a canonical record.

## Requests

**Retain**

Requests are part of audit/history.

Use status:

~~~text
cancelled
expired
completed
~~~

rather than deleting.

## Transactions

**Never hard-delete as a normal user operation.**

Preserve completed and cancelled history.

---

# 26. Audit / Timestamp Strategy

Use UTC internally.

## Standard Fields

~~~text
created_at
updated_at
~~~

## Important Event Fields

### Supplier Requests

- responded_at
- accepted_at
- cancelled_at

### Inventory

- last_quantity_changed_at
- last_updated_at

### Requests

- confirmed_at

### Transactions

- completed_at

## Timezone

Store TIMESTAMPTZ values in UTC.

The frontend converts them to the user's local timezone for display.

---

# 27. Data Validation

Validation should occur at multiple layers.

~~~text
Frontend
   ↓
FastAPI
   ↓
Database Constraints
~~~

## Quantity

Must be greater than zero for requests and transactions.

Inventory quantity may be zero when status is out_of_stock.

## Search Radius

Use a positive configurable range.

For the MVP, a practical application validation range can be:

~~~text
0.5 km ≤ radius ≤ 100 km
~~~

A default such as 10 km can be configured.

## Latitude/Longitude

When represented individually:

~~~text
latitude  ∈ [-90, 90]
longitude ∈ [-180, 180]
~~~

PostGIS point construction should validate usable coordinates.

## Urgency

~~~text
low
medium
high
~~~

## Status

Every status field must use a controlled value.

## Price

Cannot be negative.

## Strings

Backend should enforce reasonable maximum lengths.

## AI Confidence

When present:

~~~text
0 ≤ confidence ≤ 1
~~~

---

# 28. Security & Row Level Security

## Principle

**Least privilege**

Users should see and modify only the data necessary for their shop role.

## Shop Users Can

- View their shop
- Manage their own inventory
- Create requests for their shop
- View relevant requests
- Respond to supplier requests for their shop
- View their shop's transaction history
- View their own notifications

## Shop Users Cannot

- Modify another shop's inventory
- Modify another shop's transactions
- Accept requests for another shop
- View unrelated private business information

## RLS Concept

Supabase RLS should protect rows according to shop membership.

Conceptual pattern:

~~~text
Authenticated User
      ↓
users.auth_user_id
      ↓
shop_members.user_id
      ↓
shop_members.shop_id
      ↓
Owned / Allowed Rows
~~~

Do not create a blanket policy equivalent to:

~~~text
authenticated users can select everything
~~~

## FastAPI Authorization

RLS is defense in depth.

FastAPI must still perform explicit authorization checks before mutations and state transitions.

---

# 29. Sample Database Records

> **Demo Data only. All records below are fictional.**

## 2 Users

| ID | Name | Role |
|---|---|---|
| usr-001 | Rahul Kumar | shop_owner |
| usr-002 | Sneha Reddy | shop_owner |

## 2 Shops

| ID | Name | Category |
|---|---|---|
| shop-001 | Kumar Mobiles | mobile_repair |
| shop-002 | Sri Sai Mobiles | mobile_repair |

## 3 Parts

| ID | Name | Brand | Model |
|---|---|---|---|
| part-001 | Display | Samsung | Galaxy M14 |
| part-002 | Battery | Samsung | Galaxy M14 |
| part-003 | Charging Port | Redmi | Note 13 |

## 4 Inventory Records

| Shop | Part | Qty | Price | Status |
|---|---|---:|---:|---|
| Kumar Mobiles | Samsung Galaxy M14 Display | 0 | 1800 | out_of_stock |
| Sri Sai Mobiles | Samsung Galaxy M14 Display | 1 | 1850 | available |
| Sri Sai Mobiles | Samsung Galaxy M14 Battery | 2 | 650 | available |
| Kumar Mobiles | Redmi Note 13 Charging Port | 3 | 350 | available |

## 1 Spare Request

~~~text
Request ID: req-001
Shop: Kumar Mobiles
Part: Samsung Galaxy M14 Display
Quantity: 1
Urgency: high
Radius: 10 km
Status: completed
~~~

## 2 Matches

~~~text
match-001
Inventory: Sri Sai Mobiles / M14 Display
Distance: 2.4 km
Type: exact
Rank: 1

match-002
Inventory: Demo Shop Inventory / compatible display
Distance: 6.8 km
Type: possible
Rank: 2
~~~

The second inventory record can exist as controlled demo data even if it is not part of the two primary profile shops.

## 1 Supplier Request

~~~text
Supplier Request: sreq-001
Request: req-001
Supplier: Sri Sai Mobiles
Inventory: M14 Display
Requested Quantity: 1
Confirmed Quantity: 1
Status: accepted
~~~

## 1 Transaction

~~~text
Transaction: txn-001
From: Sri Sai Mobiles
To: Kumar Mobiles
Part: Samsung Galaxy M14 Display
Quantity: 1
Agreed Price: 1850
Handover: pickup
Status: completed
~~~

---

# 30. Sample PostgreSQL Schema

The following is a conceptual MVP schema designed to be valid or very close to executable PostgreSQL.

~~~sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;
CREATE EXTENSION IF NOT EXISTS postgis;

CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    auth_user_id UUID NOT NULL UNIQUE REFERENCES auth.users(id) ON DELETE RESTRICT,
    name TEXT NOT NULL,
    email TEXT,
    phone TEXT,
    global_role TEXT NOT NULL DEFAULT 'user'
        CHECK (global_role IN ('user', 'admin')),
    account_status TEXT NOT NULL DEFAULT 'active'
        CHECK (account_status IN ('active', 'inactive', 'suspended')),
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE shops (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT NOT NULL,
    category TEXT NOT NULL,
    phone TEXT,
    address TEXT,
    location GEOGRAPHY(Point, 4326),
    status TEXT NOT NULL DEFAULT 'active'
        CHECK (status IN ('active', 'inactive', 'suspended')),
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE shop_members (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    shop_id UUID NOT NULL REFERENCES shops(id) ON DELETE RESTRICT,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
    role TEXT NOT NULL
        CHECK (role IN ('owner', 'manager', 'staff')),
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    UNIQUE (shop_id, user_id)
);

CREATE TABLE parts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT NOT NULL,
    normalized_name TEXT NOT NULL,
    category TEXT,
    brand TEXT,
    model TEXT,
    manufacturer_part_number TEXT,
    aliases TEXT[] NOT NULL DEFAULT '{}',
    description TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE inventory (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    shop_id UUID NOT NULL REFERENCES shops(id) ON DELETE RESTRICT,
    part_id UUID NOT NULL REFERENCES parts(id) ON DELETE RESTRICT,
    quantity INTEGER NOT NULL DEFAULT 0
        CHECK (quantity >= 0),
    reserved_quantity INTEGER NOT NULL DEFAULT 0
        CHECK (reserved_quantity >= 0 AND reserved_quantity <= quantity),
    price NUMERIC(12,2)
        CHECK (price IS NULL OR price >= 0),
    condition TEXT NOT NULL DEFAULT 'unknown'
        CHECK (condition IN ('new', 'used', 'refurbished', 'unknown')),
    availability_status TEXT NOT NULL DEFAULT 'available'
        CHECK (availability_status IN ('available', 'low_stock', 'out_of_stock', 'inactive')),
    last_quantity_changed_at TIMESTAMPTZ,
    last_updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    UNIQUE (shop_id, part_id)
);

CREATE TABLE spare_requests (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    requesting_shop_id UUID NOT NULL REFERENCES shops(id) ON DELETE RESTRICT,
    part_id UUID REFERENCES parts(id) ON DELETE RESTRICT,
    input_type TEXT NOT NULL
        CHECK (input_type IN ('text', 'image', 'voice')),
    raw_input TEXT,
    brand TEXT,
    model TEXT,
    part_name TEXT NOT NULL,
    quantity INTEGER NOT NULL
        CHECK (quantity > 0),
    urgency TEXT NOT NULL DEFAULT 'medium'
        CHECK (urgency IN ('low', 'medium', 'high')),
    request_location GEOGRAPHY(Point, 4326),
    address_snapshot TEXT,
    search_radius_km NUMERIC(6,2) NOT NULL DEFAULT 10
        CHECK (search_radius_km > 0 AND search_radius_km <= 100),
    notes TEXT,
    status TEXT NOT NULL DEFAULT 'draft'
        CHECK (
            status IN (
                'draft',
                'analyzing',
                'confirmed',
                'searching',
                'matched',
                'request_sent',
                'accepted',
                'handover_pending',
                'completed',
                'declined',
                'cancelled',
                'expired',
                'no_match'
            )
        ),
    confirmed_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE ai_analyses (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    request_id UUID NOT NULL REFERENCES spare_requests(id) ON DELETE RESTRICT,
    attempt_number INTEGER NOT NULL CHECK (attempt_number > 0),
    ai_provider TEXT NOT NULL,
    ai_model TEXT NOT NULL,
    input_type TEXT NOT NULL,
    raw_ai_output JSONB,
    structured_output JSONB,
    confidence NUMERIC(4,3)
        CHECK (confidence IS NULL OR (confidence >= 0 AND confidence <= 1)),
    analysis_status TEXT NOT NULL
        CHECK (analysis_status IN ('completed', 'uncertain', 'failed')),
    error_code TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    UNIQUE (request_id, attempt_number)
);

CREATE TABLE request_matches (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    request_id UUID NOT NULL REFERENCES spare_requests(id) ON DELETE RESTRICT,
    inventory_id UUID NOT NULL REFERENCES inventory(id) ON DELETE RESTRICT,
    distance_m INTEGER NOT NULL CHECK (distance_m >= 0),
    match_type TEXT NOT NULL
        CHECK (match_type IN ('exact', 'possible')),
    rank_position INTEGER NOT NULL CHECK (rank_position > 0),
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    UNIQUE (request_id, inventory_id)
);

CREATE TABLE supplier_requests (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    request_id UUID NOT NULL REFERENCES spare_requests(id) ON DELETE RESTRICT,
    supplier_shop_id UUID NOT NULL REFERENCES shops(id) ON DELETE RESTRICT,
    inventory_id UUID NOT NULL REFERENCES inventory(id) ON DELETE RESTRICT,
    quantity_requested INTEGER NOT NULL CHECK (quantity_requested > 0),
    confirmed_quantity INTEGER
        CHECK (confirmed_quantity IS NULL OR confirmed_quantity > 0),
    confirmed_price NUMERIC(12,2)
        CHECK (confirmed_price IS NULL OR confirmed_price >= 0),
    handover_method TEXT,
    supplier_response TEXT,
    status TEXT NOT NULL DEFAULT 'pending'
        CHECK (status IN ('pending', 'accepted', 'declined', 'expired', 'cancelled')),
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    responded_at TIMESTAMPTZ,
    accepted_at TIMESTAMPTZ,
    cancelled_at TIMESTAMPTZ,
    UNIQUE (request_id, inventory_id)
);

CREATE TABLE transactions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    request_id UUID NOT NULL REFERENCES spare_requests(id) ON DELETE RESTRICT,
    supplier_request_id UUID NOT NULL UNIQUE REFERENCES supplier_requests(id) ON DELETE RESTRICT,
    from_shop_id UUID NOT NULL REFERENCES shops(id) ON DELETE RESTRICT,
    to_shop_id UUID NOT NULL REFERENCES shops(id) ON DELETE RESTRICT,
    inventory_id UUID NOT NULL REFERENCES inventory(id) ON DELETE RESTRICT,
    part_id UUID NOT NULL REFERENCES parts(id) ON DELETE RESTRICT,
    quantity INTEGER NOT NULL CHECK (quantity > 0),
    agreed_price NUMERIC(12,2)
        CHECK (agreed_price IS NULL OR agreed_price >= 0),
    handover_method TEXT NOT NULL,
    status TEXT NOT NULL DEFAULT 'pending'
        CHECK (status IN ('pending', 'handover_pending', 'completed', 'cancelled')),
    completed_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    CHECK (from_shop_id <> to_shop_id)
);

CREATE TABLE notifications (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
    shop_id UUID REFERENCES shops(id) ON DELETE RESTRICT,
    type TEXT NOT NULL
        CHECK (
            type IN (
                'match_found',
                'new_supplier_request',
                'request_accepted',
                'request_declined',
                'request_cancelled',
                'handover_confirmed',
                'transaction_completed',
                'request_expired'
            )
        ),
    title TEXT NOT NULL,
    message TEXT NOT NULL,
    reference_type TEXT,
    reference_id UUID,
    is_read BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE uploaded_files (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    owner_id UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
    shop_id UUID REFERENCES shops(id) ON DELETE RESTRICT,
    request_id UUID REFERENCES spare_requests(id) ON DELETE RESTRICT,
    inventory_id UUID REFERENCES inventory(id) ON DELETE RESTRICT,
    storage_path TEXT NOT NULL UNIQUE,
    file_type TEXT NOT NULL,
    mime_type TEXT NOT NULL,
    file_size INTEGER NOT NULL CHECK (file_size > 0),
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    CHECK (
        (shop_id IS NOT NULL)::INTEGER +
        (request_id IS NOT NULL)::INTEGER +
        (inventory_id IS NOT NULL)::INTEGER = 1
    )
);

CREATE UNIQUE INDEX one_owner_per_shop
    ON shop_members (shop_id)
    WHERE role = 'owner';

CREATE UNIQUE INDEX one_accepted_supplier_per_request
    ON supplier_requests (request_id)
    WHERE status = 'accepted';

CREATE INDEX shops_location_gist
    ON shops USING GIST (location);

CREATE INDEX inventory_shop_idx
    ON inventory (shop_id);

CREATE INDEX inventory_part_idx
    ON inventory (part_id);

CREATE INDEX inventory_part_status_idx
    ON inventory (part_id, availability_status, shop_id);

CREATE INDEX requests_shop_idx
    ON spare_requests (requesting_shop_id);

CREATE INDEX requests_status_created_idx
    ON spare_requests (status, created_at DESC);

CREATE INDEX request_matches_request_rank_idx
    ON request_matches (request_id, rank_position);

CREATE INDEX request_matches_inventory_idx
    ON request_matches (inventory_id);

CREATE INDEX supplier_requests_shop_status_idx
    ON supplier_requests (supplier_shop_id, status);

CREATE INDEX supplier_requests_request_idx
    ON supplier_requests (request_id);

CREATE INDEX notifications_user_read_created_idx
    ON notifications (user_id, is_read, created_at DESC);

CREATE INDEX transactions_from_shop_idx
    ON transactions (from_shop_id, created_at DESC);

CREATE INDEX transactions_to_shop_idx
    ON transactions (to_shop_id, created_at DESC);

CREATE INDEX transactions_supplier_request_idx
    ON transactions (supplier_request_id);

CREATE OR REPLACE FUNCTION set_updated_at()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER users_set_updated_at
BEFORE UPDATE ON users
FOR EACH ROW EXECUTE FUNCTION set_updated_at();

CREATE TRIGGER shops_set_updated_at
BEFORE UPDATE ON shops
FOR EACH ROW EXECUTE FUNCTION set_updated_at();

CREATE TRIGGER shop_members_set_updated_at
BEFORE UPDATE ON shop_members
FOR EACH ROW EXECUTE FUNCTION set_updated_at();

CREATE TRIGGER parts_set_updated_at
BEFORE UPDATE ON parts
FOR EACH ROW EXECUTE FUNCTION set_updated_at();

CREATE TRIGGER inventory_set_updated_at
BEFORE UPDATE ON inventory
FOR EACH ROW EXECUTE FUNCTION set_updated_at();

CREATE TRIGGER spare_requests_set_updated_at
BEFORE UPDATE ON spare_requests
FOR EACH ROW EXECUTE FUNCTION set_updated_at();

CREATE TRIGGER supplier_requests_set_updated_at
BEFORE UPDATE ON supplier_requests
FOR EACH ROW EXECUTE FUNCTION set_updated_at();

CREATE TRIGGER transactions_set_updated_at
BEFORE UPDATE ON transactions
FOR EACH ROW EXECUTE FUNCTION set_updated_at();
~~~

### Schema Note

The Auth foreign key assumes the database migration runs in the same Supabase project that owns auth.users.

Cross-table business rules such as accepted supplier state, reservation consistency, and transaction completion should be enforced in a database transaction in the FastAPI service and may also receive targeted triggers later.

---

# 31. Database Migration Strategy

Use migrations, not manual production edits.

## Recommended Flow

~~~text
Migration File
     ↓
Local / Development Database
     ↓
Test
     ↓
Review
     ↓
Deployment Migration
     ↓
Production Database
~~~

## Team Rules

- Every schema change becomes a migration.
- Migration files are committed to Git.
- Do not edit production tables manually.
- Seed/demo data is separate from structural migrations.
- Destructive changes require explicit review.

## Supabase-Oriented Structure

A practical repository can contain:

~~~text
backend/
  supabase/
    migrations/
    seed/
~~~

or keep migrations in the database/backend section chosen by the team.

---

# 32. Backup & Recovery

The MVP should use the managed database provider's available backup facilities.

In addition:

- Keep migrations in Git.
- Keep demo/seed data reproducible.
- Keep Storage paths and metadata recoverable.
- Periodically export critical development data when needed.

## Recovery Expectations

The hackathon architecture does not promise enterprise-grade availability.

The objective is to make recovery possible without rebuilding the data model from memory.

## Storage Recovery

Uploaded files should use a retention/backup policy appropriate to the deployment plan.

For the MVP, the most important safeguard is keeping the file metadata and application references consistent.

---

# 33. Database Performance

## Likely First Bottlenecks

### Nearby Search

Use PostGIS and the location GiST index.

### Inventory Search

Filter by:

- part
- availability
- shop

Use indexes matching those queries.

### Request Lookup

Index requesting_shop_id and status.

### Transaction History

Index from_shop_id/to_shop_id with created_at.

### Notifications

Index recipient + unread state + timestamp.

## Query Discipline

Avoid:

- Loading every shop for a nearby search
- Fetching all inventory before filtering
- N+1 queries in result lists
- Large unpaginated history responses

## Pagination

Paginate:

- Inventory
- Requests
- Transactions
- Notifications

For nearby results, return a bounded result set suitable for the UI.

---

# 34. Future Database Extensions

> **Future Scope only. These are not MVP tables.**

Potential future entities:

- ratings
- shop_verifications
- delivery_orders
- payments
- compatibility_rules
- demand_forecasts
- analytics_events
- business_locations

## Expansion Principle

Do not add these tables until the corresponding product feature is actually being built.

The MVP database should remain focused on the core loop.

---

# 35. Final Database Table

| Table | Purpose | Key Relationships | MVP? |
|---|---|---|---|
| users | Application user linked to Auth | auth.users, shop_members | Yes |
| shops | Repair business | shop_members, inventory, requests, transactions | Yes |
| shop_members | User-to-shop membership | users, shops | Yes |
| parts | Canonical spare-part identity | inventory, requests, transactions | Yes |
| inventory | Shop-specific stock | shops, parts, matches, supplier requests | Yes |
| spare_requests | Part requirement | shops, parts, AI analyses, matches, supplier requests | Yes |
| ai_analyses | AI attempts/output | spare_requests | Yes |
| request_matches | Potential matches | requests, inventory | Yes |
| supplier_requests | Actual requests sent to suppliers | requests, shops, inventory | Yes |
| transactions | Confirmed exchange history | supplier requests, shops, parts, inventory | Yes |
| notifications | In-app event messages | users, related references | Yes |
| uploaded_files | File metadata | users, shops, requests, inventory | Yes |

---

# 36. Final SpareLink Data Model

## SPARELINK MVP DATA MODEL

### Core Tables

~~~text
users
shops
shop_members
parts
inventory
spare_requests
ai_analyses
request_matches
supplier_requests
transactions
notifications
uploaded_files
~~~

### Relationships

~~~text
Users
  ↓
Shop Members
  ↓
Shops
  ├── Inventory ── Parts
  ├── Spare Requests
  │     ├── AI Analyses
  │     ├── Request Matches ── Inventory
  │     └── Supplier Requests ── Supplier Shop
  │                              ↓
  │                           Transactions
  ├── Notifications
  └── Uploaded Files
~~~

### Request Lifecycle

~~~text
DRAFT
  ↓
ANALYZING
  ↓
CONFIRMED
  ↓
SEARCHING
  ↓
MATCHED
  ↓
REQUEST_SENT
  ↓
ACCEPTED
  ↓
HANDOVER_PENDING
  ↓
COMPLETED
~~~

Failure states:

~~~text
DECLINED
CANCELLED
EXPIRED
NO_MATCH
~~~

### Security Model

- Supabase Auth owns authentication.
- users links application users to Auth identities.
- shop_members determines shop access.
- FastAPI performs authorization.
- RLS provides defense in depth for Supabase-accessed rows.
- Private files remain in protected object storage.
- Historical transactions are preserved.

### Location Model

- shops.location stores shop coordinates.
- spare_requests.request_location stores request-time location.
- PostGIS supports radius and distance queries.
- Search radius is configurable.
- Precise coordinates are not exposed unnecessarily.

### AI Data

AI output is stored separately in ai_analyses.

The database preserves:

- provider
- model
- raw output
- structured output
- confidence
- analysis status
- attempt number

AI output does not become trusted request data until the user confirms it.

### Inventory Model

Canonical part:

~~~text
parts
~~~

Shop-owned stock:

~~~text
inventory
~~~

This allows many shops to stock the same canonical part with different:

- quantities
- prices
- conditions
- availability

### Transaction Model

A transaction is created only from a supplier request that has been accepted.

The transaction preserves:

- supplying shop
- requesting shop
- part
- inventory source
- quantity
- agreed price when applicable
- handover method
- status
- completion timestamp

---

# 37. Database Review

## 1. Can One Part Exist in Many Shops?

**Yes.**

One parts record can be referenced by many inventory rows.

## 2. Can One Shop Have Many Parts?

**Yes.**

One shop can have many inventory records.

## 3. Can One Request Produce Multiple Potential Matches?

**Yes.**

request_matches is one-to-many from spare_requests.

## 4. Can Multiple Supplier Requests Be Sent?

**Yes.**

supplier_requests is one-to-many from spare_requests.

## 5. Can Only One Supplier Request Ultimately Become the Fulfilled Transaction?

**Yes.**

A partial unique index limits accepted supplier requests for a request, and each supplier request can create at most one transaction.

## 6. Can Inventory Become Stale?

**Yes.**

last_updated_at and last_quantity_changed_at support freshness evaluation.

## 7. Can AI Output Be Corrected by the User?

**Yes.**

AI attempts remain separate, while confirmed request fields are stored in spare_requests.

## 8. Can Transaction History Remain Preserved After Inventory Changes?

**Yes.**

Transactions hold their own relationship and are protected from destructive cascading deletes.

## 9. Can Nearby Shops Be Searched Efficiently?

**Yes.**

PostGIS plus a GiST location index supports radius/distance operations.

## 10. Can Two Users Work Under the Same Shop?

**Yes.**

shop_members provides multi-user shop membership.

## 11. Can Unauthorized Users Access Another Shop's Data?

**They should not be able to**, assuming the FastAPI authorization checks and RLS policies are implemented as designed.

## 12. Can the Schema Support Future Growth?

**Yes.**

Future features can add focused entities without redesigning the core request/inventory/transaction model.

---

# Important Implementation Rules

- Parts 1–4 remain the source of truth.
- Keep the database relational and understandable.
- Avoid unnecessary tables.
- Preserve request and transaction history.
- Never store application passwords when Supabase Auth is used.
- Store images in object storage, not large PostgreSQL fields.
- Validate AI output before it becomes confirmed request data.
- Separate canonical parts from inventory.
- Separate potential matches from supplier requests.
- Separate supplier requests from completed transactions.
- Avoid destructive cascades for business history.
- Use UTC timestamps.
- Keep demo records fictional and clearly labeled.
- Enforce important cross-table rules inside database transactions.
- Do not represent listed inventory as confirmed physical availability.
- Keep future payment, delivery, rating, verification, and analytics models out of the MVP.

---

> **Status: Part 5 Complete | Database Architecture & Data Model Defined**
