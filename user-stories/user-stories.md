# Airbnb Clone — User Stories

Repository: `alx-airbnb-project-documentation`  
Directory: `user-stories/`  
File: `user-stories.md`

## Overview

This document translates the use case diagram for the Airbnb Clone backend into actionable user stories. Each story includes a short description, acceptance criteria, and a priority label.

---

### US-01 — User Registration & Login

**As a** guest or host,  
**I want** to register an account and log in,  
**so that** I can access personalized features and create/manage listings or bookings.

**Acceptance criteria**

- User can register with email, password, and name.
- Email verification link is sent to new users.
- Registered users can log in and receive a JWT (or other session token).
- Invalid credentials produce an appropriate error message.

**Priority:** High

---

### US-02 — Profile Management & Roles

**As a** registered user,  
**I want** to view and update my profile and role (guest/host),  
**so that** my contact details, verification status, and role-specific settings are correct.

**Acceptance criteria**

- User can update name, phone number, profile photo, bio.
- Hosts can provide ID/verification documents and business info.
- System exposes role-based access controls (guest vs host vs admin).
- Changes are persisted and visible on next login.

**Priority:** High

---

### US-03 — Property (Listing) Management

**As a** host,  
**I want** to create, edit, delete property listings and upload images,  
**so that** I can offer my space for bookings with accurate details and availability.

**Acceptance criteria**

- Host can create a listing with title, description, price, location, amenities.
- Host can upload multiple images (stored via cloud service) and set availability/calendar.
- Host can edit or delete a listing.
- Validation prevents overlapping availability or missing required fields.

**Priority:** High

---

### US-04 — Search, Filter & View Listings

**As a** guest,  
**I want** to search and filter properties by location, dates, price and amenities,  
**so that** I can find listings that match my needs.

**Acceptance criteria**

- Search supports location + date range + guest count.
- Filters include price range, property type, amenities, rating.
- Search results list shows thumbnails, price per night, rating, and short description.
- Clicking a result opens a detailed listing page with calendar and booking CTA.

**Priority:** High

---

### US-05 — Booking Flow (Create / Cancel)

**As a** guest,  
**I want** to create and cancel bookings for available dates,  
**so that** I can reserve properties and manage my travel plans.

**Acceptance criteria**

- Guest can request a booking for available dates.
- System checks availability at booking time and prevents double-booking.
- Booking status and history are available in user's account.
- Guests can cancel (policy-dependent) and receive confirmation.

**Priority:** High

---

### US-06 — Payments, Refunds & Host Payouts

**As a** guest and host,  
**I want** to make payments, receive refunds when applicable, and receive payouts as a host,  
**so that** transactions are processed securely and funds flow correctly.

**Acceptance criteria**

- Payment provider integration (stripe-like) processes guest payments.
- System records transactions with status (pending, completed, refunded).
- Refunds can be issued and recorded.
- Hosts receive payouts (scheduled or on-demand) with transaction records.

**Priority:** High

---

### US-07 — Reviews & Ratings

**As a** guest or host,  
**I want** to leave and respond to reviews,  
**so that** future users can make informed decisions and hosts can reply.

**Acceptance criteria**

- Guests can leave a rating and text review for completed stays.
- Hosts can respond to guest reviews.
- Average rating is calculated and shown on listings.
- Reviews are moderated for abuse (basic validation/reporting).

**Priority:** Medium

---

### US-08 — Notifications & Email Confirmations

**As a** user (guest/host/admin),  
**I want** to receive email and in-app notifications for bookings, payments, and messages,  
**so that** I stay informed about relevant events.

**Acceptance criteria**

- Booking confirmations and payment receipts are sent via email.
- Hosts get notified on new bookings / cancellations.
- Users can opt in/out of notification types.
- Notification records are visible in the user’s dashboard.

**Priority:** Medium

---

### US-09 — Admin Controls & Reporting

**As an** admin,  
**I want** to manage users, listings, bookings and generate reports,  
**so that** I can resolve disputes, issue refunds, and maintain platform health.

**Acceptance criteria**

- Admin dashboard shows users, listings, bookings, and financial transactions.
- Admin can deactivate/reactivate users and remove listings.
- Admin can initiate refunds and adjust booking statuses.
- Basic reporting (bookings per day, revenue per period) is available.

**Priority:** Medium

---

### US-10 — Payment Provider & Email Service Integrations

**As a** system integrator,  
**I want** dedicated integration points for external payment and email services,  
**so that** payments and transactional emails are handled reliably.

**Acceptance criteria**

- Payment integration supports webhooks for events (payment succeeded, refunded).
- Email service integration handles templates for confirmations and password resets.
- Logging exists for webhook events and email delivery failures.

**Priority:** Medium

---

## Notes

- Each story should be split into smaller dev tasks during sprint planning if needed (e.g., "Upload images" can be a separate task).
- Add acceptance test cases and API contract details during backlog refinement.
