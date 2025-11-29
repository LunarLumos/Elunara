Build a complete online programming contest platform called “Elunara” with the following full set of features and technologies.

========================================
1. TECHNOLOGY STACK
========================================
- Backend: Node.js + Express
- Realtime: Socket.io (WebSockets)
- Database: MySQL (for high concurrency and multi-user contests)
- Queue/Workers: Node worker threads OR BullMQ-style job queue
- Code Execution: Run C/C++ code using Linux process sandboxing (NO Docker)
    - ulimit, non-root, isolated temp directory per submission
- Frontend: HTML + CSS + JavaScript (or minimal React if needed)

Optimized for an i5 4th Gen CPU with 16GB RAM, able to handle ~100 simultaneous submissions efficiently.

========================================
2. CORE FEATURES
========================================

2.1 ADMIN FEATURES
- Create contests (title, start/end time, registration status)
- Add problems:
  - Title
  - Description (Markdown)
  - Difficulty
  - Points
  - Testcases (input/output)
- View registered users
- View all submissions with verdicts and penalties
- Monitor real-time leaderboard

2.2 USER FEATURES
- Register/login securely
- Register for contests
- View contest problems (after registration)
- Submit C/C++ code
- Test code with custom input (pre-submission test-run)
- View submission results: AC, WA, TLE, RE, CE
- Real-time leaderboard updates via WebSockets
- Malicious code penalties (-100 points)

========================================
3. CODE JUDGING SYSTEM
========================================

3.1 Compilation & Execution
- Use gcc/g++ on the OS
- Save each submission in a temporary folder
- Compile:
    g++ -O2 -std=c++17 -o program program.cpp
- Run:
    ./program < input.txt

3.2 Sandbox
- No Docker
- Use Linux process restrictions:
  - ulimit -t 2           (CPU time limit)
  - ulimit -v 262144      (256MB memory limit)
  - ulimit -f 65536       (max output)
- Run as non-root user
- Use isolated temp directories
- Kill process if execution exceeds time limit
- No network access

3.3 Malicious Code Detection
- Penalize or reject submissions with:
  - system(), fork(), exec()
  - #include <unistd.h>, #include <sys/>
  - Dangerous shell commands (rm, kill, shutdown, etc.)
- Apply -100 points penalty if malicious

========================================
4. PERFORMANCE REQUIREMENTS
========================================
- Async job queue with worker pool (8–16 workers)
- Tiny programs (<50 lines) must finish <100ms
- 100 submissions complete ~1–2 seconds total
- MySQL handles concurrent writes efficiently
- In-memory cache for leaderboard to avoid DB bottleneck
- WebSocket updates every 50–100ms

========================================
5. DATABASE (MySQL)
========================================

Tables:

1. users
- id (PK)
- username
- password_hash
- total_score

2. contests
- id (PK)
- title
- start_time
- end_time
- registration_open (bool)

3. contest_registrations
- id (PK)
- user_id (FK)
- contest_id (FK)
- registration_time

4. problems
- id (PK)
- contest_id (FK)
- title
- statement
- difficulty
- points

5. testcases
- id (PK)
- problem_id (FK)
- input
- output

6. submissions
- id (PK)
- user_id (FK)
- problem_id (FK)
- code
- language
- verdict
- score
- penalty
- runtime
- submission_time

7. test_runs (optional)
- id (PK)
- user_id (FK)
- code
- input
- output
- runtime
- verdict

========================================
6. REAL-TIME FEATURES
========================================
- WebSockets via Socket.io
- Leaderboard updates pushed in real-time
- Only diffs sent, batched every 50–100ms
- Leaderboard cached in memory for speed
- Admin dashboard real-time monitoring

========================================
7. FRONTEND
========================================
Pages:
- Login/Register
- Contest List
- Contest Registration
- Problem List & View
- Code Editor (Monaco or textarea)
- Test Run Console
- Submission Results
- Real-Time Leaderboard

========================================
8. DELIVERABLES
========================================
- Backend:
  - Express routes, authentication
  - Submission queue and worker pool
  - Safe C/C++ sandbox execution
  - Malicious code detection
  - Leaderboard manager
- Frontend:
  - Contest pages, code editor, test-run console
  - Real-time leaderboard UI
- Database:
  - MySQL schema and migrations
  - Foreign keys and indexes for performance
- Real-time:
  - Socket.io server for updates
- Performance:
  - Worker pool for ~100 simultaneous submissions
  - Fast leaderboard and submission handling
