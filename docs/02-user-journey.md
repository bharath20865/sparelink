# SpareLink | User Journey & Product Flow

> **The missing part is just one message away.**

Part 2 defines exactly what the user does, what the system does, what information is displayed, and what happens next throughout the SpareLink MVP.

---

# 1. User Types

## A. Requesting Shop

The shop that needs a spare part.

**Goal:** Find a potentially available nearby part and complete the exchange.

**Provides:** Part requirement, brand/model, quantity, urgency, description, supported input, and location.

**Sees:** AI interpretation, confirmation controls, matches, distance, availability status, request status, notifications, and transaction history.

**Actions:** Create request, correct AI interpretation, search, view matches, send/cancel requests, track responses, confirm receipt, complete transaction.

## B. Supplying Shop

The shop that may have the requested part.

**Goal:** Review requests, verify stock, and accept or decline.

**Receives:** Part/model, quantity, requesting shop, approximate distance, urgency, and request time.

**Sees:** Request details, matching inventory, quantity, requesting shop, and status.

**Actions:** Open request, check inventory, confirm quantity, accept, decline, arrange handover, confirm completion.

## C. End Customer

The customer whose repair depends on the part.

The end customer is not a primary platform user. Their benefit is indirect through the repair shop's additional local sourcing option.

---

# 2. Complete End-to-End User Journey

~~~text
Landing Page
     ↓
Registration / Login
     ↓
Shop Profile
     ↓
Location Setup
     ↓
Inventory Setup
     ↓
Find a Spare Part
     ↓
Text / Voice / Image Input
     ↓
AI Analysis
     ↓
Confirm / Correct Requirement
     ↓
Nearby Inventory Search
     ↓
Matching Results
     ↓
View Shop Match
     ↓
Send Request
     ↓
Supplier Receives Request
     ↓
Accept / Decline
     ↓
Match Confirmed
     ↓
Part Handover / Local Delivery
     ↓
Mark Transaction Complete
     ↓
Transaction History Updated
~~~

Each stage follows:

~~~text
Step
 ↓
User Action
 ↓
System Response
 ↓
Information Displayed
 ↓
Next Action
~~~

---

# 3. Journey Stages

## Stage 1: Landing Page

**User action:** Open SpareLink.

**System response:** Show product explanation and primary actions.

**Information:** Name, tagline, short explanation, Login, Register.

**Next:** Login or register.

## Stage 2: Registration / Login

**User action:** Register or log in.

**System response:** Authenticate the shop.

**Information:** Account fields, authentication state, invalid-input errors.

**Next:** New user → shop setup. Existing user → dashboard.

## Stage 3: Shop Profile Creation

**User action:** Enter basic business information.

**System response:** Store profile.

**Important data:** Shop name, operator name, contact information, repair category, basic address/area.

**Possible error:** Missing or invalid required fields.

**Next:** Location setup.

## Stage 4: Location Setup

**User action:** Provide shop location.

**System response:** Store a usable location for local matching.

**Information:** Selected location and configurable search radius.

**Next:** Inventory setup.

## Stage 5: Inventory Setup

**User action:** Add available parts.

**System response:** Create inventory records.

**Important data:**

~~~text
Part Name
Brand
Compatible Model
Quantity
Availability
~~~

**Possible errors:** Invalid quantity, incomplete model, duplicate entry, incomplete description.

**Next:** Dashboard.

## Stage 6: Creating a Spare-Part Request

**User action:** Select Find a Spare Part.

**System response:** Open the request flow.

**Information:** Text, voice, or image input.

**Next:** Submit requirement.

## Stage 7: Text / Voice / Image Input

Supported input concepts:

- Text
- Voice
- Image

Text can be the primary working input for the MVP.

Example:

> "Samsung M14 display kavali urgent ga."

**Next:** AI analysis.

## Stage 8: AI Analysis

**System response:** Extract structured request information.

~~~text
Brand: Samsung
Model: Galaxy M14
Part: Display
Category: Mobile Display
Quantity: 1
Urgency: High
Description: Optional
~~~

The interpretation should be visible before searching.

**Next:** Confirmation/correction.

## Stage 9: User Confirmation or Correction

The shopkeeper can:

- Confirm
- Edit
- Reject

Example:

~~~text
Detected Part:
Samsung Galaxy M14 Display

[Confirm]
[Edit]
~~~

When confidence is low:

~~~text
AI uncertain
   ↓
Show possible interpretation
   ↓
User confirms / corrects
   ↓
Continue
~~~

Search proceeds only after sufficient confirmation.

## Stage 10: Nearby Inventory Search

**User action:** Confirm the requirement.

**System response:** Search participating inventory.

**Search inputs:** Part/model, compatibility, quantity, location, configured radius.

A radius such as 10 km can be an initial configuration, not a permanent limit.

**Next:** Match evaluation.

## Stage 11: Matching and Result Ranking

~~~text
Required Part
   ↓
Inventory Search
   ↓
Exact / Compatible Match
   ↓
Availability Check
   ↓
Distance Filter
   ↓
Shop Status
   ↓
Result Ranking
~~~

Factors can include:

1. Part match quality
2. Availability
3. Distance
4. Quantity
5. Shop availability
6. Response status/history where available
7. Urgency

The MVP should keep matching simple and explainable.

**Important:** A search result is potential availability, not guaranteed availability.

## Stage 12: Viewing a Shop Match

Display:

~~~text
Shop Name
Distance
Part
Compatibility
Quantity
Availability Status
Shop Type
Response Indicator
Request Button
~~~

**Next:** Send request.

## Stage 13: Sending a Request

**User action:** Select Request Part.

**System response:** Create and send the request.

**Request information:** Part, model, quantity, requesting shop, approximate location, urgency, timestamp.

**Next:** Wait for supplier response.

## Stage 14: Supplying Shop Receives Request

The supplier sees:

~~~text
Part:
Samsung Galaxy M14 Display

Quantity:
1

Requesting Shop:
Shop A

Urgency:
High
~~~

**Next:** Check actual inventory.

## Stage 15: Accept / Decline

The supplier verifies stock and chooses Accept or Decline.

- **Accept:** Request becomes accepted for the confirmed quantity.
- **Decline:** Requesting shop is notified.

## Stage 16: Match Confirmation

When accepted, show:

- Supplier name
- Confirmed part
- Confirmed quantity
- Request status
- Handover information

**Next:** Arrange handover.

## Stage 17: Part Exchange or Local Delivery

Possible options:

- Pickup
- Local delivery
- Other agreed handover

The MVP records the handover state but does not require a full logistics platform.

## Stage 18: Marking Transaction Complete

The requesting shop confirms receipt.

The state becomes:

~~~text
COMPLETED
~~~

## Stage 19: Transaction History Update

A basic record is stored:

~~~text
Part: Samsung Galaxy M14 Display
From: Shop B
To: Shop A
Quantity: 1
Status: Completed
~~~

---

# 4. Main User Flow Diagram

~~~mermaid
flowchart TD
    A[Requesting Shop] --> B[Spare Part Request]
    B --> C[AI Understanding]
    C --> D{AI Interpretation Clear?}

    D -- No --> E[User Corrects Information]
    E --> C

    D -- Yes --> F[User Confirmation]
    F --> G[Nearby Inventory Search]
    G --> H{Matching Shop Found?}

    H -- No --> I[Expand Search / Save Request]
    I --> G

    H -- Yes --> J[Matching Results]
    J --> K[View Shop Match]
    K --> L[Send Request]
    L --> M[Supplying Shop Receives Request]
    M --> N{Supplier Accepts?}

    N -- No --> O[Try Another Match]
    O --> J

    N -- Yes --> P[Match Confirmed]
    P --> Q[Part Handover / Local Delivery]
    Q --> R[Transaction Completed]
~~~

---

# 5. Requesting Shop Flow

| Step | User Action | System Action | Important Data | Possible Error | Next Step |
|---|---|---|---|---|---|
| 1 | Opens SpareLink | Loads landing page | App state | Network failure | Retry |
| 2 | Logs in | Authenticates | Credentials | Invalid credentials | Correct login |
| 3 | Opens Find a Spare Part | Opens request flow | Request fields | Missing setup | Complete profile |
| 4 | Types requirement | Captures input | Raw text | Incomplete input | Edit |
| 5 | Uploads image | Receives image | Image | Poor quality | Retake/use text |
| 6 | Uses voice | Converts input | Transcript | Recognition error | Edit |
| 7 | Reviews AI | Shows structured result | Brand/model/part | Wrong interpretation | Correct |
| 8 | Confirms requirement | Stores confirmed data | Structured request | Missing field | Complete |
| 9 | Sees matches | Searches inventory | Part + location | No matches | Expand/save |
| 10 | Chooses shop | Opens details | Shop + inventory | Shop unavailable | Return |
| 11 | Sends request | Creates request | Request payload | API/network failure | Retry |
| 12 | Waits | Tracks status | Request state | Expired | Try another |
| 13 | Receives acceptance | Updates status | Confirmed availability | Supplier cancels | Try another |
| 14 | Completes exchange | Records handover | Handover state | Part not received | Resolve |
| 15 | Marks received | Completes transaction | Transaction record | Incorrect completion | Review |

---

# 6. Supplying Shop Flow

~~~text
Notification
    ↓
Open Request
    ↓
View Part Details
    ↓
Check Inventory
    ↓
Confirm Quantity
    ↓
Accept / Decline
    ↓
Arrange Handover
    ↓
Confirm Completion
~~~

### Notification

Show part, model, quantity, requesting shop, distance/area, and urgency.

### Open Request

Show the complete request.

### Check Inventory

Compare the request with actual physical stock.

### Confirm Quantity

Confirm the quantity that can really be supplied.

### Accept / Decline

Accept only after verification. Decline when the shop cannot provide the part.

### Arrange Handover

Coordinate pickup or local delivery.

### Complete

Confirm the final handover.

---

# 7. AI Request Flow

~~~text
User Input
    ↓
Text / Voice / Image
    ↓
AI Processing
    ↓
Structured Spare-Part Data
    ↓
Confidence Check
    ↓
User Confirmation
    ↓
Search
~~~

## AI Fields

- Brand
- Model
- Part name
- Category
- Quantity
- Urgency
- Optional description

## Low Confidence

~~~text
Confidence is low
↓
Show possible interpretation
↓
Ask user to confirm/correct
↓
Continue only after confirmation
~~~

AI assists the user and must not silently make critical assumptions.

---

# 8. Matching Flow

~~~text
Required Part
↓
Inventory Search
↓
Exact / Compatible Match
↓
Availability Check
↓
Distance Filter
↓
Shop Status
↓
Result Ranking
~~~

| Factor | Purpose |
|---|---|
| Part match quality | Determines whether inventory fits |
| Availability | Filters inventory not listed as available |
| Distance | Supports local-first discovery |
| Quantity | Indicates likely quantity coverage |
| Shop availability | Helps identify responsive shops |
| Response history | Optional future ranking signal |
| Urgency | Can influence ordering of relevant results |

The MVP should avoid unnecessary algorithmic complexity.

---

# 9. Search Results Experience

A result should show:

~~~text
Shop Name
Distance
Part
Compatibility
Quantity
Approximate Price, when provided
Shop Type
Availability Status
Response Indicator
Request Button
~~~

### No Shops Found

Show no match in the current search and allow the user to expand the search, modify the requirement, or save it.

### One Shop Found

Show the result clearly and allow a request.

### Multiple Shops Found

Show relevant matches using the MVP ranking logic.

### Shop Becomes Unavailable

Update the result and allow the user to return to other matches.

### Supplier Declines

Show the decline and return the user to available matches.

---

# 10. Request Status System

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
EXPIRED
CANCELLED
NO_MATCH
~~~

| Status | Meaning |
|---|---|
| DRAFT | Request started but not complete |
| ANALYZING | AI is interpreting |
| CONFIRMED | User confirmed/corrected |
| SEARCHING | Inventory search running |
| MATCHED | Potential shop match exists |
| REQUEST_SENT | Supplier request sent |
| ACCEPTED | Supplier confirmed availability |
| HANDOVER_PENDING | Exchange is being arranged |
| COMPLETED | Receipt/handover complete |
| DECLINED | Supplier rejected |
| EXPIRED | Request remained unanswered beyond the defined window |
| CANCELLED | Requesting shop cancelled |
| NO_MATCH | No suitable inventory found |

---

# 11. Notification Flow

## Requesting Shop

- Match found
- Supplier accepted
- Supplier declined
- Handover confirmed
- Transaction completed

## Supplying Shop

- New request
- Pending request reminder
- Request cancelled
- Transaction confirmation

Notifications should remain useful and minimal.

---

# 12. Error and Edge Case Flows

## Case A: AI Cannot Identify the Part

**Problem:** Input is too unclear.

**System response:** Ask for clearer information.

**User choice:** Edit text, provide another input, or retry.

**Recovery:** Analyze again and confirm before searching.

## Case B: No Shop Within 10 km

**Problem:** No suitable participant is found in the configured local radius.

**System response:** Show no match in the current radius.

**User choice:** Expand search, modify request, or save it.

**Recovery:** Search again.

## Case C: No Shop Anywhere in the Network

**Problem:** No suitable participating inventory exists.

**System response:** Show NO_MATCH.

**User choice:** Modify or save the request.

**Recovery:** Revisit later if new inventory appears.

## Case D: Supplier Declines

**Problem:** Selected supplier cannot provide the part.

**System response:** Mark declined and notify the requesting shop.

**User choice:** Select another match.

**Recovery:** Return to search results.

## Case E: Supplier Inventory Is Outdated

**Problem:** Listed inventory is no longer physically available.

**System response:** Supplier rejects or updates availability.

**User choice:** Try another result.

**Recovery:** Correct inventory data and continue.

## Case F: Two Shops Accept Simultaneously

**Problem:** More than one supplier accepts one request.

**System response:** Prevent conflicting final assignment.

**User choice:** Select one confirmed supplier.

**Recovery:** Close the unused accepted response.

## Case G: Incomplete Information

**Problem:** Required information is missing.

**System response:** Identify missing fields.

**User choice:** Complete or correct them.

**Recovery:** Continue after sufficient information exists.

## Case H: Network/API Failure

**Problem:** Backend, AI, location, or another service fails.

**System response:** Show a clear error and preserve entered information where possible.

**User choice:** Retry or return.

**Recovery:** Reuse saved request state or controlled demo data when appropriate.

---

# 13. Customer Impact Flow

~~~text
Customer arrives
↓
Repair diagnosis
↓
Required part unavailable locally
↓
Shop uses SpareLink
↓
Nearby match found
↓
Part obtained
↓
Repair continues
↓
Customer receives service
~~~

SpareLink is intended to reduce unnecessary waiting caused by fragmented local inventory. It does not guarantee a match, exchange, or specific repair time.

---

# 14. MVP Screen List

| Screen | Purpose | Primary User | Main Action |
|---|---|---|---|
| Landing | Explain SpareLink | All | Login / Register |
| Login / Register | Authenticate | Shop | Enter / create account |
| Shop Setup | Create profile | Shop | Save profile |
| Dashboard | Main entry point | Shop | Find Part / Inventory / Requests |
| Inventory | Manage local stock | Shop | Add / edit |
| Add Part | Add inventory item | Shop | Save part |
| Find Part | Start request | Requesting Shop | Enter requirement |
| AI Analysis | Show interpretation | Requesting Shop | Review |
| Confirm Request | Confirm/correct | Requesting Shop | Confirm |
| Search Results | Show potential matches | Requesting Shop | View match |
| Shop Details | Inspect selected match | Requesting Shop | Send request |
| Request Details | Track state | Both | Respond / track |
| Notifications | Show updates | Shop | Open |
| Transaction History | Show completed exchanges | Shop | Review |
| Profile | Account/shop information | Shop | Edit |

For a small team, Login/Register, Find Part/AI Analysis, Request Details/status, and Inventory/Add Part can be combined.

---

# 15. Navigation Structure

~~~text
Dashboard
├── Find Part
├── Inventory
├── Requests
├── Transactions
└── Profile
~~~

The dashboard should provide quick access to the core actions. Labels should remain simple and understandable to shopkeepers with limited technical experience.

---

# 16. Hackathon Demo Flow

The ideal demonstration is **60–90 seconds** and shows only the core loop.

## Scenario

A customer arrives at Shop A with a damaged Samsung Galaxy M14. Shop A does not have the required display in inventory.

## Demo Sequence

1. Shop A logs in.
2. Opens Find a Spare Part.
3. Enters or uploads the requirement.
4. AI identifies the likely part.
5. Shopkeeper confirms.
6. SpareLink searches nearby inventory.
7. Matching shops appear.
8. Shop A opens a match.
9. Shop A sends a request.
10. Shop B receives the request.
11. Shop B accepts.
12. SpareLink shows the confirmed match.
13. Handover is recorded.
14. Transaction becomes completed.

## Presenter Screen Sequence

| Step | What to Show |
|---|---|
| 1 | Shop A dashboard |
| 2 | Find a Spare Part |
| 3 | Requirement input |
| 4 | AI interpretation |
| 5 | Confirmation |
| 6 | Nearby search |
| 7 | Matching shops with distance |
| 8 | Selected shop |
| 9 | Request action |
| 10 | Shop B request |
| 11 | Accept action |
| 12 | Accepted status |
| 13 | Handover state |
| 14 | Completed transaction |

Core story:

~~~text
NEED
 ↓
UNDERSTAND
 ↓
FIND
 ↓
CONNECT
 ↓
EXCHANGE
~~~

---

# 17. UX Principles

## Minimal Typing

Avoid long forms for common requests.

## Mobile-First

Design the workflow for phone use in an active repair-shop environment.

## Fast Request Creation

Keep the path from Find Part to Search short.

## Clear Status Information

Users should always understand whether a request is searching, matched, waiting, accepted, pending handover, or completed.

## Human Confirmation for AI

AI output is visible and editable before it drives the search.

## Trust and Transparency

Potential availability and confirmed availability must be visibly different.

## Simple Language

Prefer labels such as Find Part, Request Part, Accept, Decline, Confirm, and Completed.

## Failure Recovery

Each major failure should offer a useful next action.

---

# 18. Product Flow Summary

~~~text
INPUT
 ↓
UNDERSTAND
 ↓
CONFIRM
 ↓
SEARCH
 ↓
MATCH
 ↓
CONNECT
 ↓
EXCHANGE
 ↓
COMPLETE
~~~

**INPUT:** The shopkeeper provides the requirement.

**UNDERSTAND:** SpareLink interprets the request.

**CONFIRM:** The shopkeeper confirms or corrects the interpretation.

**SEARCH:** Participating inventory is searched.

**MATCH:** Potentially suitable nearby shops are identified.

**CONNECT:** A request is sent and accepted or declined.

**EXCHANGE:** Pickup or local delivery is arranged.

**COMPLETE:** Receipt is confirmed and the transaction is stored.

---

# 19. Final User Journey Definition

## SPARELINK CORE USER JOURNEY

### Primary Goal

Help a repair shop find a potentially available spare part from a nearby participating repair business and complete the exchange.

### Starting Point

A repair shop discovers that its own inventory does not contain the required part.

### Core Flow

~~~text
Shop opens SpareLink
   ↓
Creates spare-part request
   ↓
AI understands requirement
   ↓
User confirms / corrects
   ↓
Nearby inventory is searched
   ↓
Potential matches are displayed
   ↓
Requesting shop selects a match
   ↓
Request is sent
   ↓
Supplying shop checks actual inventory
   ↓
Supplier accepts
   ↓
Handover / local delivery arranged
   ↓
Part received
   ↓
Transaction completed
~~~

### Failure Recovery

~~~text
AI Uncertain
   → User corrects information

No Match
   → Expand search / save request

Supplier Declines
   → Try another match

Outdated Inventory
   → Supplier confirms unavailability / try another match

Incomplete Input
   → Complete required information

Network/API Failure
   → Retry / preserve request state
~~~

### Final Outcome

The requesting shop either completes a confirmed local exchange or receives a clear failure state with an actionable recovery path.

## Final Stage Table

| Stage | User | Action | System Response | Result |
|---|---|---|---|---|
| Landing | Shop | Opens platform | Shows entry actions | Ready |
| Registration/Login | Shop | Authenticates | Verifies account | App access |
| Shop Setup | Shop | Saves profile | Stores shop data | Profile ready |
| Location | Shop | Provides location | Stores matching location | Local search enabled |
| Inventory | Shop | Adds parts | Stores inventory | Parts searchable |
| Request | Requesting Shop | Enters requirement | Creates draft | Request started |
| AI Analysis | Requesting Shop | Reviews interpretation | Extracts fields | Requirement understood |
| Confirmation | Requesting Shop | Confirms/corrects | Stores criteria | Search ready |
| Search | System | Searches inventory | Finds candidates | Results generated |
| Matching | System | Ranks results | Considers match/location/status | Matches displayed |
| Shop Details | Requesting Shop | Views match | Shows shop and part | Match selected |
| Request Sent | Requesting Shop | Sends request | Creates supplier request | Waiting |
| Request Review | Supplying Shop | Opens request | Shows details | Stock check |
| Accept/Decline | Supplying Shop | Responds | Updates status | Accepted/declined |
| Match Confirmation | Both | Review match | Records confirmed availability | Exchange can proceed |
| Handover | Both | Arrange exchange | Records handover | Part moves |
| Completion | Requesting Shop | Confirms receipt | Marks complete | Exchange completed |
| History | Both | View transaction | Shows record | History updated |

> **Status: Part 2 Complete | Core User Journey Defined**
