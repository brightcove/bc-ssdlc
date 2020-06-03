# WIP - Secure Application Design Checklist [Security Process]

## What is the Secure Application Design Checklist?

The **Secure Application Design Checklist** is a list of requirements and suggestions that a product manager, architect, etc can go through to make sure they've fully covered security during their application design phase.

The checklist is based on the list of security requirements outlined in the [Security Requirements](./Security-Requirements.md) document.
## Checklist
### Authentication and Authorization
#### Web Pages & APIs

| Action | Requirement or Recommendation? | BC SSDLC Reference |
| ------ | ------------------------------ | ------------------ |
| Ensure all API endpoints that serve non-public data utilize BC tokens and Cathy for authentication and authorization for requests | Requirement | [API Best Practices - Access Controls](../Coding%20Practice/API-Best-Practices.md#access-controls) / [Cross-Site Request Forgery](../Coding%20Practice/Preventing-Common-Web-Attacks.md#preventing-cross-site-request-forgery) |
| If RBAC is being implemented, ensure sensitive endpoints are only accessible by the roles that require it | Requirement | [API Best Practices - Access Controls](../Coding%20Practice/API-Best-Practices.md#access-controls) |
| Authentication tokens must have a static expiration date that's enforced on the backend network | Requirement | [API Best Practices - Replay Attacks](../Coding%20Practice/API-Best-Practices.md#replay-attacks) / [AuthZ and AuthN Guidelines - Limited Token Lifetimes](../Coding%20Practice/AuthZ-AuthN-Guidelines.md#limited-token-lifetimes) |
| Ensure sensitive API actions can't be performed via click-jacking | Requirement | [Click-Jacking](../Coding%20Practice/Preventing-Common-Web-Attacks.md#preventing-clickjacking) |
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
| Ensure all incoming network requests for non-public data or sensitive actions are authenticated and authorized before being processed | Requirement | N/A |
### Input Validation/Output Encoding
#### Web Pages & APIs

| Action | Requirement or Recommendation? | BC SSDLC Reference |
| ------ | ------------------------------ | ------------------ |
| Ensure any untrusted input is properly encoded before or upon output | Requirement | [XSS Attacks](../Coding%20Practice/Preventing-Common-Web-Attacks.md#preventing-xss) |
| Add restrictions within web APIs to account for untrusted input sent to it | Requirement | [API Best Practices - Input Validation](../Coding%20Practice/API-Best-Practices.md#input-validation) |
| Set the Content-Security Policy to the proper values to restrict content usage | Requirement | [HTTP Headers - CSP](../Coding%20Practice/HTTP-Header-Security.md#content-security-policy-csp) |
| Database calls must use paramaterized queries, where applicable | Requirement | [SQL Injection](../Coding%20Practice/Preventing-Common-Web-Attacks.md#preventing-sql-injection) |
| Any service that accepts network addresses (FQDNs, IP addresses, hostnames) with the intention of initiating a connection to it should ensure proper validation | Requirement | [Server-Side Request Forgery](../Coding%20Practice/Preventing-Common-Web-Attacks.md#server-side-request-forgery-ssrf) |
| Arbitrary file uploads must be properly validated upon upload | Requirement | [Arbitrary File Uploads](../Coding%20Practice/Preventing-Common-Web-Attacks.md#arbitrary-file-uploads) |
| Applications that use untrusted input for site redirection (HTTP 30x) destinations must protect against open-redirects | Requirement | N/A (Coming Soon) |
| Prevent content-sniffing by browsers | Requirement | [HTTP Headers - X-Content-Type-Options](../Coding%20Practice/HTTP-Header-Security.md#notes-on-apis) |
| Take into account the possibility of HTTP smuggling attacks when forwarding HTTP requests through multiple services | Requirement | [HTTP Request Smuggling](../Coding%20Practice/Preventing-Common-Web-Attacks.md#http-request-smuggling-aka-http-desync-attacks) |
| Implement request integrity | Recommendation | [API Best Practices - Request Integrity](../Coding%20Practice/API-Best-Practices.md#request-integrity) |
#### Network-Accessible Applications

| Action | Requirement or Recommendation? | BC SSDLC Reference |
| ------ | ------------------------------ | ------------------ |
| When using memory-unsafe languages, ensure untrusted output isn't used for declaring buffer size, and that buffer size is appropriate for the incoming data the buffer set to hold | Requirement | N/A |
| Refrain from using object deserialization from untrusted sources | Recommendation | N/A |
### Secrets Management

| Action | Requirement or Recommendation? | BC SSDLC Reference |
| ------ | ------------------------------ | ------------------ |
| Ensure no plaintext secrets (passwords, API tokens, etc) are included within any code | Requirement | [Credential Leaks](../Coding%20Practice/Preventing-Common-Web-Attacks.md#credential-leaks) |
| Any credentials needed for the application should be independently generated for the application, where applicable | Requirement | N/A |
| Secrets needed to be statically stored for application deployment must use secure storage methods | Requirement | [Credential Leaks](../Coding%20Practice/Preventing-Common-Web-Attacks.md#credential-leaks) |
| For web applications, make sure the Referrer-Policy header is set to prevent secret data leakage | Requirement | [HTTP Headers - Referrer-Policy](../Coding%20Practice/HTTP-Header-Security.md#referrer-policy) |
### Encryption
#### Web Pages & APIs

| Action | Requirement or Recommendation? | BC SSDLC Reference |
| ------ | ------------------------------ | ------------------ |
| Ensure TLS is used for all network communications, both internal and external | Requirement | N/A |
| When configuring TLS, ensure secure ciphers are used | Requirement | See internal SSDLC document |
| Use HTTP Strict-Transport Security (HSTS) for all HTTP requests | Requirement | [HTTP Headers - HTTP Strict Transport Security](../Coding%20Practice/HTTP-Header-Security.md#http-strict-transport-security) |
| Make sure HTTP redirects (HTTP 30x) do not redirect users through HTTP endpoints before directing them to a TLS endpoint | Requirement | N/A |
#### User Management

| Action | Requirement or Recommendation? | BC SSDLC Reference |
| ------ | ------------------------------ | ------------------ |
| Ensure passwords are hashed, not encrypted | Requirement | See internal SSDLC document |
| Passwords must be salted when stored, before being hashed | Requirement | See internal SSDLC document | 
| Make sure a Brightcove-approved password hashing algorithm is used for storing passwords | Requirement | See internal SSDLC document |
| Make sure a Brightcove-approved CSPRNG is used for generating password salts | Requirement | See internal SSDLC document |
#### Persistent Data Encryption

| Action | Requirement or Recommendation? | BC SSDLC Reference |
| ------ | ------------------------------ | ------------------ |
| Sensitive data must be encrypted-at-rest when stored persistently | Requirement | See internal SSDLC document |
### Logging

| Action | Requirement or Recommendation? | BC SSDLC Reference |
| ------ | ------------------------------ | ------------------ |
| Configure log forwarding to a remote log server, SaaS service, or SIEM | Requirement | N/A |  
| Include the ability for verbose logging to log any untrusted data and associated remote identifiers (e.g. user ID) | Requirement | N/A |
| Ensure logs are retained for at least 30 days | Requirement | N/A |