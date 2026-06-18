# JWT Authentication Notes (Access Token, Refresh Token, Sessions)

## 1. Why Do We Need Tokens?

Without tokens, the user would have to send:

```text
username + password
```

with every API request.

Example:

```text
Get Profile -> username + password
Get Orders  -> username + password
Update User -> username + password
```

This is insecure and inefficient.

Instead:

```text
Login
  ↓
Server verifies credentials
  ↓
Server issues tokens
```

The client then sends tokens instead of username/password.

---

# 2. What is an Access Token?

An Access Token is a short-lived token used to access protected APIs.

Example:

```text
Access Token Validity = 15 minutes
```

Request:

```http
Authorization: Bearer <access_token>
```

Server checks:

```text
Valid?
  Yes -> Allow Request
  No  -> Reject Request
```

### Why Short-Lived?

If an attacker steals the Access Token:

```text
Damage is limited
```

because it expires quickly.

---

# 3. What is a Refresh Token?

A Refresh Token is used to obtain a new Access Token.

Example:

```text
Access Token  = 15 minutes
Refresh Token = 30 days
```

Flow:

```text
Access Token Expired
        ↓
Send Refresh Token
        ↓
Server Verifies
        ↓
New Access Token
```

The user remains logged in without entering credentials again.

---

# 4. What Does "Stateless" Mean?

Stateless means:

```text
Server does not remember logged-in users in memory.
```

Instead:

```text
Every request contains a token.
```

Example:

```http
Authorization: Bearer token
```

The server validates the token and processes the request.

Benefits:

* Easier scaling
* Less server memory usage
* Works well with multiple servers

---

# 5. Access Token vs Refresh Token

| Feature               | Access Token   | Refresh Token              |
| --------------------- | -------------- | -------------------------- |
| Purpose               | Access APIs    | Generate new Access Tokens |
| Lifetime              | Short          | Long                       |
| Sent on Every Request | Yes            | No                         |
| If Stolen             | Limited Damage | Higher Risk                |
| Typical Validity      | 5–30 min       | Days/Weeks                 |

---

# 6. Refresh Token Rotation

Without Rotation:

```text
Refresh1
```

is used for the entire session.

With Rotation:

```text
Refresh1
   ↓
Refresh2
   ↓
Refresh3
   ↓
Refresh4
```

Every refresh operation:

1. Invalidates old Refresh Token
2. Creates new Refresh Token
3. Creates new Access Token

Benefits:

```text
Stolen old refresh tokens become useless.
```

---

# 7. Sliding Expiration

Every refresh generates a new Refresh Token with a fresh expiry.

Example:

```text
Day 1  -> Refresh1 (30 days)
Day 5  -> Refresh2 (30 days)
Day 10 -> Refresh3 (30 days)
```

Expiry keeps moving forward.

Result:

```text
Active users remain logged in.
```

Common in:

* Social Media
* Streaming Platforms
* Consumer Applications

---

# 8. Absolute Session Expiration

Server tracks:

```text
Session Start Time
```

Example:

```text
Maximum Session = 30 Days
```

Even if Refresh Tokens keep rotating:

```text
Day 1  -> Login
Day 29 -> Refresh Token Rotated
Day 30 -> Force Login
```

User must authenticate again.

Common in:

* Banking Applications
* Financial Systems
* Enterprise Software

---

# 9. Difference Between Refresh Expiry and Session Expiry

Many developers confuse these.

## Refresh Token Expiry

Question:

```text
Can I obtain a new Access Token?
```

If refresh token expired:

```text
No
```

User logs in again.

---

## Session Expiry

Question:

```text
Am I allowed to stay logged in?
```

If absolute session limit reached:

```text
No
```

User logs in again.

Even if refresh token rotation is working.

---

# 10. Bank Example

Configuration:

```text
Access Token = 15 min
Refresh Rotation = Yes
Absolute Session = 30 days
```

Flow:

```text
Login
  ↓
Access + Refresh
  ↓
Tokens keep rotating
  ↓
Day 30 reached
  ↓
Force Login
```

Priority:

```text
Security
```

---

# 11. Instagram Example

Configuration:

```text
Access Token = Short
Refresh Rotation = Yes
Sliding Expiration = Yes
Absolute Session = Very Long or Less Strict
```

Flow:

```text
Login
  ↓
Access1 + Refresh1
  ↓
Access2 + Refresh2
  ↓
Access3 + Refresh3
  ↓
...
```

User can stay logged in for months.

Priority:

```text
User Convenience
```

---

# 12. Storage Options

## Memory (React State)

```js
const [token, setToken] = useState();
```

Pros:

```text
Very Secure
```

Cons:

```text
Lost on Refresh
```

---

## Session Storage

```js
sessionStorage.setItem(...)
```

Pros:

```text
Survives Page Refresh
```

Cons:

```text
Lost When Tab Closes
```

---

## Local Storage

```js
localStorage.setItem(...)
```

Pros:

```text
Survives Browser Restart
```

Cons:

```text
Can Be Read By JavaScript
```

Higher XSS risk.

---

## Production Standard

Common Secure Setup:

```text
Access Token  -> Memory
Refresh Token -> HttpOnly Secure Cookie
```

Benefits:

```text
JavaScript cannot read HttpOnly cookies.
```

---

# Interview Answer

Q: Why not make Access Token valid for 30 days?

Answer:

Because if an attacker steals the token:

```text
They get 30 days of access.
```

Instead:

```text
Short Access Token
+
Long Refresh Token
+
Refresh Rotation
```

provides better security while maintaining user experience.

---

# Final Mental Model

Access Token:

```text
Temporary ID Card
```

Refresh Token:

```text
Membership Card Used To Get New ID Cards
```

Session:

```text
Overall Permission To Stay Inside The Building
```

Access Token Expires:

```text
Get New Access Token Using Refresh Token
```

Refresh Token Expires:

```text
Login Again
```

Absolute Session Expires:

```text
Login Again
```

Even if Refresh Tokens are rotating successfully.
