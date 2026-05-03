# OverTheWire: Bandit — Complete Write-up (Levels 0–33)

> **Platform:** [OverTheWire Bandit](https://overthewire.org/wargames/bandit/)  
> **Difficulty:** Beginner → Intermediate  
> **Purpose:** Linux fundamentals, networking, cryptography, and privilege escalation practice  
> **Status:** ✅ Completed all 34 levels

---

## Table of Contents

| Level | Key Concept |
|-------|-------------|
| [0 → 1](#level-0--1) | Basic file navigation |
| [1 → 2](#level-1--2) | Files with special names (dash) |
| [2 → 3](#level-2--3) | Files with spaces in name |
| [3 → 4](#level-3--4) | Hidden files |
| [4 → 5](#level-4--5) | File type identification |
| [5 → 6](#level-5--6) | find with multiple filters |
| [6 → 7](#level-6--7) | find by user/group |
| [7 → 8](#level-7--8) | grep in large files |
| [8 → 9](#level-8--9) | sort + uniq pipeline |
| [9 → 10](#level-9--10) | strings + grep |
| [10 → 11](#level-10--11) | Base64 decoding |
| [11 → 12](#level-11--12) | ROT13 cipher |
| [12 → 13](#level-12--13) | Recursive decompression |
| [13 → 14](#level-13--14) | SSH key authentication |
| [14 → 15](#level-14--15) | Netcat + network sockets |
| [15 → 16](#level-15--16) | SSL/TLS encrypted connections |
| [16 → 17](#level-16--17) | Port scanning + automated interaction |
| [17 → 18](#level-17--18) | File differencing (diff) |
| [18 → 19](#level-18--19) | Non-interactive SSH commands |
| [19 → 20](#level-19--20) | SUID binaries |
| [20 → 21](#level-20--21) | Client-server localhost interaction |
| [21 → 22](#level-21--22) | Cron jobs |
| [22 → 23](#level-22--23) | MD5 dynamic path generation |
| [23 → 24](#level-23--24) | Privilege escalation via cron |
| [24 → 25](#level-24--25) | Brute-force + pipe efficiency |
| [25 → 26](#level-25--26) | Restricted shell escape |
| [26 → 27](#level-26--27) | SUID binary exploitation |
| [27 → 28](#level-27--28) | Git over SSH |
| [28 → 29](#level-28--29) | Git history exposure |
| [29 → 30](#level-29--30) | Git remote branches |
| [30 → 31](#level-30--31) | Git tags as hidden references |
| [31 → 32](#level-31--32) | Git commit and push |
| [32 → 33](#level-32--33) | Restricted shell + positional parameters |

---

## Level 0 → 1

**Goal:** Find the password stored in a readme file.

```bash
ls -la
cat readme
```

> **Key concept:** Basic Linux file navigation — `ls` lists files, `cat` reads them.

---

## Level 1 → 2

**Goal:** Read a file named `-` (dash).

```bash
cat ./-
```

> **Key concept:** Files starting with `-` are interpreted as flags by bash. Using `./` prefix tells the shell to treat it as a path, not an option.

---

## Level 2 → 3

**Goal:** Read a file with spaces in the filename.

```bash
cat ./"--spaces in this filename--"
```

> **Key concept:** Filenames with spaces must be wrapped in quotes or have spaces escaped with `\`. Using `./` + quotes handles both the dash prefix and spaces simultaneously.

---

## Level 3 → 4

**Goal:** Find a hidden file inside a directory.

```bash
cd inhere
ls -la
cat ./...Hiding-From-You
```

> **Key concept:** Hidden files in Linux start with `.` and are only visible with `ls -la`. The `-a` flag shows all files including hidden ones.

---

## Level 4 → 5

**Goal:** Find the only human-readable file among 10 binary files.

```bash
file ./-file0*
cat ./-file07
```

> **Key concept:** The `file` command identifies file types based on magic bytes (file signatures), not extensions. Look for the entry marked `ASCII text`.

---

## Level 5 → 6

**Goal:** Find a file that is human-readable, exactly 1033 bytes, and not executable.

```bash
find . -type f -size 1033c ! -executable
```

> **Key concept:** `find` supports multiple simultaneous filters: `-type f` (regular file), `-size 1033c` (exactly 1033 bytes), `! -executable` (not executable). This is more efficient than checking files one by one.

---

## Level 6 → 7

**Goal:** Find a file owned by user `bandit7`, group `bandit6`, size 33 bytes — anywhere on the filesystem.

```bash
find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
```

> **Key concept:** Searching from `/` covers the entire filesystem. `2>/dev/null` suppresses permission denied errors to keep output clean.

---

## Level 7 → 8

**Goal:** Find the password stored next to the word `millionth` in `data.txt`.

```bash
grep "millionth" data.txt
```

> **Key concept:** `grep` efficiently searches for a string pattern in large files without reading line by line manually.

---

## Level 8 → 9

**Goal:** Find the one unique line that appears only once in `data.txt`.

```bash
sort data.txt | uniq -u
```

> **Key concept:** `uniq -u` only works correctly on sorted input — it compares adjacent lines. Piping `sort` output into `uniq -u` identifies lines that appear exactly once.

---

## Level 9 → 10

**Goal:** Find human-readable strings in a binary file, specifically those preceded by `=` characters.

```bash
strings data.txt | grep "="
```

> **Key concept:** `strings` extracts printable character sequences from binary files. Piping into `grep` narrows results to the relevant pattern.

---

## Level 10 → 11

**Goal:** Decode base64-encoded content in `data.txt`.

```bash
base64 -d data.txt
```

> **Key concept:** Base64 encodes binary data as ASCII text. The `-d` flag decodes it back to the original content. Recognizable by the `=` padding at the end.

---

## Level 11 → 12

**Goal:** Decode ROT13-encoded text in `data.txt`.

```bash
tr 'a-zA-Z' 'n-za-mN-ZA-M' < data.txt
```

> **Key concept:** ROT13 substitutes each letter with the one 13 positions ahead in the alphabet. Since the alphabet has 26 letters, applying ROT13 twice returns the original text. `tr` performs character-by-character translation.

---

## Level 12 → 13

**Goal:** Recover the password from a hexdump of a repeatedly compressed file.

```bash
# Reverse the hexdump
xxd -r data.txt > data

# Iteratively identify and decompress
file data
mv data data.gz && gunzip data.gz      # if gzip
bzip2 -d data                           # if bzip2
tar -xf data                            # if tar archive

# Repeat until ASCII text
cat data
```

> **Key concept:** The `file` utility identifies actual file type from magic bytes regardless of extension. Multiple compression layers must be peeled back one at a time using the appropriate tool for each format (`gzip`, `bzip2`, `tar`).

---

## Level 13 → 14

**Goal:** Use a provided SSH private key to authenticate as `bandit14` and read the password.

```bash
# Copy the key content to local machine, save as bandit14.key
chmod 600 bandit14.key
ssh -i bandit14.key bandit14@bandit.labs.overthewire.org -p 2220
cat /etc/bandit_pass/bandit14
```

> **Key concept:** SSH public key authentication uses a cryptographic key pair instead of passwords. The private key must have `600` permissions — SSH refuses to use keys readable by others as a security enforcement.

---

## Level 14 → 15

**Goal:** Submit the current password to port 30000 on localhost to receive the next password.

```bash
nc localhost 30000
# paste current password and press Enter
```

> **Key concept:** `nc` (netcat) opens raw TCP connections to any port. Unlike SSH, it sends and receives unformatted data — useful for interacting with custom services directly.

---

## Level 15 → 16

**Goal:** Submit the password to port 30001 on localhost using SSL/TLS encryption.

```bash
ncat --ssl localhost 30001
# paste current password and press Enter
```

> **Key concept:** SSL/TLS wraps communication in an encrypted layer. Standard `nc` sends plaintext — `ncat --ssl` performs the TLS handshake before transmitting data, protecting it from interception.

---

## Level 16 → 17

**Goal:** Find the correct SSL service among ports 31000–32000 and submit the password.

```bash
# Scan for open ports with SSL
nmap -p 31000-32000 --script ssl-enum-ciphers localhost

# Or automate submission across all open ports
for p in $(nmap -p 31000-32000 localhost | grep open | cut -d/ -f1); do
    echo "CURRENT_PASSWORD" | ncat --ssl localhost $p
done
```

> **Key concept:** Port scanning identifies open services. Scripted loops automate credential submission across multiple targets — a common penetration testing technique for efficient service validation.

---

## Level 17 → 18

**Goal:** Find the one line that differs between `passwords.old` and `passwords.new`.

```bash
diff passwords.old passwords.new
# Alternative:
grep -vFf passwords.old passwords.new
```

> **Key concept:** `diff` compares files line by line. `<` marks lines only in the first file, `>` marks lines only in the second. In security, `diff` detects unauthorized modifications in config files or codebases.

---

## Level 18 → 19

**Goal:** Read `readme` despite `.bashrc` logging you out immediately on login.

```bash
ssh bandit18@bandit.labs.overthewire.org -p 2220 "cat readme"
```

> **Key concept:** SSH allows executing commands on remote hosts without starting an interactive shell. Appending a command to the SSH connection string bypasses `.bashrc` initialization and its logout trap.

---

## Level 19 → 20

**Goal:** Use the SUID binary to read the password for `bandit20`.

```bash
ls -la                                        # spot the 's' in permissions: -rwsr-x---
./bandit20-do cat /etc/bandit_pass/bandit20
```

> **Key concept:** The `setuid` bit causes an executable to run with the **file owner's** privileges rather than the caller's. The `s` in `-rwsr-x---` indicates this. Commonly exploited in privilege escalation.

---

## Level 20 → 21

**Goal:** Use `suconnect` binary — act as a server holding the current password, let the binary validate it and return the next.

```bash
# Step 1: start listener serving the current password
echo "CURRENT_PASS" | nc -l -p 1234 &

# Step 2: connect the binary to your listener
./suconnect 1234
```

> **Key concept:** The user acts as the **server**, the binary as the **client**. `&` sends the listener to the background. Demonstrates client-server interaction on localhost and how SUID binaries can interact with user-controlled services.

---

## Level 21 → 22

**Goal:** Discover what cron is running automatically as `bandit22`.

```bash
ls -la /etc/cron.d/
cat /etc/cron.d/cronjob_bandit22
cat /usr/bin/cronjob_bandit22.sh
cat /tmp/<path revealed by script>
```

> **Key concept:** `cron` is a time-based job scheduler. System jobs in `/etc/cron.d/` run automatically — inspecting them can reveal privileged operations or sensitive data exposure.

---

## Level 22 → 23

**Goal:** Predict the dynamically generated filename where the password is stored by mimicking the script's MD5 logic.

```bash
echo I am user bandit23 | md5sum
cat /tmp/<resulting_hash>
```

> **Key concept:** Scripts often use hashing to generate obscure filenames. By understanding and replicating the logic, the output path becomes predictable even without execution rights.

---

## Level 23 → 24

**Goal:** Write a script, place it in the cron spool directory, and have `bandit24` execute it with elevated privileges.

```bash
# Create script
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

> **Key concept:** Cron runs scripts in the spool directory as the owning user (`bandit24`). By placing your script there, you delegate execution to a more privileged context — a core privilege escalation pattern.

---

## Level 24 → 25

**Goal:** Brute-force a 4-digit PIN combined with the current password against port 30002.

```bash
# Generate all combinations and pipe into single connection
for i in $(seq -w 0 9999); do
    echo "CURRENT_PASS $i"
done | nc localhost 30002 | grep -v "Wrong"
```

> **Key concept:** Piping a pre-generated wordlist into one persistent connection is far more efficient than opening a new connection per attempt. A fundamental brute-force optimization.

---

## Level 25 → 26

**Goal:** Escape a restricted shell that logs out immediately. Shell is `more`, not bash.

```bash
# Shrink terminal until 'more' pauses
stty rows 3

# SSH in — 'more' will pause
# Inside 'more':
v                    # opens vi
:set shell=/bin/bash
:shell               # escape to bash
```

> **Key concept:** Restricted shell escapes via interactive pagers. `more` can invoke `vi`, and `vi` can spawn a shell. A classic GTFOBins technique.

---

## Level 26 → 27

**Goal:** Use the SUID binary to read `bandit27`'s password.

```bash
./bandit27-do cat /etc/bandit_pass/bandit27
```

> **Key concept:** Same SUID concept as Level 19 — the binary runs as its owner (`bandit27`), allowing access to restricted files.

---

## Level 27 → 28

**Goal:** Clone a git repository over SSH and find the password.

```bash
git clone ssh://bandit27-git@bandit.labs.overthewire.org:2220/home/bandit27-git/repo
cat repo/README
```

> **Key concept:** Git over SSH requires specifying the non-standard port (2220) in the URL format `ssh://user@host:port/path`.

---

## Level 28 → 29

**Goal:** Recover a password that was censored in the latest commit by inspecting git history.

```bash
git log
git log -p          # shows diffs for each commit
git show <commit>   # inspect specific commit
```

> **Key concept:** Removing sensitive data from the latest commit does **not** erase it from history. Git permanently stores all previous states. Proper remediation requires rewriting history with `git filter-branch` or BFG.

---

## Level 29 → 30

**Goal:** Find the password hidden in a non-default remote branch.

```bash
git branch -a           # list all branches including remote
git checkout dev        # switch to the branch with the password
cat README.md
```

> **Key concept:** `git clone` tracks `master` by default. Remote branches must be explicitly checked out. Sensitive data is often left in feature or development branches.

---

## Level 30 → 31

**Goal:** Find the password stored in a git tag.

```bash
git tag
git show <tag_name>
```

> **Key concept:** Git tags are persistent references that survive branch deletions. They can point to commits not reachable from any branch — a common place for sensitive data to hide.

---

## Level 31 → 32

**Goal:** Commit and push a specific file to trigger a server-side response.

```bash
git config user.name "bandit31"
git config user.email "bandit31@bandit"
echo "May I come in?" > key.txt
git add -f key.txt      # -f bypasses .gitignore
git commit -m "add key"
git push origin master
```

> **Key concept:** Git requires identity (name + email) for every commit. The `-f` flag forces `git add` to include files listed in `.gitignore`.

---

## Level 32 → 33

**Goal:** Escape a shell that converts all input to UPPERCASE.

```bash
$0
cat /etc/bandit_pass/bandit33
```

> **Key concept:** `$0` is a special shell variable holding the name of the current shell/script. Because it's a variable expansion (not typed text), the uppercase filter doesn't apply. The shell expands it and executes the referenced shell binary.

---

## Level 34 — Complete ✅

```
DONE.
```

---

## Skills Demonstrated

| Category | Skills |
|----------|--------|
| **Linux Fundamentals** | File navigation, permissions, hidden files, special filenames |
| **File Analysis** | `file`, `strings`, `xxd`, recursive decompression |
| **Text Processing** | `grep`, `sort`, `uniq`, `tr`, `diff`, pipelines |
| **Networking** | `nc`, `ncat --ssl`, port scanning with `nmap`, SSH |
| **Cryptography** | Base64, ROT13, SSL/TLS |
| **Privilege Escalation** | SUID binaries, cron abuse, restricted shell escapes |
| **Version Control** | Git history, branches, tags, remote repos |
| **Automation** | Bash loops, brute-forcing, background processes |
