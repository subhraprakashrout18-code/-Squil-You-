# Challenge Write-Up: Squil You!

**Category:** Web / SQL Injection
**Target:** http://172.168.168.114:5000
**Flag Format:** `IIITBh{...}`

---

## 1. Reconnaissance

Accessed the web application running at:

```
http://172.168.168.114:5000
```

The app features an input field (such as a login or search form) that accepts user-supplied text and passes it into a backend SQL query without proper sanitization — making it vulnerable to SQL Injection.

---

## 2. Exploitation

Entered the standard authentication bypass payload into the vulnerable input field:

```sql
' OR '1'='1
```

(Depending on the field, this was submitted as the username, password, or search parameter — e.g. `SQL' OR '1'='1`.)

### Why it works

The application likely builds a query similar to:

```sql
SELECT * FROM users WHERE username = '<input>' AND password = '<input>';
```

By injecting `' OR '1'='1`, the resulting query becomes:

```sql
SELECT * FROM users WHERE username = '' OR '1'='1' AND password = '';
```

- The injected `'` closes the original string literal early.
- `OR '1'='1'` appends a **tautology** — a condition that is always `TRUE`.
- This causes the `WHERE` clause to evaluate as true for **every row** in the table, regardless of the actual credentials supplied.

As a result, the backend database authenticates the request (or returns all matching records) without requiring valid credentials, bypassing the intended access control logic.

---

## 3. Result

Submitting the payload successfully bypassed authentication / exposed backend data, revealing the flag:

```
IIITBh{...}
```

---

## 4. Root Cause & Remediation

**Root cause:** User input was concatenated directly into a SQL query string instead of being treated as data.

**Recommended fixes:**
- Use **parameterized queries / prepared statements** instead of string concatenation.
- Apply an **ORM** with built-in query sanitization.
- Enforce **input validation** and **least-privilege** database accounts.
- Implement **WAF rules** or input filtering as a defense-in-depth measure (not a substitute for fixing the root cause).

---

## 5. Summary

| Step | Action |
|------|--------|
| 1 | Identified vulnerable input field on the login/search form |
| 2 | Injected `' OR '1'='1` payload |
| 3 | Query tautology bypassed authentication logic |
| 4 | Flag retrieved: `IIITBh{...}` |
