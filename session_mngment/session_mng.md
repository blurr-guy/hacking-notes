# Session Management

---
## Session Management Lifecycle:
```session management

session creation ---> session tracking --> session timeout/expiry ---> session termination


```
## IAAA in Session Management

### identification 
Identification is the process of verifying who the user is. This starts with the user claiming to be a specific identity. 

### Authentication
Authentication is the process of ensuring that the user is who they say they are. Where in identification, you provide a username, for authentication, you provide proof that you are who you say you are.

### Authorization
Authorisation is the process of ensuring that the specific user has the rights required to perform the action requested. For example - A normal user trying to access admin panel

### Accountability
Accountability is the process of creating a record of the actions performed by users. We should track the user's session and log all actions performed using the specific session. 

## Cookie-Based Session Management
After authentication server sent a set-cookie header in response ---↓
Then user's browser create a session cookie field and store the value of the session ----> 

set-cookie ( A response header has many attributes which are responsible for safe and secure cookie storage and save it from theft)

```request
POST /api/login HTTP/1.1
Host: example.com
Content-Type: application/json

{"username": "dhakad", "password": "hunter2"}
```
      ↓

```Response
HTTP/1.1 200 OK
Set-Cookie: sessionid=8f14e45fceea167a5a36dedd4bad3a7d; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=3600
Content-Type: application/json

{"message": "Login successful", "user": "dhakad"}
```

## Token-Based Session Management
Token-based session management is a relatively new concept. Instead of using the browser's automatic cookie management features, it relies on client-side code for the process. 
After authentication, the web application provides a token within the request body. Using client-side JavaScript code, this token is then stored in the browser's **LocalStorage**. 

JSON Web Tokens (JWT)---> One of the token based session management mechanism. It is send as **Authorization: Bearer <token>** in the request body...

```Request
POST /api/login HTTP/1.1
Host: example.com
Content-Type: application/json

{"username": "dhakad", "password": "hunter2"}
```
       ↓

```Response
HTTP/1.1 200 OK
Content-Type: application/json

{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjoxMDQyLCJyb2xlIjoidXNlciIsImlhdCI6MTcyMzM4MjQwMCwiZXhwIjoxNzIzMzg2MDAwfQ.9f8a7d6c5b4a3e2f1d0c9b8a7f6e5d4c3b2a1908",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

## vulnerabilities in session creation
- Weak session vakues
- Controllable Session Values	
- Session Fixation
- Insecure Session Transmission

## Session Tracking
- Authorisation Bypass
- Insufficient Logging
- Session Expiry
- Session Termination




































