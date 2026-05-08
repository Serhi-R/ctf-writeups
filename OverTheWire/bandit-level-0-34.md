# OverTheWire: Bandit - Complete Write-up (Levels 0-33)

**Platform:** [OverTheWire Bandit](https://overthewire.org/wargames/bandit/)
**Difficulty:** Beginner - Intermediate
**Status:** Completed all 34 levels

---

I worked through all 34 levels of Bandit as a way to sharpen my Linux fundamentals and get hands-on experience with real security concepts - file permissions, privilege escalation, network sockets, cryptography, and version control forensics. Every solution below reflects how I actually approached each challenge, not just the answer.

---

## Table of Contents

| Level | Core Skill |
|-------|------------|
| [0 - 1](#level-0--1) | SSH login, basic file reading |
| [1 - 2](#level-1--2) | Special filename handling (dash) |
| [2 - 3](#level-2--3) | Filenames with spaces |
| [3 - 4](#level-3--4) | Hidden files |
| [4 - 5](#level-4--5) | File type identification with `file` |
| [5 - 6](#level-5--6) | `find` with multiple filters |
| [6 - 7](#level-6--7) | `find` by user, group, size |
| [7 - 8](#level-7--8) | `grep` in large files |
| [8 - 9](#level-8--9) | `sort` + `uniq` pipeline |
| [9 - 10](#level-9--10) | `strings` + `grep` on binaries |
| [10 - 11](#level-10--11) | Base64 decoding |
| [11 - 12](#level-11--12) | ROT13 cipher |
| [12 - 13](#level-12--13) | Recursive decompression from hexdump |
| [13 - 14](#level-13--14) | SSH key authentication |
| [14 - 15](#level-14--15) | Netcat and raw TCP sockets |
| [15 - 16](#level-15--16) | SSL/TLS with `ncat` |
| [16 - 17](#level-16--17) | Port scanning + scripted interaction |
| [17 - 18](#level-17--18) | File diffing |
| [18 - 19](#level-18--19) | Non-interactive SSH command execution |
| [19 - 20](#level-19--20) | SUID binaries |
| [20 - 21](#level-20--21) | Localhost client-server interaction |
| [21 - 22](#level-21--22) | Cron job inspection |
| [22 - 23](#level-22--23) | MD5-based dynamic path prediction |
| [23 - 24](#level-23--24) | Privilege escalation via cron spool |
| [24 - 25](#level-24--25) | Brute-force with piped input |
| [25 - 26](#level-25--26) | Restricted shell escape via `more` and `vi` |
| [26 - 27](#level-26--27) | SUID binary exploitation |
| [27 - 28](#level-27--28) | Git clone over SSH |
| [28 - 29](#level-28--29) | Git commit history forensics |
| [29 - 30](#level-29--30) | Git remote branch enumeration |
| [30 - 31](#level-30--31) | Git tags as hidden references |
| [31 - 32](#level-31--32) | Git commit and push |
| [32 - 33](#level-32--33) | Uppercase shell escape via `$0` |

---

## Level 0 - 1

**Goal:** Read the password from a `readme` file in the home directory.

```bash
ls -la
cat readme
```

This is the entry point - SSH in, look around, read a file. Simple, but it sets up the habit of always running `ls -la` first to see everything including hidden files and permissions.

---

## Level 1 - 2

**Goal:** Read a file literally named `-`.

```bash
cat ./-
```

The shell interprets a lone `-` as "read from stdin", not as a filename. Prefixing with `./` forces bash to treat it as a path. This is a common gotcha when dealing with files created by tools that output to `-` by convention.

---

## Level 2 - 3

**Goal:** Read a file with spaces in its name.

```bash
cat ./"--spaces in this filename--"
```

Quoting the full path handles both the leading dash and the embedded spaces in one shot. Alternatively, you can escape each space with a backslash - either works.

---

## Level 3 - 4

**Goal:** Find a hidden file inside the `inhere` directory.

```bash
cd inhere
ls -la
cat ./...Hiding-From-You
```

Hidden files in Linux are just files prefixed with `.` - they don't show up in `ls` without the `-a` flag. Nothing cryptographic, just a convention the shell respects.

---

## Level 4 - 5

**Goal:** Find the only human-readable file among 10 binary files.

```bash
file ./-file0*
cat ./-file07
```

`file` reads magic bytes at the start of each file to determine its actual type, completely ignoring the extension. I used a glob to check all ten at once and looked for the one marked `ASCII text`.

---

## Level 5 - 6

**Goal:** Locate a file that is human-readable, exactly 1033 bytes, and non-executable.

```bash
find . -type f -size 1033c ! -executable
```

`find` lets you stack filters in a single pass. `-size 1033c` means exactly 1033 bytes (`c` = bytes). The `!` negates the `-executable` test. Combining all three conditions avoids any manual inspection.

---

## Level 6 - 7

**Goal:** Find a file owned by user `bandit7`, group `bandit6`, 33 bytes - anywhere on the whole filesystem.

```bash
find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
```

Starting at `/` searches the entire system. Without `2>/dev/null`, the output is flooded with "Permission denied" noise for every directory I can't read. Redirecting stderr to `/dev/null` keeps the signal clean.

---

## Level 7 - 8

**Goal:** Find the password in a large file, stored next to the word `millionth`.

```bash
grep "millionth" data.txt
```

The file has tens of thousands of lines. `grep` scans the whole thing and returns only the matching line - no need to open it manually. Pattern matching at the shell level is something I reach for constantly.

---

## Level 8 - 9

**Goal:** Find the one line that appears exactly once in `data.txt`.

```bash
sort data.txt | uniq -u
```

`uniq` only deduplicates adjacent lines, so sorting first is essential. The `-u` flag filters to lines that appear exactly once. This pipeline pattern - sort, then deduplicate - comes up repeatedly in log analysis and data processing.

---

## Level 9 - 10

**Goal:** Extract human-readable strings from a binary file and find the one preceded by `=` signs.

```bash
strings data.txt | grep "="
```

`strings` pulls printable character sequences from binary files. Most of the output is noise - piping into `grep` narrows it to the relevant pattern immediately.

---

## Level 10 - 11

**Goal:** Decode base64-encoded content.

```bash
base64 -d data.txt
```

Base64 is an encoding scheme, not encryption - it just represents binary data as printable ASCII. The trailing `=` padding is the giveaway. `-d` decodes it back.

---

## Level 11 - 12

**Goal:** Decode ROT13-encoded text.

```bash
tr 'a-zA-Z' 'n-za-mN-ZA-M' < data.txt
```

ROT13 shifts each letter 13 positions through the alphabet. Since there are 26 letters, applying it twice is a no-op - it's its own inverse. `tr` does character-to-character substitution in a single pass, no external tools needed.

---

## Level 12 - 13

**Goal:** Reverse a hexdump and unpack multiple layers of compression to get to a plain text file.

```bash
xxd -r data.txt > data

# Then iteratively:
file data
mv data data.gz && gunzip data.gz   # gzip
bzip2 -d data                        # bzip2
tar -xf data                         # tar
# Repeat until: data: ASCII text
cat data
```

This one required patience. The file had been compressed with gzip, bzip2, and tar multiple times in sequence, then hexdumped. I reversed the hexdump with `xxd -r`, then used `file` after each extraction to identify the next layer. The key insight is that `file` reads magic bytes - extension means nothing here.

The compression stack looked like this:

```
hexdump (data.txt)
  └── gzip
        └── bzip2
              └── gzip
                    └── tar
                          └── bzip2
                                └── tar
                                      └── gzip
                                            └── ASCII text (password)
```

Each layer had to be identified separately with `file` and unpacked with the matching tool. There is no shortcut - you peel it one layer at a time.

> **Real-world link:** Malware packers use the same technique - stacking multiple compression and encoding layers to slow down static analysis. The workflow here (identify type, unpack, repeat) is exactly how you approach it manually.

---

## Level 13 - 14

**Goal:** Use an SSH private key to log in as `bandit14` and read its password.

```bash
chmod 600 bandit14.key
ssh -i bandit14.key bandit14@bandit.labs.overthewire.org -p 2220
cat /etc/bandit_pass/bandit14
```

SSH refuses to use a private key if it's world-readable - it exits with a permissions error. `chmod 600` sets it to owner-read/write only. The `-i` flag specifies the identity file. This is standard practice for any SSH key-based auth setup.

---

## Level 14 - 15

**Goal:** Submit the current level's password to a service on port 30000 via TCP.

```bash
nc localhost 30000
```

`nc` (netcat) opens a raw TCP connection. No encryption, no protocol overhead - just a direct socket. I used it here to interact with a custom service that expects the password and returns the next one.

---

## Level 15 - 16

**Goal:** Same as above, but the service requires TLS.

```bash
ncat --ssl localhost 30001
```

Standard `nc` can't do TLS - it sends plaintext. `ncat` is the modern replacement from the nmap suite and supports `--ssl` to perform the TLS handshake before any data flows. The workflow is identical to the previous level once the connection is established.

---

## Level 16 - 17

**Goal:** Find which port in the 31000-32000 range runs an SSL service that responds correctly, then submit the password.

```bash
nmap -p 31000-32000 localhost

for p in $(nmap -p 31000-32000 localhost | grep open | cut -d/ -f1); do
    echo "CURRENT_PASSWORD" | ncat --ssl localhost $p
done
```

First I scanned for open ports, then looped over each one and piped the password into `ncat --ssl`. Most ports either refused the connection or echoed back an error - the correct one returned the next SSH key. Automating credential submission across a port range is a standard recon technique.

---

## Level 17 - 18

**Goal:** Find the single changed line between two password files.

```bash
diff passwords.old passwords.new
```

`diff` outputs lines prefixed with `<` (only in old) and `>` (only in new). The `>` line is the new password. In real-world security work, `diff` is how you detect unauthorized changes in config files or source code between revisions.

---

## Level 18 - 19

**Goal:** Read `readme` even though `.bashrc` immediately logs you out on login.

```bash
ssh bandit18@bandit.labs.overthewire.org -p 2220 "cat readme"
```

SSH lets you pass a command directly after the connection string. When you do that, it executes the command without starting a login shell - meaning `.bashrc` never runs, the logout trap never triggers, and the output comes back to your terminal cleanly.

---

## Level 19 - 20

**Goal:** Use a setuid binary to read a file you normally have no access to.

```bash
ls -la                # -rwsr-x--- means setuid is set
./bandit20-do cat /etc/bandit_pass/bandit20
```

The `s` in the owner execute position of the permissions string means setuid - the binary runs with the privileges of its owner (`bandit20`), not the caller (`bandit19`). So when I pass `cat /etc/bandit_pass/bandit20` to it, it runs that command as `bandit20` and can read the file. This is the foundational concept behind most UNIX privilege escalation.

---

## Level 20 - 21

**Goal:** Act as a server that holds the current password, connect a SUID binary to it, and receive the next password.

```bash
echo "CURRENT_PASS" | nc -l -p 1234 &
./suconnect 1234
```

The binary connects to a port you specify and validates whatever it receives against the current password. If it matches, it sends back the next one. I set up a listener in the background with `&`, then ran the binary to connect to it. Understanding who is the client and who is the server here is what makes this level click.

---

## Level 21 - 22

**Goal:** Find out what `bandit22` is running via cron and read the output it produces.

```bash
ls /etc/cron.d/
cat /etc/cron.d/cronjob_bandit22
cat /usr/bin/cronjob_bandit22.sh
cat /tmp/<path from the script>
```

Cron jobs in `/etc/cron.d/` run on a schedule as specific users. Reading the script told me it was writing `bandit22`'s password to a predictable path in `/tmp`. I just needed to read that file.

---

## Level 22 - 23

**Goal:** Predict the filename where a cron script stores the password by replicating its MD5 hashing logic.

```bash
echo I am user bandit23 | md5sum
cat /tmp/<resulting_hash>
```

The script generates a path by hashing the string `"I am user <username>"`. I replicated that logic manually to compute what the path would be for `bandit23`, then read it directly. Predictable "obscure" paths are not security.

---

## Level 23 - 24

**Goal:** Write a script, drop it in the cron spool for `bandit24`, wait for it to execute, and read the output.

```bash
mkdir -p /tmp/mydir
cat > /tmp/mydir/exploit.sh << 'EOF'
#!/bin/bash
cat /etc/bandit_pass/bandit24 > /tmp/mydir/pass.txt
EOF

chmod 777 /tmp/mydir
chmod 777 /tmp/mydir/exploit.sh
cp /tmp/mydir/exploit.sh /var/spool/bandit24/foo/

# Wait ~60 seconds
cat /tmp/mydir/pass.txt
```

The cron job runs every script placed in the spool directory as `bandit24`. I wrote a script that copies `bandit24`'s password to a location I can read, set the permissions so the cron process can write there, dropped the script in, and waited. This is exactly how cron-based privilege escalation works in practice - if a privileged user runs arbitrary scripts from a directory you can write to, you own their access.

> **Real-world link:** This level permanently changed how I think about directory permissions. Never give write access to a directory from which a privileged process executes scripts. It doesn't matter how obscure the path is - writable execution directories are a direct privilege escalation vector.

---

## Level 24 - 25

**Goal:** Brute-force a 4-digit PIN combined with the current password against a daemon on port 30002.

```bash
for i in $(seq -w 0 9999); do
    echo "CURRENT_PASS $i"
done | nc localhost 30002 | grep -v "Wrong"
```

Opening 10,000 individual connections would be slow and likely rate-limited. Instead I pre-generated all combinations, piped the entire list into a single `nc` connection, and filtered out the "Wrong!" responses with `grep -v`. The one line that isn't "Wrong!" is the answer.

---

## Level 25 - 26

**Goal:** Escape a restricted shell - the assigned shell for `bandit26` is `more`, not bash, and it exits immediately.

```bash
# Shrink the terminal window until `more` is forced to paginate
stty rows 3

# SSH in - more pauses for input
# Inside more:
v               # opens vi
:set shell=/bin/bash
:shell          # spawns bash
```

I checked `/etc/passwd` to confirm the shell was `more`. Since `more` only paginates when output exceeds the screen, shrinking the terminal forces it to pause. From there I opened `vi` with `v`, set the shell to `/bin/bash` using a vim command, and called `:shell` to spawn it. This is a documented GTFOBins escape path - if you can open a pager or text editor, you can usually escape to a shell.

---

## Level 26 - 27

**Goal:** Use a SUID binary to read `bandit27`'s password.

```bash
./bandit27-do cat /etc/bandit_pass/bandit27
```

Same setuid mechanic as Level 19-20. Once I was in `bandit26`'s bash (from the escape in the previous level), the binary was right there with the same pattern.

---

## Level 27 - 28

**Goal:** Clone a Git repository over SSH and find the password.

```bash
git clone ssh://bandit27-git@bandit.labs.overthewire.org:2220/home/bandit27-git/repo
cat repo/README
```

Git over SSH with a non-standard port requires the full `ssh://user@host:port/path` URL format. The password was sitting in the README of the cloned repo.

---

## Level 28 - 29

**Goal:** Recover a password that was redacted in the latest commit by digging through git history.

```bash
git log
git log -p
git show <commit-hash>
```

The latest commit showed `****` in place of the password. But git stores every previous state permanently - `git log -p` showed me the diff for each commit, and the one before the redaction had the real value. This is why "I'll just remove it in the next commit" is not a valid remediation for leaked secrets in version control.

> **Real-world link:** The correct fix is `git filter-branch` or the BFG Repo Cleaner to rewrite history, followed by a force push and rotating the leaked credential immediately. Anything short of that and the secret is still in the repo for anyone with clone access.

---

## Level 29 - 30

**Goal:** Find a password hidden in a non-default branch of the repository.

```bash
git branch -a
git checkout dev
cat README.md
```

`git clone` only tracks `origin/master` by default. `git branch -a` lists all remote branches. The `dev` branch had the password that `master` explicitly stated was "not in production." Sensitive data left on feature branches is a real and common issue.

> **Real-world link:** Pre-commit hooks and tools like `truffleHog` or `git-secrets` scan for credentials before they ever land in a branch. Auditing branches is something that belongs in any repo security review - not just `main`.

---

## Level 30 - 31

**Goal:** Find a password stored in a git tag.

```bash
git tag
git show secret
```

There were no useful commits and no other branches. Git tags can point to any object in the repository - including blobs that aren't reachable from any branch. Running `git show` on the tag revealed the password stored directly as a tag message, not attached to any commit.

---

## Level 31 - 32

**Goal:** Push a specific file to the remote to trigger a server-side hook that returns the password.

```bash
git config user.name "bandit31"
git config user.email "bandit31@bandit"
echo "May I come in?" > key.txt
git add -f key.txt
git commit -m "add key"
git push origin master
```

The `.gitignore` was blocking `key.txt` from being staged. The `-f` flag to `git add` overrides that. Git needs a user identity configured before it will commit - set with `git config`. The server-side hook validated the file content and printed the password in the push output.

---

## Level 32 - 33

**Goal:** Escape a shell that converts all typed input to uppercase, making every command fail.

```bash
$0
cat /etc/bandit_pass/bandit33
```

`$0` is a special positional parameter that holds the name of the currently running shell or script. Since it's a variable expansion rather than typed text, the uppercase filter never touches it. The shell expands `$0` to `/bin/sh` (or similar) and executes it, dropping me into a normal shell with `bandit33`'s privileges via the setuid bit on the shell binary.

---

---

## Lessons Learned

**Hardest level: 25-26 (restricted shell escape)**

Not because the technical steps were complex, but because the solution required thinking outside the problem entirely. The instinct is to try different commands, different flags, different encodings. The actual solution was to resize the terminal window. That shift - from "what command do I run" to "what is the environment doing and how can I change it" - is the mental model that separates scripted attacks from real security thinking. Once I had `more` paused, the path through `vi` to a shell was straightforward. Getting there was the hard part.

**Most "real-world" level: 28-29 (git history forensics)**

This one stuck with me the most because it mirrors an actual mistake developers make all the time. The credential was already removed from the latest commit - the repo looked clean. But git's entire value proposition is that it never loses history, which means it also never loses secrets. I have gone back and audited my own repos after this level. The fix is not a new commit - it is a history rewrite and a credential rotation.

**Biggest mindset shift: 23-24 (cron privilege escalation)**

Before this level, cron was just a scheduler. After it, cron is a trust boundary. Any time a privileged process executes something from a location another user can write to, that is not a scheduler - that is an escalation path waiting to be used. I now check directory permissions on anything cron-related the same way I check setuid bits.

**Most underrated level: 8-9 (sort + uniq)**

It looks trivial but the pipeline pattern - sort first, then deduplicate - is something I use constantly now for log analysis. The level teaches you to think in composable steps rather than monolithic tools.

---

## Skills Demonstrated

| Category | Tools and Concepts |
|----------|--------------------|
| Linux Fundamentals | File navigation, permissions, special filenames, hidden files |
| File Analysis | `file`, `strings`, `xxd`, magic bytes, recursive decompression |
| Text Processing | `grep`, `sort`, `uniq`, `tr`, `diff`, shell pipelines |
| Networking | `nc`, `ncat --ssl`, `nmap`, SSH, raw TCP sockets |
| Cryptography | Base64, ROT13, SSL/TLS handshake |
| Privilege Escalation | SUID binaries, cron abuse, restricted shell escapes, GTFOBins |
| Version Control Security | Git history forensics, branch enumeration, tag inspection, push hooks |
| Automation | Bash loops, brute-force optimization, background processes |
