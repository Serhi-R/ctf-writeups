# OverTheWire Bandit — Levels 0-5

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
**Key concept:** `file` command identifies file types — look for ASCII text

## Level 5 → 6
**Goal:** Find file that is human-readable, 1033 bytes, not executable  
**Commands:** `find . -type f -size 1033c ! -executable`  
**Key concept:** `find` with multiple filters is powerful for locating specific files
