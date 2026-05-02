# OverTheWire Bandit - Levels 0-20

## Level 0 → 1
**Goal:** Find password stored in readme file  
**Commands:** `ls -la`, `cat readme`  
**Key concept:** Basic Linux file navigation

## Level 1 → 2
**Goal:** Read file named `-` (dash)  
**Commands:** `cat ./-`  
**Key concept:** Files starting with `-` need `./` prefix to avoid bash interpreting as flag

## Level 2 → 3
**Goal:** Read file with spaces in filename  
**Commands:** `cat ./"--spaces in this filename--"`  
**Key concept:** Files with spaces need quotes or backslash escaping

## Level 3 → 4
**Goal:** Find hidden file in directory  
**Commands:** `cd inhere`, `ls -la`, `cat ./...Hiding-From-You`  
**Key concept:** Hidden files start with `.` and only visible with `ls -la`

## Level 4 → 5
**Goal:** Find human-readable file among 10 files  
**Commands:** `file ./-file0*`  
**Key concept:** `file` command identifies file types - look for ASCII text

## Level 5 → 6
**Goal:** Find file that is human-readable, 1033 bytes, not executable  
**Commands:** `find . -type f -size 1033c ! -executable`  
**Key concept:** `find` with multiple filters is powerful for locating specific files

## Level 6 → 7
**Goal:** Find file that size 33 bytes,  user bandit 7 , group bandit 6  
**Commands:** `find / -user bandit7 -group bandit6 -size 33c`
**Key concept:** `find` with multiple filter is powerful for locating hidden file  

## Level 7 → 8
**Goal:** Find file the file **data.txt** next to the word **millionth**
**Commands:** `grep "millionth" data.txt`
**Key concept:** `grep` useful for finding string in large files  

## Level 8 → 9
**Goal:** Find unique string that occurs only once in the file **data.txt** 
**Commands:** `sort data.txt | uniq -u`
**Key concept:** Using pipes `|` to chain sort and uniq filters

## Level 9 → 10
**Goal:** Find one of the few human-readable strings in the file **data.txt** 
**Commands:** `strings data.txt | grep "="`
**Key concept:** Extract text, then search for pattern


## Level 10 → 11
**Goal:** Find the password for the next level that stored in the file **data.txt**, which contains base64 encoded data
**Commands:** `base64 -d data.txt`
**Key concept:** Decode base64-encoded information to reveal plain text

## Level 11 → 12
**Goal:** Find the password for the next level that stored in the file **data.txt**, where all lowercase (a-z) and uppercase (A-Z) letters have been rotated by 13 positions
**Commands:** `tr 'a-zA-Z' 'n-za-mN-ZA-M' < data.txt`
**Key concept:** ROT13: Replace each letter with the letter 13 positions ahead. Apply the same transformation twice to decode.


## Level 12 → 13
**Goal:** The password for the next level is stored in the file **data.txt**, which is a hexdump of a file that has been repeatedly compressed.
**Commands:** - **Reverse Hexdump:** `xxd -r data.txt > data`
- **Identify Type:** `file data`
- **Decompress GZIP:** `mv data data.gz && gunzip data.gz`
- **Decompress BZIP2:** `bzip2 -d data` (or `bunzip2 data`)
- **Extract TAR:** `tar -xf data`
- **Read Result:** `cat data` (once it is identified as **ASCII text**)

**Key concept:** **Recursive Decompression and File Identification:** The process of using the `file` utility to identify the actual file type based on "magic bytes" (signatures), regardless of its extension. It requires the iterative use of various decompression tools (`gzip`, `bzip2`, `tar`) to peel back multiple layers of nested archives until the final payload is reached.


## Level 13 → 14
**Goal:** The password for the next level is stored in `/etc/bandit_pass/bandit14` and can only be read by user **bandit14**. Use the provided private SSH key (`sshkey.private`) to log in as bandit14 and retrieve the password.
**Commands:** 
- `cat sshkey.private` – Displays the key's content so it can be copied to your local machine.
- `chmod 600 bandit14.key` – Sets the critical file permissions (read/write only for the owner), which are required for SSH to accept the key.
- `ssh -i bandit14.key bandit14@bandit.labs.overthewire.org -p 2220` – Connects to the server using the identity file (private key) instead of a password.
- `cat /etc/bandit_pass/bandit14` – Reads the password file once successfully logged in as bandit14.
**Key concept:** **SSH Public Key Authentication:** A method of logging into an SSH server using a cryptographic key pair (private and public keys) instead of a password. The security relies on the **private key** staying on the client side. SSH clients enforce strict security policies: the private key file must have restricted permissions (`600`) to prevent other users from reading it, and the `-i` flag must be used to explicitly point the client to this specific identity file.

## Level 14 → 15
**Goal:** Submit the current level's password to **port 30000** on **localhost** to retrieve the password for the next level.
**Commands:**
- `nc localhost 30000` - Opens a raw network connection to the specified port on the local machine.
**Key Concept:** **Network Sockets and Netcat:** A network socket is an endpoint in a communication flow. While SSH is used for secure remote management, tools like **netcat (`nc`)** allow for direct communication with services on specific ports by sending and receiving raw data. In this level, the service on port 30000 acts as a simple "echo" server that validates the current password and returns the next one.


## Level 15 → 16
**Goal:** The password for the next level can be retrieved by submitting the password of the current level to **port 30001 on localhost** using SSL/TLS encryption.
**Commands:** `ncat --ssl localhost 30001` – Connects to the port using `ncat` with SSL encryption enabled.
**Key concept:**  **SSL/TLS Encrypted Network Communication:** Unlike the previous level which used a raw TCP socket (plaintext), this level introduces **Secure Sockets Layer (SSL)** and **Transport Layer Security (TLS)**. These protocols wrap the communication in an encrypted layer, protecting the data from being intercepted or read by unauthorized parties during transit. Tools like `openssl s_client` or `ncat --ssl` perform a "handshake" to establish this secure tunnel before any data (like your password) is sent.

## Level 16 → 17
**Goal:** Efficiently identify the correct SSL service among multiple open ports without waiting for slow version scans.
**Commands:**
- `nmap -p 31000-32000 --script ssl-enum-ciphers localhost` - Uses Nmap scripts to specifically target SSL/TLS services.
- **Bash One-Liner:**
  Bash
    ```
    for p in $(nmap -p 31000-32000 localhost | grep open | cut -d/ -f1); do 
        echo "CURRENT_PASSWORD" | ncat --ssl localhost $p
    done
    ```
**Key Concept:** **Automated Service Interaction:** Instead of manual testing or waiting for deep service inspection (`-sV`), you can combine **Port Scanning** with **Scripted Loops**. By piping the output of a fast scan into a loop that attempts SSL authentication, you automate the "trial and error" process. This is a common technique in penetration testing to quickly validate credentials across multiple services.

## Level 17 → 18
**Goal:** Identify the only line that has been changed between `passwords.old` and `passwords.new`. The changed line in `passwords.new` is the password for the next level.
**Commands:**
- `diff passwords.old passwords.new` - Standard tool to compare files line by line.
- `grep -vFf passwords.old passwords.new` - An alternative way to filter out all lines present in the old file, leaving only the unique new password.
- 
**Key Concept:** **File Differencing and Patching:** The `diff` utility is fundamental in Linux for tracking changes. In the output, `<` indicates lines from the first file (old), and `>` indicates lines from the second file (new). In a security context, `diff` is often used to detect unauthorized modifications in configuration files or codebases.

## Level 18 → 19
**Goal:** Retrieve the password from the `readme` file. The `.bashrc` file has been modified to log the user out immediately upon login. 
**Commands:** * `ssh bandit18@... "ls"` — Executes the `ls` command on the remote server without starting an interactive shell session. * `ssh bandit18@... "cat readme"` — Reads the password file directly. 

**Key Concept:** **Non-Interactive SSH Commands:** SSH allows users to execute single commands on a remote host by appending the command to the connection string. This bypasses the initialization of an interactive login shell, preventing the execution of certain logout scripts (like a modified `.bashrc`) that would otherwise terminate the session.

## Level 19 → 20
**Goal:** Use the `setuid` binary to read the password for `bandit20` located in `/etc/bandit_pass/bandit20`.
**Commands:**
- `ls -la` — View file permissions and identify the `setuid` bit (indicated by the `s` in `-rwsr-x---`).
- `./bandit20-do [command]` — Executes a specified command with the privileges of the file owner (`bandit20`).
- `./bandit20-do cat /etc/bandit_pass/bandit20` — Uses the binary to bypass your current user's restrictions and read the next level's password.
  
**Key Concept:** **Setuid (set user ID):** A special type of file permission in Unix-like systems. When an executable with the `setuid` bit is run, it executes with the privileges of the **file's owner** rather than the user who started it. This is often used to allow normal users to perform specific tasks that require higher privileges (like changing a password or, in this case, reading a restricted file).
