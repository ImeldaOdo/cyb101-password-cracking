# Project 2: Password Hash Cracking with John the Ripper

This project documents my hands-on experience with password hash cracking using John the Ripper, modeled after real-world data breaches like the 2012 LinkedIn Hack and 2016 Yahoo Data Breach. Beyond simply running commands, I worked to understand *why* certain attack strategies work — and what that means for how organizations should protect user credentials.

This helped me move from "I ran a tool" to **"I understand how attackers think — and how defenders should respond."**

---

## Overview

Working with a fictional dataset of 1,000 leaked username/password hashes (`cp_leak.txt`), I used multiple cracking strategies to recover plaintext passwords from their md5crypt hashes. The goal was to understand how real password cracking works so those techniques can inform better security practices.

---

## Dataset Statistics

| Password Length | Count |
|----------------|-------|
| 1 character | 2 |
| 2 characters | 13 |
| 3 characters | 87 |
| 4 characters | 284 |
| 5 characters | 487 |
| 6 characters | 107 |
| 7 characters | 18 |
| 8 characters | 2 |

**Key insight:** 77% of passwords were 4-5 characters long — which directly shaped my attack strategy. Shorter passwords have exponentially fewer combinations, making them the lowest-hanging fruit.

---

## Attack Strategy

Rather than blindly running every possible combination, I analyzed the dataset statistics first and attacked strategically.

### Attack 1: Wordlist Attack with rockyou.txt

The `lower.lst` wordlist provided was intentionally small. I sourced `rockyou.txt` — a 14 million password wordlist compiled from real data breaches — as a significantly more effective alternative.

```bash
john --wordlist=/home/codepath/unit3/rockyou.txt /home/codepath/unit3/cp_leak.txt
```

**Why this works:** Many users choose passwords that already exist in previous breach datasets. If a password appeared in the LinkedIn or Adobe breach, it likely appears in rockyou.txt.

**Result:** Cracked ~97 hashes.

---

### Attack 2: Built-in Ruleset (Single)

John's built-in rulesets apply transformations to wordlist entries — capitalizing letters, appending numbers, substituting characters (e.g., `hello` → `Hello`, `h3llo`, `hello1`). This dramatically expands coverage without needing a larger wordlist.

```bash
john --wordlist=/home/codepath/unit3/rockyou.txt --rules=Single /home/codepath/unit3/cp_leak.txt
```

**Why this works:** Users frequently make predictable modifications to common words to satisfy password requirements (uppercase first letter, number at the end). Rules automate guessing these patterns.

---

### Attack 3: Custom Mask Attack (4-character passwords)

Given that 284 passwords were exactly 4 characters long, a targeted mask attack was more efficient than a full brute force. `?l` represents any lowercase letter.

```bash
john --mask=?l?l?l?l /home/codepath/unit3/cp_leak.txt
```

**Why this works:** 4-letter lowercase combinations = 26⁴ = 456,976 possibilities. On md5crypt this is feasible in minutes, whereas 8-character combinations (26⁸ = 208 billion) would take years.

**Result:** Cracked ~57 additional hashes.

---

## Results

| Attack Method | Hashes Cracked |
|--------------|---------------|
| rockyou.txt wordlist | ~97 |
| 4-letter mask attack | ~57 |
| Additional attempts | ~9 |
| **Total** | **163 / 1000** |

![John Show Output](screenshots/john-show-output.png)

---

## Key Security Takeaways

**What makes a password hard to crack:**
- **Length** — every extra character multiplies combinations exponentially
- **Complexity** — mixing uppercase, lowercase, numbers and symbols increases the character pool
- **Uniqueness** — passwords not found in any breach dataset can't be cracked by wordlist attacks

**This maps directly to standard password requirements:**

Most websites require 8+ characters, mixed case, numbers, and symbols — not arbitrarily, but because each requirement specifically counters a cracking technique. Short passwords fall to mask attacks. Common words fall to wordlist attacks. Predictable variations (Hello1!) fall to rule-based attacks.

**The real-world implication:**
Even with industry-standard hashing (md5crypt in this case), weak passwords are recoverable in minutes. This is why modern security practices include breach monitoring, password managers, and multi-factor authentication — hashing alone is not enough.

---

## Tools Used

- **John the Ripper** — password cracking framework
- **rockyou.txt** — real-world breach wordlist (14M passwords)
- **Ubuntu VM** (Azure Labs) — cracking environment

---

## Connection to Cloud Security

This project connects directly to cloud security practices. In AWS environments, credential exposure is one of the leading causes of breaches — whether through leaked IAM keys, weak console passwords, or compromised EC2 instance credentials. Understanding how quickly weak passwords are recovered reinforces why cloud security frameworks like AWS IAM enforce strong password policies, credential rotation, and MFA by default.

See also: [Project 1 — EC2 Web Server Deployment & Troubleshooting](../project-1-troubleshooting.md)

---

*Part of my CYB101 coursework at CodePath — building toward a career in cloud security.*
