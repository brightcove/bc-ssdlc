# Security in Testing Environments (QA/Staging) [Security Process]

## What is Considered a Testing Environment?

A **testing environment** is any non-production application environment. This may be a local developer environment on an engineer's workstation, to dedicated resources that mirror production used for staging new released before being released to production.

## Best Practices

When building, updating, or using a testing environment at Brightcove, it's important to consider several best practices:
- Whenever possible, engineers should generate fake/test data values instead of migrating from production
- If production data _is_ required, engineers must make sure that all data being transferred is either:
  - Unrecoverable (e.g. data is hashed or encrypted instead of plaintext)
  - Transformed in such a way that it becomes common data (e.g. Setting all last names to 'Smith' for a database table containing [**only**] first and last names)
- Use separate authentication credentials (especially tokens) for testing environments than you do in the production environment
