# API Testing — Examples & Postman Notes

> Examples of API test scenarios and curl commands used in manual API testing.

---

## Tools Used

- **Postman** — collections, environments, pre-request scripts, test assertions
- **Swagger UI** — contract verification, schema exploration
- **curl** — quick checks, CI-friendly scripting

---

## 1. Authentication Flow

### Get access token
```bash
curl -X POST https://api.staging.example.com/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "qa_test@example.com",
    "password": "SecurePass123!"
  }'
```

**Postman Test Script:**
```javascript
pm.test("Status is 200", () => {
    pm.response.to.have.status(200);
});

pm.test("Response has access_token", () => {
    const json = pm.response.json();
    pm.expect(json).to.have.property("access_token");
    pm.expect(json.access_token).to.be.a("string").and.not.empty;
});

// Save token for next requests
const token = pm.response.json().access_token;
pm.environment.set("access_token", token);
```

---

## 2. Payment Processing

### Happy path
```bash
curl -X POST https://api.staging.example.com/v1/payments/process \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "terminal_id": "TRM-00123",
    "amount": 25.99,
    "currency": "USD",
    "card_token": "tok_visa_success"
  }'
```

### Postman Assertions
```javascript
pm.test("Status 200", () => pm.response.to.have.status(200));

pm.test("Transaction approved", () => {
    const json = pm.response.json();
    pm.expect(json.status).to.eql("approved");
    pm.expect(json).to.have.property("transaction_id");
});

pm.test("Amount matches request", () => {
    const json = pm.response.json();
    pm.expect(json.amount).to.eql(25.99);
});

pm.test("Response time < 2000ms", () => {
    pm.expect(pm.response.responseTime).to.be.below(2000);
});
```

---

## 3. Schema Validation Example

```javascript
// Validate response matches expected schema
const schema = {
    type: "object",
    required: ["transaction_id", "status", "amount", "currency"],
    properties: {
        transaction_id: { type: "string" },
        status: { type: "string", enum: ["approved", "declined", "pending"] },
        amount: { type: "number" },
        currency: { type: "string", minLength: 3, maxLength: 3 }
    }
};

pm.test("Response schema is valid", () => {
    pm.response.to.have.jsonSchema(schema);
});
```

---

## 4. Error Handling Checks

### 401 — No token
```bash
curl -X GET https://api.staging.example.com/v1/transactions \
  -H "Content-Type: application/json"
# Expected: 401 Unauthorized
```

### 403 — Wrong role
```bash
curl -X DELETE https://api.staging.example.com/v1/admin/users/123 \
  -H "Authorization: Bearer $OPERATOR_TOKEN"
# Expected: 403 Forbidden (operator cannot delete users)
```

### 404 — Non-existent resource
```bash
curl -X GET https://api.staging.example.com/v1/transactions/txn_nonexistent \
  -H "Authorization: Bearer $TOKEN"
# Expected: 404 Not Found
```

---

## 5. Edge Cases Checklist

| Case | Input | Expected |
|------|-------|----------|
| Empty string body | `{}` | `400 Bad Request` |
| Extra unknown fields | `{"foo": "bar", ...valid}` | `200` or `400` (per contract) |
| Very large amount | `9999999999.99` | `400` or handled gracefully |
| Special chars in string fields | `"name": "<script>alert(1)</script>"` | Sanitized / `400` |
| Concurrent requests | 10 simultaneous payments | All processed, no duplicates |

---

## 6. Environment Variables (Postman)

```
BASE_URL      = https://api.staging.example.com
access_token  = (set dynamically after login)
terminal_id   = TRM-00123
test_email    = qa_test@example.com
```
