# OverTheWire: Leviathan — Notes (Levels 0–7, No Spoilers)

**Platform:** [OverTheWire Leviathan](https://overthewire.org/wargames/leviathan/)  
**Difficulty:** Beginner (with “real” Linux security patterns)  
**Status:** Completed all levels (0–7)

---

I worked through Leviathan to get hands-on practice with classic Linux security concepts: **information disclosure**, **SUID behavior**, **dynamic analysis**, **TOCTOU race conditions**, **symlink pitfalls**, **encoding vs encryption**, and **small keyspace brute-force**.  
These notes intentionally avoid step-by-step solutions, exact commands, and spoilers.

---

## Table of Contents

| Level | Core Skill |
|-------|------------|
| [0 - 1](#level-0---1) | Information disclosure via leftover artifacts |
| [1 - 2](#level-1---2) | SUID discovery + dynamic analysis mindset |
| [2 - 3](#level-2---3) | TOCTOU (check vs use) + symlink race concepts |
| [3 - 4](#level-3---4) | Tracing tools vs SUID behavior |
| [4 - 5](#level-4---5) | Bits-to-ASCII decoding (encoding vs encryption) |
| [5 - 6](#level-5---6) | Insecure `/tmp` usage + symlink redirection |
| [6 - 7](#level-6---7) | Small keyspace brute-force + operational tradeoffs |

---

## Level 0 - 1

**Goal:** Progress to the next user account.

**Core idea:** Information disclosure through leftover user data / backups.

**What I practiced:**
- Enumerating a home directory properly (including hidden paths).
- Prioritizing “interesting” files by **ownership** and **permissions**.
- Extracting useful hints from user-generated artifacts without brute force.

**Real-world link:** Backups, browser artifacts, and forgotten files often leak credentials or internal URLs in real environments.

---

## Level 1 - 2

**Goal:** Progress via a privileged program.

**Core idea:** SUID binaries + dynamic analysis mindset.

**What I practiced:**
- Spotting SUID permissions (`s` bit) during enumeration.
- Identifying binary properties (ELF, architecture, dynamic linking).
- Observing runtime behavior to understand input validation (without source).
- Recognizing when a spawned shell may drop privileges (security hardening).

**Real-world link:** Misconfigured SUID utilities are a common local privilege escalation path.

---

## Level 2 - 3

**Goal:** Progress via a privileged helper program.

**Core idea:** **TOCTOU** (Time-of-check → time-of-use) + SUID.

**What I practiced:**
- Recognizing the risky pattern: “check permissions first, use later”.
- Confirming the flow conceptually:
  - a “check” happens under one identity/context
  - the “use” happens later (sometimes under higher privileges)
- Understanding why the same path can be “safe during check” but “different during use”.

**Why it works (high-level):**
If a program checks an object at time \(T_1\) and uses it at time \(T_2\), an attacker may replace what that path points to in the window between \(T_1\) and \(T_2\). With SUID involved, the “use” step can run with higher privileges than the “check” step.

**Real-world link:** TOCTOU bugs show up in installers, log readers, backup scripts, and “helper” tools that touch user-controlled paths.

---

## Level 3 - 4

**Goal:** Progress via a SUID binary.

**Core idea:** Tracing tools vs SUID (why behavior differs).

**What I practiced:**
- Extracting expected values by observing comparisons at runtime.
- Verifying identity/privileges after success (`whoami` / `id`).
- Understanding that running SUID programs under tracers (e.g., `ltrace`/`strace`) can **disable or alter** privilege escalation for security reasons.

**Real-world link:** Debugging/observability can change behavior in privileged contexts (important for both attackers and defenders).

---

## Level 4 - 5

**Goal:** Progress via a SUID binary.

**Core idea:** Binary/bit encoding vs encryption.

**What I practiced:**
- Recognizing that groups of `0`/`1` often represent **ASCII bytes** (encoding).
- Converting 8-bit groups into readable text.
- Distinguishing:
  - **Encoding** = reversible representation (no secret key)
  - **Encryption** = secrecy with a key

**Real-world link:** Logs/IoT/protocol dumps often carry data in encoded forms (base64, hex, bitstrings) that aren’t “crypto”.

---

## Level 5 - 6

**Goal:** Progress via a SUID binary.

**Core idea:** Insecure log handling in `/tmp` + symlink redirection.

**What I practiced:**
- Noticing a privileged program reads a predictable file from a world-writable directory.
- Understanding why temp paths in `/tmp` are dangerous without safe patterns (secure creation, ownership checks, avoiding symlink-follow).
- Using a symlink conceptually to redirect a privileged read from an attacker-controlled name to a protected target.

**Real-world link:** This is a common class of bugs in “helper” tools that read/write predictable files in `/tmp`.

---

## Level 6 - 7

**Goal:** Progress via a SUID binary.

**Core idea:** Small keyspace brute-force + operational tradeoffs.

**What I practiced:**
- Recognizing a fixed-size numeric keyspace (4 digits) and estimating brute-force cost.
- Automating repeated attempts safely:
  - stop conditions
  - avoiding terminal spam / performance issues
- Confirming privilege change after success (new shell / identity checks).

**Real-world link:** Brute-forcing is often limited by lockouts/rate limits in real systems, but small local keyspaces still matter.

---

## Lessons Learned

- **Most realistic concept:** TOCTOU + SUID mismatches (Levels 2–3 style) — it’s a real software class of bug.
- **Most important mindset shift:** “What is the program *checking* vs what does it *actually use*?”
- **Practical habit:** Always verify effective identity/privileges after “success”, and remember tooling (tracers) can change behavior.

---

## Skills Demonstrated

- **Linux fundamentals:** permissions, ownership, enumeration
- **Binary awareness:** ELF basics, dynamic behavior observation
- **Privilege escalation patterns:** SUID risks, symlink pitfalls, TOCTOU
- **Data handling:** encoding vs encryption
- **Automation:** brute-force scripting and operational hygiene
