
# Architectural Proposal: Refactoring Paddock Capacity State

## 1. The Problem: Denormalized State
Currently, the `paddocks` table contains an `animal_count` column. This is an architectural anti-pattern known as denormalization. By storing a hardcoded integer count instead of computing it dynamically, the application creates a dual-source of truth. 

As discovered during the codebase audit, this design choice directly led to state corruption bugs. When an animal was transferred between paddocks, the system incremented the new paddock but failed to decrement the old one. While I have temporarily patched this by adding transactions and proper decrement logic to the API routes, the underlying architecture remains brittle. Every future feature that moves, deletes, or adds an animal will need to remember to manually sync this counter.

## 2. The Proposed Solution: Dynamic Computed Counts
I propose dropping the `animal_count` column from the `paddocks` table entirely and moving to a dynamic computation model to ensure a Single Source of Truth. 

Whenever a client requests the paddock list, the database will compute the current count on the fly using an aggregated `COUNT()` query against the `animals` table. 

**Updated SQL Query for `GET /api/paddocks`:**
\`\`\`sql
SELECT 
    paddocks.id, 
    paddocks.name, 
    paddocks.capacity,
    COUNT(animals.id) as animal_count
FROM paddocks
LEFT JOIN animals ON animals.paddock_id = paddocks.id
GROUP BY paddocks.id;
\`\`\`

## 3. Migration Strategy
To implement this safely without breaking the frontend, I propose a three-step migration:

1. **Schema Update:** Create a database migration script to drop the `animal_count` column from the `paddocks` schema in `db.js`.
2. **Read Logic Update:** Update the `GET /api/paddocks` route to utilize the `LEFT JOIN` and `COUNT()` SQL query shown above. Because we alias the aggregate as `animal_count`, the frontend API contract remains completely unchanged.
3. **Write Logic Cleanup:** Refactor the `POST`, `PUT`, and `DELETE` routes in `animals.js` to completely remove all `UPDATE paddocks SET animal_count...` queries. 

## 4. Benefits
* **Absolute Data Integrity:** It becomes mathematically impossible for a paddock to report the wrong number of animals.
* **Simplified Route Logic:** The `animals.js` route controllers become much leaner and easier to test because they no longer have to manage multi-table state updates.
* **Future-Proofing:** Future developers can add bulk-import or bulk-delete features without worrying about accidentally desyncing the paddock counts.
