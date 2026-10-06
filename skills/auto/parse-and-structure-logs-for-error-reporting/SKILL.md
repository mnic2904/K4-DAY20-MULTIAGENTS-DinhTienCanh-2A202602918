---
name: parse-and-structure-logs-for-error-reporting
description: Use this skill when extracting error entries from service logs to produce structured error reports following strict schema and formatting rules.
---
- Read log lines sequentially, identifying entries by timestamp, log level, service name, and message.
- Filter entries to include only ERROR and CRITICAL levels (case-insensitive).
- Normalize service names to lowercase and replace dashes with underscores.
- Convert all timestamps to UTC in ISO 8601 format with 'Z' suffix (YYYY-MM-DDTHH:MM:SSZ).
- Extract the main message text after the service name and colon on the first line.
- If a traceback follows, extract the last line as the exception message; otherwise, set exception to null.
- Detect and sum repeated messages indicated by lines like "-- last message repeated N times --" to compute repeat_count.
- Sort the final error list by service name, then timestamp ascending.
- Compute counts_by_service as the sum of repeat_count per service.
- Include required top-level metadata fields: schema_version and generated_by.
- Validate the final JSON output against the schema before submission.
