# CTF Writeups

This repository contains my **CTF / wargame writeups** and learning notes as I build hands-on cybersecurity skills. I focus on documenting **how I approached each challenge** (not just the final answer), with short “why it works” explanations and real-world security takeaways.

## What’s inside (current)
- **OverTheWire — Bandit (Levels 0–33)** (completed)  
  - Full writeup: [`bandit-level-0-34.md`](./bandit-level-0-34.md)

## Highlights from my Bandit writeup
These are the core skills demonstrated across the levels:

- **Linux fundamentals**: permissions, special filenames, hidden files, finding files safely
- **Text processing & pipelines**: `grep`, `sort | uniq`, `diff`, `tr`, redirections
- **File analysis**: `file`, `strings`, `xxd`, recursive unpacking
- **Networking basics**: `ssh`, `nc`, `ncat --ssl`, `nmap`, raw TCP interactions
- **Privilege escalation concepts**: SUID mechanics, cron trust boundaries, shell escapes (GTFOBins-style)
- **Git / version-control security**: history forensics, branches, tags, “secrets remain in history” lessons
- **Automation mindset**: loops, narrowing signals, reducing manual work

## Writeup format (how I structure solutions)
For each challenge, I try to include:
- **Goal**: what needs to be achieved
- **Steps**: minimal commands/actions that solved it
- **Why it works**: short explanation of the underlying mechanism
- **Real-world takeaway**: what this maps to in real systems
- **Defense / Mitigation**: how to prevent or reduce this class of issue

## Roadmap (what I’m adding next)
Based on what I’ve already done (Bandit), the next logical steps are:
- **More web security labs** (OWASP Top 10 basics): XSS, SQLi, SSRF, IDOR, auth/session issues — on legal training platforms only
- **More “defender mindset” notes**: mitigations, secure configs, and common mistakes
- **Better navigation**: splitting large writeups into smaller files by level/topic for easier reading

## Legal / Ethical disclaimer
All content here is for **legal security training only** (CTFs, wargames, and labs with explicit permission).  
I do not publish or target real systems, and I do not include private data from any real environment.
