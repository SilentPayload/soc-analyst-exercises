# Datasets

Larger synthetic files for SIEM, timeline and reporting work. Not tied to a
specific worksheet yet.

## `attack-timeline.csv`

A labelled incident timeline. Columns: `_time`, `sourcetype`, `event_label`,
`AccountName`, `SourceIP`, `Amount`, `CommandLine`. Events run from SQL
injection activity through to the actions that followed, each tagged with a
label.

Useful for teaching timeline reconstruction and for demonstrating what a
labelled dataset looks like when students later build detections.

## `siem-enterprise.csv`

An enterprise event log. Columns: `timestamp`, `source_ip`, `destination_ip`,
`username`, `event_type`, `action`, `severity`, `protocol`, `port`, `bytes_in`,
`bytes_out`, `country`, `user_agent`, `status_code`, `message`.

Contains successful and failed logins, file access, severity grading and
country codes, which makes it suitable for exercises on filtering by severity,
spotting access from unexpected locations, and correlating a failed login run
against a later success.

## Note

Both files are synthetic. Addresses use private and documentation ranges.
Nothing here is from a real environment or a real engagement.
