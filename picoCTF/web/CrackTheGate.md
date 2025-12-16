# Login Bypass via SQL Injection

**CTF:** picoCTF  
**Category:** Web Exploitation  
**Difficulty:** Easy  

---

## Challenge Overview
The challenge presented a basic login form requiring authentication to access
a protected admin page.

---

## Initial Recon
- Login form contained `username` and `password` fields
- Form submitted a POST request to a `/login` endpoint
- Error messages differed depending on input values, suggesting server-side
  input handling issues

---

## Vulnerability Analysis
The application constructed SQL queries using unsanitized user input. This allowed
SQL injection via the `username` parameter, as logical conditions could be injected
into the authentication query.

This behavior was inferred from inconsistent error responses and the lack of
input validation.

---

## Exploitation
By supplying the following payload in the username field:
the SQL query’s authentication condition always evaluated to true.  
The trailing comment prevented the password check from executing.

---

## Result
Authentication was bypassed successfully, granting access to the protected admin
page and revealing the flag.

---

## Mitigation / Lessons Learned
- Use parameterized queries (prepared statements)
- Avoid exposing detailed database error messages
- Implement strict server-side input validation

