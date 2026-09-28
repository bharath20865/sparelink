# SpareLink | Technical Architecture

> **The missing part is just one message away.**

Part 4 defines the technical architecture for the SpareLink MVP. It uses Parts 1–3 as the product source of truth and favors a simple, secure, maintainable architecture that a small student hackathon team can actually build and demonstrate.

The architecture is intentionally a **modular monolith** rather than a collection of microservices. External services are isolated behind small adapters so they can be replaced or mocked during the hackathon.

---

# 1. Architecture Objective

The architecture must support the complete core flow:

\`\`\`text
Shopkeeper
    ↓
Web Application
    ↓
Authentication
    ↓
Spare-Part Request
    ↓
AI Processing
    ↓
Request Confirmation
    ↓
Matching Engine
    ↓
Nearby Inventory
    ↓
Supplier Request
    ↓
Acceptance
    ↓
Transaction
\`\`\`

## Layer Responsibilities

| Layer | Responsibility |
|---|---|
| Web Application | Mobile-first interface, forms, request flow, inventory, status display |
| Authentication | Sign-in, sign-out, session/token management, identity |
| API Backend | Business rules, validation, authorization, orchestration |
| AI Service | Text/image understanding and structured spare-part extraction |
| Matching Engine | Inventory matching, location filtering, ranking |
| PostgreSQL | Persistent users, shops, parts, inventory, requests, matches, transactions, notifications |
| Storage | Request images and optional shop/part images |
| Location | Capture shop coordinates and perform radius/distance queries |
| Notifications | Inform shops about important request lifecycle events |

## Architectural Principles

1. **Keep the request path short.**
2. **Validate every external input.**
3. **Treat AI output as untrusted input.**
4. **Keep supplier confirmation separate from listed availability.**
5. **Keep external services behind adapters.**
6. **Make the core demo work with controlled local data if a dependency fails.**
7. **Prefer simple modules over distributed infrastructure.**

---

# 2. Recommended Tech Stack

| Technology | Purpose | Why it fits SpareLink |
|---|---|---|
| **Next.js + React + TypeScript** | Frontend web application | Strong React-based framework, routing, server/client rendering options, and a practical foundation for a mobile-first web app. |
| **Tailwind CSS** | UI styling | Fast to implement consistently and supports the lightweight design system from Part 3. |
| **FastAPI + Python** | Backend REST API | Concise Python API development, automatic request validation, and useful API documentation. |
| **Supabase** | Managed Postgres, Auth, Storage | Combines the database, authentication, and object storage needed by the MVP. |
| **PostgreSQL** | Main database | Relational model fits shops, inventory, requests, matches, and transactions. |
| **PostGIS** | Geospatial matching | Supports radius filtering and distance calculation close to the database. |
| **OpenAI Responses API** | AI text/image understanding | Supports text and image inputs and structured output patterns suitable for normalized spare-part data. |
| **Browser Geolocation API** | Device location capture | Simple location capture without adding a dedicated location SDK. |
| **Mapbox GL JS** | Optional map visualization | Useful for displaying nearby shops; it is not required for the core matching query. |
| **Vercel** | Frontend deployment | Directly supports Next.js deployment. |
| **Render** | Backend deployment | Practical hosting option for a Python/FastAPI service. |

### Important Scope Choice

The MVP does **not** require a separate cache server, message broker, Kubernetes cluster, service mesh, or microservice fleet.

The database remains the source of truth for the MVP.

---

# 3. Architecture Style

## Choice: Modular Monolith

SpareLink should start as a **modular monolith**.

A single FastAPI backend contains clearly separated business modules:

\`\`\`text
FastAPI Application
│
├── Auth
├── Users / Shops
├── Parts
├── Inventory
├── Requests
├── AI
├── Matching
├── Transactions
├── Notifications
└── Location
\`\`\`

## Why Not Microservices?

Microservices would add:

- More deployments
- Network communication between services
- Distributed debugging
- More authentication boundaries
- More infrastructure
- More failure points

None of those are needed to prove the SpareLink MVP.

## Why Not a Completely Unstructured Monolith?

A single codebase can still become difficult to maintain if every feature shares the same files.

The modular monolith avoids that by keeping business responsibilities separate while preserving one deployable backend.

## Future Extraction Path

If a particular module eventually becomes large, it can later be extracted into a dedicated service.

Potential future candidates:

- AI processing
- Matching/search
- Notifications

That is a future scaling decision, not an MVP requirement.

---

# 4. High-Level System Architecture

\`\`\`mermaid
flowchart TD
    U[Shopkeeper / Repair Shop] --> FE[Next.js Web Application]

    FE --> AUTH[Supabase Auth]
    FE --> API[FastAPI API]

    API --> AUTHV[Auth Token Validation]
    API --> AI[AI Adapter]
    API --> MATCH[Matching Engine]
    API --> DB[(Supabase PostgreSQL + PostGIS)]
    API --> STORAGE[Supabase Storage]
    API --> LOC[Location Services]
    API --> NOTIFY[Notification Module]

    AI --> OPENAI[OpenAI Multimodal Model]
    LOC --> GEO[Browser Geolocation / Mapbox]
    NOTIFY --> INAPP[In-App Notifications]

    DB --> MATCH
    STORAGE --> AI
\`\`\`

### Request Path

\`\`\`text
Browser
  ↓
Next.js
  ↓
FastAPI
  ↓
Validate user
  ↓
AI / Matching modules
  ↓
PostgreSQL
  ↓
Response
  ↓
Next.js UI
\`\`\`

---

# 5. Frontend Architecture

## Application Framework

**Next.js + React + TypeScript**

Next.js provides routing and application infrastructure around React. The App Router should be used for the new SpareLink frontend.

## Routing

Recommended routes:

\`\`\`text
/
├── /login
├── /register
├── /onboarding
├── /dashboard
├── /find-part
├── /requests
├── /requests/[id]
├── /inventory
├── /inventory/new
├── /inventory/[id]
├── /transactions
└── /profile
\`\`\`

Notifications can be a global UI surface rather than a mandatory dedicated route on mobile.

## Pages

Pages should map to the MVP screen inventory from Part 3.

## Reusable Components

Components should be domain-aware where useful:

\`\`\`text
components/
├── layout/
├── forms/
├── ui/
├── parts/
├── shops/
├── requests/
├── inventory/
├── transactions/
└── notifications/
\`\`\`

## State Management

Do not introduce a heavy global state library initially.

Use:

- React state for local form/UI state
- URL state for filters/search parameters
- Server/API data hooks for remote state
- Auth session provider for authentication state

A dedicated global state library can be added later only if real application complexity requires it.

## API Communication

Create one typed API layer:

\`\`\`text
Frontend UI
   ↓
Feature service
   ↓
Typed API client
   ↓
FastAPI
\`\`\`

Do not scatter raw fetch calls throughout UI components.

## Authentication State

The frontend receives the authenticated Supabase session/token and uses it for authenticated backend requests.

The frontend should never contain:

- Service-role keys
- Database passwords
- OpenAI API keys
- Private Mapbox/server secrets

## Form Handling

Forms should:

- Validate required inputs
- Preserve user input on failure
- Display field-level errors
- Prevent duplicate submissions
- Show loading state on submit

## Loading States

Use meaningful states such as:

- Loading dashboard
- Analyzing request
- Searching nearby shops
- Sending request

Do not use fake percentage progress.

## Error States

Errors should:

- Explain what happened
- Preserve useful input
- Provide a recovery action

## Mobile Responsiveness

Mobile is the primary layout.

Desktop should expand the same information architecture rather than introduce a completely different workflow.

---

## Recommended Frontend Folder Structure

\`\`\`text
frontend/
├── app/
│   ├── (auth)/
│   │   ├── login/
│   │   └── register/
│   ├── onboarding/
│   ├── dashboard/
│   ├── find-part/
│   ├── requests/
│   ├── inventory/
│   ├── transactions/
│   └── profile/
│
├── components/
│   ├── layout/
│   ├── ui/
│   ├── forms/
│   ├── ai/
│   ├── shops/
│   ├── parts/
│   ├── requests/
│   ├── inventory/
│   └── transactions/
│
├── features/
│   ├── auth/
│   ├── find-part/
│   ├── inventory/
│   ├── requests/
│   └── transactions/
│
├── hooks/
├── lib/
│   ├── supabase/
│   ├── api/
│   └── validation/
├── services/
├── types/
└── public/
\`\`\`

### Folder Responsibilities

| Folder | Responsibility |
|---|---|
| app | Routes and page composition |
| components | Reusable UI |
| features | Feature-specific UI and orchestration |
| hooks | Reusable React hooks |
| lib | Clients, validation, and infrastructure helpers |
| services | API-facing application services |
| types | Shared TypeScript types |
| public | Static assets |

---

# 6. Backend Architecture

FastAPI is the backend framework.

The backend owns business rules rather than allowing the browser to directly decide critical transaction states.

## Core Modules

\`\`\`text
Auth
Users
Shops
Parts
Inventory
Requests
AI
Matching
Transactions
Notifications
Location
\`\`\`

### Auth

Validates the authenticated identity and provides authorization context.

### Users

Stores application-level user/profile information linked to the authentication identity.

### Shops

Creates and updates shop profiles and categories.

### Parts

Normalizes reusable part/model information where appropriate.

### Inventory

Creates, updates, removes, and queries shop inventory.

### Requests

Owns the spare-part request lifecycle.

### AI

Sends approved inputs to the AI provider, validates structured output, and returns an interpretation.

### Matching

Searches inventory, applies compatibility/location/availability rules, and ranks results.

### Transactions

Owns accepted exchanges, handover state, and completion records.

### Notifications

Creates and retrieves in-app notifications.

### Location

Stores coordinates and calculates distance/radius relationships.

---

## Recommended Backend Folder Structure

\`\`\`text
backend/
├── app/
│   ├── main.py
│   ├── api/
│   │   ├── deps.py
│   │   └── routes/
│   │       ├── shops.py
│   │       ├── inventory.py
│   │       ├── requests.py
│   │       ├── ai.py
│   │       ├── matching.py
│   │       ├── transactions.py
│   │       └── notifications.py
│   │
│   ├── models/
│   ├── schemas/
│   ├── services/
│   │   ├── ai/
│   │   ├── matching/
│   │   ├── location/
│   │   └── notifications/
│   │
│   ├── repositories/
│   ├── core/
│   │   ├── config.py
│   │   ├── security.py
│   │   └── logging.py
│   └── utils/
│
├── tests/
├── requirements.txt
└── .env.example
\`\`\`

---

# 7. API Architecture

Supabase Auth owns user authentication. FastAPI owns application-specific business operations and authorization checks.

## Authentication

No custom password-login API is required if Supabase Auth is used.

The frontend obtains the user session from Supabase Auth and sends the access token to FastAPI.

FastAPI verifies the token and derives the authenticated application user.

## Core REST API

| Method | Endpoint | Purpose | Authentication |
|---|---|---|---|
| GET | /health | Service health check | Public |
| GET | /me | Current application user/profile | Required |
| GET | /shops/me | Current shop profile | Required |
| PATCH | /shops/me | Update current shop | Required |
| POST | /shops/me/location | Save/update shop location | Required |
| GET | /inventory | List current shop inventory | Required |
| POST | /inventory | Add inventory item | Required |
| PATCH | /inventory/{id} | Edit own inventory | Required |
| DELETE | /inventory/{id} | Remove own inventory | Required |
| POST | /requests | Create spare-part request | Required |
| POST | /requests/{id}/analyze | Analyze request input | Required |
| POST | /requests/{id}/confirm | Confirm AI interpretation | Required |
| POST | /requests/{id}/search | Search potential matches | Required |
| GET | /requests/{id} | View request and status | Required |
| POST | /requests/{id}/cancel | Cancel own request | Required |
| POST | /requests/{id}/supplier-request | Send request to selected shop | Required |
| GET | /requests/incoming | List incoming supplier requests | Required |
| POST | /supplier-requests/{id}/accept | Accept supplier request | Required |
| POST | /supplier-requests/{id}/decline | Decline supplier request | Required |
| POST | /supplier-requests/{id}/handover | Record handover state | Required |
| POST | /transactions/{id}/complete | Complete transaction | Required |
| GET | /transactions | List transactions visible to current shop | Required |
| GET | /notifications | List current notifications | Required |
| POST | /notifications/{id}/read | Mark notification read | Required |

Endpoints should be added only when the feature requires a separate operation.

---

# 8. API Request / Response Contracts

The examples below are conceptual contracts for implementation.

## Create Spare Request

### Request

\`\`\`json
{
  "input_type": "text",
  "raw_input": "Samsung M14 display kavali urgent ga.",
  "quantity": 1,
  "notes": null
}
\`\`\`

### Required

- input_type
- raw_input or uploaded input reference
- quantity

### Optional

- notes
- urgency if explicitly entered

### Response

\`\`\`json
{
  "id": "req_123",
  "status": "DRAFT",
  "raw_input": "Samsung M14 display kavali urgent ga.",
  "quantity": 1
}
\`\`\`

---

## AI Analyze Request

### Request

\`\`\`json
{
  "request_id": "req_123"
}
\`\`\`

The backend retrieves the request input and sends the appropriate content to the AI adapter.

### Response

\`\`\`json
{
  "request_id": "req_123",
  "status": "ANALYZING",
  "interpretation": {
    "brand": "Samsung",
    "model": "Galaxy M14",
    "part_name": "Display",
    "category": "Mobile Display",
    "quantity": 1,
    "urgency": "high",
    "confidence": 0.94,
    "notes": null
  }
}
\`\`\`

The AI output remains provisional until user confirmation.

---

## Search Matches

### Request

\`\`\`json
{
  "request_id": "req_123",
  "radius_km": 10
}
\`\`\`

### Response

\`\`\`json
{
  "request_id": "req_123",
  "results": [
    {
      "shop_id": "shop_42",
      "shop_name": "Sri Sai Mobiles",
      "distance_km": 2.4,
      "part_name": "Samsung M14 Display",
      "match_type": "EXACT",
      "listed_quantity": 1,
      "availability": "LISTED",
      "price": 1850
    }
  ]
}
\`\`\`

The result represents a potential match until the supplier confirms it.

---

## Send Supplier Request

### Request

\`\`\`json
{
  "shop_id": "shop_42",
  "inventory_id": "inv_987",
  "quantity": 1,
  "note": "Needed urgently for customer repair."
}
\`\`\`

### Response

\`\`\`json
{
  "supplier_request_id": "sreq_88",
  "status": "REQUEST_SENT"
}
\`\`\`

---

## Accept Supplier Request

### Request

\`\`\`json
{
  "confirmed_quantity": 1,
  "confirmed_price": 1850,
  "handover_method": "PICKUP"
}
\`\`\`

### Response

\`\`\`json
{
  "supplier_request_id": "sreq_88",
  "status": "ACCEPTED",
  "confirmed_quantity": 1,
  "confirmed_price": 1850,
  "handover_method": "PICKUP"
}
\`\`\`

---

## Complete Transaction

### Request

\`\`\`json
{
  "received_quantity": 1,
  "confirmation": true
}
\`\`\`

### Response

\`\`\`json
{
  "transaction_id": "txn_1001",
  "status": "COMPLETED"
}
\`\`\`

---

## Standard Error Response

\`\`\`json
{
  "error": {
    "code": "PART_NOT_FOUND",
    "message": "No matching part was found nearby.",
    "action": "Expand the search radius or edit the request."
  }
}
\`\`\`

### Common Error Codes

| Code | Meaning |
|---|---|
| VALIDATION_ERROR | Request data is invalid |
| UNAUTHORIZED | Authentication missing/invalid |
| FORBIDDEN | User cannot access the resource |
| NOT_FOUND | Requested record does not exist |
| AI_UNAVAILABLE | AI provider unavailable |
| AI_UNCERTAIN | AI could not confidently interpret the requirement |
| PART_NOT_FOUND | No matching inventory found |
| INVENTORY_OUTDATED | Inventory could not be confirmed |
| REQUEST_ALREADY_HANDLED | Supplier request already resolved |
| LOCATION_UNAVAILABLE | Required location information is missing |
| RATE_LIMITED | Too many requests |
| INTERNAL_ERROR | Unexpected server error |

Never expose API keys, database credentials, stack traces, or private provider errors to the browser.

---

# 9. AI Architecture

## Text Request

\`\`\`text
Natural Language
      ↓
AI Model
      ↓
Structured JSON
      ↓
Schema Validation
      ↓
Confidence Assessment
      ↓
User Confirmation
      ↓
Database Search
\`\`\`

A server-side AI adapter should call the multimodal model. The browser must never contain the provider secret.

## Image Request

\`\`\`text
Image
  ↓
Upload to Storage
  ↓
Generate temporary access/reference
  ↓
Vision-capable AI
  ↓
Possible Part Identification
  ↓
Structured Output
  ↓
Validation
  ↓
User Confirmation
  ↓
Search
\`\`\`

## Voice Request

For the MVP, voice should be treated as an input adapter rather than a separate AI pipeline.

Preferred flow:

\`\`\`text
Voice
  ↓
Speech-to-Text
  ↓
Normalized Text
  ↓
Text Understanding Pipeline
  ↓
Structured Request
\`\`\`

To keep the MVP simple, browser-supported speech recognition may be used where available, with text input as the fallback. A dedicated transcription provider can be added later without changing the core request model.

---

# 10. AI Output Schema

Recommended schema:

\`\`\`json
{
  "brand": "Samsung",
  "model": "Galaxy M14",
  "part_name": "Display",
  "category": "Mobile Display",
  "quantity": 1,
  "urgency": "high",
  "confidence": 0.94,
  "notes": null
}
\`\`\`

## Field Rules

| Field | Required | Null Allowed | Validation |
|---|---|---|---|
| brand | No | Yes | Trimmed string |
| model | No | Yes | Trimmed string |
| part_name | Yes for searchable request | No | Non-empty string |
| category | No | Yes | Known category or null |
| quantity | Yes | No | Integer > 0 |
| urgency | Yes | No | low / medium / high |
| confidence | Yes | No | Number between 0 and 1 |
| notes | No | Yes | Length-limited string |

### Urgency Values

\`\`\`text
low
medium
high
\`\`\`

### Confidence Interpretation

Confidence is a model-provided signal, not ground truth.

Recommended product behavior:

- **High confidence:** Show interpretation and request normal user confirmation.
- **Moderate confidence:** Show interpretation with a stronger correction prompt.
- **Low confidence:** Require additional user clarification before search.
- **Missing critical field:** Do not create a searchable confirmed request.
- **AI failure:** Preserve input and offer retry/manual entry.

The exact numeric thresholds should be configurable and validated through prototype testing rather than treated as universal truth.

---

# 11. AI Safety / Reliability

AI output must follow:

\`\`\`text
AI Result
   ↓
Schema Validation
   ↓
Confidence Assessment
   ↓
Business Validation
   ↓
User Confirmation
   ↓
Database Search
\`\`\`

## Malformed Output

If the AI returns invalid JSON or an invalid field:

1. Reject the response.
2. Log a safe error event.
3. Ask the AI adapter to retry only when appropriate.
4. Otherwise return an AI error state to the user.

## Hallucinated Part / Model

The AI must not invent a part catalog ID simply because it can produce a plausible string.

The backend should:

- Treat AI names as untrusted text.
- Match against known part/model data where available.
- Preserve unknown text for manual correction.
- Require user confirmation.
- Avoid claiming compatibility without enough evidence.

## High Confidence

Still show the interpretation for confirmation.

## Moderate Confidence

Highlight that the interpretation may need correction.

## Low Confidence

Ask for more information and do not silently search.

---

# 12. Matching Engine Architecture

The matching engine is a separate logical module inside the backend.

## Input

\`\`\`text
Required Part
Requester Location
Quantity
Urgency
\`\`\`

## Output

\`\`\`text
Matching Shops
Distance
Availability
Match Quality
\`\`\`

## Matching Pipeline

\`\`\`text
Request
   ↓
Normalize Part Information
   ↓
Search Inventory
   ↓
Check Availability
   ↓
Calculate Distance
   ↓
Filter
   ↓
Rank
   ↓
Return Results
\`\`\`

## Exact Match

An exact match should require enough normalized information to support the claim, such as matching manufacturer/model/part information.

Example:

\`\`\`text
Request: Samsung Galaxy M14 Display

Inventory:
Samsung Galaxy M14 Display

Result:
EXACT
\`\`\`

## Compatible / Possible Match

Use this only when the system has enough evidence to consider the inventory potentially usable, but not enough to claim exact identity.

Example:

\`\`\`text
Request: Samsung Galaxy M14 Display

Inventory:
Samsung Galaxy M14 LCD Assembly

Result:
POSSIBLE / COMPATIBLE
\`\`\`

Compatibility should not be asserted merely because two strings look similar.

## No Match

Return:

\`\`\`text
NO_MATCH
\`\`\`

when no suitable participating inventory remains after filtering.

---

# 13. Location Architecture

## Stored Location Data

Each shop should have:

- latitude
- longitude
- address
- search radius preference

The database should store the shop coordinate in a geospatial type supported by PostGIS.

## Location Capture

Primary method:

**Browser Geolocation API**

Fallback:

**Manual address/area selection**

The shop should explicitly provide or confirm the location.

## Radius Search

A configurable initial radius such as **10 km** can be stored as a preference.

Conceptually:

\`\`\`text
Requester Location
      ↓
Configured Radius
      ↓
PostGIS Radius Query
      ↓
Candidate Shops
\`\`\`

For the actual database query, a geography column can be used with \`ST_DWithin\` for radius filtering and \`ST_Distance\` for distance output.

## Privacy

Do not expose precise coordinates publicly unless needed.

The UI should primarily show:

- Approximate area
- Distance
- Map position only when useful

The backend can use precise coordinates for matching while the frontend receives only the data needed for the transaction.

## Manual Location Fallback

If browser geolocation is denied:

1. Ask the shop to enter an area/address.
2. Resolve to coordinates only when needed.
3. Store the resulting shop location.
4. Allow the shop to correct it.

---

# 14. Database Architecture

Part 5 defines the full schema. Part 4 only defines the architectural entities and ownership.

## Major Entities

\`\`\`text
Users
Shops
Parts
Inventory
Requests
Matches
Supplier Requests
Transactions
Notifications
\`\`\`

### Interaction

\`\`\`text
User
  ↓
Shop
  ↓
Inventory

User
  ↓
Request
  ↓
Match
  ↓
Supplier Request
  ↓
Transaction

Request / Transaction
  ↓
Notifications
\`\`\`

The backend should access the database through repositories/services rather than embedding SQL in route handlers.

PostgreSQL remains the system of record.

---

# 15. Storage Architecture

Uploaded images should live in object storage rather than in large database fields.

## Flow

\`\`\`text
User
  ↓
Upload Image
  ↓
Supabase Storage
  ↓
Secure File Reference
  ↓
AI Processing
\`\`\`

## Suggested Buckets

Keep buckets separated by security purpose:

\`\`\`text
request-images
inventory-images
shop-images
\`\`\`

The exact bucket names can be changed, but the security boundary should remain clear.

## File Restrictions

MVP should accept only required image types, for example:

- JPEG
- PNG
- WebP

Avoid accepting arbitrary executable/document formats when they are not needed.

### Recommended Maximum Size

Use a configurable maximum such as **5 MB per image** for the MVP. Enforce the limit in both the UI and backend.

### Access Control

Uploaded files should default to private access.

Use short-lived access/reference mechanisms for AI processing where necessary.

Do not make all user-uploaded files public merely to simplify the first implementation.

### Retention

The MVP should define a simple retention rule:

- Keep images needed for active/completed requests.
- Remove abandoned temporary uploads when practical.
- Avoid retaining unnecessary copies.

### Naming

Use generated identifiers rather than original user filenames:

\`\`\`text
{entity}/{shop_id}/{uuid}.{extension}
\`\`\`

Example:

\`\`\`text
request-images/shop_42/550e8400-e29b-41d4-a716-446655440000.jpg
\`\`\`

Do not trust original filenames as storage paths.

---

# 16. Authentication & Authorization

Supabase Auth handles identity.

FastAPI handles application authorization.

## Roles

The MVP can keep roles simple:

### Shop Owner / Account

Owns the shop profile and can manage shop-level data.

### Requesting Shop

A contextual role for the shop acting as requester in a request.

### Supplying Shop

A contextual role for the shop acting as supplier.

The same authenticated shop can play both requesting and supplying roles at different times.

## Permissions

| Action | Current Shop |
|---|---|
| Create inventory | Yes |
| Edit own inventory | Yes |
| Remove own inventory | Yes |
| Create requests | Yes |
| View own requests | Yes |
| Accept supplier request sent to own shop | Yes |
| View own transactions | Yes |
| Modify own shop details | Yes |
| Modify another shop's inventory | No |
| View another shop's private data | No |
| Modify another shop's transactions | No |

## Authorization Rule

For every resource mutation:

\`\`\`text
Authenticated User
      ↓
Resolve Shop
      ↓
Check Resource Ownership / Role
      ↓
Allow or Reject
\`\`\`

Do not trust a shop ID supplied by the client without checking ownership.

---

# 17. Security Architecture

## Essential MVP Controls

### Authentication

Use Supabase Auth rather than implementing password storage manually.

### Authorization

Check shop ownership and action permissions on every protected mutation.

### Input Validation

Validate:

- String lengths
- IDs
- Quantities
- Enum values
- Coordinates
- Request states
- File metadata

### SQL Injection Prevention

Use parameterized queries/ORM mechanisms. Never construct raw SQL by concatenating user input.

### Rate Limiting

At minimum rate-limit:

- AI analysis
- Search
- Login-related operations handled by the chosen auth provider
- Request creation

The exact limits should be configured from prototype usage rather than invented as business guarantees.

### File Upload Validation

Validate:

- MIME type
- Extension
- Size
- Image parsing where appropriate

Do not trust client-supplied MIME type alone.

### Secret Management

Keep secrets in deployment environment variables.

Never commit:

- OpenAI API keys
- Database passwords
- Supabase service keys
- Private Mapbox tokens

### HTTPS

Production traffic must use HTTPS.

### CORS

Allow only the deployed frontend origin(s) required by the MVP.

### Error Handling

Return safe application errors. Keep provider details and stack traces in server-side logs.

### Logging

Use structured logs without passwords, access tokens, or unnecessary personal data.

---

# 18. Data Flow

## Main Request Flow

\`\`\`mermaid
sequenceDiagram
    participant A as Shop A
    participant F as Next.js Frontend
    participant B as FastAPI Backend
    participant AI as AI Adapter
    participant D as PostgreSQL
    participant M as Matching Engine

    A->>F: Enter spare-part request
    F->>B: Create request
    B->>D: Store draft request
    D-->>B: Request ID

    F->>B: Analyze request
    B->>AI: Send text/image
    AI-->>B: Structured interpretation
    B->>D: Store provisional interpretation
    B-->>F: Show AI result

    A->>F: Confirm interpretation
    F->>B: Confirm request
    B->>D: Store confirmed request

    F->>B: Search nearby
    B->>M: Match requirement
    M->>D: Search inventory + location
    D-->>M: Candidate inventory
    M-->>B: Ranked matches
    B-->>F: Matching results
    F-->>A: Show nearby shops
\`\`\`

## Supplier Flow

\`\`\`mermaid
sequenceDiagram
    participant A as Shop A
    participant B as FastAPI Backend
    participant S as Shop B

    A->>B: Send supplier request
    B->>S: New request notification
    S->>B: Review inventory
    S->>B: Accept or decline
    B->>A: Update request status
    A->>B: Confirm receipt
    B->>A: Transaction completed
    B->>S: Transaction completed
\`\`\`

---

# 19. Core Transaction Flow

The backend lifecycle is:

\`\`\`text
REQUEST CREATED
      ↓
AI PROCESSED
      ↓
USER CONFIRMED
      ↓
MATCHES FOUND
      ↓
SUPPLIER REQUEST SENT
      ↓
SUPPLIER ACCEPTED
      ↓
HANDOVER PENDING
      ↓
COMPLETED
\`\`\`

## Transition Rules

| Transition | Backend/System Action |
|---|---|
| Create | Insert request with DRAFT state |
| AI Processed | Store provisional interpretation; state ANALYZING until processing completes |
| User Confirmed | Persist confirmed fields; state CONFIRMED |
| Search | Query inventory; store match candidates; state SEARCHING/MATCHED |
| Supplier Request Sent | Create supplier-request record; state REQUEST_SENT |
| Supplier Accepted | Verify supplier ownership, confirm quantity/availability, update state ACCEPTED |
| Handover Pending | Store agreed handover state |
| Completed | Require authorized completion action; create/finalize transaction record |

## Failure Transitions

\`\`\`text
Supplier Declines → DECLINED
User Cancels → CANCELLED
Response Window Ends → EXPIRED
Search Returns Nothing → NO_MATCH
\`\`\`

State changes should be validated by the backend so a client cannot arbitrarily jump from DRAFT to COMPLETED.

---

# 20. Notification Architecture

## MVP Choice: In-App Notifications

The simplest reliable MVP notification method is an in-app notification table plus dashboard/request indicators.

No push-notification infrastructure is required to prove the core workflow.

## Event Model

\`\`\`text
Event
 ↓
Notification Service
 ↓
Notification Record
 ↓
Recipient
 ↓
Unread Indicator
\`\`\`

## Required Events

| Event | Recipient |
|---|---|
| Match found | Requesting shop |
| Supplier request sent | Supplying shop |
| Supplier accepted | Requesting shop |
| Supplier declined | Requesting shop |
| Request cancelled | Supplying shop |
| Handover state updated | Relevant shop |
| Transaction completed | Both shops |

Email/push/WhatsApp can be added later behind the same notification interface.

---

# 21. Real-Time Requirements

The MVP does **not** require a custom WebSocket infrastructure.

## Requirement Analysis

| Feature | Real-Time Required? | MVP Approach |
|---|---|---|
| Supplier request | Helpful, not mandatory | In-app notification + refresh |
| Acceptance | Helpful | Status refresh |
| Inventory changes | No | Read latest inventory on search |
| Notifications | No | Store in DB and refresh |
| Request status | No | Poll/refresh request details |

## Optional Supabase Realtime

Supabase Realtime can be considered later if the demo needs a more immediate two-screen experience.

The first implementation should work without it.

This keeps the backend easier to debug and avoids unnecessary infrastructure.

---

# 22. Error Handling Architecture

The backend should return consistent errors:

\`\`\`json
{
  "error": {
    "code": "PART_NOT_FOUND",
    "message": "No matching part was found nearby.",
    "action": "Expand the search radius or edit the request."
  }
}
\`\`\`

## Categories

### Validation

User-provided data is invalid.

### Authentication

User is not signed in or the token is invalid.

### Authorization

User does not own or have access to the requested resource.

### Business Rule

The requested state transition is not allowed.

### External Service

AI, maps, storage, or another provider failed.

### Not Found

Requested resource does not exist.

### Conflict

Another operation changed the record before the current action.

### Internal Error

Unexpected backend failure.

The UI maps these codes to friendly messages and next actions.

---

# 23. Observability

MVP logging should be enough to answer:

> What happened to this request?

## Events

- Request created
- AI analysis started
- AI analysis completed
- AI analysis failed
- Request confirmed
- Match search performed
- Match returned
- Supplier request sent
- Supplier accepted
- Supplier declined
- Handover updated
- Transaction completed
- API failure

## Safe Logging

Do not log:

- Passwords
- Access tokens
- API keys
- Full private image contents
- Unnecessary personal data

## Recommended Request Identifier

Use a request ID in logs and API responses:

\`\`\`text
req_123
\`\`\`

This makes debugging much easier during the hackathon.

---

# 24. Performance Considerations

The MVP does not need artificial numerical performance promises.

Focus on obvious bottlenecks.

## Dashboard

- Load summary data efficiently.
- Avoid fetching full inventory on every dashboard visit.

## Inventory Search

- Index searchable part/model fields.
- Keep queries scoped to available inventory.

## Nearby Lookup

Use PostGIS geospatial indexing for radius queries.

Conceptually:

\`\`\`text
Requester Coordinate
      ↓
ST_DWithin
      ↓
Candidate Shops
      ↓
ST_Distance
      ↓
Rank
\`\`\`

## Images

Compress images before upload where practical.

Reject unnecessarily large files.

## AI Latency

Show a proper loading state.

Do not block unrelated parts of the application while AI is processing.

## Pagination

Paginate:

- Inventory
- Requests
- Transactions
- Notifications

A search result can use a small result limit for the MVP.

---

# 25. Scalability Path

## 10 Shops

Current modular monolith is sufficient.

## 100 Shops

Still use the same architecture.

Focus on:

- Better indexes
- Clean geospatial queries
- Pagination
- Avoiding N+1 queries

## 1,000 Shops

Consider:

- Search indexes
- Query optimization
- Caching frequently requested reference data
- Background processing for slow AI operations

## 10,000+ Shops

Potential future changes:

- Dedicated search service
- More advanced geospatial indexing
- Queue/background workers
- AI job processing
- Read caching
- Dedicated notification infrastructure

The key principle is:

> **Scale the bottleneck only when real usage demonstrates that the bottleneck exists.**

---

# 26. External Services & Dependencies

| Service | Purpose | MVP Required? | Failure Impact | Fallback |
|---|---|---:|---|---|
| Supabase Postgres | Persistent data | Yes | Core app unavailable | Controlled demo dataset/local fallback only |
| Supabase Auth | Authentication | Yes | Login unavailable | Demo account/session for controlled presentation |
| Supabase Storage | Image storage | Only for image flow | Image requests fail | Text input |
| OpenAI API | AI understanding | For AI demo | AI analysis unavailable | Mock structured response for demo |
| Browser Geolocation | Device location | Yes for automatic location | Local matching needs manual location | Manual location |
| Mapbox GL JS | Map visualization | No | Map unavailable | Result list with distances |
| Vercel | Frontend hosting | Deployment | Frontend unavailable | Local dev build for rehearsal |
| Render | Backend hosting | Deployment | API unavailable | Local backend/demo environment |
| In-app Notifications | Request updates | Yes | Reduced UX | Request status page |

## Dependency Rule

No external service should be deeply embedded inside domain logic.

Use adapters:

\`\`\`text
Domain Logic
   ↓
Interface / Adapter
   ↓
External Provider
\`\`\`

This makes failures easier to mock.

---

# 27. Development Environment

## Local Development

Recommended:

\`\`\`text
Frontend:
Next.js dev server

Backend:
FastAPI + Uvicorn

Database:
Supabase development project
or local Postgres for advanced local testing
\`\`\`

A shared Supabase development project is practical for a small hackathon team, provided test data is clearly separated from real data.

## Environment Variables

Example:

\`\`\`text
# Frontend
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=
NEXT_PUBLIC_API_BASE_URL=
NEXT_PUBLIC_MAPBOX_TOKEN=

# Backend
SUPABASE_URL=
SUPABASE_SERVICE_ROLE_KEY=
DATABASE_URL=
OPENAI_API_KEY=
OPENAI_MODEL=
CORS_ORIGINS=
\`\`\`

### Public Variables

These may be exposed to browser code when designed as public client configuration:

\`\`\`text
NEXT_PUBLIC_SUPABASE_URL
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY
NEXT_PUBLIC_API_BASE_URL
NEXT_PUBLIC_MAPBOX_TOKEN
\`\`\`

Only expose a public/client-safe Mapbox token configured with appropriate restrictions.

### Secret Variables

These must remain server-only:

\`\`\`text
SUPABASE_SERVICE_ROLE_KEY
DATABASE_URL
OPENAI_API_KEY
\`\`\`

Never commit \`.env\` files containing real credentials.

## Test Data

The repository should include scripts or fixtures for controlled demo data:

- Shop A
- Shop B
- Sample inventory
- Sample Samsung M14 display
- Sample request

Demo data must be clearly labeled as test/demo data.

---

# 28. Git Repository Architecture

Recommended repository:

\`\`\`text
sparelink/
│
├── README.md
│
├── docs/
│   ├── 01-idea-foundation.md
│   ├── 02-user-journey.md
│   ├── 03-ui-ux.md
│   ├── 04-technical-architecture.md
│   ├── 05-database.md
│   ├── 06-backend.md
│   ├── 07-ai-engine.md
│   ├── 08-matching-engine.md
│   ├── 09-testing.md
│   └── 10-hackathon-presentation.md
│
├── frontend/
│
├── backend/
│
├── tests/
│
├── scripts/
│
├── .env.example
├── .gitignore
└── LICENSE
\`\`\`

## Folder Purpose

| Folder | Purpose |
|---|---|
| docs | Product and engineering specifications |
| frontend | Next.js application |
| backend | FastAPI application |
| tests | Cross-layer/integration tests |
| scripts | Seed/demo/setup utilities |

The structure can be expanded in later parts without changing the architecture principle.

---

# 29. Development Order

Recommended implementation order:

\`\`\`text
1. Repository + project setup
2. Environment/configuration
3. Supabase project and database foundation
4. Authentication integration
5. Shop profile
6. Location
7. Inventory CRUD
8. Spare request CRUD
9. AI adapter + structured validation
10. Request confirmation
11. Matching engine
12. Supplier request flow
13. Transactions
14. In-app notifications
15. Frontend integration/refinement
16. Testing
17. Demo fallback
18. Deployment
\`\`\`

## Why This Order Works

Authentication and database identity come before shop-owned data.

Shop profile and inventory must exist before meaningful matching can happen.

The spare request exists before AI interpretation.

The confirmed request exists before matching.

Matching exists before supplier requests.

Supplier acceptance exists before a transaction.

This order follows the actual dependency graph rather than building screens in isolation.

---

# 30. Hackathon Fallback Strategy

The final demo should not depend on one external provider working perfectly.

## AI Unavailable

Use a predefined mock response through the same AI adapter interface:

\`\`\`text
MockAIProvider
     ↓
Structured AI Output
\`\`\`

The presenter should not describe this as a live AI call.

## Maps Unavailable

Show:

- Calculated distance
- Shop list
- Static map placeholder if necessary

The map is not the source of truth for matching.

## Notifications Unavailable

Use the in-app notification list and request status page.

## Network/API Issue

Keep a local or pre-seeded demo dataset and a rehearsed local environment.

## Storage Failure

Use text input instead of image input.

## Demo Principle

> **A mock should preserve the same interface and workflow as the real dependency, while clearly remaining labeled as mock functionality.**

Never present mock output as a live production integration.

---

# 31. Architectural Decisions

| Decision | Choice | Reason |
|---|---|---|
| Frontend | Next.js + React + TypeScript | Strong fit for the mobile-first React web app and rapid implementation |
| Styling | Tailwind CSS | Small, consistent design system can be implemented quickly |
| Backend | FastAPI + Python | Fast API development, validation, and clean separation from frontend |
| Database | Supabase PostgreSQL | Relational data model plus managed database services |
| Geospatial | PostGIS | Efficient radius and distance queries near the data |
| Authentication | Supabase Auth | Avoid building password/session infrastructure from scratch |
| AI | OpenAI Responses API via backend adapter | Multimodal text/image flow with structured output capability |
| Location | Browser Geolocation + manual fallback | Smallest practical location capture path |
| Map | Mapbox GL JS, optional | Useful visualization without making maps part of the matching core |
| Storage | Supabase Storage | Integrated object storage for request/part images |
| Notifications | In-app DB-backed notifications | Simplest reliable MVP |
| Real-time | Deferred | Core workflow works without WebSockets |
| Deployment | Vercel + Render + Supabase | Practical split for Next.js frontend, Python backend, and managed data services |
| Architecture Style | Modular monolith | Lowest unnecessary operational complexity |
| External Dependencies | Adapter-based | Makes mock/fallback implementations possible |

---

# 32. Final Technical Architecture

## Frontend

**Next.js + React + TypeScript + Tailwind CSS**

Responsibility:

- Mobile-first UI
- Authentication experience
- Request creation
- AI confirmation
- Search/results
- Inventory
- Requests
- Transactions
- Notifications

## Backend

**FastAPI + Python**

Responsibility:

- REST API
- Authorization
- Validation
- Business rules
- Request lifecycle
- AI orchestration
- Matching orchestration
- Transaction rules
- Notifications

## Database

**Supabase PostgreSQL + PostGIS**

Responsibility:

- Application data
- Relationships
- Inventory
- Requests
- Matching candidates
- Transactions
- Notifications
- Geospatial search

## AI

**OpenAI multimodal model through a server-side adapter**

Responsibility:

- Natural-language understanding
- Image interpretation
- Structured spare-part extraction

The backend validates AI output before it becomes searchable data.

## Matching

**A FastAPI module using PostgreSQL/PostGIS**

Responsibility:

- Normalize requirement fields
- Search inventory
- Check listed availability
- Calculate distance
- Filter candidates
- Rank exact/possible matches

## Location

**Browser Geolocation API + PostGIS**

Responsibility:

- Capture shop coordinates
- Support manual fallback
- Radius filtering
- Distance output

Mapbox is optional for visualization only.

## Storage

**Supabase Storage**

Responsibility:

- Request images
- Inventory images
- Optional shop images

Files remain separate from relational records.

## Authentication

**Supabase Auth + FastAPI authorization**

Responsibility:

- Identity
- Session/token issuance
- Backend authorization
- Resource ownership checks

## Deployment

**Vercel + Render + Supabase**

\`\`\`text
Next.js
   ↓
Vercel

FastAPI
   ↓
Render

PostgreSQL / Auth / Storage
   ↓
Supabase
\`\`\`

---

# 33. Final Architecture Diagram

\`\`\`mermaid
flowchart TB
    USER[Repair Shop User]

    subgraph CLIENT[Web Client]
        NEXT[Next.js + React + TypeScript]
        UI[SpareLink UI]
    end

    subgraph BACKEND[FastAPI Modular Monolith]
        API[REST API]
        AUTHZ[Authorization]
        SHOP[Shop Module]
        INV[Inventory Module]
        REQ[Request Module]
        AI[AI Module]
        MATCH[Matching Module]
        TXN[Transaction Module]
        NOTIF[Notification Module]
        LOC[Location Module]
    end

    subgraph SUPABASE[Supabase]
        AUTH[Supabase Auth]
        DB[(PostgreSQL + PostGIS)]
        STORE[Supabase Storage]
    end

    OPENAI[OpenAI Multimodal API]
    MAP[Mapbox GL JS - Optional]

    USER --> NEXT
    NEXT --> UI
    NEXT --> AUTH
    NEXT --> API

    API --> AUTHZ
    AUTHZ --> AUTH

    API --> SHOP
    API --> INV
    API --> REQ
    API --> AI
    API --> MATCH
    API --> TXN
    API --> NOTIF
    API --> LOC

    SHOP --> DB
    INV --> DB
    REQ --> DB
    MATCH --> DB
    TXN --> DB
    NOTIF --> DB
    LOC --> DB

    AI --> STORE
    AI --> OPENAI
    INV --> STORE

    LOC --> MAP
\`\`\`

### Core Technical Path

\`\`\`text
User
  ↓
Next.js
  ↓
FastAPI
  ↓
Request Module
  ↓
AI Module
  ↓
Validated Request
  ↓
Matching Module
  ↓
PostgreSQL + PostGIS
  ↓
Potential Matches
  ↓
Supplier Request
  ↓
Transaction
\`\`\`

---

# 34. Architecture Review

## 1. Can a Small Student Team Build It?

**Yes.**

The MVP is one frontend, one backend, and one managed data platform.

## 2. Is It Overcomplicated?

**No.**

The architecture avoids microservices, custom real-time infrastructure, dedicated queues, and unnecessary cloud components.

## 3. Is Every Technology Needed?

**Core:** Next.js, FastAPI, PostgreSQL/Supabase, Auth, Storage, AI, PostGIS.

**Optional:** Mapbox for map visualization.

## 4. Can the Demo Survive External API Failure?

**Yes, with the planned fallback adapters and controlled demo data.**

## 5. Are Authentication and Authorization Separated?

**Yes.**

Supabase Auth establishes identity. FastAPI checks what the authenticated shop is allowed to do.

## 6. Is AI Safely Integrated?

**Yes.**

AI output is schema-validated, treated as untrusted, checked for uncertainty, and confirmed by the user.

## 7. Can Nearby Inventory Be Searched Efficiently?

**Yes.**

PostGIS is used for radius filtering and distance calculations.

## 8. Can the System Grow?

**Yes.**

Modules can be optimized or extracted only when real scale requires it.

## 9. Does It Match Parts 1–3?

**Yes.**

The architecture supports the established mobile-first request, AI confirmation, nearby matching, supplier acceptance, handover, and transaction completion flow.

## 10. Can Part 5 Use It?

**Yes.**

Part 5 can derive the detailed database schema directly from the architectural entities and relationships defined here.

---

# 35. Technical Reference Notes

The selected architecture aligns with the current capabilities documented by the vendors:

- Next.js provides the React application framework and App Router for modern applications.
- Supabase provides managed Postgres, Auth, Storage, and related services.
- FastAPI provides Python API and security tooling.
- OpenAI's current API supports text and image inputs through the Responses API and structured-output patterns.
- PostGIS provides \`ST_DWithin\` for radius filtering and \`ST_Distance\` for distance calculation.
- Mapbox GL JS can provide browser map rendering and a geolocation control.
- Vercel provides direct Next.js deployment support.
- Render provides a practical deployment path for FastAPI services.

These references support the technology choices; they are not performance or business guarantees.

---

> **Status: Part 4 Complete | Technical Architecture Defined**
