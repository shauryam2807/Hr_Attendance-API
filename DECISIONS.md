# DECISIONS.md

1. **Indexes.**
We created a unique index on `employees.emp_code`, a compound unique index on `attendance_logs(emp_code, date)` to enforce a single log per employee-day, and compound indexes `attendance_logs(date, emp_code)` and `(date, late_minutes)` to support fast sorted pagination and late analytics (`IXSCAN`). We considered an index on `history`, but rejected it because history is an embedded audit trail never queried directly, and multikey array indexing adds unnecessary write overhead.

2. **Punch-in race.**
When two identical punch-in requests arrive concurrently, both validate input and execute `insert_one()`. Because of the unique compound index on `(emp_code, date)`, MongoDB's storage engine atomically accepts only the first document (HTTP 201). The second concurrent insert immediately raises a `DuplicateKeyError`, which our code catches to return HTTP 409, eliminating race conditions without vulnerable check-then-insert lookups.

3. **Ties.**
Employees tied on late minutes share the identical rank number (e.g., 1, 2, 2, 4). At the pagination cutoff of `limit`, our pipeline deterministically breaks ties by sorting `emp_code` ascending. This ensures stable, predictable page slices without records shifting between pages.

4. **Headcount.**
The department summary aggregation starts from the `employees` collection (filtering `joined_on <= month_end`) and uses `$lookup` to join monthly `attendance_logs`. Employees with zero attendance records are preserved as parent documents with empty logs, ensuring `$sum: 1` accurately counts them into the department headcount.

5. **One thing you would change.**
At 100x data (~10 million records), we would shard `attendance_logs` on `emp_code` and pre-aggregate daily and monthly metrics into rollup collections via MongoDB Change Streams, avoiding expensive runtime multi-collection aggregations.
