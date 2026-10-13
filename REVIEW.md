# Starter code review

| # | Where | What is wrong | How to notice | Fix in revised implementation |
|---|---|---|---|---|
| 1 | `health()` | Reports ready without contacting MongoDB. | Stop MongoDB and call `/health`. | Ping MongoDB and return 503 on failure. |
| 2 | `create_employee()` | Check-then-insert is unsafe for concurrent duplicate codes. | Send parallel POSTs with the same `emp_code`. | Unique index plus handling `DuplicateKeyError`. |
| 3 | `EmployeeIn` | Missing contract validation and shift equality check. | Send malformed employee code, email, date or equal shift times. | Add Pydantic constraints and validators. |
| 4 | `create_employee()` | Uses naive local `datetime.now()` and returns no epoch-ms conversion. | Inspect `created_at` across timezone settings. | Store UTC-aware server time and serialize to epoch milliseconds. |
| 5 | `list_employees()` | `skip=page * page_size` skips the first page. | Request page 1 with page size 2. | Use `(page - 1) * page_size`. |
| 6 | `list_employees()` | `total` ignores department filter. | Filter a department and compare total to matching rows. | Count with the same query. |
| 7 | `list_employees()` | No sort and no pagination bounds. | Repeat pages or request page size over 100. | Sort by `emp_code`; validate page and page size. |
| 8 | `compute_late_minutes()` | Uses floored minutes to decide whether grace was exceeded. | Punch in at 09:40:00.900 for a 09:30 shift. | Compare elapsed seconds strictly to 600 before flooring. |
| 9 | `compute_late_minutes()` | Does not account for overnight shift start dates. | Use a 22:00–06:00 shift and punch in at 05:00. | Resolve the shift's actual start date in IST. |
| 10 | `compute_work_hours()` | Python `round()` uses ties-to-even, not half-up. | Test values exactly at a half-cent boundary. | Use `Decimal` with `ROUND_HALF_UP`. |
| 11 | `compute_overtime()` | Always uses the same calendar date for shift end. | Punch out after midnight on an overnight shift. | Move shift end to the next day for overnight shifts. |
| 12 | `punch_in()` | Unknown employee leads to dereferencing `None`; timestamp uses machine local time. | Punch in for an unknown code or run under another timezone. | Return 404; parse epoch ms as UTC and derive date in IST. |
| 13 | `punch_in()` | Check-then-insert can create duplicates under concurrency. | Parallel punch-in requests for the same employee/date. | Unique compound index and duplicate-key handling. |
| 14 | `punch_in()` | Accepts unsupported statuses and does not truncate/validate timestamps consistently. | Send `ABSENT`, seconds, float, string or sub-second timestamp. | Restrict status and validate/truncate epoch-ms input. |
| 15 | `list_attendance()` | Fetches the entire matching result set into Python. | Load 100k records and inspect memory/latency. | Filter, sort, count and paginate in MongoDB. |
| 16 | `list_attendance()` | Returns Mongo `_id` as a response `id`, contrary to the contract. | Inspect the response schema. | Exclude `_id`; identify records by `emp_code` and `date`. |
| 17 | `list_attendance()` | Does not reject an inverted date range and does not use deterministic database sorting. | Request `date_from` later than `date_to`. | Validate range and sort by date descending, employee ascending. |
| 18 | Missing endpoints | Punch-out, regularization, analytics and explain are absent. | Call each endpoint from the OpenAPI contract. | Implement each route and add indexes at startup. |

**Reviewed and retained:** the use of separate `employees` and `attendance_logs` collections matches the supplied data model; `emp_code` is the business key; derived attendance fields are stored so analytics can use the authoritative stored values. These choices are intentional, not defects.
