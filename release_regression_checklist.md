# Checklist — Release Regression (API & UI)

> Use before every production release.  
> Mark each item: ✅ Pass | ❌ Fail | ⏭️ Skip (with reason)

---

## 🔐 Authentication

- [ ] Login with valid credentials → success, token returned
- [ ] Login with wrong password → 401, no token
- [ ] Login with expired token → 401, proper error
- [ ] Logout invalidates token
- [ ] Protected routes redirect to login when unauthenticated

---

## 💳 Payments

- [ ] Successful payment → `approved` status, DB updated
- [ ] Declined payment → `declined` status, no charge
- [ ] Payment to inactive terminal → error returned
- [ ] Zero/negative amount → 400 validation error
- [ ] Duplicate payment (same idempotency key) → single transaction in DB
- [ ] Payment response contains: `transaction_id`, `status`, `amount`, `currency`

---

## 🔍 Transactions

- [ ] Transactions list loads without errors
- [ ] Filter by status works (approved / declined / pending)
- [ ] Filter by date range works
- [ ] Transaction detail page shows correct data
- [ ] Transaction data in UI matches DB record

---

## 🧾 Receipts

- [ ] Receipt generated after approved payment
- [ ] Amount formatted correctly (2 decimal places)
- [ ] Receipt contains: terminal name, date, amount, transaction ID
- [ ] Receipt PDF opens without errors

---

## 👤 User Management (Admin)

- [ ] Create new user → appears in list, receives invite email
- [ ] Update user role → new permissions applied immediately
- [ ] Deactivate user → user cannot log in
- [ ] Delete user → user removed from DB, no orphaned records

---

## 🌐 API Health

- [ ] `GET /api/v1/health` → 200 OK
- [ ] All endpoints return correct Content-Type: `application/json`
- [ ] Error responses include meaningful `message` field
- [ ] No stack traces exposed in error responses
- [ ] Response times under 2s for core endpoints

---

## 🖥️ UI Smoke

- [ ] Dashboard loads without JS errors (DevTools console clean)
- [ ] Navigation works: all main menu items accessible
- [ ] Tables: pagination works, data loads on page change
- [ ] Forms: required field validation shows inline errors
- [ ] UI works in: Chrome latest, Firefox latest

---

## 📋 Notes

| Item | Status | Comment |
|------|--------|---------|
| | | |
