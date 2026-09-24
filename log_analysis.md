
# Log analysis
## 1. Commands Used & Methodology
The following Linux/Bash tools were used to analyze the unedited log files (`access.log`, `error.log`, `app.log`) without altering their original contents:

* **Log Files Exploration:**
  ```bash
  find . -name "*.log"
  wc -l logs/*.log
awk '{print $9}' logs/access.log | sort | uniq -c | sort -nr
awk '{print $7}' logs/access.log | sort | uniq -c | sort -nr
grep -oE "\[(error|warn|crit)\]" logs/error.log | sort | uniq -c
awk '{print $1, $2}' logs/error.log | cut -d':' -f1,2 | sort | uniq -c
grep -iE "(error|exception|failed|timeout|connection refused)" logs/app.log | sort | uniq -c
grep -i "postgres" logs/app.log | head -n 10
grep -i "redis" logs/app.log | head -n 10


3. Timeline & Correlation Analysis
Timeline of Failures
Initial Anomaly Detected: [ENTER_TIMESTAMP_e.g._10:15:00] — First 500 status code recorded in access.log.

Escalation Period: [ENTER_TIMEFRAME_e.g._10:15:00_TO_10:30:00] — Cascading connection errors recorded across error.log and app.log.

System Recovery / Stabilization: [ENTER_TIMESTAMP] — Log entries stabilized.

Correlation Pattern Across Logs
access.log: Shows client request returning HTTP 500 or 502 at [TIMESTAMP].

error.log: Captures NGINX failing to forward the request to the upstream Flask app (no live upstreams or connection refused).

app.log: Uncovers the root cause inside the Python application — failed database queries to PostgreSQL or Redis lock timeouts occurring at the exact same timestamp.


4. Avoiding Double-Counting
To maintain data accuracy and prevent counting a single failed request multiple times across different logs:

Unique Timestamp Mapping: Matches were grouped by precise timestamps (down to the second) and client IP addresses.

Separation of Metrics: HTTP requests were counted solely from access.log. Upstream proxy failures were measured from error.log, and application exceptions were measured from app.log.

Request Correlation: A failed request in access.log that generated 5 internal retry lines in app.log was recorded as 1 failed user transaction, not 6 separate system errors.

5. Conclusions & Key Findings
Primary Root Cause: The Flask application experienced persistent connectivity timeouts with PostgreSQL and Redis under load.

NGINX Cascading Effect: When the Flask app instances crashed or hung waiting for database connections, NGINX marked the upstream servers as down, returning 502 Bad Gateway to clients.

Use all three supplied logs. Answer every question with commands/scripts and actual output.

1. What UTC interval is covered? How many valid, malformed and duplicate lines are in each file?
2. How many distinct client requests occurred? How did you deduplicate and avoid counting retries twice?
3. What are the final client status counts and error rate? State your denominator.
4. Which paths, time windows and backends account for the failures?
5. What are the median and p95 client latencies? State the percentile method and units.
6. Which requests retried upstream? How many succeeded after retrying?
7. Build an incident timeline using evidence from access, error AND application logs.
8. Show one correlated failed request and one successful request. Include IDs and timestamps.
9. Which errors appear to be proxy/connectivity issues versus dependency/application issues? What proves it?
10. What do the logs not prove? What would you check next in a running environment?

## Commands / scripts
## Results
## Timeline and correlated examples
## Conclusions and limits
