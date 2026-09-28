# SpareLink | UI/UX & Screen Design

> **The missing part is just one message away.**

Part 3 defines the complete UI/UX specification for the SpareLink MVP, using Parts 1 and 2 as the product foundation.

The product should feel **professional, trustworthy, modern, fast, practical, mobile-first, and easy for small-business owners to understand**.

---

# 1. PRODUCT DESIGN DIRECTION

SpareLink should use a clean operational interface with strong hierarchy, practical cards, clear actions, and restrained visual effects.

Avoid unnecessary animations, complicated navigation, decorative elements that do not improve usability, and unexplained technical language.

### Core Design Message

> **Find the part you need quickly.**

Every important screen should make the next useful action obvious.

---

# 2. DESIGN PERSONALITY

## Visual Personality

SpareLink should feel like a focused business tool rather than a generic marketplace or social app.

Priority information:

1. Required part
2. Match quality
3. Availability state
4. Distance
5. Next action

## Brand Character

**Practical · Local · Trusted · Fast · Clear · Helpful**

## Emotional Goal

The shopkeeper should feel that SpareLink reduces uncertainty during a sourcing task and clearly shows what to do next.

## Central Design Principle

> **Every screen should move the shopkeeper one step closer to finding, confirming, or exchanging the required part.**

---

# 3. INFORMATION ARCHITECTURE

~~~text
SPARELINK
│
├── Dashboard
├── Find Spare Part
│   ├── Input
│   ├── AI Analysis
│   ├── Confirm Request
│   ├── Search
│   ├── Results
│   ├── Shop Details
│   └── Request Status
├── Inventory
│   └── Add / Edit Part
├── Requests
│   ├── Sent
│   ├── Received
│   └── Active
├── Transactions
├── Notifications
└── Profile
~~~

## Main Navigation

- Dashboard
- Find Part
- Inventory
- Requests
- Transactions
- Profile

Notifications should be globally accessible and can use a badge instead of taking a permanent full navigation slot on small screens.

## Primary CTA

> **Find a Spare Part**

## Mobile Navigation

~~~text
[Home] [Find Part] [Requests] [Inventory] [Profile]
~~~

---

# 4. SCREEN INVENTORY

| # | Screen | User | Purpose | Priority |
|---|---|---|---|---|
| 1 | Landing | All | Explain SpareLink | MUST HAVE |
| 2 | Login | Shop | Authenticate | MUST HAVE |
| 3 | Register | Shop | Create account | MUST HAVE |
| 4 | Shop Setup | Shop | Create business profile | MUST HAVE |
| 5 | Dashboard | Shop | Main operating hub | MUST HAVE |
| 6 | Find Part | Requesting Shop | Start a spare-part request | MUST HAVE |
| 7 | AI Analysis / Confirm | Requesting Shop | Review AI interpretation | MUST HAVE |
| 8 | Search / Results | Requesting Shop | Find local matches | MUST HAVE |
| 9 | Shop Details | Requesting Shop | Review a potential supplier | MUST HAVE |
| 10 | Request Details | Both | Track/respond | MUST HAVE |
| 11 | Inventory | Shop | Manage stock | MUST HAVE |
| 12 | Add / Edit Part | Shop | Create/update stock | MUST HAVE |
| 13 | Requests | Both | View requests | MUST HAVE |
| 14 | Transactions | Both | View completed exchanges | MUST HAVE |
| 15 | Notifications | Both | Important updates | SHOULD HAVE |
| 16 | Profile / Settings | Shop | Manage account/shop | SHOULD HAVE |
| 17 | Dedicated Voice Flow | Shop | Voice-first experience | FUTURE |
| 18 | Advanced Analytics | Shop | Detailed analytics | FUTURE |

Related screens should be combined where practical for the hackathon MVP.

---

# 5. LANDING PAGE

## Header

### Desktop

- SpareLink logo/name
- How It Works
- Login
- Register

### Mobile

- SpareLink logo/name
- Compact authentication controls
- Minimal navigation

## Hero

### Headline

> **Find the spare part you need from nearby repair shops.**

### Supporting Text

> SpareLink helps repair shops discover potentially available spare parts from nearby participating businesses and coordinate a local request.

### Primary CTA

> **Find a Spare Part**

### Secondary CTA

> **How It Works**

## Problem Section

~~~text
Part required
    ↓
Not in own inventory
    ↓
Manual calls / supplier search
    ↓
Local inventory is difficult to discover
~~~

## How SpareLink Works

~~~text
Request
   ↓
AI Understands
   ↓
Find Nearby
   ↓
Connect
~~~

## Benefits

- Discover potential local inventory
- Reduce repeated manual searching
- Coordinate repair-to-repair requests
- Keep availability confirmation visible

## Supported Business Types

**Initial focus:** Mobile Repair Shops

**Future scope:** Bike Repair, Car Workshops, Appliance Repair, Electrical Technicians, Electronics Repair, Machinery Workshops.

## Footer

Include:

- SpareLink name
- Short description
- Project/hackathon label where appropriate
- Project/GitHub link
- Basic navigation

Do not add unsupported partnerships, customer counts, or market claims.

---

# 6. AUTHENTICATION

## Login

### Fields

- Phone/email identifier
- Password

### Actions

> **Login**

Secondary actions:

- Forgot Password
- Create Account

### Error

> **We couldn't sign you in. Check your details and try again.**

## Registration

### Fields

- Name
- Phone/email
- Password
- Confirm password

Then continue to shop registration.

## Forgot Password

~~~text
Enter registered contact
      ↓
Send recovery instruction
      ↓
Reset password
~~~

## Shop Registration

| Field | Required | Reason |
|---|---|---|
| Shop name | Yes | Business identification |
| Owner name | Yes | Basic identity |
| Phone number | Yes | Business coordination |
| Shop category | Yes | Repair category |
| Address/area | Yes | Shop information |
| Location | Yes | Nearby matching |
| Shop description | No | Optional context |

---

# 7. SHOP ONBOARDING

~~~text
Create Account
      ↓
Create Shop Profile
      ↓
Set Location
      ↓
Choose Shop Category
      ↓
Add Initial Inventory
      ↓
Enter Dashboard
~~~

Keep onboarding short by using one focused task per step.

Use simple progress information such as **Step 2 of 5**.

Do not force optional fields or future features during onboarding.

---

# 8. DASHBOARD

The dashboard is the shopkeeper's operating home.

## Primary Action

> **Find a Spare Part**

## Useful Information

- Inventory count
- Active requests
- Incoming requests
- Recent transactions
- Nearby activity when available

## Recommended Hierarchy

~~~text
Header
   ↓
Find a Spare Part
   ↓
Active Requests
   ↓
Inventory Summary
   ↓
Recent Transactions
~~~

### Mobile

Show the primary action, active requests, inventory summary, and recent activity in that order.

### Desktop

Use a sidebar plus a wider main content area.

---

# 9. FIND A SPARE PART

This is the most important screen in SpareLink.

## Text Input

Example:

> “Samsung M14 display kavali urgent ga.”

Primary action:

> **Analyze Request**

## Image Input

Actions:

> **Upload Photo**

> **Take Photo**

## Voice Input

Action:

> **Speak Request**

## Suggested Layout

~~~text
What spare part do you need?

[ Type your request... ]

[ Photo ] [ Voice ]

[ Analyze Request ]
~~~

Recent searches are optional and should only appear when genuinely useful.

---

# 10. AI ANALYSIS SCREEN

### Loading Copy

> **Analyzing your request...**

Meaningful processing labels can include:

- Understanding spare part
- Processing image
- Preparing search

Do not show fake percentage progress.

## Interpreted Result

~~~text
Brand: Samsung
Model: Galaxy M14
Part: Display
Quantity: 1
Urgency: High
~~~

Actions:

> **Confirm**

> **Edit**

Supporting copy:

> **AI suggestion. Please verify the details before searching.**

## Low Confidence

~~~text
We may need more information.

Possible match:
Samsung Galaxy M14 Display

[ Edit Details ]
[ Try Again ]
~~~

The user must be able to correct the interpretation before search.

---

# 11. REQUEST CONFIRMATION

Show:

| Field | Example |
|---|---|
| Part | Display |
| Brand | Samsung |
| Model | Galaxy M14 |
| Quantity | 1 |
| Urgency | High |
| Search radius | 10 km |
| Additional notes | Optional |

Primary action:

> **Find Nearby Parts**

Secondary action:

> **Edit Request**

Do not begin the search automatically when AI finishes. Require explicit confirmation.

---

# 12. MATCHING / SEARCH SCREEN

Loading copy:

> **Searching nearby SpareLink shops...**

Display:

- Search radius
- Current search state
- Match count only when actually known

Example:

~~~text
Searching within 10 km

Checking participating inventory...
~~~

Do not use invented progress percentages.

---

# 13. SEARCH RESULTS SCREEN

Example result:

~~~text
Sri Sai Mobiles
2.4 km

Samsung M14 Display
1 listed

Approx. ₹1,850

Mobile Repair
Potentially Available

[ Request Part ]
~~~

## Result Fields

| Field | Purpose |
|---|---|
| Shop Name | Identifies supplier |
| Distance | Supports local-first decision |
| Part Availability | Shows matching inventory |
| Quantity | Shows listed quantity |
| Price, if provided | Helps transaction planning |
| Shop Category | Adds business context |
| Availability Status | Shows inventory state |
| Request Action | Starts coordination |

## Match States

### Exact Match

Use when stored inventory directly corresponds to the confirmed request.

### Possible / Compatible Match

Use when the part may satisfy the requirement but needs supplier confirmation.

### Unavailable

> **Currently Unavailable**

Do not present it as an active request option.

### Pending Confirmation

> **Awaiting Supplier Confirmation**

### Critical Rule

> **Listed availability is not confirmed availability.**

---

# 14. SHOP DETAILS

Show only transaction-relevant information:

- Shop name
- Distance
- Shop category
- Matching part
- Quantity
- Approximate price when provided
- Location/area
- Contact option
- Request action

## Trust Signals

~~~text
Inventory updated: [timestamp]
Availability: Listed
~~~

After supplier confirmation:

~~~text
Availability: Confirmed
~~~

Do not add an unvalidated ratings system to the MVP.

---

# 15. REQUEST SCREEN

Before sending:

~~~text
Requested Part
Samsung Galaxy M14 Display

Requesting Shop
Kumar Mobiles

Quantity
1

Urgency
High

Distance
2.4 km

Optional Note
...
~~~

Primary action:

> **Send Request**

After sending:

> **Request Sent**

Show:

~~~text
Request Sent
Waiting for supplier response
~~~

---

# 16. SUPPLIER REQUEST SCREEN

~~~text
NEW SPARE REQUEST

Samsung Galaxy M14 Display

Requested by:
Kumar Mobiles

Distance:
2.4 km

Quantity:
1

Urgency:
HIGH

[ Accept ]
[ Decline ]
~~~

After accepting, confirm:

- Quantity
- Price if applicable
- Actual availability
- Handover/delivery method

The supplier should verify physical stock before accepting.

---

# 17. REQUEST STATUS UI

Use these states:

~~~text
Draft
Analyzing
Confirmed
Searching
Matched
Request Sent
Accepted
Handover Pending
Completed
Declined
Cancelled
Expired
No Match
~~~

| Status | Meaning | Available Action |
|---|---|---|
| Draft | Request not submitted | Continue editing |
| Analyzing | AI processing | Wait/retry |
| Confirmed | User verified request | Start search |
| Searching | Inventory search running | Wait |
| Matched | Potential match found | View match |
| Request Sent | Supplier request sent | Wait/cancel |
| Accepted | Supplier confirmed | Arrange handover |
| Handover Pending | Exchange being arranged | Confirm receipt |
| Completed | Transaction finished | View history |
| Declined | Supplier rejected | Try another |
| Cancelled | Request cancelled | Create new request |
| Expired | Response window ended | Try another |
| No Match | No suitable inventory | Expand/save |

### Visual Rule

Every status must have visible text. Color must never be the only status signal.

---

# 18. INVENTORY SCREEN

Inventory supports:

- Part name
- Brand
- Model
- Category
- Quantity
- Price
- Condition
- Availability
- Last updated

Actions:

> **Add Part**

> **Edit**

> **Remove**

> **Update Quantity**

Example:

~~~text
Samsung M14 Display
Samsung · Galaxy M14
Qty: 1
Condition: New
Availability: Available
Updated: Today

[ Edit ]
~~~

Use cards on mobile and a table on larger screens.

---

# 19. ADD PART SCREEN

Fields:

~~~text
Part Name
Brand
Model
Category
Quantity
Price
Condition
Notes
~~~

### Minimum Searchable Information

- Part name
- Model/compatibility where applicable
- Quantity
- Availability

### Validation

- Quantity must be valid.
- Required text cannot be blank.
- Model information should be captured when compatibility depends on it.
- Price must use the selected valid format when entered.
- Failed image uploads should support retry.

Primary action:

> **Save Part**

---

# 20. REQUESTS SCREEN

Use one page with simple filters/tabs:

~~~text
Sent | Received | Active | Completed
~~~

Each request card shows:

- Part
- Other shop
- Distance when relevant
- Date/time
- Quantity
- Status
- Next action

### Sent Example

~~~text
Samsung M14 Display
To: Kumar Mobiles
Status: Request Sent

[ View Request ]
~~~

### Received Example

~~~text
Samsung M14 Display
From: Kumar Mobiles
Status: New Request

[ Review ]
~~~

---

# 21. TRANSACTIONS SCREEN

Display:

- Transaction ID
- Part
- From shop
- To shop
- Date
- Quantity
- Amount if applicable
- Status

Example:

~~~text
TXN-00124
Samsung M14 Display
Shop B → Shop A
28 Sep 2026
Qty: 1
Status: Completed
~~~

Provide a simple detail view.

---

# 22. NOTIFICATIONS

Only important events should generate notifications:

~~~text
New spare request
Match found
Request accepted
Request declined
Request cancelled
Transaction completed
~~~

A notification should answer:

1. What happened?
2. Which request is affected?
3. What can I do next?

Urgent requests can be surfaced more prominently without creating unnecessary notification noise.

---

# 23. PROFILE & SHOP SETTINGS

Include:

- Shop information
- Contact details
- Business category
- Location
- Notification preferences
- Account settings

Keep the MVP settings minimal.

---

# 24. EMPTY STATES

## No Inventory

> **Your inventory is empty.**

> Add the parts your shop currently has so nearby shops can discover them.

CTA:

> **Add Part**

## No Requests

> **No requests yet.**

> Spare-part requests you send or receive will appear here.

CTA:

> **Find a Spare Part**

## No Transactions

> **No completed transactions yet.**

> Completed spare-part exchanges will appear here.

## No Nearby Matches

> **No matching shops found nearby.**

> Try expanding the search area or changing the requirement.

CTAs:

> **Expand Search**

> **Edit Request**

## No Notifications

> **You're all caught up.**

Do not use generic no-data messages.

---

# 25. ERROR STATES

| Error | User Sees | Next Action |
|---|---|---|
| AI failure | We couldn't analyze that request. | Try Again |
| Image upload failure | We couldn't upload the image. Try again or continue with text. | Retry / Use Text |
| Voice failure | We couldn't capture the voice request. | Try Again / Type |
| Network failure | Connection problem. Please try again. | Retry / Go Back |
| Search failure | We couldn't complete the nearby search. | Try Again |
| Location unavailable | We need your shop location to find nearby matches. | Set Location |
| Inventory unavailable | This inventory information could not be confirmed. | Try Another Match |
| Request submission failure | Your request wasn't sent. | Retry Request |

Error messages should be human-readable and always provide a recovery action.

---

# 26. MOBILE-FIRST DESIGN

SpareLink should prioritize phone use and one-handed interaction where practical.

## Mobile

Use:

- Single-column layouts
- Large primary CTA
- Compact cards
- Short forms
- Bottom navigation
- Large touch targets
- Visible primary action

### Find-Part Layout

~~~text
Header
   ↓
Request Input
   ↓
Photo / Voice
   ↓
Analyze
   ↓
AI Result
   ↓
Confirm
~~~

### Results

Use full-width stacked cards. Avoid side-by-side comparison.

## Desktop

Use:

- Sidebar navigation
- Wider dashboard
- Larger search/result panels
- Optional list/map split when implemented
- Wider request details

---

# 27. RESPONSIVE DESIGN RULES

| Component | Mobile | Tablet/Desktop |
|---|---|---|
| Navigation | Bottom nav | Sidebar/top nav |
| Cards | Full-width stacked | Grid/horizontal |
| Search | Stacked controls | Horizontal groups |
| Forms | One column | Two columns for related fields where useful |
| Tables | Cards or limited horizontal scroll | Full table |
| Maps | Secondary | Optional side-by-side |
| Modals | Full-screen/bottom sheet | Centered dialog |
| Buttons | Full-width primary where practical | Compact |
| Images | Container width | Bounded larger container |

The design should reflow instead of simply shrinking.

---

# 28. DESIGN SYSTEM

Keep the design system small enough for fast implementation.

## Typography

~~~text
H1 = Page/hero heading
H2 = Section heading
H3 = Card/feature heading
Body = Main information
Label = Field/status label
Supporting = Helper text
~~~

Suggested direction:

| Type | Size |
|---|---:|
| H1 | 28–40 px |
| H2 | 24–30 px |
| H3 | 18–22 px |
| Body | 15–17 px |
| Label | 13–14 px |
| Supporting | 12–14 px |

Use a readable modern sans-serif font.

## Colors

Use a restrained palette with roles for:

- Primary
- Primary Dark
- Background
- Surface
- Text
- Muted Text
- Border
- Success
- Warning
- Error
- Info

## Buttons

### Primary

Find a Spare Part, Confirm, Send Request, Accept, Save Part.

### Secondary

Edit, Back, View Details.

### Destructive

Remove and Decline where appropriate.

### Disabled

Clearly show that the action cannot currently be used.

## Inputs

Support:

- Default
- Focus
- Error
- Disabled

Use visible labels rather than placeholder-only labels.

## Status Indicators

Use text plus an icon/shape where useful:

~~~text
● Confirmed
~~~

Color can reinforce the meaning but never replace the status text.

---

# 29. ACCESSIBILITY

Requirements:

- Readable text
- Adequate contrast
- Comfortable touch targets
- Keyboard navigation on desktop
- Meaningful form labels
- Human-readable errors
- Accessible labels for icons
- Text alternatives for important images
- Visible focus states
- Logical heading structure
- No color-only status communication
- No icon-only critical communication

Camera, microphone, notification, and navigation controls need accessible names.

---

# 30. MICROCOPY

| Situation | Copy |
|---|---|
| Main CTA | **Find a Spare Part** |
| Search CTA | **Find Nearby Parts** |
| AI confirmation | **Review the AI suggestion before searching.** |
| No match | **No matching shops found nearby.** |
| Request sent | **Request sent. Waiting for supplier response.** |
| Accepted | **Request accepted. Availability confirmed by the supplier.** |
| Declined | **This supplier declined the request. Try another match.** |
| Completed | **Transaction completed. Part receipt confirmed.** |
| AI error | **We couldn't analyze that request. Try again.** |
| Search error | **We couldn't complete the nearby search. Try again.** |
| Location error | **Set your shop location to find nearby matches.** |
| Network error | **Connection problem. Please try again.** |
| Upload error | **We couldn't upload the image. Try again or use text.** |
| Voice error | **We couldn't capture the voice request. Try again or type it.** |
| Submission error | **Your request wasn't sent. Please try again.** |

---

# 31. HACKATHON DEMO UI

Scenario:

~~~text
Shop A needs Samsung M14 Display
        ↓
Upload image
        ↓
AI identifies part
        ↓
Shopkeeper confirms
        ↓
Nearby matches appear
        ↓
Request sent
        ↓
Supplier accepts
        ↓
Transaction completed
~~~

## Presenter Sequence

### 1. Dashboard

Show:

> **Find a Spare Part**

### 2. Find Part

Enter/upload the requirement.

### 3. AI Analysis

Show:

~~~text
Samsung
Galaxy M14
Display
Urgency: High
~~~

Click:

> **Confirm**

### 4. Results

Show controlled demo matches with distances.

Mark inventory as potential/listed until supplier confirmation.

### 5. Request

Open the selected shop and click:

> **Request Part**

### 6. Supplier

Switch to supplier view and show:

> **NEW SPARE REQUEST**

Click:

> **Accept**

### 7. Completion

Show:

~~~text
Request Sent
      ↓
Accepted
      ↓
Handover Pending
      ↓
Completed
~~~

Avoid unrelated screens during the live demo.

---

# 32. TRUST & TRANSPARENCY UX

## MVP Trust Features

- Shop profile
- Inventory last-updated timestamp
- Availability status
- Supplier confirmation
- Request status
- Transaction history

Example:

~~~text
Samsung M14 Display

Listed availability
Updated: Today

Supplier confirmation:
Pending
~~~

After confirmation:

~~~text
Availability:
Confirmed by supplier
~~~

## Future Trust Features

- Business verification
- Ratings/reputation
- Reliability indicators
- More detailed transaction history

These are future scope and must not be presented as current MVP features.

---

# 33. UI COMPONENT INVENTORY

| Component | Purpose |
|---|---|
| Navbar | Desktop/global navigation |
| Sidebar | Desktop navigation |
| MobileNavigation | Mobile primary navigation |
| Button | Main interaction |
| Input | Form input |
| SearchBar | Search/filter |
| UploadBox | Image input |
| VoiceInput | Voice input |
| AIAnalysisCard | AI interpretation |
| PartSummaryCard | Confirmed requirement |
| ShopCard | Shop information |
| MatchCard | Potential supplier result |
| RequestCard | Request summary/status |
| StatusBadge | Status label |
| InventoryTable | Desktop inventory |
| TransactionCard | Transaction history |
| NotificationItem | Notification |
| Modal | Confirmation/dialog |
| Toast | Feedback |
| EmptyState | Zero-data state |
| ErrorState | Recoverable error |
| LoadingState | Processing state |

Shared components should use the same spacing, typography, button, and status patterns.

---

# 34. FINAL SCREEN MAP

~~~mermaid
flowchart TD
    A[Landing] --> B[Login / Register]
    B --> C[Shop Setup]
    C --> D[Dashboard]

    D --> E[Find Part]
    E --> F[AI Analysis]
    F --> G[Confirm Request]
    G --> H[Search Results]
    H --> I[Shop Details]
    I --> J[Send Request]
    J --> K[Request Status]
    K --> L[Completed]

    D --> M[Inventory]
    M --> N[Add / Edit Part]

    D --> O[Requests]
    D --> P[Transactions]
    D --> Q[Profile]
    D --> R[Notifications]
~~~

### Core Request Branch

~~~text
Dashboard
   ↓
Find Part
   ↓
AI Analysis
   ↓
Confirm
   ↓
Search Results
   ↓
Shop Details
   ↓
Send Request
   ↓
Request Status
   ↓
Completed
~~~

---

# 35. FINAL MVP UI SPECIFICATION

## SPARELINK MVP UI SPECIFICATION

### Core Screens

1. Landing
2. Login/Register
3. Shop Setup
4. Dashboard
5. Find Part
6. AI Analysis / Confirmation
7. Search Results
8. Shop Details
9. Request Details / Status
10. Inventory
11. Add/Edit Part
12. Requests
13. Transactions
14. Notifications
15. Profile

Related screens can be combined where useful.

### Primary User Flow

~~~text
Dashboard
   ↓
Find Part
   ↓
Input
   ↓
AI Analysis
   ↓
Confirm
   ↓
Search
   ↓
Match Results
   ↓
Shop Details
   ↓
Send Request
   ↓
Supplier Accepts
   ↓
Handover
   ↓
Completed
~~~

### Primary CTA

> **Find a Spare Part**

### Design Principle

> **Every screen should make the next useful action obvious while keeping part identity, availability, distance, and request status clear.**

### Hackathon Demo Path

~~~text
Need Samsung M14 Display
        ↓
Input request
        ↓
AI identifies part
        ↓
Confirm
        ↓
Nearby matches
        ↓
Request Part
        ↓
Supplier Accepts
        ↓
Transaction Completed
~~~

### Developer Handoff

The team now has enough UI/UX definition to begin implementation of:

- Information architecture
- Main navigation
- MVP screen set
- Screen hierarchy
- Primary CTA placement
- Request input methods
- AI confirmation behavior
- Search-result structure
- Supplier request handling
- Request statuses
- Inventory fields
- Transaction information
- Empty states
- Error states
- Responsive behavior
- Lightweight design system
- Accessibility baseline
- Hackathon demo flow
- Reusable component inventory

The next implementation stages can now focus on technical architecture, database design, backend behavior, AI engine integration, and matching logic.

> **Status: Part 3 Complete | MVP UI/UX Specification Defined**
