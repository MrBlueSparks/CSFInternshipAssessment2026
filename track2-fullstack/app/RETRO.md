# Assessment Retrospective

## What went well
* **Full-Stack Integration:** Successfully building the Weight Tracking feature from the database layer (with proper constraints) all the way up to the frontend UI went very smoothly.
* **Catching Critical Bugs:** Discovering the paddock transfer state corruption and wrapping the fix in a SQLite transaction was a major win for data integrity.

## What was challenging
* **Working with Denormalized Data:** The `animal_count` on the `paddocks` table made simple CRUD operations much more complex than they needed to be. Having to manually sync state across tables is risky, which is why I focused my Architecture Proposal on fixing this.
* **Hidden UI Gaps:** Discovering that the previous developer built backend routes for `POST` and `DELETE` but forgot to build the frontend HTML forms for them was a funny surprise. 

## What I would do differently with more time
If this wasn't a time-boxed sprint, I would have paused the Weight Tracking feature implementation to run a proper database migration first. Dropping the `animal_count` column and refactoring the Paddock routes to use dynamic `COUNT()` queries would have permanently eliminated the state bugs rather than just patching the immediate route logic.
