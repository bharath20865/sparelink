# SPARELINK | PART 10: REAL-TIME COMMUNICATION, NOTIFICATIONS & FULFILLMENT

## MASTER INSTRUCTION

Continue building the **SpareLink** project from Parts 1–9.

Treat Parts 1–9 as the existing source of truth.

Do not contradict:

- the original SpareLink idea
- the user journey
- the UI/UX architecture
- the technical architecture
- the database model
- the backend architecture
- the AI Engine
- the Matching Engine
- the Frontend Integration layer

Part 10 defines how SpareLink handles **real-time communication, notifications, supplier responses, reservations, and final fulfillment** after a match is discovered.

The system should make a request feel alive:

```text
Request created
      ↓
Nearby suppliers found
      ↓
Supplier notified
      ↓
Supplier responds
      ↓
Inventory confirmed
      ↓
Reservation created
      ↓
Pickup / delivery coordination
      ↓
Fulfillment
      ↓
Completion
```

The user should never have to repeatedly refresh the application to understand what is happening.

---

# 1. PART 10 OBJECTIVE

Design the real-time communication and fulfillment layer of SpareLink.

Part 10 covers:

- real-time architecture
- supplier notifications
- buyer notifications
- request events
- reservation events
- supplier responses
- accept/decline flows
- reservation expiration
- fulfillment states
- pickup coordination
- optional delivery coordination
- in-app notifications
- push notifications
- WhatsApp integration strategy
- WebSocket/SSE architecture
- polling fallback
- event-driven backend design
- notification preferences
- notification reliability
- duplicate prevention
- idempotency
- concurrency
- security
- privacy
- audit logs
- monitoring
- testing
- MVP priorities

---

# 2. CORE PRINCIPLE

SpareLink is not simply a search engine.

It is a **coordination system**.

Finding a shop is only half the problem.

The real workflow is:

```text
Find
 ↓
Notify
 ↓
Respond
 ↓
Confirm
 ↓
Reserve
 ↓
Fulfill
```

Part 10 therefore focuses on what happens **after matching**.

---

# 3. REAL-TIME ARCHITECTURE

A realistic MVP architecture is:

```text
                     ┌───────────────┐
                     │    Frontend   │
                     └───────┬───────┘
                             │
                     WebSocket / SSE
                             │
                     ┌───────▼───────┐
                     │ Realtime Layer│
                     └───────┬───────┘
                             │
                     ┌───────▼───────┐
                     │ Event System   │
                     └───────┬───────┘
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
          ▼                  ▼                  ▼
      Requests           Inventory        Notifications
          │                  │                  │
          └──────────────────┼──────────────────┘
                             ▼
                         PostgreSQL
```

Keep the first implementation realistic.

Do not introduce unnecessary Kafka, RabbitMQ, or complex distributed infrastructure for the MVP.

---

# 4. EVENT-DRIVEN THINKING

Important system actions should produce events.

Examples:

```text
REQUEST_CREATED
MATCH_FOUND
SUPPLIER_NOTIFIED
SUPPLIER_VIEWED
SUPPLIER_ACCEPTED
SUPPLIER_DECLINED
RESERVATION_STARTED
RESERVATION_CONFIRMED
RESERVATION_EXPIRED
REQUEST_CANCELLED
FULFILLMENT_STARTED
FULFILLMENT_COMPLETED
REQUEST_COMPLETED
```

Events should have:

- event ID
- event type
- timestamp
- request ID
- actor
- relevant entity ID
- metadata
- correlation/request ID

Example:

```json
{
  "event_id": "evt_123",
  "event_type": "SUPPLIER_ACCEPTED",
  "request_id": "req_456",
  "actor_id": "shop_789",
  "timestamp": "2026-09-28T10:30:00Z"
}
```

---

# 5. REQUEST EVENT LIFECYCLE

The state machine should be explicit:

```text
OPEN
 ↓
MATCHED
 ↓
SUPPLIER_NOTIFIED
 ↓
RESPONSE_PENDING
 ↓
RESERVED
 ↓
ACCEPTED
 ↓
FULFILLMENT_IN_PROGRESS
 ↓
COMPLETED
```

Alternative states:

```text
DECLINED
EXPIRED
CANCELLED
NO_MATCH
```

Do not allow arbitrary state transitions from the frontend.

---

# 6. SUPPLIER NOTIFICATION FLOW

When a suitable match is found:

```text
Matching Engine
      ↓
Eligible Supplier
      ↓
Notification Service
      ↓
Supplier
```

### In-App

```text
New Spare Request

Samsung M14 Display
Quantity: 1
Distance: 0.8 km

[View Request]
```

### Push

```text
New nearby spare-part request
Samsung M14 Display
```

### WhatsApp

Optional future integration:

```text
New SpareLink request:
Samsung M14 Display × 1

[View Request]
```

The first MVP should prioritize in-app notifications.

---

# 7. BUYER NOTIFICATION FLOW

The requester should receive updates.

Examples:

```text
3 nearby suppliers found.
```

```text
Sai Mobile Care accepted your request.
```

```text
Your part has been reserved.
```

```text
Your request is ready for pickup.
```

```text
Request completed.
```

Avoid overwhelming users with every internal event.

Only user-relevant events should generate visible notifications.

---

# 8. NOTIFICATION PRIORITY

### Critical

Examples:

- reservation failure
- reservation expiration
- supplier acceptance
- request cancellation

### Important

Examples:

- new match
- supplier response
- fulfillment update

### Informational

Examples:

- inventory reminder
- completed request history
- general updates

The notification system should prioritize actionable information.

---

# 9. NOTIFICATION CHANNELS

Channels:

```text
In-App
Push
WhatsApp
SMS
Email
```

MVP:

```text
In-App
+
Push where supported
```

P1:

```text
WhatsApp
SMS
```

P2:

```text
Email
Advanced notification preferences
```

Do not make the MVP dependent on third-party messaging providers.

---

# 10. NOTIFICATION SERVICE

Create a dedicated backend notification module.

Suggested structure:

```text
backend/app/notifications/
├── __init__.py
├── service.py
├── schemas.py
├── repository.py
├── channels/
│   ├── in_app.py
│   ├── push.py
│   ├── whatsapp.py
│   └── sms.py
└── templates/
```

Responsibilities:

- generate notifications
- choose channel
- store notifications
- deliver notifications
- retry failures
- track delivery status
- prevent duplicates

---

# 11. NOTIFICATION DATABASE MODEL

Example:

```text
notifications
------------------------
id
user_id
shop_id
event_id
type
title
body
channel
status
created_at
sent_at
read_at
metadata
```

Possible statuses:

```text
PENDING
SENT
DELIVERED
READ
FAILED
```

Avoid storing unnecessary sensitive information inside notification payloads.

---

# 12. NOTIFICATION TEMPLATES

Use structured templates.

Example:

```text
Title:
New Spare Request

Body:

Samsung M14 Display × 1
A nearby repair shop needs this part.
```

Supplier acceptance:

```text
Title:
Request Accepted

Body:

Sai Mobile Care has accepted your request for
Samsung M14 Display.
```

Reservation:

```text
Title:
Part Reserved

Body:

1 Samsung M14 Display has been reserved for your request.
```

Templates should be centrally managed.

---

# 13. REAL-TIME CONNECTION

Use WebSockets or Server-Sent Events where appropriate.

Example:

```text
Frontend
   │
   │ WebSocket
   ▼
Realtime Gateway
   │
   ▼
Event Stream
```

Clients subscribe to relevant request updates.

Example channel:

```text
request:req_123
```

Events are delivered only to authorized participants.

---

# 14. AUTHORIZATION OF REAL-TIME EVENTS

A user must never receive another shop's private events.

Before subscribing:

```text
Authenticate
 ↓
Identify user
 ↓
Verify request membership
 ↓
Allow subscription
```

Never trust:

```text
request_id
shop_id
user_id
```

sent by the client without backend verification.

---

# 15. POLLING FALLBACK

If WebSockets/SSE are unavailable:

```text
Realtime connection
      ↓
Failed
      ↓
Polling fallback
```

Example:

```text
GET /api/v1/requests/{request_id}
```

Poll only when necessary.

Avoid aggressive polling.

---

# 16. SUPPLIER RESPONSE FLOW

When a supplier receives a request:

```text
New Request
 ↓
View Details
 ↓
Check Inventory
 ↓
Accept / Decline
```

Supplier should see:

- requested part
- quantity
- relevant compatibility information
- requester context needed for fulfillment
- response deadline if applicable

Do not expose unnecessary requester personal information.

---

# 17. ACCEPTANCE FLOW

When supplier selects:

```text
[Accept Request]
```

Backend must:

1. authenticate supplier
2. verify supplier is eligible
3. verify request is still active
4. re-check inventory
5. lock relevant inventory row
6. verify quantity
7. reserve inventory
8. update request state
9. record event
10. create notifications
11. return updated state

The frontend must display progress during this operation.

---

# 18. DECLINE FLOW

Supplier may decline.

Example:

```text
Decline Request

Reason:
○ Out of stock
○ Wrong compatibility
○ Cannot fulfill now
○ Other
```

Reasons are useful for analytics and debugging.

Do not require a reason for every decline if that creates unnecessary friction.

---

# 19. INVENTORY RACE CONDITION

Two repair shops may attempt to reserve the same final part.

```text
Shop A → Reserve
Shop B → Reserve
```

Only one should succeed.

Backend transaction:

```text
BEGIN
 ↓
LOCK inventory row
 ↓
CHECK quantity
 ↓
UPDATE quantity
 ↓
CREATE reservation
 ↓
COMMIT
```

If inventory is insufficient:

```text
ROLLBACK
 ↓
Reservation Failed
```

Frontend:

```text
This part was just taken by another request.

We're checking the next available match.
```

---

# 20. RESERVATION MODEL

Example:

```text
reservations
------------------------
id
request_id
inventory_id
supplier_shop_id
requester_shop_id
quantity
status
expires_at
created_at
confirmed_at
cancelled_at
```

Statuses:

```text
PENDING
CONFIRMED
EXPIRED
CANCELLED
FULFILLED
```

---

# 21. RESERVATION EXPIRATION

Reservations should not remain locked forever.

Example:

```text
Reservation created
      ↓
Timer starts
      ↓
Supplier/requester completes next step
      ↓
Reservation confirmed
```

If timeout occurs:

```text
Reservation
 ↓
EXPIRED
 ↓
Inventory released
 ↓
Request can search again
```

The timeout should be configurable.

Do not hard-code a single business duration throughout the system.

---

# 22. RESERVATION COUNTDOWN UI

If an active reservation has a deadline, display it clearly.

Example:

```text
Part Reserved

Samsung M14 Display
Quantity: 1

Reservation expires in:
08:42

[Continue]
```

The countdown is informational.

The backend timestamp remains authoritative.

---

# 23. FULFILLMENT OPTIONS

SpareLink should support different fulfillment methods.

### Pickup

```text
Buyer collects part from supplier.
```

### Local Delivery

```text
Supplier delivers part.
```

### Third-Party Delivery

Potential future integration.

MVP should prioritize pickup because it is simpler.

---

# 24. PICKUP FLOW

Example:

```text
Reservation Confirmed
 ↓
Supplier prepares part
 ↓
Buyer travels to supplier
 ↓
Part handed over
 ↓
Fulfillment confirmed
 ↓
Request completed
```

UI:

```text
Pickup Location

Sai Mobile Care
0.8 km away

Status:
Ready for pickup
```

Only show location details appropriate for the transaction and privacy settings.

---

# 25. DELIVERY FLOW

If delivery is enabled:

```text
Reservation
 ↓
Delivery requested
 ↓
Supplier confirms
 ↓
Delivery initiated
 ↓
Delivery in progress
 ↓
Delivered
```

Keep third-party logistics optional.

Do not make the core SpareLink architecture dependent on a delivery provider.

---

# 26. FULFILLMENT STATUS

Define:

```text
RESERVED
 ↓
PREPARING
 ↓
READY
 ↓
PICKUP / DELIVERY
 ↓
FULFILLED
```

Alternative:

```text
CANCELLED
```

Every state change must be authorized.

---

# 27. COMPLETION CONFIRMATION

Both sides may need confirmation.

Buyer:

```text
Did you receive the part?

[Yes, Complete Request]
[Report Problem]
```

Supplier:

```text
Was the part handed over?

[Confirm Fulfillment]
```

For MVP, define a clear primary confirmation flow.

Do not create unnecessary confirmation loops.

---

# 28. DISPUTE / PROBLEM FLOW

If something goes wrong:

```text
Report a Problem
```

Possible categories:

```text
Part not received
Wrong part
Damaged part
Wrong quantity
Supplier unavailable
Other
```

Create a support/problem record.

Do not automatically modify inventory based solely on a user report.

---

# 29. CANCELLATION

Allow cancellation only where business rules permit.

Possible flow:

```text
Cancel Request
 ↓
Confirm
 ↓
Backend verifies state
 ↓
Reservation released if applicable
 ↓
Inventory restored
 ↓
Events recorded
 ↓
Participants notified
```

Cancellation must be idempotent.

---

# 30. DUPLICATE EVENT PREVENTION

The same event may be delivered more than once.

Example:

```text
SUPPLIER_ACCEPTED
SUPPLIER_ACCEPTED
```

The system must not:

- reserve twice
- decrement inventory twice
- send duplicate critical notifications unnecessarily

Use:

- event IDs
- idempotency keys
- unique database constraints
- state-transition checks

---

# 31. EVENT OUTBOX PATTERN

For reliable event publishing, consider an outbox pattern.

Example:

```text
Database Transaction
      │
      ├── Update Request
      │
      ├── Update Inventory
      │
      └── Write Event to Outbox
                ↓
             Event Worker
                ↓
          Notifications / Realtime
```

This prevents the database state from changing successfully while the event is silently lost.

For the MVP, implement a lightweight version if full event infrastructure is unnecessary.

---

# 32. MESSAGE DELIVERY RELIABILITY

Notifications may fail.

Examples:

- phone offline
- push provider unavailable
- WhatsApp API failure
- WebSocket disconnected

The system should:

```text
Attempt
 ↓
Record result
 ↓
Retry when appropriate
 ↓
Fallback where configured
```

Never retry unsafe business operations blindly.

Notifications can be retried.

Inventory reservations must remain transactionally controlled.

---

# 33. NOTIFICATION PREFERENCES

Allow users to control non-critical notifications.

Example:

```text
Notifications

New Requests       ON
Match Updates      ON
Inventory Alerts   ON
Marketing          OFF
```

Critical transaction notifications should follow defined product rules.

Do not allow preferences to break essential transaction communication.

---

# 34. WHATSAPP INTEGRATION

SpareLink's original concept includes WhatsApp-oriented coordination.

Treat WhatsApp as a communication channel, not the source of truth.

Architecture:

```text
SpareLink Backend
      ↓
WhatsApp Adapter
      ↓
WhatsApp Business Platform
```

The database remains authoritative.

A WhatsApp message must never directly modify inventory without backend validation.

---

# 35. WHATSAPP REQUEST EXAMPLE

Possible future experience:

```text
User:
Samsung M14 display kavali urgent ga

SpareLink:
I found 3 nearby shops.

1. Sai Mobile Care - 0.8 km
2. Ravi Mobiles - 2.1 km
3. Kiran Electronics - 4.7 km

Reply with 1, 2, or 3.
```

The WhatsApp layer sends the user's selection to the backend.

Backend validates the selection before taking action.

---

# 36. WHATSAPP SECURITY

Never trust incoming WhatsApp commands blindly.

Validate:

- sender identity
- linked account
- request state
- authorization
- command validity
- expiration

Example:

```text
"ACCEPT 123"
```

must not directly execute a reservation without backend verification.

---

# 37. NOTIFICATION DEDUPLICATION

Prevent notification spam.

If five internal events occur within seconds, the user may only need:

```text
Your request has been successfully matched.
```

rather than five separate notifications.

Define notification grouping/coalescing rules where appropriate.

---

# 38. REAL-TIME UI BEHAVIOUR

Frontend should update automatically.

Example:

```text
Finding suppliers...

        ↓

3 suppliers found

        ↓

Supplier accepted

        ↓

Part reserved

        ↓

Ready for pickup
```

The user should not need to repeatedly reload the page.

---

# 39. EVENT TO UI MAPPING

Define a mapping:

```text
REQUEST_CREATED
→ Request status: Finding suppliers

MATCH_FOUND
→ Match list updated

SUPPLIER_ACCEPTED
→ Supplier accepted badge

RESERVATION_CONFIRMED
→ Reservation screen

RESERVATION_EXPIRED
→ Match search restarted

FULFILLMENT_READY
→ Ready for pickup

FULFILLMENT_COMPLETED
→ Completed state
```

This becomes the contract between realtime events and frontend state.

---

# 40. EVENT ORDERING

Events may arrive late or out of order.

Example:

```text
FULFILLMENT_COMPLETED
```

could theoretically arrive before another older notification.

Frontend must not blindly apply every event.

Use:

- server timestamps
- sequence numbers where necessary
- authoritative request state
- state-transition validation

When uncertain, refresh the authoritative request state.

---

# 41. OFFLINE REAL-TIME HANDLING

When the user reconnects:

```text
Reconnect
 ↓
Authenticate
 ↓
Resubscribe
 ↓
Fetch latest request state
 ↓
Fetch missed notifications
 ↓
Update UI
```

Do not assume that reconnecting automatically reconstructs every missed event.

The backend state remains authoritative.

---

# 42. AUDIT LOG

Record important actions.

Examples:

```text
REQUEST_CREATED
MATCH_SELECTED
SUPPLIER_ACCEPTED
SUPPLIER_DECLINED
RESERVATION_CREATED
RESERVATION_EXPIRED
REQUEST_CANCELLED
FULFILLMENT_COMPLETED
```

Audit records help with:

- debugging
- disputes
- security
- analytics
- accountability

Do not expose internal audit data to ordinary users.

---

# 43. SECURITY REQUIREMENTS

Protect:

- request information
- shop identity
- inventory
- contact details
- transaction details
- notifications
- real-time channels

Every operation must verify authorization server-side.

Prevent:

- unauthorized subscription
- fake supplier responses
- replay attacks
- duplicate reservations
- forged events
- notification injection
- account takeover

---

# 44. PRIVACY

Only reveal information necessary for the current transaction.

Before acceptance:

```text
Shop name
Approximate distance
Part availability
```

After acceptance, additional information may become available according to product rules.

Avoid exposing personal phone numbers unnecessarily.

---

# 45. MONITORING

Track:

```text
notification_delivery_success
notification_delivery_failure
websocket_connections
realtime_disconnects
reservation_success
reservation_failure
reservation_expiration
supplier_response_rate
average_response_time
fulfillment_completion_rate
cancellation_rate
```

Track failures by event type.

---

# 46. ALERTING

Engineering alerts should trigger for abnormal behavior.

Examples:

```text
Reservation failures suddenly increase
Notification delivery drops
Realtime connection failures spike
Database transaction failures increase
```

Do not alert on every individual user error.

Use thresholds and aggregation.

---

# 47. TESTING STRATEGY

### Unit Tests

Test:

- event creation
- state transitions
- notification templates
- expiration logic
- idempotency

### Integration Tests

Test:

```text
Request
 ↓
Match
 ↓
Notification
 ↓
Supplier response
 ↓
Reservation
 ↓
Fulfillment
```

### Realtime Tests

Test:

- connection
- authorization
- event delivery
- reconnect
- duplicate events
- event ordering

### End-to-End Tests

Test complete real-world workflows.

---

# 48. CRITICAL FAILURE TESTS

### Supplier accepts after reservation expires

Expected:

```text
Reject action
 ↓
Explain request is no longer active
```

### Two suppliers compete for one part

Expected:

```text
Only valid reservation succeeds.
```

### Buyer cancels during reservation

Expected:

```text
Reservation released according to state rules.
```

### WebSocket disconnects

Expected:

```text
Reconnect
 ↓
Refresh authoritative state
```

### Notification provider fails

Expected:

```text
Record failure
 ↓
Retry when safe
 ↓
Use fallback where configured
```

---

# 49. MVP PRIORITIES

## P0

Implement:

- request state machine
- supplier responses
- reservation workflow
- inventory re-check
- in-app notifications
- basic realtime updates
- polling fallback
- reservation expiration
- fulfillment states
- cancellation
- event logging
- idempotency
- authorization

## P1

Add:

- push notifications
- WebSockets
- supplier communication
- advanced notification preferences
- WhatsApp integration
- pickup coordination
- richer audit tools

## P2

Add:

- delivery integrations
- intelligent notification prioritization
- advanced event infrastructure
- predictive response timing
- automated supplier reminders
- advanced communication analytics

---

# 50. PERFORMANCE REQUIREMENTS

Real-time features should remain lightweight.

Prioritize:

- efficient subscriptions
- limited event payloads
- reconnect handling
- notification batching
- database indexing
- efficient reservation transactions
- minimal frontend re-rendering

Do not broadcast every event to every connected user.

Use targeted channels.

---

# 51. END-TO-END EXAMPLE

A repair shop requests:

```text
Samsung M14 Display × 1
Urgency: High
```

Matching Engine finds:

```text
Sai Mobile Care
0.8 km
2 available
Exact match
```

System:

```text
MATCH_FOUND
```

Supplier receives:

```text
New nearby request
Samsung M14 Display × 1
```

Supplier opens request.

Event:

```text
SUPPLIER_VIEWED
```

Supplier accepts.

Backend:

```text
BEGIN TRANSACTION
 ↓
Lock inventory
 ↓
Check quantity
 ↓
Reserve 1 unit
 ↓
Update request
 ↓
Write events
 ↓
COMMIT
```

Buyer receives:

```text
Sai Mobile Care accepted your request.

Part reserved.
```

Supplier marks:

```text
Ready for pickup
```

Buyer receives:

```text
Your part is ready for pickup.
```

Buyer collects the part.

Both sides complete the transaction.

System:

```text
FULFILLMENT_COMPLETED
```

Buyer sees:

```text
Request completed ✓
```

Inventory becomes:

```text
Previous: 2
Reserved: 1
Fulfilled: 1
Remaining: 1
```

The complete workflow is recorded.

---

# 52. SYSTEM ARCHITECTURE SUMMARY

The final Part 10 architecture:

```text
                    USER
                     │
                     ▼
                  FRONTEND
                     │
           ┌─────────┴─────────┐
           │                   │
        REST API          WebSocket/SSE
           │                   │
           ▼                   ▼
        BACKEND           REALTIME LAYER
           │                   │
     ┌─────┼─────┐             │
     │     │     │             │
     ▼     ▼     ▼             │
  Request Match Inventory      │
     │     │     │             │
     └─────┼─────┘             │
           ▼                   │
        PostgreSQL ◄────────────┘
           │
           ▼
        Event / Outbox
           │
      ┌────┼─────────────┐
      ▼    ▼             ▼
   Notify Push       WhatsApp
      │
      ▼
   SUPPLIER
```

The architecture must remain modular enough that notification providers can be changed without rewriting the core business logic.

---

# 53. DEFINITION OF DONE

Part 10 is complete when:

- real-time architecture is defined
- event model is defined
- request lifecycle is defined
- supplier notification flow is defined
- buyer notification flow is defined
- reservation lifecycle is defined
- inventory race conditions are addressed
- supplier acceptance is defined
- supplier decline is defined
- reservation expiration is defined
- fulfillment flow is defined
- pickup flow is defined
- delivery extension is defined
- cancellation is defined
- dispute flow is defined
- realtime frontend behavior is defined
- polling fallback is defined
- WhatsApp integration strategy is defined
- notification preferences are defined
- notification deduplication is defined
- idempotency is defined
- event ordering is addressed
- offline reconnection is defined
- audit logging is defined
- security requirements are defined
- privacy requirements are defined
- monitoring is defined
- testing is defined
- MVP priorities are defined

---

# 54. CONNECTION TO PART 11

Part 11 should continue from the communication and fulfillment foundation and define the **deployment, DevOps, infrastructure, scalability, reliability, CI/CD, monitoring, and production-readiness architecture** of SpareLink.

Parts 1–10 define what SpareLink is and how it works.

Part 11 should define how SpareLink is **built, deployed, monitored, secured, and operated in the real world.**

---

# FINAL PRINCIPLE

SpareLink should not stop at:

> “We found a shop.”

It should complete the entire chain:

```text
Need
 ↓
Understand
 ↓
Find
 ↓
Notify
 ↓
Respond
 ↓
Reserve
 ↓
Fulfill
 ↓
Complete
```

The user should never wonder:

> “What happened to my request?”

SpareLink should always have an answer.

**The system knows the state.**
**The user can see the state.**
**The backend controls the state.**

That is the foundation of reliable real-time coordination.