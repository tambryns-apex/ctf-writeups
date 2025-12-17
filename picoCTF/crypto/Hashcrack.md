 # Weak Password Hashes

**Category:** Cryptography  
**Difficulty:** Easy  
**Platform:** picoCTF

---

## Challenge Overview

A company stored a secret message on a server that was compromised due to the use of weakly hashed passwords. By connecting to the service, we are given multiple password hashes and must crack them to ultimately retrieve the hidden flag.

---

## 🛠️ Tools Used

* `nc` (netcat)
* Basic hash recognition
* Common weak password knowledge

---

## 🔍 Connecting to the Server

```bash
nc verbal-sleep.picoctf.net 51328
```

Once connected, the server interactively prompts for passwords corresponding to given hashes.

---

## Hash Cracking Walkthrough

### MD5 Hash

```
482c811da5d5b4bc6d497ffa98491e38
```

* **Identified as:** MD5 (32 hex characters)
* **Password:** `password123`
* **Reasoning:** Extremely common weak password; MD5 is fast and unsalted

Result:

```
Correct! You've cracked the MD5 hash with no secret found!
```

---

### SHA-1 Hash

```
b7a875fc1ea228b9061041b7cec4bd3c52ab3ce3
```

* **Identified as:** SHA-1 (40 hex characters)
* **Password:** `letmein`
* **Reasoning:** SHA-1 is deprecated and vulnerable; password appears in common wordlists

Result:

```
Correct! You've cracked the SHA-1 hash with no secret found!
```

---

### SHA-256 Hash (Final)

```
916e8c4f79b25028c9e467f1eb8eee6d6bbdff965f9928310ad30a8d88697745
```

* **Identified as:** SHA-256 (64 hex characters)
* **Password:** `qwerty098`
* **Reasoning:** Strong hash algorithm, but completely undermined by a weak password and no salting

Result:

```
Correct! You've cracked the SHA-256 hash with a secret found.
```

---

## Flag

```
picoCTF{UseStr0nG_h@shEs_&PaSswDs!_93e052d7}
```

---

## Lessons Learned

* Weak passwords negate the security of even strong hashing algorithms
* MD5 and SHA-1 should never be used for password storage
* Password hashes must be salted and processed with slow, adaptive algorithms

  * Examples: bcrypt, scrypt, Argon2

---

## Conclusion

This challenge demonstrates how attackers can trivially recover sensitive information when weak passwords are used. Proper password hygiene and modern hashing practices are critical for real-world security.


