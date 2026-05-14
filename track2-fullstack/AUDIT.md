# FarmTracker Codebase Audit

## 1. Issues Identified
During my initial review of the codebase, I identified several critical bugs and architectural concerns affecting data integrity, performance, and security:

* **Data State Corruption (Transfer Bug):** In `PUT /api/animals/:id`, when an animal's `paddock_id` is updated, the new paddock's `animal_count` is incremented, but the old paddock's count is never decremented. 
* **Missing Database Transactions:** In `POST` and `DELETE` animal routes, multi-step writes (updating paddock counts and inserting/deleting animals) are not wrapped in transactions. If the animal query fails, the paddock count becomes permanently desynced.
* **Ignored Business Constraints:** The system never validates if a paddock's `capacity` has been reached before assigning a new animal to it.
* **Performance (N+1 Query) & Pagination Logic:** In `GET /api/animals`, the `OFFSET` calculation is incorrectly using the raw `page` integer rather than `page * limit`. Furthermore, the route executes a new database query inside a `.map()` loop to fetch health events, creating an N+1 performance bottleneck.
* **Missing Error Handling:** `server.js` lacks a global error-handling middleware. Uncaught database exceptions currently risk hanging the API.
* **Version Control Security:** Live SQLite database files (`.db`, `-shm`, `-wal`) were originally committed directly to the repository.

## 2. Immediate Priorities
My immediate priority is fixing data integrity and security:
1. **Remove tracked DB files:** I immediately untracked the local database files and updated the `.gitignore` to prevent exposing live data and causing merge conflicts.
2. **Fix Data State Corruption:** I will fix the `PUT`, `POST`, and `DELETE` animal routes by ensuring the old paddock is decremented properly during transfers, and I will wrap these operations in SQLite transactions to ensure atomic writes. 
3. **Fix Pagination Offset:** I will correct the math in the `GET /api/animals` offset query so the frontend displays data correctly.

## 3. Deferred Items
While the `animal_count` denormalization in the `paddocks` table is the root cause of the state corruption bugs, fixing it properly requires a database schema migration to drop the column and rewriting several API routes to use dynamic `COUNT()` queries. Given the time constraints of implementing the new Weight Tracking feature, I will patch the route logic for now and leave the schema refactor as an architectural proposal. Additionally, I would defer fixing the N+1 query and enforcing the paddock capacity constraint to a future sprint. (See `ARCH_PROPOSAL.md` for details).
