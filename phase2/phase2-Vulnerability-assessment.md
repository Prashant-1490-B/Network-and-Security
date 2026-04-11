# Vulnerability & Exploitation

This repository documents the identification, exploitation, and remediation of security vulnerabilities discovered in a controlled lab environment.

---
# Vulnerabilities:


# 1. SQLi

## Vulnerability Report: SQL Injection (SQLi) - DVWA

## 1. Vulnerability Details
| Attribute | Details |
| :--- | :--- |
| **Vulnerability Type** | SQL Injection (Error-Based / Union-Based) |
| **Location** | `/DVWA/vulnerabilities/sqli/` |
| **Vulnerable Parameter** | `id` (GET Request) |
| **Severity** | **CRITICAL** (High Impact) |
| **Status** | Open / Proof of Concept Provided |

---

## 2. Description
The web application fails to properly sanitize the `id` parameter before passing it to the database query. An attacker can break the SQL syntax by injecting a single quote `'` and appending malicious SQL commands. This allows for unauthorized data access, database schema enumeration, and extraction of sensitive user credentials.

---

## 3. Proof of Concept (PoC)

### Step 1: Initial Authentication Bypass / Boolean Test
By injecting a simple OR logic, the query evaluates to `TRUE` for every row, forcing the application to display all user records in the database.
* **Payload:** `1' OR '1'='1`

<img width="1340" height="776" alt="Screenshot 2026-03-31 194116" src="https://github.com/user-attachments/assets/f3fd0a37-871d-43ad-a08e-109198ff64c8" />


---

### Step 2: Database Name Enumeration
Using a `UNION SELECT` statement, the current database name was identified to confirm the scope of the attack.
* **Payload:** `1' UNION SELECT null, database()-- -`
* **Result:** The database name is `dvwa`.

<img width="1339" height="769" alt="Screenshot 2026-03-31 194354" src="https://github.com/user-attachments/assets/7c7378f9-bc2b-4382-8042-b0144a1c5498" />


---

### Step 3: Current Database User Identification
To assess the level of privileges, the `user()` function was executed to identify the system user interacting with the database.
* **Payload:** `1' UNION SELECT null, user()-- -`
* **Result:** Database is running as `dvwa@localhost`.

<img width="1332" height="805" alt="Screenshot 2026-03-31 194438" src="https://github.com/user-attachments/assets/e3708dd0-45d2-4b86-b750-a1f753a9d197" />

<img width="1686" height="741" alt="Screenshot 2026-04-04 170438" src="https://github.com/user-attachments/assets/32f8008d-1910-4db2-918f-19fcadab4879" />


---

### Step 4: Database Schema Enumeration (Column Discovery)
Accessing the `information_schema` allows an attacker to map the structure of the database. Here, column names for the `users` table were extracted.
* **Payload:** `1' UNION SELECT null, column_name FROM information_schema.columns WHERE table_name='users' LIMIT 0,1-- -`

<img width="1705" height="803" alt="Screenshot 2026-04-04 170514" src="https://github.com/user-attachments/assets/6942eb2e-80fe-4657-b423-083e01549c19" />


---

### Step 5: Final Data Exfiltration (User Credentials)
With the table and column names known, a final payload was used to dump the usernames and MD5 password hashes of all registered users.
* **Payload:** `1' UNION SELECT user, password FROM users-- -`

<img width="1344" height="734" alt="Screenshot 2026-04-04 165811" src="https://github.com/user-attachments/assets/c443dea7-c7d9-4bae-b3d9-9d41e6e75639" />


---

## 4. Impact
* **Full Data Leakage:** Unauthorized access to the entire `users` table.
* **Credential Theft:** Exposure of admin and user password hashes.
* **System Reconnaissance:** Disclosure of database versions, users, and internal structures.

---

## 5. Remediation

### 1. Use Parameterized Queries (Prepared Statements)
Instead of concatenating user input, use prepared statements. This ensures the database treats input as data, not executable code.
```php
$stmt = $pdo->prepare('SELECT first_name, last_name FROM users WHERE user_id = :id');
$stmt->execute(['id' => $_GET['id']]);
$user = $stmt->fetch();
