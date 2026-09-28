# SPARELINK | PART 9: FRONTEND & USER EXPERIENCE INTEGRATION

## MASTER INSTRUCTION

Continue building the **SpareLink** project from Parts 1–8.

Treat Parts 1–8 as the existing source of truth. Do not contradict their architecture, data model, backend responsibilities, AI boundaries, or Matching Engine logic.

Part 9 must define the **Frontend & User Experience Integration layer** that connects the user-facing application to the backend, AI Engine, database-backed inventory system, and Matching Engine.

The goal is to design a frontend that feels extremely simple to a repair-shop owner while hiding the technical complexity underneath.

SpareLink should feel like:

> **“Tell us what you need. We find who can help.”**

The frontend must not expose unnecessary technical complexity to users.

---

# 1. PART 9 OBJECTIVE

Define how SpareLink transforms backend capabilities into a practical, mobile-first user experience.

Part 9 must cover:

- frontend architecture
- user interface structure
- navigation
- authentication
- shop onboarding
- inventory management
- spare-part request creation
- AI-assisted input
- request confirmation
- match-result presentation
- shop cards
- match explanation
- accept/decline flow
- reservation state
- fulfillment state
- notifications
- real-time updates
- loading states
- empty states
- error states
- offline handling
- responsive design
- accessibility
- API integration
- frontend state management
- security
- performance
- testing
- MVP priorities

The result must be implementation-ready.

---

# 2. FRONTEND PRODUCT PHILOSOPHY

SpareLink is designed for real repair-shop environments.

Users may:

- be busy with customers
- have limited time
- use low-end or mid-range phones
- type quickly
- use Telugu, English, or mixed language
- make spelling mistakes
- have incomplete part information
- have unreliable internet connectivity

Therefore:

### Principle 1: Minimum typing

The system should minimize manual typing.

### Principle 2: Fast actions

Important actions should require very few steps.

### Principle 3: Progressive information

Show important information first.

### Principle 4: AI assistance

Allow AI to understand messy input without forcing users to learn technical terminology.

### Principle 5: Human confirmation

AI suggestions must be editable and confirmed by the user.

### Principle 6: Clear status

Users should always understand what is happening.

### Principle 7: Trust

Never present an uncertain AI interpretation as confirmed fact.

---

# 3. FRONTEND ARCHITECTURE

Define a production-ready frontend architecture.

Recommended direction:

- React
- Next.js
- TypeScript
- Tailwind CSS or equivalent styling system
- React Query / TanStack Query or equivalent server-state library
- lightweight client state management where required
- responsive mobile-first UI

Architecture:

```text
User
 ↓
Frontend Application
 ↓
UI Components
 ↓
Feature Modules
 ↓
State / Query Layer
 ↓
API Client
 ↓
FastAPI Backend
 ↓
AI / Database / Matching Engine
```

Keep frontend business logic separate from backend business logic.

The frontend must never become the source of truth for:

- inventory
- authorization
- pricing
- compatibility
- fulfillment
- reservation
- shop ownership

---

# 4. FRONTEND PROJECT STRUCTURE

Design a clean structure such as:

```text
frontend/
├── app/
│   ├── login/
│   ├── onboarding/
│   ├── dashboard/
│   ├── inventory/
│   ├── requests/
│   ├── matches/
│   ├── suppliers/
│   └── settings/
│
├── components/
│   ├── ui/
│   ├── navigation/
│   ├── forms/
│   ├── inventory/
│   ├── requests/
│   ├── matches/
│   └── notifications/
│
├── features/
│   ├── auth/
│   ├── onboarding/
│   ├── inventory/
│   ├── spare-requests/
│   ├── matching/
│   └── notifications/
│
├── lib/
│   ├── api/
│   ├── auth/
│   ├── validation/
│   └── utilities/
│
├── hooks/
├── types/
├── constants/
└── styles/
```

Explain the responsibility of every major directory.

---

# 5. APPLICATION NAVIGATION

Design navigation around the most common repair-shop tasks.

Recommended primary navigation:

```text
Home
Requests
Inventory
Messages / Notifications
Profile
```

The interface should make the main action highly visible:

> **Find a Spare Part**

The dashboard should not become a collection of unnecessary analytics.

Prioritize actions over information overload.

---

# 6. LOGIN AND AUTHENTICATION

Define the authentication experience.

Possible flow:

```text
Open App
 ↓
Login / Register
 ↓
Phone / Email Verification
 ↓
Shop Profile
 ↓
Dashboard
```

Authentication must integrate with the backend authentication system defined in Part 6.

Frontend responsibilities:

- login forms
- registration forms
- validation
- loading states
- authentication persistence
- logout
- protected routes
- session expiration handling

Never store sensitive credentials insecurely.

---

# 7. SHOP ONBOARDING

Define a simple onboarding process.

Collect only necessary information initially.

Example:

```text
Shop Name
Owner Name
Phone
Location
Shop Category
Services
```

Optional:

```text
Address
Working Hours
Description
Business Registration Details
```

Location should support geographic matching.

Explain how the frontend obtains and sends location data to the backend.

Do not expose unnecessary technical GIS information to the user.

---

# 8. HOME DASHBOARD

Design the main dashboard.

The dashboard should answer:

> “What can I do right now?”

Recommended sections:

### Primary Action

**Find a Spare Part**

### Secondary Actions

- Add Inventory
- View Requests
- Check Matches

### Recent Activity

Show:

- active requests
- recent matches
- pending supplier responses
- completed requests

Keep the interface visually clean.

---

# 9. SPARE REQUEST CREATION

This is one of the most important frontend flows.

The user should be able to type naturally.

Example:

```text
Samsung M14 display kavali urgent ga
```

The frontend sends the text to the AI-enabled backend.

Flow:

```text
User Input
 ↓
AI Processing
 ↓
Structured Information
 ↓
Frontend Preview
 ↓
User Confirmation/Edit
 ↓
Create Request
```

The frontend must never assume the AI result is automatically correct.

---

# 10. AI RESULT CONFIRMATION UI

After AI processing, display a clear confirmation card.

Example:

```text
We understood:

Brand: Samsung
Model: Galaxy M14
Part: Display
Quantity: 1
Urgency: High

[Confirm Request]
[Edit]
```

If confidence is low:

```text
We need a little more information.

Which model do you mean?

Galaxy M14
Galaxy M14 5G
Other
```

AI uncertainty should produce clarification, not silent guessing.

---

# 11. IMAGE-BASED REQUESTS

If Part 7's image capability is enabled, allow users to upload or capture a photo.

Flow:

```text
Camera / Upload
 ↓
Image Preview
 ↓
AI Analysis
 ↓
Detected Part Information
 ↓
User Confirmation
 ↓
Request Creation
```

Show uncertainty clearly.

Example:

```text
Possible match:
Samsung M14 Display

Confidence: Medium

[Confirm]
[Correct]
```

Never claim that image recognition guarantees compatibility.

---

# 12. REQUEST STATUS UI

Define a clear request lifecycle.

Use the backend state machine from Parts 6 and 8.

Example:

```text
OPEN
 ↓
MATCHED
 ↓
RESERVED
 ↓
ACCEPTED
 ↓
COMPLETED
```

Alternative states:

```text
DECLINED
EXPIRED
CANCELLED
```

Frontend should translate technical states into human-friendly labels.

For example:

```text
OPEN
Finding nearby shops...

MATCHED
3 nearby shops found

RESERVED
Part temporarily reserved

ACCEPTED
Supplier accepted your request

COMPLETED
Request completed
```

---

# 13. MATCH RESULTS SCREEN

This is the central user-facing output of Part 8.

The frontend receives ranked match results from:

```text
POST /api/v1/matches/search
```

Display useful information such as:

- shop name
- distance
- part availability
- compatibility
- quantity
- response status
- estimated availability
- relevant shop information

Avoid exposing raw matching scores unless useful.

Instead show understandable explanations.

Example:

```text
Sai Mobile Care

0.8 km away

Samsung M14 Display
Available: 2

✓ Exact part
✓ Nearby
✓ Available now

[Request Part]
```

---

# 14. MATCH EXPLANATION

The Matching Engine is explainable.

Use that capability in the frontend.

Example:

```text
Why this shop?

✓ Exact model match
✓ Part currently available
✓ Within 1 km
✓ Recently updated inventory
```

Do not expose internal numerical weights such as:

```text
Compatibility = 40%
Distance = 15%
```

unless there is a strong product reason.

Users need understandable reasoning, not the scoring formula.

---

# 15. SHOP MATCH CARD

Design a reusable match card.

Suggested structure:

```text
┌─────────────────────────────┐
│ Sai Mobile Care             │
│ ⭐ Trusted supplier signal   │
│                             │
│ Samsung M14 Display         │
│ 2 available                 │
│                             │
│ ✓ Exact match               │
│ ✓ 0.8 km away               │
│ ✓ Updated recently          │
│                             │
│ [Request Part]              │
└─────────────────────────────┘
```

The design should remain compact on mobile screens.

---

# 16. ACCEPT / REQUEST FLOW

Define what happens when the user selects a match.

Flow:

```text
User selects shop
 ↓
Frontend requests reservation/acceptance
 ↓
Backend re-checks inventory
 ↓
Transaction begins
 ↓
Inventory is safely reserved
 ↓
Request state changes
 ↓
Frontend receives updated status
```

Important:

The frontend must never assume that the part is still available.

The backend remains authoritative.

---

# 17. RACE CONDITION UX

A match can disappear because another shop or customer fulfilled the inventory first.

Example:

```text
This part is no longer available
at this shop.

We're checking the next best
available option.
```

Then offer:

```text
[View Other Matches]
```

Do not show a generic technical error.

Turn backend failures into understandable user feedback.

---

# 18. INVENTORY UI

Define inventory management.

Users should be able to:

- add part
- edit part
- remove part
- update quantity
- mark availability
- search inventory
- filter inventory

Example:

```text
Samsung M14 Display
Available: 2

[-] 2 [+]

[Edit]
```

Inventory updates must call backend APIs.

Never trust frontend quantities.

---

# 19. FAST INVENTORY UPDATE

Because repair shops may receive parts or sell parts frequently, provide quick quantity controls.

Example:

```text
Samsung M14 Display

Available: 4

[−]     4     [+]
```

Backend validates every update.

Prevent:

```text
Available = -1
```

The database transaction remains the final authority.

---

# 20. NOTIFICATION SYSTEM

Define frontend notification handling.

Notifications may include:

- new match
- supplier accepted
- supplier declined
- reservation expired
- request completed
- inventory warning
- supplier response
- fallback supplier found

Use:

- in-app notifications
- push notifications when supported
- optional WhatsApp/SMS integration later

Notifications must not expose sensitive information unnecessarily.

---

# 21. REAL-TIME UPDATES

Where useful, use:

- WebSockets
- Server-Sent Events
- polling fallback

Potential real-time events:

```text
MATCH_FOUND
SUPPLIER_ACCEPTED
SUPPLIER_DECLINED
RESERVATION_CONFIRMED
RESERVATION_EXPIRED
REQUEST_COMPLETED
```

If real-time connectivity fails, fall back gracefully to periodic refresh.

---

# 22. LOADING STATES

Every asynchronous operation needs a meaningful loading state.

Examples:

Instead of:

```text
Loading...
```

Use:

```text
Finding nearby suppliers...
```

For AI:

```text
Understanding your request...
```

For matching:

```text
Checking nearby inventory...
```

For reservation:

```text
Confirming availability...
```

Avoid long blank screens.

---

# 23. EMPTY STATES

Design useful empty states.

### No inventory

```text
Your inventory is empty.

Add your first spare part to start
helping nearby repair shops.

[Add Inventory]
```

### No matches

```text
No nearby shops currently have this part.

Try:
• expanding the search area
• checking the part details
• asking supplier network
```

### No requests

```text
No active requests.

Need a spare part?

[Find a Spare Part]
```

---

# 24. ERROR STATES

Errors must be actionable.

Bad:

```text
500 Internal Server Error
```

Better:

```text
We couldn't check nearby inventory.

Please try again.

[Retry]
```

Technical details should be logged for developers, not dumped onto users.

---

# 25. OFFLINE AND WEAK NETWORK SUPPORT

Design for unstable connectivity.

The frontend should:

- detect network loss
- preserve unsent form data where safe
- show connection state
- retry safe requests
- prevent accidental duplicate submissions
- refresh stale information after reconnecting

Example:

```text
You're offline.

Your request is saved locally.
We'll reconnect when your internet returns.
```

Only claim queued submission when the implementation actually supports it.

---

# 26. API CLIENT ARCHITECTURE

Create a centralized API layer.

Example:

```text
lib/api/
├── client.ts
├── auth.ts
├── shops.ts
├── inventory.ts
├── requests.ts
├── matches.ts
└── notifications.ts
```

Every API request should handle:

- authentication
- request IDs
- validation
- timeout
- errors
- response parsing

Do not scatter raw fetch calls throughout UI components.

---

# 27. FRONTEND DATA VALIDATION

Frontend validation improves UX.

Validate:

- required fields
- quantity
- phone format
- image size
- supported file types
- text length

However:

> Frontend validation is for user experience. Backend validation is for security and correctness.

Never rely only on frontend validation.

---

# 28. STATE MANAGEMENT

Separate:

### Server state

Examples:

- inventory
- requests
- matches
- notifications
- shop profile

Use a server-state solution such as TanStack Query.

### Local UI state

Examples:

- modal open/close
- selected shop
- form fields
- temporary filters

Do not put the entire application into one giant global state object.

---

# 29. SECURITY

Frontend security requirements:

- protected routes
- secure authentication handling
- no secrets in frontend source
- input sanitization
- safe file upload handling
- HTTPS
- controlled API access
- safe error messages
- no exposure of internal database identifiers unless required

Never put:

```text
AI_API_KEY
DATABASE_PASSWORD
JWT_SIGNING_SECRET
```

inside client-side code.

---

# 30. RESPONSIVE DESIGN

The primary target is mobile.

Design for:

```text
Mobile
 ↓
Tablet
 ↓
Desktop
```

The mobile interface should not simply be a shrunken desktop interface.

Prioritize:

- large tap targets
- readable text
- simple navigation
- fast actions
- low cognitive load

---

# 31. ACCESSIBILITY

Follow practical accessibility principles:

- sufficient contrast
- readable typography
- keyboard navigation
- semantic HTML
- labels for inputs
- accessible buttons
- screen-reader support
- visible focus states
- avoid color-only status indicators

Example:

Do not show only:

🔴

Instead:

🔴 Unavailable

---

# 32. DESIGN SYSTEM

Create a consistent visual language.

Define:

- typography
- spacing
- buttons
- cards
- forms
- status badges
- icons
- alerts
- navigation
- modal styles

The visual identity should communicate:

**Fast + Local + Trustworthy + Practical**

Avoid excessive decoration.

---

# 33. PERFORMANCE

Optimize for real-world mobile conditions.

Priorities:

- small bundles
- lazy loading
- image compression
- caching
- pagination
- debounced search
- optimized API calls
- skeleton loading
- minimal unnecessary re-renders

Do not load an entire inventory database into the browser.

---

# 34. ACCESSIBILITY + PERFORMANCE + UX BALANCE

Do not optimize only for visual appearance.

The product should satisfy:

```text
Usability
+
Speed
+
Accessibility
+
Reliability
```

A beautiful interface that takes 10 seconds to load is not a successful SpareLink interface.

---

# 35. FRONTEND REQUEST FLOW

Define the complete flow:

```text
User opens app
 ↓
Dashboard
 ↓
Find Spare Part
 ↓
Natural-language input
 ↓
AI interpretation
 ↓
Confirmation
 ↓
Request created
 ↓
Matching Engine searches
 ↓
Match results displayed
 ↓
User selects supplier
 ↓
Backend re-checks availability
 ↓
Reservation
 ↓
Supplier response
 ↓
Acceptance
 ↓
Fulfillment
 ↓
Completion
```

Every transition must have a visible user state.

---

# 36. FRONTEND ↔ BACKEND CONTRACT

Define the frontend contract with backend APIs from Parts 6–8.

Example:

```text
POST /api/v1/requests
POST /api/v1/ai/analyze
POST /api/v1/matches/search
POST /api/v1/matches/{match_id}/accept
GET  /api/v1/requests/{request_id}
GET  /api/v1/inventory
POST /api/v1/inventory
PATCH /api/v1/inventory/{inventory_id}
GET  /api/v1/notifications
```

The exact endpoints must remain consistent with the backend implementation.

Do not invent duplicate APIs unnecessarily.

---

# 37. ERROR CONTRACT

Define a consistent backend error format.

Example:

```json
{
  "error": {
    "code": "INVENTORY_UNAVAILABLE",
    "message": "The requested quantity is no longer available.",
    "request_id": "req_123"
  }
}
```

Frontend maps technical codes to user-friendly messages.

---

# 38. REQUEST IDEMPOTENCY

Prevent accidental duplicate requests.

For important actions such as:

```text
Create Request
Reserve Part
Accept Match
Complete Request
```

use idempotency mechanisms where appropriate.

Example:

User taps:

```text
[Request Part]
```

twice rapidly.

The system should not create two reservations.

---

# 39. MATCH RESULT REFRESH

Match results can become stale.

Provide refresh behavior.

Example:

```text
Last checked just now

[Refresh]
```

When the user attempts to reserve a stale match:

```text
Checking current availability...
```

Backend performs the authoritative check.

---

# 40. USER JOURNEY CONSISTENCY

The frontend must preserve the user journey defined in Part 2.

The user should never feel like they are moving through separate disconnected systems.

AI, matching, inventory, notifications, and backend operations should appear as one unified product.

---

# 41. COMPONENT REUSABILITY

Create reusable components such as:

```text
SpareRequestForm
AIResultCard
MatchCard
InventoryCard
StatusBadge
NotificationItem
LoadingState
EmptyState
ErrorState
ConfirmDialog
QuantityControl
SearchBar
```

Avoid duplicating UI logic.

---

# 42. TESTING STRATEGY

Define:

### Unit Tests

Test:

- validation
- state transformations
- formatting
- API parsing

### Component Tests

Test:

- forms
- match cards
- inventory controls
- status components

### Integration Tests

Test:

```text
Request → AI → Confirmation → Match Results
```

### End-to-End Tests

Test complete workflows:

```text
Login
 ↓
Create Request
 ↓
Find Match
 ↓
Reserve
 ↓
Accept
 ↓
Complete
```

---

# 43. IMPORTANT FAILURE TESTS

Test:

- AI returns invalid output
- no matching shop
- inventory disappears
- network disconnects
- duplicate button clicks
- expired reservation
- unauthorized request
- backend timeout
- invalid image
- stale match
- supplier declines
- user cancels request

The UI must respond gracefully in every case.

---

# 44. MVP PRIORITIES

### P0

Must have:

- authentication
- shop onboarding
- dashboard
- spare request creation
- AI result confirmation
- inventory management
- match results
- match cards
- request status
- accept/request flow
- loading states
- error states
- mobile responsive UI
- backend API integration

### P1

Add:

- image upload
- real-time notifications
- push notifications
- advanced inventory filters
- multilingual UI
- offline improvements
- supplier communication

### P2

Add:

- voice requests
- advanced analytics
- predictive UI
- personalized recommendations
- advanced supplier network visualization
- intelligent notification prioritization

---

# 45. PERFORMANCE TARGETS

Define realistic engineering targets.

Examples:

```text
Initial UI load: optimized for mobile networks
API interaction: minimal unnecessary round trips
Match results: fast enough for interactive use
Inventory search: responsive with pagination
Image uploads: compressed before transmission where appropriate
```

Do not invent unrealistic guarantees.

Measure actual performance during implementation.

---

# 46. OBSERVABILITY

Frontend should capture useful diagnostics.

Track:

- request ID
- API latency
- failed API calls
- UI errors
- match-search duration
- reservation failures
- network failures

Do not collect unnecessary personal information.

Respect privacy requirements.

---

# 47. ANALYTICS

Product analytics may track events such as:

```text
REQUEST_CREATED
AI_CONFIRMATION_EDITED
MATCH_VIEWED
MATCH_SELECTED
RESERVATION_STARTED
RESERVATION_SUCCESS
RESERVATION_FAILED
REQUEST_COMPLETED
```

Analytics must not replace operational logging.

Avoid collecting sensitive information unnecessarily.

---

# 48. USER TRUST DESIGN

SpareLink is coordinating real businesses.

Therefore, the interface must clearly distinguish:

```text
AI interpretation
```

from:

```text
Confirmed information
```

and:

```text
Current inventory
```

Example:

```text
AI detected:
Samsung M14 Display
```

Then:

```text
Confirmed request:
Samsung M14 Display
```

Then:

```text
Current availability:
2 units
```

These are three different pieces of information and should not be visually confused.

---

# 49. END-TO-END FRONTEND EXAMPLE

User types:

```text
Bro Samsung M14 display kavali urgent ga
```

Frontend:

```text
Understanding your request...
```

AI returns structured information.

Frontend displays:

```text
Samsung Galaxy M14
Display
Quantity: 1
Urgency: High

[Confirm]
[Edit]
```

User confirms.

Frontend:

```text
Finding nearby shops...
```

Matching Engine returns:

```text
Sai Mobile Care
0.8 km
Exact match
2 available

Ravi Mobiles
2.1 km
Exact match
1 available

Kiran Electronics
4.7 km
Compatible part
3 available
```

User selects Sai Mobile Care.

Frontend:

```text
Checking current availability...
```

Backend confirms inventory.

Frontend:

```text
Part reserved successfully.

Supplier: Sai Mobile Care
Quantity: 1

[View Request]
```

Supplier accepts.

Frontend:

```text
Supplier accepted your request.
```

After fulfillment:

```text
Request completed ✓
```

This complete experience should feel simple even though multiple backend systems are working underneath.

---

# 50. DEFINITION OF DONE

Part 9 is complete when:

- frontend architecture is defined
- mobile-first UX is defined
- authentication flow is defined
- onboarding is defined
- dashboard is defined
- request creation is defined
- AI confirmation is defined
- match results are defined
- match cards are defined
- reservation flow is defined
- inventory UI is defined
- notifications are defined
- real-time behavior is defined
- loading states are defined
- empty states are defined
- error states are defined
- offline behavior is defined
- API integration is defined
- state management is defined
- security requirements are defined
- accessibility requirements are defined
- performance requirements are defined
- testing strategy is defined
- MVP priorities are defined

---

# 51. CONNECTION TO PART 10

Part 10 should continue from the frontend foundation and define the **real-time communication, notifications, supplier interaction, and fulfillment experience** in greater depth.

Part 9 establishes how users interact with SpareLink.

Part 10 should establish how SpareLink keeps users and suppliers synchronized while a request is being fulfilled.

The architecture must remain consistent with Parts 1–9.

---

# FINAL PRINCIPLE

SpareLink should hide complexity from the user.

Behind the screen:

```text
AI
+
Backend
+
Database
+
Geolocation
+
Matching Engine
+
Inventory
+
Transactions
+
Notifications
```

But to the repair-shop owner, it should simply feel like:

> **“I need this part.”**

SpareLink responds:

> **“Here are the nearby shops that can help.”**

That simplicity is the frontend's most important job.