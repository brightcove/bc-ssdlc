# WIP - Secure Application Design Checklist [Security Process]

## What is the Secure Application Design Checklist?

The **Secure Application Design Checklist** is a list of requirements and suggestions that a product manager, architect, etc can go through to make sure they've fully covered security during their application design phase.

The checklist is based on the list of security requirements outlined in the [Security Requirements](./Security-Requirements.md) document.
## Checklist
### Authentication and Authorization
#### Web APIs

| Action | Requirement or Recommendation? | BC SSDLC Reference |
| ------ | ------------------------------ | ------------------ |
| Ensure all API endpoints that serve non-public data utilize BC tokens and Cathy for authentication and authorization for requests | Requirement | [API Best Practices - Access Controls](./Coding%20Practice/API-Best-Practices.md#access-controls) |
| If RBAC is being implemented, ensure sensitive endpoints are only accessible by the roles that require it | Requirement | [API Best Practices - Access Controls](./Coding%20Practice/API-Best-Practices.md#access-controls) |
#### Network-Accessible Applications

**Ex:** apache2, an encoding application listening on port 12345 for encoding requests

| Action | Requirement or Recommendation? | BC SSDLC Reference |
| ------ | ------------------------------ | ------------------ |
| Ensure all incoming network requests for non-public data are authenticated and authorized before being processed | Requirement |  |
| |||
### Input Validation/Output Encoding

### Secrets Management

### Encryption

### Logging