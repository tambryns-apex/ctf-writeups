# XOR + Hour-Based Encoding

**Category:** Cryptography  
**Difficulty:** Easy/Medium  
**Platform:** Custom/Unknown  

---

# Challenge Overview

We were given a Go program that encodes an input string into a list of bytes. The encoding process involves XORing each byte of the input with the current hour of the day, then adding the same hour value to the result. The encoded output was provided, and the task was to recover the original input (the flag).

The challenge tested understanding of bitwise operations, reversible encryption, and brute forcing small keyspaces.

---

## Tools Used

- Python 3
    
---

## Given Data

- \[149, 139, 122, 136, 131, 60, 59, 95, 138, 141, 133, 123, 95, 122, 149, 146, 146, 95, 107, 150, 124, 149, 123, 122, 145, 141, 123, 77, 129\]
    
- Encoding Go code snippet:
```
package main

import (
	"fmt"
	"time"
)

func main() {
	var input string
	hour := time.Now().Hour()
	fmt.Print("Enter input: ")
	fmt.Scan(&input)
	xorBytes := make([]byte, len(input))
	for i := range input {
		xorBytes[i] = input[i] ^ byte(hour)
	}
	enc := make([]byte, len(input))
	for i := range input {
		enc[i] = xorBytes[i] + byte(hour)
	}

	fmt.Printf("Encoded bytes: %v\n", enc)

}
```

---

## Vulnerability Analysis

The encoding process can be described mathematically for each byte b of the input:

enc\[i\]=(input\[i\]⊕hour)+hourenc\[_i_\]=(input\[_i_\]⊕hour)+hour

Where ⊕ denotes the XOR operation and hour is the current hour (0–23).

To decode, we reverse the process:

input\[i\]=(enc\[i\]−hour)⊕hourinput\[_i_\]=(enc\[_i_\]−hour)⊕hour

The main challenge was the unknown hour value used during encoding. Since the hour ranges from 0 to 23, brute forcing all possibilities is feasible.

---

## Exploitation

I wrote a Python script to brute force all possible hour values and decode the bytes accordingly:
```
encoded_bytes = [149 139 122 136 131 60 59 95 138 141 133 123 95 122 149 146 146 95 107 150 124 149 123 122 145 141 123 77 129]
for hour in range(24):
	decoded = ''.join(chr((b - hour) ^ hour) for b in encoded_bytes)
	print(f"Hour {hour}: {decoded}")
```
---

## Flag

At hour = 23, the decoded output was:

```
Hour 23: ictf{23_days_till_Christmas!}
```
---

## Key Takeaways

- XOR is a reversible operation widely used in cryptography.
    
- Small keyspaces (like the hour of the day) can be brute forced easily.
    
- Time-dependent keys, if not handled carefully, can lead to vulnerabilities.
    
- Understanding the encoding logic and reversing it mathematically is crucial in cryptanalysis.
    
---

## Conclusion

This challenge demonstrated how a simple XOR-based encoding combined with a small keyspace can be reversed by brute force. It highlights the importance of strong, unpredictable keys in encryption and the risks of relying on time-dependent values for security.
