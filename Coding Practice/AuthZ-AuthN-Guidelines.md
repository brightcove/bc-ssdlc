# Authorization and Authentication Guidelines [Coding Practice]

## Overview

This document provides guidelines for how to implement authentication and authorization in Brightcove applications.

These guidelines will cover general points like:
- [Passwords and Other Forms of Authentication](#authentication-guidelines)
- [Require Multi-Factor Authentication for Sensitive Applications](#require-multi-factor-authentication-for-sensitive-applications)
- [Limit OAuth2 Scope by the Principle of Least Privilege](#limit-oauth2-scope-by-the-principle-of-least-privilege)
- [Limited Token Lifetimes](#limited-token-lifetimes)
- [Validating OAuth2 Redirect URIs](#validating-oauth2-redirect-uris)
- [Validating the OAuth2 State Parameter](#validating-the-oauth2-state-parameter)

**Authentication** is the process of verifying the user’s identity typically through an initial login and subsequently by verifying a unique secret identifier on each HTTP request made by the user’s browser.

**Authorization** is the process of ensuring a user can only access resources they are supposed to, usually based on some role or Risk Rating that is defined according to business logic.

## User Management
Even before authenticating and authorizing users comes the problem of user management. Bots, credential stuffing, compromised accounts...all of it combined can be tough to manage securely. Below are several recommendations on how to help combat these issues.

### Account Verification
Almost every application that allows public account creation will need to defend against the creation of non-human accounts, a.k.a. bot accounts. These accounts can be used for a variety of purposes: farming discount codes, hosting illegal content, getting around limits, etc.

To better improve verification of accounts, you can:

1. Require a manual action to enable the account
    1. This is typiaclly done via a tokenized URL sent to the email associated with the account. But you can also utilize SMS, or even snail mail as well
2. Implement a robust internally-available method for blocking specific emails/phone numbers/whatever is being used as a canonical user identifier
    1. For example, a security or operations engineer can use a CLI tool to update a WAF that blocks any signups from `*@fakeemail.ru`
3. Add machine learning models to help detect fake accounts during or after creation
    1. An in-house model can be built for this, or there are commercial services available

### Credential Stuffing/Compromised Accounts
Credential stuffing is when an attacker uses previously-compromised credentials to log into an account on a separate service that uses those same credentials. Typically the end goal of this is the same as creating bot accounts. However, since these are typcally associated with actual humans (or were at one time) they are better at fooling machine learning models and other programmatic defenses.

The best way to help prevent account compromise via credential stuffing is to add multi-factor authentication as an option for accounts (as described below) and urge your user's to take advantage of it. The next best way is to compare a user's credentials when they login to an internal database of known compromised credentials. Lsts of compromised credentals can be found on various sites across the web. However, be aware that storing these lists comes with an additional responsbility to keep them safely guarded from attackers.

## Authentication Guidelines
The US NIST maintains a very detailed white paper on digital authentication - [NIST-800 63](https://pages.nist.gov/800-63-3/sp800-63b.html) - that is consistently updated as authentication requirements in technology change. 

The guidelines for authentication mechanisms in particular (e.g. passwords, OTP devices, etc) - including recommended password configurations and minimum requirements - can be found in [Section 5](https://pages.nist.gov/800-63-3/sp800-63b.html#sec5).

## Recommendations
### Require Multi-Factor Authentication for Sensitive Applications
###### Description

During login, the user should be prompted for at least one second factor of authentication for sensitive services. Sensitive services typically include, but are not limited to: administrative panels, services that handle PII or payment information, and flagship services.
###### Why We Care

By adding multiple layers of verification, we can help ensure that a user’s account is not compromised, especially for sensitive applications and services. This is helpful in the case where a password (the most common initial factor of authentication) is reused by a user across services or has otherwise been leaked, and an attacker is now using that password for Brightcove services, an attack commonly referred to as "password stuffing".
###### How to Fix?

Contact the appropriate identity provider to be used at Brightcove, and require 2FA if needed.
###### Risk Rating

Medium

---
### Limit OAuth2 Scope by the Principle of Least Privilege
###### Description

When an OAuth2 client, a scope is defined for the client. This scope determines which backend APIs are accessible to the client after performing authentication via the auth provider. When setting up a new OAuth2 client, the team requesting the client should identify what APIs/data they need access to, and limit the scope to those specific APIs.
###### Why We Care

Limiting the scope of our APIs is a best practice that can limit the impact of a successful attack.
###### How to Fix?

During the requirements phase, determine what information is needed for the OAuth2 client, and limit the APIs to only what is necessary. If possible, work with the auth provider to build a custom scope that will limit access to wider APIs. 
###### Risk Rating

Medium
###### References

- https://tools.ietf.org/html/rfc6819#section-3.1.1

--- 
### Limited Token Lifetimes
###### Description

When authenticating, a variety of tokens are used to prove the identity of a user. This is typically through a session cookie, or an access token (often a "BC Token") attached to a user’s account. These tokens should have the shortest lifetime that balances the needs of the users with security. Tokens should not be valid for an infinite length of time, and should expire within a reasonable timeframe.
###### Why We Care

This is primarily a defense-in-depth measure, focusing on limiting the impact of an attack if these tokens are compromised. This also decreases the attack surface of a resulting application, reducing the number of valid tokens at any given time.
###### How to Fix?

Check all tokens related to authentication, and review their lifetimes. If you are unsure, contact security for recommendations on their lifetimes. When possible, use the shortest lifetime within a reasonable use.
###### Risk Rating

Medium

---
### Validating OAuth2 Redirect URIs
###### Description

When performing the OAuth2 login flow, a redirect URI is used to securely send the access token to the client web application, after it has authenticated with the auth provider. This is provided by the client during the OAuth2 flow. This URI should be validated to ensure it is a known URI that is used by the OAuth2 client.

VideoCloud Studio supports this, as well as allowing restriction of the redirect URI.
###### Why We Care

This prevents a potential information disclosure attack when there is an open redirect on a valid redirect URI. This results in a user’s OAuth token being potentially leaked, compromising their account.
###### How to Fix?

Validate the entire URI against a whitelist of known valid URIs. Avoid using regular expressions or substring matching.
###### Examples:

_Good Example:_
    
    def validate_redirect_uri(redirect_uri):
        if redirect_uri == "https://newservice.unity3d.com/validate_oauth" :
            return True;
        return False;

_Bad Example_
    
    def validate_redirect_uri(redirect_uri):
        if "unity3d.com" in redirect_uri:
            return True;
        return False;


###### Risk Rating

High

---
### Validating the OAuth2 State Parameter
###### Description

When starting the initial OAuth2 flow, the client has the option of providing a state parameter. This state parameter is then sent back with the authorization to be validated by the client later in the flow.

This state parameter should be generated upon the initial OAuth2 request, be randomly-generated and universally-unique, and stored to validate once the authorization from the auth provider has occurred.
###### Why We Care

This is to ensure that the OAuth2 flow is initiated by the user, and is not vulnerable to a potential CSRF (Cross-Site Request Forgery). By ensuring that the user has initiated the OAuth2 flow, they will be performing actions within their account with known permissions.
###### How to Fix?

Generate a cryptographically-secure random state parameter when the OAuth2 flow is initialized. The client should validate this when it is returned through the redirect URI callback. The parameter should only be valid for the same OAuth2 flow that initiated it.
###### Risk Rating

Low
###### References

- https://tools.ietf.org/html/rfc6819#section-3.6
