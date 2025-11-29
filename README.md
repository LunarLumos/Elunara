Build a fully functional online programming contest platform named “Elunara” with the following technologies and complete set of features.

file structure should:

elunara/
├── backend/
│   ├── config/
│   │   ├── db.js
│   │   └── jwt.js
│   ├── controllers/
│   │   ├── authController.js
│   │   ├── contestController.js
│   │   ├── problemController.js
│   │   ├── submissionController.js
│   │   └── adminController.js
│   ├── middleware/
│   │   ├── auth.js
│   │   └── roleGuard.js
│   ├── models/
│   │   ├── User.js
│   │   ├── Contest.js
│   │   ├── Problem.js
│   │   ├── Submission.js
│   │   └── index.js
│   ├── routes/
│   │   ├── auth.js
│   │   ├── contests.js
│   │   ├── problems.js
│   │   ├── submissions.js
│   │   └── admin.js
│   ├── services/
│   │   ├── sandbox/
│   │   │   ├── execute.js
│   │   │   ├── detectMalicious.js
│   │   │   └── utils.js
│   │   ├── queue/
│   │   │   └── judgeQueue.js
│   │   └── leaderboard.js
│   ├── utils/
│   │   └── tempCleanup.js
│   ├── workers/
│   │   └── judgeWorker.js
│   ├── sockets/
│   │   └── index.js
│   ├── .env.example
│   └── server.js
├── frontend/
│   ├── public/
│   │   └── index.html
│   ├── src/
│   │   ├── components/
│   │   │   ├── CodeEditor.jsx
│   │   │   ├── Leaderboard.jsx
│   │   │   └── ProtectedRoute.jsx
│   │   ├── pages/
│   │   │   ├── Login.jsx
│   │   │   ├── Register.jsx
│   │   │   ├── ContestList.jsx
│   │   │   ├── ContestDetail.jsx
│   │   │   ├── ProblemPage.jsx
│   │   │   ├── AdminDashboard.jsx
│   │   │   └── LeaderboardPage.jsx
│   │   ├── services/
│   │   │   ├── api.js
│   │   │   └── socket.js
│   │   ├── App.js
│   │   └── index.js
│   ├── package.json
│   └── .env.local
├── sql/
│   └── schema.sql
├── deploy/
│   ├── setup-sandbox-user.sh
│   └── systemd-elunara.service
├── .gitignore
├── package.json
├── README.md
└── LICENSE


========================================
1. TECHNOLOGY STACK
========================================
- Backend: Node.js + Express
- Real-time: Socket.io
- Database: MySQL
- Asynchronous Judging Queue: Worker Threads or BullMQ-style queue
- Code Execution: Linux process sandbox (NO Docker)
- Frontend: HTML/CSS/JS (or React)
- Authentication: JWT or sessions

Must run efficiently on:
- Intel i5 4th Gen CPU
- 16GB RAM
- Linux environment

========================================
2. USER ROLES & ACCESS CONTROL
========================================

2.1 ADMIN
- Admin login
- Create & manage contests
- Add/edit problems with testcases
- Approve/reject user contest applications
- Ban users (banned users cannot log in or participate)
- Unban users
- Remove users from contest
- Hide/show leaderboard
- Hide/show admin dashboard from users
- View all submissions
- See real-time submission stats
- Apply penalties manually if needed

2.2 USER
- Register/login
- Apply to join contest
- Access problems ONLY AFTER approval
- Test-run code safely before submission
- Submit code for judging
- View verdicts (AC, WA, TLE, RE, CE, MAL)
- View real-time leaderboard (if visible)
- Check their own submission history

========================================
3. CONTEST WORKFLOW
========================================
- Users must apply to join contests.
- Admin approves before they can solve problems.
- Contest has:
  - Name
  - Description
  - Start & end time
  - Registration status
  - Leaderboard visibility toggle

========================================
4. CODE JUDGING SYSTEM (NO DOCKER)
========================================

4.1 Compilation
- Use gcc/g++:
    g++ -O2 -std=c++17 program.cpp -o program

4.2 Execution Sandbox
- NO Docker
- Use Linux process controls:
    ulimit -t 2           (CPU time)
    ulimit -v 262144      (256 MB RAM)
    ulimit -f 65536       (max output)
- Run under restricted non-root user
- Isolated temp directory per submission
- No network access

4.3 Infinite Loop Detection
Detect infinite loops using:

A. **CPU time limit exceeded (TLE)**  
   If process hits ulimit -t or is still running after time limit → treat as infinite loop.

B. **Instruction limit check via /proc/\<pid\>/stat**  
   If instructions increase without producing output → identify infinite loop.

C. **Runtime watchdog timer**  
   Kill process if:
   - Running continuously
   - No data written to stdout/stderr for N milliseconds
   → classify as harmful loop

If infinite loop detected:
- verdict = “MAL (Harmful — Infinite Loop)”
- penalty = -100 points

========================================
5. MALICIOUS CODE DETECTION
========================================

5.1 Forbidden Keywords
Reject or penalize code containing:
- system()
- fork()
- exec()
- popen()
- mmap(…)
- #include <unistd.h>
- #include <sys/>
- asm volatile
- access to forbidden filepaths (/, /etc/, /home/, etc.)
- Attempts to spawn child processes
- Writing outside temp directory
- Dangerous shell commands:
  rm, reboot, shutdown, kill, chmod, chown, mv, wget, curl

5.2 Harmful Code Behavior
If suspicious behavior detected:
- Infinite loop (logic bombs)
- Excessive memory usage
- Excessive output flooding
- Attempt to run external programs
- Attempt to escape sandbox

→ verdict = MAL  
→ penalty = -100 points  
→ submission blocked  

========================================
6. VERDICTS
========================================

- **AC** — Accepted  
- **WA** — Wrong Answer  
- **TLE** — Time Limit Exceeded  
- **RE** — Runtime Error  
- **CE** — Compilation Error  
- **MAL** — Malicious/Harmful Code (includes infinite loops)  

========================================
7. PERFORMANCE GOALS
========================================
- 8–16 worker threads
- Handle 100 simultaneous submissions
- Compile time < 50ms for small code
- Execution < 30ms
- MySQL write < 5ms
- Leaderboard update < 20ms
- Real-time WebSockets push every 50–100ms
- Judge 100 submissions in ~1–2 seconds total

========================================
8. REAL-TIME SYSTEM (WebSockets)
========================================

Socket.io events:
- submissionResult → send submission verdict instantly
- leaderboardUpdate → send updated ranks
- dashboardUpdate → admin-only live system stats
- contestStatus → notify users of admin changes

Features:
- Leaderboard can be toggled visible/hidden by admin
- Dashboard visible only to admin
- Only diff updates sent to reduce bandwidth
- Leaderboard cached in memory for speed

========================================
9. DATABASE SCHEMA (MYSQL)
========================================

TABLE: users
- id
- username
- password_hash
- banned (bool)
- role (user/admin)
- created_at

TABLE: contests
- id
- title
- description
- start_time
- end_time
- registration_open (bool)
- leaderboard_visible (bool)
- created_at

TABLE: contest_applications
- id
- user_id
- contest_id
- status (pending/approved/rejected)
- applied_at

TABLE: problems
- id
- contest_id
- title
- statement
- difficulty
- points

TABLE: testcases
- id
- problem_id
- input
- output

TABLE: submissions
- id
- user_id
- problem_id
- code
- language
- verdict
- score
- penalty
- runtime
- created_at

TABLE: test_runs
- id
- user_id
- input
- output
- runtime
- verdict

========================================
10. FRONTEND REQUIREMENTS
========================================

User interface:
- Login/Register
- Contest list
- Apply to join contest
- Wait for approval page
- Problem list
- Problem detail page
- Code editor with syntax highlighting
- Test-run console
- Submission results page
- Real-time leaderboard page

Admin interface:
- Admin login
- Contest manager
- User application approval panel
- Add/edit problems
- Add/edit testcases
- All submissions overview
- Ban/unban users
- Toggle leaderboard visibility
- Toggle dashboard visibility
- Real-time admin dashboard

========================================
11. DELIVERABLES
========================================
Deliver a complete running project with:

BACKEND:
- All API routes
- Authentication
- Admin permission system
- User apply/approval system
- Ban/unban system
- MySQL models
- Code sandbox runner
- Infinite loop detection
- Malicious code detection
- Submission queue & workers
- Leaderboard engine

FRONTEND:
- All UI pages
- Code editor page
- Admin panel
- Test-run interface
- Leaderboard UI

REAL-TIME:
- Socket.io event system
- Admin-only metrics
- Leaderboard & result pushes

SYSTEM:
- Worker pool
- Logging
- Config
- Temp folder cleanup
