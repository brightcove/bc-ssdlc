<!-- markdownlint-disable-file MD024 -->
# Secure Application Design Checklist [Security Process]

## What is the Secure Application Design Checklist?

The **Secure Application Design Checklist** is a list of requirements and suggestions that a product manager, architect, or other development personnel can run through to make sure they've fully covered all security requirements during their SDLC process.

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
| If redirect URIs are being used with OAuth2 authentication requests, ensure the URIs are validated as pointing to the expected FQDN | Required | [AuthZ and AuthN Guidelines - Validating OAuth2 Redirect URIs](../Coding%20Practice/AuthZ-AuthN-Guidelines.md#validating-oauth2-redirect-uris) |
| Limit OAuth2 scopes in line with the Principle of Least Privilege | Recommendation | [AuthZ and AuthN Guidelines - Limit OAuth2 Scope](../Coding%20Practice/AuthZ-AuthN-Guidelines.md#limit-oauth2-scope-by-the-principle-of-least-privilege) |
| Use and validate the state parameter with OAuth2 quthentication requests | Recommendation | [AuthZ and AuthN Guidelines - Validating the OAuth2 State Parameter](../Coding%20Practice/AuthZ-AuthN-Guidelines.md#validating-the-oauth2-state-parameter) |

#### User Management

| Action | Requirement or Recommendation? | BC SSDLC Reference |
| ------ | ------------------------------ | ------------------ |
| If passwords are being used, ensure the password policy complies with NIST 800-63 guidelines | Requirement | [AuthZ and AuthN Guidelines - Auth Guidelines](../Coding%20Practice/AuthZ-AuthN-Guidelines.md#authentication-guidelines) |
| Disable/delete user account records after a specified amount of inactivity | Recommendation | [Data Retention](../Coding%20Practice/Data-Retention.md) |
| Support SSO authentication for users (e.g. Okta, Google, Auth0) | Recommendation | N/A |
| Allow local accounts to be configured for use with Multi-Factor Authentication | Required for sensitive applications | [AuthZ and AuthN Guidelines - Require MFA](../Coding%20Practice/AuthZ-AuthN-Guidelines.md#require-multi-factor-authentication-for-sensitive-applications) |

#### Network-Accessible Applications

**Ex:** an encoding application listening on port 12345 for encoding requests and serving up completed encoding jobs

| Action | Requirement or Recommendation? | BC SSDLC Reference |
| ------ | ------------------------------ | ------------------ |
| Ensure all incoming network requests for non-public data or sensitive actions are authenticated and authorized before being processed | Requirement | N/A |
| Configure network access such that only the IP addresses/network hosts that need access to it, have access to it, and only for the ports and protocols in use (e.g. configure Security Groups to only allow web traffic from Brightcove IP addresses for internal applications) | Requirement | N/A |

### Input Validation/Output Encoding

#### Web Pages & APIs

| Action | Requirement or Recommendation? | BC SSDLC Reference |
| ------ | ------------------------------ | ------------------ |
| Ensure any untrusted input is properly encoded upon output | Requirement | [XSS Attacks](../Coding%20Practice/Preventing-Common-Web-Attacks.md#preventing-xss) |
| Add restrictions within web APIs to account for untrusted input sent to it | Requirement | [API Best Practices - Input Validation](../Coding%20Practice/API-Best-Practices.md#input-validation) |
| Set the Content-Security Policy to the proper values to restrict content usage | Requirement | [HTTP Headers - CSP](../Coding%20Practice/HTTP-Header-Security.md#content-security-policy-csp) |
| Database calls must use paramaterized queries, where applicable | Requirement | [SQL Injection](../Coding%20Practice/Preventing-Common-Web-Attacks.md#preventing-sql-injection) |
| Any service that accepts network addresses (FQDNs, IP addresses, hostnames) with the intention of initiating a connection to that address should ensure proper validation is in place to prevent malicious addresses from being used | Requirement | [Server-Side Request Forgery](../Coding%20Practice/Preventing-Common-Web-Attacks.md#server-side-request-forgery-ssrf) |
| Arbitrary file uploads must be properly validated upon upload | Requirement | [Arbitrary File Uploads](../Coding%20Practice/Preventing-Common-Web-Attacks.md#arbitrary-file-uploads) |
| Applications that use untrusted input for site redirection (HTTP 30x) destinations must protect against open-redirects | Requirement | [Open Redirect](https://github.com/brightcove/bc-ssdlc/blob/bc-master/Coding%20Practice/Preventing-Common-Web-Attacks.md#open-redirect) |
| Prevent content-sniffing by browsers | Requirement | [HTTP Headers - X-Content-Type-Options](../Coding%20Practice/HTTP-Header-Security.md#notes-on-apis) |
| Take into account the possibility of HTTP smuggling attacks when forwarding HTTP requests through multiple services | Requirement | [HTTP Request Smuggling](../Coding%20Practice/Preventing-Common-Web-Attacks.md#http-request-smuggling-aka-http-desync-attacks) |
| Implement request integrity | Recommendation | [API Best Practices - Request Integrity](../Coding%20Practice/API-Best-Practices.md#request-integrity) |

#### Networked Applications

| Action | Requirement or Recommendation? | BC SSDLC Reference |
| ------ | ------------------------------ | ------------------ |
| When using memory-unsafe languages, ensure untrusted intput isn't used for declaring buffer size, and that buffer size is appropriate for the incoming data the buffer set to hold | Requirement | N/A |
| Refrain from using object deserialization with untrusted data | Recommendation | N/A |

### Secrets Management

| Action | Requirement or Recommendation? | BC SSDLC Reference |
| ------ | ------------------------------ | ------------------ |
| Ensure no plaintext secrets (passwords, API tokens, etc) are included within any code | Requirement | [Credential Leaks](../Coding%20Practice/Preventing-Common-Web-Attacks.md#credential-leaks) |
| Any credentials needed for the application should be independently generated for the application, where applicable | Requirement | N/A |
| Secrets that needed to be statically stored for application deployment must use secure storage methods in accordance with [Brightcove cryptogaphic standards](https://brightcove.atlassian.net/wiki/spaces/IS/pages/905592/Cryptographic+Standards+Systems+Protocols+Algorithms+Primitives+and+More#Encryption-At-Rest) | Requirement | [Credential Leaks](../Coding%20Practice/Preventing-Common-Web-Attacks.md#credential-leaks) |
| For web applications, make sure the Referrer-Policy header is set to prevent unnecessary data leakage | Requirement | [HTTP Headers - Referrer-Policy](../Coding%20Practice/HTTP-Header-Security.md#referrer-policy) |

### Encryption

#### Web Pages & APIs

| Action | Requirement or Recommendation? | BC SSDLC Reference |
| ------ | ------------------------------ | ------------------ |
| Ensure TLS is used for all network communications, both internal and external | Requirement | [BC Internal - Guide to TLS - Should I Use TLS?](https://brightcove.atlassian.net/wiki/spaces/IS/pages/905819/Guide+to+TLS#GuidetoTLS-ShouldIUseTLSForMyApplication'sNetworkCommunications%3F) |
| When configuring TLS, ensure secure ciphers are used | Requirement | [BC Internal - Guide to TLS - Protocol and Ciphers](https://brightcove.atlassian.net/wiki/spaces/IS/pages/905819/Guide+to+TLS#GuidetoTLS-ProtocolsandCiphers) |
| Use HTTP Strict-Transport Security (HSTS) for all HTTP requests | Requirement | [HTTP Headers - HTTP Strict Transport Security](../Coding%20Practice/HTTP-Header-Security.md#http-strict-transport-security) |
| Make sure HTTP redirects (HTTP 30x) do not redirect users through HTTP endpoints before directing them to a TLS endpoint | Requirement | N/A |

#### User Management

| Action | Requirement or Recommendation? | BC SSDLC Reference |
| ------ | ------------------------------ | ------------------ |
| Ensure passwords are hashed, not encrypted | Requirement | [BC Internal - SSDLC - Cryptography - Storing Passwords](https://brightcove.atlassian.net/wiki/spaces/IS/pages/905592/Recommended+Encryption+Standards+Algorithms+and+Primitives#RecommendedEncryptionStandards%2CAlgorithms%2CandPrimitives-StoringofPasswords) |
| Passwords must be salted when stored, before being hashed | Requirement | [BC Internal - SSDLC - Cryptography - Salting Passwords](https://brightcove.atlassian.net/wiki/spaces/IS/pages/905592/Recommended+Encryption+Standards+Algorithms+and+Primitives#RecommendedEncryptionStandards%2CAlgorithms%2CandPrimitives-Salting) |
| Make sure a Brightcove-approved password hashing algorithm is used for storing passwords | Requirement | [BC Internal - SSDLC - Cryptography - Storing Passwords - Algorithms](https://brightcove.atlassian.net/wiki/spaces/IS/pages/905592/Recommended+Encryption+Standards+Algorithms+and+Primitives#RecommendedEncryptionStandards%2CAlgorithms%2CandPrimitives-argon2vsbcryptvsscryptvsPBKDF2) |
| Make sure a Brightcove-approved CSPRNG is used for generating password salts | Requirement | [BC Internal - SSDLC - Cryptography - Storing Passwords - Salting Procedures](https://brightcove.atlassian.net/wiki/spaces/IS/pages/905592/Recommended+Encryption+Standards+Algorithms+and+Primitives#RecommendedEncryptionStandards%2CAlgorithms%2CandPrimitives-SaltingProcedures) |

#### Persistent Data Encryption

| Action | Requirement or Recommendation? | BC SSDLC Reference |
| ------ | ------------------------------ | ------------------ |
| Confirm sensitive data is encrypted-at-rest when stored persistently | Requirement | [BC Internal - SSDLC - Cryptography - Encryption-at-Rest](https://brightcove.atlassian.net/wiki/spaces/IS/pages/905592/Recommended+Encryption+Standards+Algorithms+and+Primitives#RecommendedEncryptionStandards%2CAlgorithms%2CandPrimitives-Encryption-At-Rest) |

### Infrastructure

#### Cloud Security

| Action | Requirement or Recommendation? | BC SSDLC Reference |
| ------ | ------------------------------ | ------------------ |
| Ensure Brightcove Cloud Security Standards are met when creating or updating any cloud infrastructure | Requirement | [BC Internal - Cloud Security Standards](https://brightcove.atlassian.net/wiki/spaces/IS/pages/14031749252/Cloud+Security+Standards) |
| Cloud resources must be configured securely, meaning as little access as needed | Requirement | [BC Internal - Guide to Security in AWS](https://brightcove.atlassian.net/wiki/spaces/IS/pages/905669/Guide+to+Security+in+AWS) |

#### Container Security

| Action | Requirement or Recommendation? | BC SSDLC Reference |
| ------ | ------------------------------ | ------------------ |
| Make sure to only use official, up-to-date official images | Requirement | [BC Internal - Docker Security Guide](https://brightcove.atlassian.net/wiki/spaces/IS/pages/1889862929/Docker+Security+Guide#Use-minimal,-trusted-base-images) |
| Run applications as service account users as opposed to the default `root` user | Requirement | [BC Internal - Docker Security Guide](https://brightcove.atlassian.net/wiki/spaces/IS/pages/1889862929/Docker+Security+Guide#Don%E2%80%99t-run-the-container-as-root) |
| Only use `EXPOSE` with ports that are in use and need to be exposed outside of the container | Requirement | [BC Internal - Docker Security Guide](https://brightcove.atlassian.net/wiki/spaces/IS/pages/1889862929/Docker+Security+Guide#Only-expose-the-ports-you-need-when-using-the-EXPOSE-instruction) |
| Do not include any secrets or sensitive data within a container | Requirement | [BC Internal - Docker Security Guide](https://brightcove.atlassian.net/wiki/spaces/IS/pages/1889862929/Docker+Security+Guide#Don%E2%80%99t-store-secrets-directly-in-Docker-files) |
| Any static secrets in use by the container during runtime must be encrypted when used within the Kubernetes manifest in accordance with Brightcove cryptographic standards | Requirement | [BC Internal - SSDLC - Container Security](https://brightcove.atlassian.net/wiki/spaces/IS/pages/905780/Brightcove+Secure+Software+Development+Lifecycle+SSDLC+Guide#BrightcoveSecureSoftwareDevelopmentLifecycle(SSDLC)Guide-ContainerSecurity) |
| Review the Brightcove Docker Security Guide to ensure your container image is as secure as possible | Recommended | [BC Internal - Docker Security Guide](https://brightcove.atlassian.net/wiki/spaces/IS/pages/1889862929/Docker+Security+Guide) |

#### Kubernetes Security

| Action | Requirement or Recommendation? | BC SSDLC Reference |
| ------ | ------------------------------ | ------------------ |
| Run the container as non-root | Required | [BC Internal - Kubernetes Security Guide](https://brightcove.atlassian.net/wiki/spaces/IS/pages/1891041307/Kubernetes+Security+Guide#Implement-a-Seccomp-Policy) |
| Disallow container privilege escalation | Required | [BC Internal - Kubernetes Security Guide](https://brightcove.atlassian.net/wiki/spaces/IS/pages/1891041307/Kubernetes+Security+Guide#Disallow-Privilege-Escalation) |
| Do not run the container in privileged mode | Required | [BC Internal - Kubernetes Security Guide](https://brightcove.atlassian.net/wiki/spaces/IS/pages/1891041307/Kubernetes+Security+Guide#Refrain-From-Running-Container-as-Privileged) |
| Implement a Seccomp policy | Recommended | [BC Internal - Kubernetes Security Guide](https://brightcove.atlassian.net/wiki/spaces/IS/pages/1891041307/Kubernetes+Security+Guide#Implement-a-Seccomp-Policy) |
| Implement kernel security defenses | Recommended | [BC Internal - Kubernetes Security Guide](https://brightcove.atlassian.net/wiki/spaces/IS/pages/1891041307/Kubernetes+Security+Guide#Kernel-Security) |
| Review the Brightcove Kubernetes Security Guide to ensure your container image is deployed as securely as possible | Recommended | [BC Internal - Kubernetes Security Guide](https://brightcove.atlassian.net/wiki/spaces/IS/pages/1891041307/Kubernetes+Security+Guide) |

### Logging

| Action | Requirement or Recommendation? | BC SSDLC Reference |
| ------ | ------------------------------ | ------------------ |
| Configure log forwarding to a remote log server, SaaS service, or SIEM | Requirement | [BC Internal - SSDLC - Logging](https://brightcove.atlassian.net/wiki/spaces/IS/pages/1887208183/Logging+At+Brightcove#Logging-Setup) |  
| Include the ability for verbose logging to log any untrusted data and associated remote identifiers (e.g. user ID) | Requirement | N/A |
| Ensure logs are retained for at least 30 days | Requirement | N/A |

### Data Retention

| Action | Requirement or Recommendation? | BC SSDLC Reference |
| ------ | ------------------------------ | ------------------ |
| Ensure all PII-related data points that are collected and persistently stored include a method for automated or on-demand removal | Requirement | [Data Retention](../Coding%20Practice/Data-Retention.md) |
| Make sure to set a static expiration date for all data points of a given set | Requirement | [Data Retention](../Coding%20Practice/Data-Retention.md) |
| Expiration dates set for data points must be set only as long as the data's usable lifespan to the business | Requirement | [Data Retention](../Coding%20Practice/Data-Retention.md) |

### Security Tools

| Action | Requirement or Recommendation? | BC SSDLC Reference |
| ------ | ------------------------------ | ------------------ |
| Configure source code to be scanned by SAST platform | Requirement | [BC Internal - Tools - SAST](https://brightcove.atlassian.net/wiki/x/GgCBWgM) |
| Integrate source code with dependency management application | Requirement | [BC Internal - Tools - Dependency Management](https://brightcove.atlassian.net/wiki/x/GgCBWgM) |

### Vulnerability Scanning/Patch Management

| Action | Requirement or Recommendation? | BC SSDLC Reference |
| ------ | ------------------------------ | ------------------ |
| Ensure application containers are being scanned by vulnerability management platform | Requirement | [BC Internal - Tools - Vulnerability Scanning/Management](https://brightcove.atlassian.net/l/cp/C0A77W9k) |
| Confirm application containers are patched before being deployed to production environments | Requirement | [BC Internal - SSDLC - Patching Guide](https://brightcove.atlassian.net/l/cp/NimyhFn4) |
| Ensure that patching standards are followed and a plan is in place to patch the application and its infrastructure on a regular basis | Requirement | [BC Internal - SSDLC - Patching Standard](https://brightcove.atlassian.net/wiki/spaces/IS/pages/14210891915/Patch+Management+Standard) |

### SCM Security

| Action | Requirement or Recommendation? | BC SSDLC Reference |
| ------ | ------------------------------ | ------------------ |
| Ensure source code organizations and repos are only accessible by users who truly require access | Required | N/A |
| Implement branch protections on production and main branches | Recommended | [Github Docs - Protected Branches](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/defining-the-mergeability-of-pull-requests/about-protected-branches) |
| Refrain from checking in any secret or confidential data to source code repos | Requirement | [BC Internal - SSDLC - Securing Development Environments](https://brightcove.atlassian.net/l/cp/Xy5bLWRF) |
| Require at least two reviews for PRs to be merged | Recommended | [Github Docs - PR Review Requirements](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/defining-the-mergeability-of-pull-requests/about-protected-branches#require-pull-request-reviews-before-merging) |
| Don't create repos in personal organizations. If this _is_ needed, ensure that it is set to `private` visibility upon creation | Recommended | [Github Docs - Repo Visibility](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/managing-repository-settings/setting-repository-visibility) |

### CI/CD Security

| Action | Requirement or Recommendation? | BC SSDLC Reference |
| ------ | ------------------------------ | ------------------ |
| Only use CI/CD platforms approved for use by Brightcove | Required | N/A |
| Refrain from using hard-code secrets for deployments | Required | [BC Internal - SSDLC - Using Application Secrets Securely](https://brightcove.atlassian.net/l/cp/6Gp9fWaM) |
