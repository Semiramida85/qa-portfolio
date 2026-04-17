# Bug Reports — Examples

> Real-world style bug reports from fintech QA practice.  
> Company-specific data is anonymized.

---

## BUG-001 · Payment status not updated in UI after successful transaction

| Field | Value |
|-------|-------|
| **ID** | BUG-001 |
| **Severity** | Major |
| **Priority** | High |
| **Status** | Fixed |
| **Environment** | Staging |
| **Module** | Payments / UI |

### Summary
After a successful payment transaction, the UI continues to show status **"Pending"** instead of **"Approved"**, even though the backend correctly updates the DB record.

### Steps to Reproduce
1. Log in as operator user
2. Navigate to **Transactions** page
3. Initiate a payment via terminal `TRM-00123`
4. Wait for transaction to complete (~3 sec)
5. Observe transaction status in the list

### Expected Result
Transaction status changes to **"Approved"** in the UI within 5 seconds.

### Actual Result
Transaction status remains **"Pending"** indefinitely. Hard page refresh shows correct "Approved" status.

### Evidence
- **Kibana log:** `transaction_id=txn_9987` → `status=approved` at `2025-07-14T11:23:05Z`
- **DB query result:**
  ```sql
  SELECT id, status, updated_at FROM transactions WHERE id = 'txn_9987';
  -- Returns: approved | 2025-07-14 11:23:05
  ```
- **UI screenshot:** [screenshot attached]

### Root Cause (after investigation)
WebSocket subscription was not re-established after token refresh, causing the UI to miss real-time status update events.

### Notes
Reproducible 100% of the time when session token was refreshed during the transaction.

---

## BUG-002 · API returns 500 on payment with currency code in lowercase

| Field | Value |
|-------|-------|
| **ID** | BUG-002 |
| **Severity** | Major |
| **Priority** | High |
| **Status** | Fixed |
| **Environment** | Staging + Production |
| **Module** | Payments API |

### Summary
`POST /api/v1/payments/process` returns `500 Internal Server Error` when `currency` field is sent in lowercase (e.g., `"usd"` instead of `"USD"`).

### Steps to Reproduce
```bash
curl -X POST https://api.staging.example.com/v1/payments/process \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "terminal_id": "TRM-00123",
    "amount": 10.00,
    "currency": "usd"
  }'
```

### Expected Result
- Status: `400 Bad Request` with validation message, OR
- Input normalized to uppercase and processed successfully

### Actual Result
- Status: `500 Internal Server Error`
- Kibana error: `CurrencyNotFoundException: unknown currency 'usd'`
- Unhandled exception with full stack trace exposed in response body ⚠️

### Evidence
**Response body (500):**
```json
{
  "error": "Internal Server Error",
  "trace": "CurrencyNotFoundException at PaymentService.java:234..."
}
```

### Impact
Stack trace exposure is a **security concern** — internal service structure visible to the client.

---

## BUG-003 · SQL injection not sanitized in search endpoint

| Field | Value |
|-------|-------|
| **ID** | BUG-003 |
| **Severity** | Critical |
| **Priority** | Critical |
| **Status** | Fixed |
| **Environment** | Staging |
| **Module** | Admin / Search |

### Summary
The `GET /api/v1/admin/transactions?search=` parameter is not sanitized, allowing SQL injection via the search input.

### Steps to Reproduce
```
GET /api/v1/admin/transactions?search=' OR '1'='1
```

### Expected Result
- Input sanitized / parameterized query used
- Status: `200 OK` with empty or normal results

### Actual Result
- Returns all transactions in the database regardless of user permissions
- Confirms SQL injection vulnerability

### Notes
Reported immediately to dev team lead. Not included in any public test run. Fixed via prepared statements in the same sprint.

---

## BUG-004 · Incorrect rounding of transaction amount in receipt

| Field | Value |
|-------|-------|
| **ID** | BUG-004 |
| **Severity** | Minor |
| **Priority** | Medium |
| **Status** | Open |
| **Environment** | Staging |
| **Module** | Receipts |

### Summary
Transaction receipt displays `$10.1` instead of `$10.10` — trailing zero is dropped, inconsistent with financial formatting standards.

### Steps to Reproduce
1. Process payment for exactly `$10.10`
2. View generated receipt PDF

### Expected Result
Amount displayed as `$10.10`

### Actual Result
Amount displayed as `$10.1`

### Notes
Affects only amounts ending in `0` in the cents position (e.g., $5.10, $20.30). Amounts like $5.11 display correctly.
