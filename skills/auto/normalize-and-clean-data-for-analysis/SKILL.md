---
name: normalize-and-clean-data-for-analysis
description: Use this skill when preparing raw data files for analysis to ensure consistent formatting, deduplication, and correct data types.
---
- Normalize categorical fields by trimming whitespace and standardizing capitalization (e.g., capitalize first letter only).
- Parse date/time fields robustly by supporting multiple known formats and converting all to a single canonical timezone (usually UTC).
- Detect and remove exact duplicate rows.
- Remove duplicate entries by unique keys (e.g., order_id), keeping the first occurrence.
- Identify and handle missing or sentinel values (e.g., -999 for amounts) by excluding or flagging them as needed.
- Convert monetary values to integer cents if required by specification.
- Write cleaned data to output files with correct headers and formats.
- Document metadata such as source file name, total rows read, and rows used.
- Validate output files against rules before submission.
