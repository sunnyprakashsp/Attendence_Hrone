# Design decisions — review and rewrite in your own words before submission

1. **Indexes.** I create a unique index on `employees.emp_code` so duplicate employee codes cannot be inserted concurrently, plus a department/join-date index for headcount queries. Attendance has a unique `(emp_code, date)` index for the one-record-per-day rule, a `(date desc, emp_code asc)` index for listing, an employee/date/status index for employee history, and a status/date index for filtered attendance. The best compound index depends on the query's equality and sort fields, so I would validate each plan with `explain` against the grader-sized dataset.

2. **Punch-in race.** Both requests may initially find the employee, but both attempt to insert the same natural key. MongoDB's unique index permits one insert only. The winner returns 201; the loser catches `DuplicateKeyError` and returns 409. A pre-insert existence check alone would not be sufficient.

3. **Ties.** `$rank` assigns the same rank to equal totals and skips the next rank. I filter on rank only after calculating it, so ties at the cutoff are all returned. I then order equal totals by employee code.

4. **Headcount.** Department summary starts from employees and looks up monthly logs, preserving employees with no matching records. The joined-on filter excludes employees who had not joined by month end.

5. **100x data.** I would measure explain plans and query latency, add integration/load tests, and consider pre-aggregated daily summaries or a reporting store if repeated cross-collection analytics became expensive.
