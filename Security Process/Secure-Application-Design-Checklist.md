# WIP - Secure Application Design Checklist [Security Process]

## What is the Secure Application Design Checklist?

The **Secure Application Design Checklist** is a list of requirements and suggestions that a product manager, architect, etc can go through to make sure they've fully covered security during their application design phase.

The checklist is based on the list of security requirements outlined in the [Security Requirements](./Security-Requirements.md) document.
## Checklist
### Authentication and Authorization
#### Web APIs

| Action | Requirement or Recommendation? | BC SSDLC Reference |
| ------ | ------------------------------ | ------------------ |
| Ensure all API endpoints that serve non-public data utilize BC tokens and Cathy for authentication and authorization for requests | Requirement | [API Best Practices - Access Controls](../Coding%20Practice/API-Best-Practices.md#access-controls) |
| If RBAC is being implemented, ensure sensitive endpoints are only accessible by the roles that require it | Requirement | [API Best Practices - Access Controls](../Coding%20Practice/API-Best-Practices.md#access-controls) |
| Authentication tokens must have a static expiration date that's enforced on the backend network | Requirement | [API Best Practices - Replay Attacks](../Coding%20Practice/API-Best-Practices.md#replay-attacks) / [AuthZ and AuthN Guidelines - Limited Token Lifetimes](../Coding%20Practice/AuthZ-AuthN-Guidelines.md#limited-token-lifetimes) |
| Limit OAuth2 scopes in line with the Principle of Least Privilege | Recommendation | [AuthZ and AuthN Guidelines - Limit OAuth2 Scope](../Coding%20Practice/AuthZ-AuthN-Guidelines.md#limit-oauth2-scope-by-the-principle-of-least-privilege) |
| Validate redirect URIs used with OAuth2 quthentication requests | Recommendation | [AuthZ and AuthN Guidelines - Validating OAuth2 Redirect URIs](../Coding%20Practice/AuthZ-AuthN-Guidelines.md#validating-oauth2-redirect-uris) |
| Use and validate the state parameter with OAuth2 quthentication requests | Recommendation | [AuthZ and AuthN Guidelines - Validating the OAuth2 State Parameter](../Coding%20Practice/AuthZ-AuthN-Guidelines.md#validating-the-oauth2-state-parameter) |
#### User Management

| Action | Requirement or Recommendation? | BC SSDLC Reference |
| ------ | ------------------------------ | ------------------ |
| If passwords are used, ensure the password policy complies with NIST-800 63 guidelines | Requirement | [AuthZ and AuthN Guidelines - Auth Guidelines](../Coding%20Practice/AuthZ-AuthN-Guidelines.md#authentication-guidelines) |
| Disable/delete user account records after a specified amount of inactivity | Recommendation | [Data Retention](../Coding%20Practice/Data-Retention.md) |
| Allow SSO authentication for users (e.g. Okta, Google, Auth0) | Recommendation | N/A |
| Allow local accounts to be configured for use with Multi-Factor Authentication | Required for sensitive applications | [AuthZ and AuthN Guidelines - Require MFA](../Coding%20Practice/AuthZ-AuthN-Guidelines.md#require-multi-factor-authentication-for-sensitive-applications) |
#### Network-Accessible Applications

**Ex:** an encoding application listening on port 12345 for encoding requests and serving up completed encoding jobs

| Action | Requirement or Recommendation? | BC SSDLC Reference |
| ------ | ------------------------------ | ------------------ |
| Ensure all incoming network requests for non-public data are authenticated and authorized before being processed | Requirement |  |
### Input Validation/Output Encoding

### Secrets Management

### Encryption

### Logging