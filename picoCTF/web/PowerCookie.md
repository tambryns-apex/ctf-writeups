# Power Cookie

**Category:** Web Exploitation  
**Difficulty:** Medium  
**Platform:** Custom CTF

---

## Challenge Overview
The challenge involved a website where user privileges appeared to be controlled via cookies.

---

## Initial Recon
- Visited the website and inspected the browser cookies
- Noticed a cookie named `isAdmin` with the value set to `0`
- The cookie name suggested it might control administrative access

---

## Vulnerability Analysis
The application relied on a client-side cookie (`isAdmin`) to determine user privileges without proper server-side validation. This allowed privilege escalation by simply modifying the cookie value.

---

## Exploitation
Changed the `isAdmin` cookie value from `0` to `1` in the browser’s developer tools and refreshed the page. The site recognized the user as an admin and revealed the flag.

---

## Result
Successfully gained administrative access and retrieved the flag by manipulating the cookie value.

# Flag
```
picoCTF{gr4d3_A_c00k13_5d2505be}
```

---

## Key Takeaways
- Never trust client-side data for authorization decisions
- Implement server-side checks for user roles and permissions
- Use secure, tamper-proof methods such as signed cookies or session tokens
