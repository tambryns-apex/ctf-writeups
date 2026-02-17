# Vault Door Training: Password Extraction from Source Code

**Category:** Reverse Engineering / Binary Exploitation  
**Difficulty:** Very Easy  
**Platform:** picoCTF

---

## Challenge Overview
The challenge simulates a vault door controlled by a Java program requiring a password for access. The source code for the vault door's program is provided, and the task is to determine the correct password to open the vault.

---

## Initial Recon
- The program prompts the user to enter a vault password.
- It expects the password to be in the format `picoCTF{...}`.
- The program extracts the inner content of the flag by removing the prefix `picoCTF{` and the trailing `}`.
- The extracted string is then passed to the `checkPassword` method for verification.

---

## Source Code Analysis
```java
public boolean checkPassword(String password) {
    return password.equals("w4rm1ng_Up_w1tH_jAv4_000HPpgh7Ph");
}
```
- The password check compares the input string against a hardcoded string: "w4rm1ng_Up_w1tH_jAv4_000HPpgh7Ph"
- The check is case-sensitive and exact.
- Since the program strips the picoCTF{} wrapper before checking, the full flag format is: picoCTF{w4rm1ng_Up_w1tH_jAv4_000HPpgh7Ph}

---

## Exploitation
- No actual exploitation is required beyond reading and understanding the source code.
- The password is explicitly present in the source, making this a warmup to demonstrate how source code disclosure can lead to compromise.

---

## Flag
```
picoCTF{w4rm1ng_Up_w1tH_jAv4_000HPpgh7Ph}
```

---

## Key Takeaways
- Embedding passwords directly in source code is insecure, especially if the code can be leaked.
- Always avoid hardcoding secrets; consider secure storage or hashing methods.
- This warmup highlights the importance of code confidentiality and secure authentication design.
