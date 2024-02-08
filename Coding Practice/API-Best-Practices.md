<!-- markdownlint-disable-file MD024 -->
# API Best Practices Guidelines [Coding Practice]

## Overview

This document covers general security guidelines for API endpoints within Brightcove. These guidelines will cover general points like:

- [Access control best practices](#access-controls)
- [Input validation](#input-validation)
- [Request verification](#request-integrity)
- [Replay attacks](#replay-attacks)
- [Logging and Error Management](#logging-and-error-management)

### Recommendations

#### Access Controls

##### Description

API endpoints should follow the principle of least privilege. Services with protected information should serve to the smallest group possible.

##### Why We Care

APIs with misconfigured access controls can lead to unintentional information leaks, or unauthorized and malicious state-changing actions on sensitive data.

##### Example of Issue (Optional)

A POST that allows the user to modify information on an account without checking if the user owns the account being modified.

A GET request that returns sensitive informative information without authentication.

##### How to Fix?

Determine which API actions should be considered sensitive or public.

For Internal (to Brightcove) APIs: Along with the same steps required for public sensitive APIs, try to also limit network access as much as possible. Ideally, these endpoints should be restricted to a closed network, and require multi-factor authentication or tie-in with our internal SSO provider.

For Public APIs with sensitive information: Require authentication before performing any action being requested. API keys should be both revocable and renewable.

For Public APIs providing public information: Ensure no state changing actions are being performed through a public API without authentication. Consider rate limiting to prevent a single host making too many requests in a small amount of time.

##### Risk Rating

Incorrect access controls can range from High to Low Severity.

##### References

- <https://www.owasp.org/index.php/REST_Security_Cheat_Sheet>

---

#### Input Validation

##### Description

Incoming data can be malformed or crafted to cause unintended behavior when it is parsed.

##### Why We Care

Depending on how input is parsed, it is possible for unvalidated input to contain command injections, or other harmful actions.

##### Example of Issue

Accepting a DELETE request for an API that should only accept GET and PUT requests, causing data to be deleted or malformed
in the backend database

##### How to Fix?

Type checking - Ensure input is of the expected data type, reject anything else.

Length and size checking - Input should be within an expected length or size. Reject anything larger or smaller than expected.

Whitelist accepted content-types.

Restrict http methods.

Parsing - Third party parsers should be kept up to date, changes to internal parsers should be carefully reviewed.

##### Risk Rating

Input Validation issues could range from Low to High depending on how the error can be leveraged.

---

#### Request Integrity

##### Description

It is possible that a request could be modified in-transit between the original requester and the API endpoint.

##### Why We Care

Modified requests may cause state changing actions to the original requester’s data, or cause incorrect, modified, or 
unexpected data to be served.

##### Example of Issue

A JSON Web Token is utilized in a web-application, but does not include an HMAC as the specification requires to pretect
token integrity

##### How to Fix?

Utilize TLS for all network requests

Integrate signing of requests, e.g. by using a recommended HMAC algorithm

---

#### Replay Attacks

##### Description

An attacker sends a previous, genuine request to cause an action to happen again at a later time.

Modified requests in-transit: An attacker modifies data in the genuine request as it is sent.

##### Why We Care

Some API requests (e.g. login-related requests) are time-sensitive, meaning they are only valid for a specific period of time. Not taking this time restriction into account could allow unauthorized usage or changes to resources.

##### How to Fix?

Include timestamps within signed requests (see above) and deny all requests that are relatively too old.

##### Risk Rating

Depending on what actions the request can take, severity could range from Low to High.

##### References

- <https://docs.aws.amazon.com/general/latest/gr/signing_aws_api_requests.html#why-requests-are-signed>

---

#### Logging and Error Management

##### Description

Security practices that are relevant to an application's logging and error management techniques.

##### Why We Care

Logging is important for ensuring security events are being effectively monitored for, as well an helps with forensics activity, should the need for that ever arise.

Sensitive data can accidentally be exposed via error messages and other outputs.

##### Example of Issue

No logging, or logging that's not verbose enough (or even too verbose).

Returning stack traces or other descriptive information of the service backend.

##### How to Fix?

Ensure that your application is coded and configured to log to our central log repository, as this helps with monitoring and investigations.

Return relatively vague error responses. Put as little information as possible when returning an error to the user. Do not return any configuration data, information about the server environment, or debug information like stack traces.

##### Risk Rating

Ranging from Low to High depending on context.
