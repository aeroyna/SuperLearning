# HTTP Methods, Headers & Status Codes 📨

**HTTP (HyperText Transfer Protocol)** is a stateless, text-based request-response protocol used to communicate between clients (browsers) and servers.

---

## 1. The Anatomy of a Request/Response 🧬

### Request
```http
GET /index.html HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0
Accept: text/html
```
*   **Method**: `GET`
*   **Path**: `/index.html`
*   **Version**: `HTTP/1.1`
*   **Headers**: Key-value pairs.

### Response
```http
HTTP/1.1 200 OK
Date: Mon, 23 May 2022 22:38:34 GMT
Content-Type: text/html; charset=UTF-8
Content-Length: 138

<html>...</html>
```
*   **Status Line**: Version, Code (`200`), Reason Phrase (`OK`).

---

## 2. HTTP Methods (Verbs) 🗣️

| Method | Description | Safe? | Idempotent? |
| :--- | :--- | :--- | :--- |
| **GET** | Retrieve a representation of a resource. No side effects. | ✅ Yes | ✅ Yes |
| **POST** | Submit data to be processed (e.g., create a new user). | ❌ No | ❌ No |
| **PUT** | Replace a resource entirely (update). | ❌ No | ✅ Yes |
| **PATCH** | Partial modification of a resource. | ❌ No | ❌ No* |
| **DELETE** | Delete a resource. | ❌ No | ✅ Yes |
| **HEAD** | Same as GET but returns NO body (headers only). Useful for checking validity/size. | ✅ Yes | ✅ Yes |
| **OPTIONS**| Returns supported HTTP methods (used in CORS). | ✅ Yes | ✅ Yes |

*   **Safe**: The operation does not modify the state of the server (Read-only).
*   **Idempotent**: Making the same request multiple times has the same effect as making it once. (e.g., `DELETE /user/1` is idempotent; the user is gone whether you do it once or 10 times. `POST /user` is NOT; it creates 10 users).
*   *\*PATCH is theoretically not idempotent, but often implemented as such.*

---

## 3. Status Codes 🚦

### 1xx: Informational
*   `100 Continue`: Client should continue sending the body.

### 2xx: Success
*   `200 OK`: Standard success.
*   `201 Created`: Resource created (usually after POST).
*   `204 No Content`: Success, but no body returned (common in DELETE).

### 3xx: Redirection
*   `301 Moved Permanently`: The URL has changed forever. Browser caches this redirection.
*   `302 Found` (Temporary): The URL changed temporarily. Don't cache.
*   `304 Not Modified`: Cached version is still valid (saves bandwidth).

### 4xx: Client Error
*   `400 Bad Request`: Malformed syntax.
*   `401 Unauthorized`: Authentication required.
*   `403 Forbidden`: Authenticated, but no permission.
*   `404 Not Found`: Resource doesn't exist.
*   `429 Too Many Requests`: Rate limiting.

### 5xx: Server Error
*   `500 Internal Server Error`: Generic server crash/bug.
*   `502 Bad Gateway`: Upstream server sent an invalid response.
*   `503 Service Unavailable`: Overloaded or maintenance.

---

## 4. Key Headers 🏷️

### Content Negotiation
*   `Accept`: What format the client wants (`application/json`).
*   `Content-Type`: What format the body is in.

### Caching
*   `Cache-Control`: Directives for caching mechanisms (`max-age=3600`, `no-store`).
*   `ETag`: A unique fingerprint of the resource version for validation.

### Security
*   `Authorization`: Credentials (`Bearer <token>`).
*   `Cookie`: Stored state sent back to server.
