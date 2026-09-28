# SpareLink | Matching Engine

> **Part 8 of the SpareLink MVP documentation.** This document defines the core matching engine that converts a confirmed spare-part request into a ranked set of nearby shops that may be able to supply the required part.

## 1. Matching Engine Objective

The Matching Engine is the decision layer between a normalized spare request and the shops that can potentially fulfill it.

Its core responsibility is simple:

> **Find the right spare part, from the right nearby shop, with the right availability, and return the most useful matches first.**

The engine must not blindly return the nearest shop. Distance is only one factor.

A useful match considers:

- Part identity.
- Brand and model compatibility.
- Part category.
- Current inventory availability.
- Quantity available.
- Shop status.
- Geographic distance.
- Estimated pickup/travel time.
- Seller/shop reliability signals.
- Request urgency.
- Optional shop preferences.
- Inventory freshness.
- Confidence and uncertainty.

The engine should prefer deterministic database rules for inventory and availability while using AI only where semantic interpretation is required.

---

## 2. Position in the SpareLink Architecture

The complete request path is:

```text
User
  ↓
Frontend
  ↓
Backend API
  ↓
AI Engine
  ↓
Structured + Confirmed Spare Request
  ↓
Matching Engine
  ↓
Candidate Retrieval
  ↓
Eligibility Filtering
  ↓
Compatibility Check
  ↓
Match Scoring
  ↓
Ranking
  ↓
Availability Re-check
  ↓
Match Results
  ↓
User / Requester
```

The Matching Engine therefore sits after Part 7's AI interpretation layer.

### Important boundary

The AI Engine answers:

> "What part is the user asking for?"

The Matching Engine answers:

> "Which shops can potentially provide that part right now?"

The database answers:

> "What inventory and shop data does SpareLink actually have?"

The backend remains responsible for authorization, transactions, and safe state changes.

---

## 3. Matching Principles

SpareLink follows six core principles.

### 3.1 Exactness before distance

A shop 500 metres away with the wrong part is not a useful match.

A shop 2 kilometres away with the exact confirmed part and available quantity may be useful.

Therefore:

```text
Part Compatibility
      >
Inventory Availability
      >
Shop Eligibility
      >
Distance / Travel
      >
Secondary Ranking Signals
```

### 3.2 Availability must be real

The matching engine must not treat an old inventory record as guaranteed availability.

A result may initially be discovered from indexed inventory, but before fulfillment is created, the backend must re-check the current inventory state.

### 3.3 AI does not control inventory

AI may normalize:

```text
"M14 display"
"Samsung M14 screen"
"M14 LCD"
```

into a candidate canonical representation.

AI must not independently decide that an arbitrary display is compatible with a device.

Compatibility rules and trusted inventory data remain authoritative.

### 3.4 Nearby is contextual

"Nearby" should be calculated from the shop's stored geographic location and the requester's relevant location.

PostGIS is used for geographic filtering and distance calculation.

### 3.5 Ranking must be explainable

The system should be able to explain why a shop appeared near the top.

Example:

```text
Exact part
✓ Available: 2
✓ 1.4 km away
✓ Open now
✓ Compatible model
```

The ranking algorithm should never become an unexplained black box.

### 3.6 Race conditions must be expected

Two repair shops may request the same spare part at almost the same time.

Therefore:

```text
Search result
    ≠
Guaranteed reservation
```

Only a transactional inventory operation can reserve or consume stock.

---

## 4. Matching Engine Inputs

The engine receives a normalized request.

Example:

```json
{
  "request_id": "req_123",
  "brand": "Samsung",
  "model": "Galaxy M14",
  "part_name": "Display",
  "category": "Mobile Display",
  "quantity": 1,
  "urgency": "high",
  "requester_location": {
    "latitude": 17.45,
    "longitude": 78.38
  }
}
```

### Required input fields

| Field | Purpose |
|---|---|
| request_id | Identifies the spare request |
| normalized part identity | Main matching key |
| quantity | Required quantity |
| requester location | Geographic search |
| urgency | Optional ranking modifier |

### Optional input fields

- Brand.
- Model.
- Variant.
- Part category.
- Part condition.
- Preferred radius.
- Notes.
- Compatibility requirements.
- Pickup preference.

---

## 5. Candidate Retrieval

The engine should not score every shop in the database.

Instead, it first retrieves a candidate set.

### Candidate retrieval flow

```text
Normalized Request
      ↓
Geographic Radius Filter
      ↓
Relevant Inventory Filter
      ↓
Active Shop Filter
      ↓
Candidate Shops
```

Example:

A request may begin with a 5 km search radius.

If too few useful candidates are found, the system may expand the search according to configured rules.

```text
0–5 km
   ↓
5–10 km
   ↓
10–20 km
```

Radius expansion should be controlled by configuration, not hard-coded throughout the application.

---

## 6. Geographic Search

SpareLink uses PostgreSQL with PostGIS for location-aware queries.

A shop location can be represented using a geographic point.

Conceptually:

```text
shop.location = POINT(longitude, latitude)
```

The engine calculates:

```text
distance(requester, shop)
```

using geographic distance rather than simple latitude/longitude subtraction.

### Why PostGIS

PostGIS provides:

- Radius searches.
- Accurate geographic distance calculations.
- Spatial indexing.
- Ordering by distance.
- Efficient location-aware queries.

The MVP should avoid loading every shop into application memory just to calculate distance.

---

## 7. Eligibility Filtering

Before scoring, obviously unsuitable candidates should be removed.

A candidate shop should normally satisfy:

```text
Shop is active
AND
Inventory is active
AND
Required part is compatible
AND
Available quantity >= requested quantity
```

Additional rules may include:

- Shop has not disabled receiving requests.
- Inventory item has not expired.
- Shop is not suspended.
- Item is not already fully reserved.
- Shop is inside the allowed search radius.
- Request is still open.

### Example

If the user needs:

```text
Samsung Galaxy M14 Display × 1
```

and a shop has:

```text
Samsung Galaxy M14 Display
available_quantity = 0
```

that shop must not be presented as an available match.

---

## 8. Compatibility Levels

Not every candidate should be treated equally.

SpareLink defines compatibility levels.

### Level A: Exact

All important identifiers match.

```text
Brand ✓
Model ✓
Part ✓
Variant ✓ when required
```

### Level B: Strong compatible

The inventory item is explicitly mapped as compatible with the requested device/model.

### Level C: Possible

The item may be compatible based on known aliases or compatibility rules but needs confirmation.

### Level D: Unknown

The system does not have enough trusted information.

The MVP should prioritize A and B.

C may be shown only with a clear confirmation requirement.

D should not be silently treated as a valid exact match.

---

## 9. Matching Score

After filtering, candidates are scored.

A practical MVP scoring model can be:

```text
Match Score =
  Compatibility Score
+ Availability Score
+ Distance Score
+ Freshness Score
+ Reliability Score
+ Response Score
+ Urgency Adjustment
```

The exact weights should remain configurable.

### Example weighted model

```text
Compatibility      40%
Availability       20%
Distance           15%
Inventory Freshness 10%
Reliability         5%
Response History    5%
Urgency Adjustment  5%
```

These are initial engineering weights, not permanent business rules. They should be validated with real usage data.

---

## 10. Compatibility Score

Compatibility should dominate the score.

Example:

| Compatibility | Score |
|---|---:|
| Exact | 100 |
| Explicitly compatible | 85 |
| Possible / needs confirmation | 50 |
| Unknown | 0 |

A candidate with score 100 should not normally lose to a candidate with score 50 simply because the latter is slightly closer.

---

## 11. Availability Score

Availability represents whether the shop can satisfy the requested quantity.

Example:

```text
Required = 1
Available = 3
```

This is stronger than:

```text
Required = 1
Available = 1
```

However, excessive inventory should not dominate the entire ranking.

A simple normalized score can be used:

```text
availability_score =
min(available_quantity / requested_quantity, 2) / 2 × 100
```

This gives additional confidence to shops with some stock while preventing extremely large inventory from overpowering compatibility.

---

## 12. Distance Score

Distance should reward practical proximity.

For example:

```text
0–1 km     → very high
1–3 km     → high
3–5 km     → medium
5–10 km    → lower
10+ km     → only when radius expansion is allowed
```

The actual score should be calculated continuously rather than through arbitrary hard buckets where possible.

A simple normalized approach is:

```text
distance_score =
max(0, 100 × (1 - distance / maximum_radius))
```

The engine should use actual geographic distance returned by PostGIS.

---

## 13. Inventory Freshness

Inventory becomes dangerous when it is stale.

Example:

```text
Inventory updated 2 minutes ago
→ high freshness

Inventory updated 4 days ago
→ lower confidence
```

Freshness should influence ranking, but stale inventory should also be handled by eligibility rules when it exceeds the configured threshold.

### Example policy

```text
Fresh        → normal candidate
Aging        → lower score
Stale        → confirmation required / exclude
```

The exact thresholds belong in configuration.

---

## 14. Shop Reliability

SpareLink can maintain lightweight operational signals.

Possible signals:

- Successful completed exchanges.
- Cancellation rate.
- Average response time.
- Recent activity.
- Disputed transactions.
- Inventory accuracy.

These should never override a hard compatibility failure.

Reliability is a secondary signal, not permission to sell unavailable or incompatible parts.

---

## 15. Response Behaviour

A shop that regularly responds quickly can be more useful during an urgent repair.

Possible response signal:

```text
fast response history → higher secondary score
slow response history → lower secondary score
unknown history       → neutral
```

This signal should be carefully bounded so that new shops are not permanently disadvantaged.

---

## 16. Urgency Handling

SpareLink supports urgency without destroying the core ranking logic.

Example:

```text
Normal request
→ optimize for compatibility + availability + distance

Urgent request
→ increase importance of nearby, active, responsive shops
```

Urgency must not cause the engine to recommend an incompatible part.

The rule is:

> **Urgency changes ranking, not correctness.**

---

## 17. Final Ranking Pipeline

The complete ranking pipeline is:

```text
1. Receive confirmed request
          ↓
2. Validate request
          ↓
3. Determine search radius
          ↓
4. Retrieve nearby active shops
          ↓
5. Retrieve relevant inventory
          ↓
6. Remove unavailable candidates
          ↓
7. Check compatibility
          ↓
8. Calculate distance
          ↓
9. Calculate freshness
          ↓
10. Calculate reliability signals
          ↓
11. Calculate match score
          ↓
12. Sort candidates
          ↓
13. Limit result count
          ↓
14. Re-check availability
          ↓
15. Return matches
```

---

## 18. Deterministic vs AI Responsibilities

This boundary is critical.

| Responsibility | AI | Deterministic System |
|---|---:|---:|
| Understand natural language | ✓ | |
| Normalize aliases | ✓ | ✓ |
| Extract brand/model | ✓ | ✓ validation |
| Verify inventory count | | ✓ |
| Calculate distance | | ✓ |
| Check shop status | | ✓ |
| Check authorization | | ✓ |
| Final inventory reservation | | ✓ |
| Compatibility from trusted rules | | ✓ |
| Explain natural-language request | ✓ | |
| Rank candidates | optional assistance | ✓ |

The MVP ranking algorithm should remain deterministic and testable.

AI may improve semantic understanding, but it must not secretly rewrite the ranking rules.

---

## 19. Example Matching Scenario

### Request

A repair shop sends:

> "Bro Samsung M14 display kavali urgent ga."

Part 7 converts this into:

```text
Brand: Samsung
Model: Galaxy M14
Part: Display
Quantity: 1
Urgency: High
```

The Matching Engine finds four candidates.

| Shop | Compatibility | Stock | Distance | Status |
|---|---|---:|---:|---|
| Shop A | Exact | 2 | 1.2 km | Open |
| Shop B | Exact | 1 | 0.7 km | Open |
| Shop C | Compatible | 3 | 2.1 km | Open |
| Shop D | Exact | 0 | 0.4 km | Open |

Shop D is removed because available quantity is zero.

The remaining candidates are scored using the configured ranking model.

The user sees useful information rather than an opaque number.

Example result:

```text
1. Shop A
   ✓ Exact match
   ✓ 2 available
   ✓ 1.2 km away
   ✓ Open

2. Shop B
   ✓ Exact match
   ✓ 1 available
   ✓ 0.7 km away
   ✓ Open

3. Shop C
   ✓ Compatible
   ✓ 3 available
   ✓ 2.1 km away
   ✓ Open
```

The ordering is generated by the configured score, not manually selected.

---

## 20. Availability Re-check

This is one of the most important backend protections.

Suppose the engine returns:

```text
Shop A → 1 display available
```

Before reservation, another shop may have already taken it.

Therefore:

```text
Candidate discovery
       ↓
User selects / accepts
       ↓
Transactional inventory re-check
       ↓
Reserve or reject
```

The backend must use a safe database transaction.

Conceptually:

```sql
BEGIN;

SELECT available_quantity
FROM inventory
WHERE id = :inventory_id
FOR UPDATE;

-- verify quantity

UPDATE inventory
SET available_quantity = available_quantity - :quantity
WHERE id = :inventory_id;

-- create fulfillment / reservation

COMMIT;
```

The exact SQL implementation belongs to the backend layer, but the matching engine must be designed around this reality.

---

## 21. Preventing Duplicate Fulfillment

A request should not accidentally create multiple successful fulfillments.

The backend should enforce request state transitions such as:

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

Possible failure path:

```text
OPEN
 ↓
MATCHED
 ↓
DECLINED / EXPIRED
 ↓
OPEN or CLOSED
```

Only valid state transitions are allowed.

This protects both shops and inventory.

---

## 22. Match Result Schema

The API should return a clean representation.

Example:

```json
{
  "request_id": "req_123",
  "matches": [
    {
      "shop_id": "shop_001",
      "shop_name": "ABC Mobiles",
      "inventory_id": "inv_101",
      "part_name": "Samsung Galaxy M14 Display",
      "compatibility": "exact",
      "available_quantity": 2,
      "distance_km": 1.2,
      "shop_status": "open",
      "match_score": 91.4,
      "last_inventory_update": "2026-09-28T10:20:00Z"
    }
  ]
}
```

The frontend should receive enough information to display the match clearly without exposing internal scoring implementation details unnecessarily.

---

## 23. API Design

### Find matches

```http
POST /api/v1/matches/search
```

Request:

```json
{
  "request_id": "req_123",
  "radius_km": 5,
  "limit": 10
}
```

Response:

```json
{
  "request_id": "req_123",
  "matches": [],
  "search_radius_km": 5,
  "expanded_radius": false
}
```

### Get one match

```http
GET /api/v1/matches/{match_id}
```

### Accept a match

```http
POST /api/v1/matches/{match_id}/accept
```

The accept operation must trigger the backend's transactional availability validation.

---

## 24. Search Radius Expansion

If the initial radius contains no valid candidates:

```text
Search 5 km
   ↓
No useful match
   ↓
Search 10 km
   ↓
No useful match
   ↓
Search 20 km
   ↓
Return available candidates
```

The response should tell the user when the radius was expanded.

Example:

> "No exact match was found within 5 km. We found 2 possible matches within 10 km."

This makes the system transparent.

---

## 25. No-Match Handling

No match is a valid system state.

The application should not fabricate a result.

Possible response:

```text
No exact match found nearby.

We can:
• expand the search radius
• notify nearby shops
• let you create a supplier request
• allow manual part details
```

This connects the Matching Engine with the supplier-request workflow defined in the backend architecture.

---

## 26. Fallback Supplier Request

When inventory search fails, SpareLink can convert the request into a broader supplier request.

```text
No direct inventory match
          ↓
Supplier request
          ↓
Notify eligible nearby shops
          ↓
Shop responds
          ↓
Validate part + quantity
          ↓
Create fulfillment
```

This preserves SpareLink's core purpose even when no shop has already published the required item.

---

## 27. Performance Architecture

The Matching Engine should be fast enough for an interactive repair-shop workflow.

### Database responsibilities

- Spatial filtering.
- Inventory filtering.
- Shop status filtering.
- Basic compatibility lookup.
- Sorting candidates where practical.

### Application responsibilities

- Build the scoring model.
- Apply business rules.
- Normalize signals.
- Produce explainable match data.
- Coordinate final validation.

### Cache candidates only when safe

Caching may be used for:

- Static compatibility mappings.
- Common aliases.
- Shop metadata with safe expiry.

Live inventory should not rely on a long-lived cache.

---

## 28. Suggested Internal Modules

```text
backend/
└── app/
    ├── matching/
    │   ├── __init__.py
    │   ├── service.py
    │   ├── candidate_retrieval.py
    │   ├── compatibility.py
    │   ├── scoring.py
    │   ├── ranking.py
    │   ├── distance.py
    │   ├── freshness.py
    │   └── schemas.py
    │
    ├── inventory/
    ├── shops/
    ├── requests/
    ├── notifications/
    └── api/
```

The matching module should remain modular without becoming a separate microservice.

---

## 29. Pseudocode

```python
def find_matches(request):
    validate_request(request)

    candidates = retrieve_candidates(
        part=request.part,
        location=request.location,
        radius=request.radius
    )

    eligible = []

    for candidate in candidates:
        if not candidate.shop.is_active:
            continue

        if candidate.available_quantity < request.quantity:
            continue

        compatibility = check_compatibility(
            request,
            candidate.inventory
        )

        if compatibility.level == "unknown":
            continue

        score = calculate_match_score(
            compatibility=compatibility,
            availability=candidate.available_quantity,
            distance=candidate.distance_km,
            freshness=candidate.inventory_updated_at,
            reliability=candidate.shop.reliability,
            response_history=candidate.shop.response_history,
            urgency=request.urgency
        )

        eligible.append(
            build_match(candidate, compatibility, score)
        )

    ranked = sort_by_score(eligible)

    return ranked[:request.limit]
```

This pseudocode describes the architecture only. Production code must include authorization, transactions, validation, logging, error handling, and concurrency protection.

---

## 30. Match Score Explainability

Internally, the engine can retain a score breakdown.

Example:

```json
{
  "compatibility": 40,
  "availability": 18,
  "distance": 13,
  "freshness": 9,
  "reliability": 4,
  "response": 4,
  "urgency": 3,
  "total": 91
}
```

The application does not need to expose every internal number to the user.

Instead, it can generate a human-readable explanation:

```text
Exact match • 2 available • 1.2 km away • recently updated
```

This gives users confidence without exposing unnecessary implementation details.

---

## 31. Security Requirements

The Matching Engine must respect backend security rules.

### Never expose

- Private shop credentials.
- Internal database identifiers unless required.
- Hidden scoring configuration.
- Other shops' private inventory beyond what is necessary.
- Internal reliability calculations.
- Sensitive business analytics.

### Validate

- Request ownership.
- Shop permissions.
- Inventory visibility.
- Match acceptance permissions.
- Quantity values.
- Radius values.
- Request state.

The client must never be trusted to submit a final match score.

---

## 32. Abuse Protection

Potential abuse includes:

- Creating thousands of fake requests.
- Repeatedly querying the same shops.
- Scraping inventory.
- Requesting unrealistic quantities.
- Sending malformed location data.

Controls should include:

- Authentication.
- Rate limiting.
- Request validation.
- Per-user/shop limits.
- Audit logs.
- Suspicious activity detection.

These controls belong primarily to the backend but must be considered by the matching design.

---

## 33. Testing Strategy

The Matching Engine requires deterministic tests.

### Unit tests

Test:

- Exact compatibility.
- Compatible inventory.
- Zero inventory.
- Insufficient inventory.
- Distance scoring.
- Freshness scoring.
- Urgency adjustment.
- Score calculation.
- Ranking order.
- Radius expansion.

### Integration tests

Test:

```text
Request
 ↓
Database
 ↓
Candidate retrieval
 ↓
Matching service
 ↓
Ranked response
```

### Concurrency tests

Test:

```text
Request A ──┐
            ├── same inventory item
Request B ──┘
```

Only the valid transaction should successfully reserve stock.

---

## 34. Example Test Cases

| Test | Expected Result |
|---|---|
| Exact part + available stock | Match returned |
| Exact part + zero stock | Excluded |
| Exact part + insufficient quantity | Excluded |
| Compatible part + stock | Lower-priority match |
| Unknown compatibility | Excluded from exact results |
| Shop outside radius | Excluded |
| Inactive shop | Excluded |
| Stale inventory | Lower score or excluded according to policy |
| Two requests for one item | Transaction protects inventory |
| No candidates | No-match state |
| No candidate within first radius | Radius expansion |
| Urgent request | Nearby/active signals receive configured adjustment |

---

## 35. MVP Priorities

### P0 — Required

- Candidate retrieval.
- Geographic filtering.
- Inventory availability filtering.
- Exact compatibility matching.
- Deterministic scoring.
- Ranking.
- Availability re-check.
- Safe reservation integration.
- No-match handling.
- Basic explainability.
- Unit and integration tests.

### P1 — Strong improvements

- Explicit compatibility mappings.
- Inventory freshness scoring.
- Reliability signals.
- Response-time signals.
- Automatic radius expansion.
- Supplier request fallback.
- Match analytics.

### P2 — Future

- Learning-to-rank.
- Personalized shop recommendations.
- Demand-aware matching.
- Predictive inventory.
- Advanced compatibility reasoning.
- Dynamic ranking based on historical outcomes.

The MVP should not depend on P2 intelligence to work.

---

## 36. Observability

Every matching operation should produce useful logs.

Example:

```text
request_id
search_radius
candidate_count
eligible_count
exact_match_count
expanded_radius
ranking_duration_ms
final_match_count
```

Metrics may include:

- Average matching latency.
- No-match rate.
- Exact-match rate.
- Match acceptance rate.
- Match cancellation rate.
- Inventory mismatch rate.
- Reservation failure rate.

These metrics will help improve the system after real-world testing.

---

## 37. Failure Handling

The engine should fail safely.

### Database unavailable

Return a temporary service error.

### Location unavailable

Use a supported fallback only when the product flow explicitly permits it.

### Inventory service failure

Do not report stock as confirmed.

### AI normalization uncertain

Send the request back to confirmation rather than performing an unsafe exact match.

### Candidate disappears during reservation

Return a clear availability-change message and allow the user to continue with other candidates.

---

## 38. End-to-End Example

```text
REPAIR SHOP A
"I need Samsung M14 display urgently."

        ↓

PART 7: AI ENGINE

Brand      → Samsung
Model      → Galaxy M14
Part       → Display
Quantity   → 1
Urgency    → High

        ↓

USER CONFIRMS

        ↓

PART 8: MATCHING ENGINE

Find nearby shops
        ↓
Check inventory
        ↓
Check compatibility
        ↓
Calculate distance
        ↓
Score candidates
        ↓
Rank candidates

        ↓

RESULT

Shop B
✓ Exact match
✓ 2 available
✓ 1.1 km away
✓ Open

Shop C
✓ Exact match
✓ 1 available
✓ 2.4 km away
✓ Open

        ↓

USER SELECTS SHOP B

        ↓

BACKEND TRANSACTION

Re-check stock
        ↓
Reserve inventory
        ↓
Create fulfillment
        ↓
Notify supplier

        ↓

SPARE PART MOVES

        ↓

REQUEST COMPLETED
```

This is the central operational loop of SpareLink.

---

## 39. Definition of Done

Part 8 is complete when the system specification supports:

- [x] Structured spare requests from Part 7.
- [x] Nearby candidate discovery.
- [x] Inventory filtering.
- [x] Compatibility evaluation.
- [x] Geographic distance calculation.
- [x] Match scoring.
- [x] Candidate ranking.
- [x] Explainable results.
- [x] Radius expansion.
- [x] No-match handling.
- [x] Supplier-request fallback.
- [x] Availability re-check.
- [x] Transaction-safe fulfillment.
- [x] Duplicate fulfillment protection.
- [x] Security boundaries.
- [x] Testing strategy.
- [x] Observability.
- [x] P0/P1/P2 implementation priorities.

---

## 40. Part 8 Architecture Summary

The Matching Engine can be summarized as:

```text
              CONFIRMED REQUEST
                      │
                      ▼
             ┌─────────────────┐
             │ Candidate Search│
             └────────┬────────┘
                      │
                      ▼
             ┌─────────────────┐
             │ Eligibility     │
             │ Filtering       │
             └────────┬────────┘
                      │
                      ▼
             ┌─────────────────┐
             │ Compatibility   │
             │ Evaluation      │
             └────────┬────────┘
                      │
                      ▼
             ┌─────────────────┐
             │ Match Scoring   │
             └────────┬────────┘
                      │
                      ▼
             ┌─────────────────┐
             │ Ranking         │
             └────────┬────────┘
                      │
                      ▼
             ┌─────────────────┐
             │ Availability    │
             │ Re-check        │
             └────────┬────────┘
                      │
                      ▼
                MATCH RESULTS
                      │
                      ▼
             TRANSACTIONAL
                FULFILLMENT
```

### Final principle

> **SpareLink should never ask "Which shop is closest?"**
>
> It should ask:
>
> **"Which nearby shop can actually help this repair shop right now?"**

That distinction is what turns a location search into a real spare-part coordination engine.

---

## 41. Connection to the Next Part

Part 8 produces the ranked match results.

The next layer must turn those results into a usable product experience.

### Part 9 will define:

- Frontend integration.
- Match-result UI.
- Live request states.
- Shop cards.
- Accept / decline interactions.
- Reservation feedback.
- Loading and empty states.
- Error states.
- Real-time status updates.
- Frontend-to-backend API integration.
- Mobile-first implementation.
- Final user journey from request to fulfillment.

**Part 8 complete.**
