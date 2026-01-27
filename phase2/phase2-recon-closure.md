### Scope Validation
All reconnaissance activities were performed strictly within the defined engagement scope.
Testing was limited to the DVWA application exposed on TCP port 80, with no interaction outside
authorized URLs or services.

### Asset Understanding
The target is a PHP-based web application hosted on Apache/2.4.66 (Debian), exposed over
unencrypted HTTP. The application implements form-based authentication with credentials
submitted via HTTP POST and session management handled through a cookie-based mechanism
(PHPSESSID).

### Authentication & Session Observations
Authentication requests are processed by the `/DVWA/login.php` endpoint. Successful login
results in an HTTP 302 redirect and issuance of a session cookie. Credentials and session
identifiers are observable in plaintext network traffic due to the absence of TLS.

### Reconnaissance Outcome
No vulnerabilities are concluded in this phase. Observations related to transport security,
credential handling, session management, and input parameters have been documented to
support controlled testing in subsequent phases.

### Phase Closure
PHASE 2.1 successfully established the application stack, authentication flow, session behavior,
and visible attack surface, enabling systematic input testing and exploitation in PHASE 2.2.
