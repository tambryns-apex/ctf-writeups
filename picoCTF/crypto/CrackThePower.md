# RSA Low-Exponent Attack

**Category:** Cryptography  
**Difficulty:** Medium  
**Platform:** picoCTF  

---

## Challenge Overview

We are given an RSA-encrypted message. The modulus `n` is extremely large, making factorization infeasible with current methods. Instead, the vulnerability lies in how RSA was configured.

The goal is to analyze the provided parameters consider alternative attacks and recover the plaintext flag.

---

## Tools Used

- Python 3
- `gmpy2`
- `pycryptodome`

---

## Given Parameters

- RSA modulus `n`
- Public exponent `e = 20`
- Ciphertext `c`

---

## Vulnerability Analysis

RSA encryption is defined as:

```
c = m^e mod n
```

If the plaintext `m` is small enough such that:

```
m^e < n
```

then no modular reduction occurs, meaning:

```
c = m^e
```


In this case, the plaintext can be recovered by computing the e-th integer root of `c`.

Key issues:
- The public exponent `e = 20` is unusually small
- The message is unpadded (textbook RSA)
- The plaintext fits entirely below `n`

This makes the system vulnerable to a low-exponent RSA attack.

---

## Exploitation

Using Python and the `gmpy2` library, the 20th integer root of the ciphertext was computed.

```python
import gmpy2
from Crypto.Util.number import long_to_bytes

m, exact = gmpy2.iroot(c, e)
print(long_to_bytes(m))
```

The root was exact, confirming that the ciphertext was simply m^e without modular reduction.

---

## Flag

```
picoCTF{UseStr0nG_h@shEs_&PaSswDs!_93e052d7}
```

---

## Key Takeaways

- RSA without padding is insecure
- Small public exponents can completely break RSA encryption
- Proper RSA implementations must use padding schemes such as OAEP
- Cryptographic strength depends on correct configuration, not just key size

---

## Conclusion

This challenge demonstrates how RSA can be compromised without factoring the modulus when weak parameters are used. Even strong cryptographic algorithms fail when implemented incorrectly.
