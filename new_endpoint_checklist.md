# Checklist — New API Endpoint Testing

> Use when a new endpoint is added or significantly changed.

---

## 📄 Contract & Documentation

- [ ] Endpoint is documented in Swagger / OpenAPI
- [ ] Request/response schema matches implementation
- [ ] Required vs optional fields are correctly marked
- [ ] All possible response codes documented (200, 400, 401, 403, 404, 500)

---

## ✅ Positive Scenarios

- [ ] Happy path with valid data → expected success response
- [ ] Response structure matches schema (all required fields present)
- [ ] Response data is correct (validated against DB)
- [ ] Correct HTTP status code returned

---

## ❌ Negative Scenarios

- [ ] Missing required fields → 400 with field-level errors
- [ ] Invalid field types (string instead of number, etc.) → 400
- [ ] Empty string where value required → 400
- [ ] Boundary values (0, -1, max int, very long strings)

---

## 🔐 Security

- [ ] Unauthenticated request → 401
- [ ] Wrong role / insufficient permissions → 403
- [ ] User cannot access other users' data (IDOR check)
- [ ] SQL injection attempt → handled gracefully
- [ ] XSS in string fields → sanitized

---

## 🚀 Performance

- [ ] Response time < 500ms under normal conditions
- [ ] Endpoint handles 10 concurrent requests without errors

---

## 📦 Data Integrity

- [ ] Created/updated record in DB matches API request
- [ ] No orphaned records after operations
- [ ] Timestamps (created_at, updated_at) are populated correctly
- [ ] Audit log entries created where applicable
