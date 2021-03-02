# Preventing Common Web Attacks [Coding Practice]

## Overview

This guideline covers how to prevent some common vulnerability classes that can be eradicated, such as:
- [Open-Redirect](#open-redirect)
- [Clickjacking](#preventing-clickjacking)
- [HTTPS Downgrade (HTTP Security Header)](#all-the-cool-kids-use-http-security-headers)
- [HTTP Request Smuggling](#http-request-smuggling-aka-http-desync-attacks)
- [Cross-Site Scripting (XSS)](#preventing-xss)
- [SQL Injection](#preventing-sql-injection)
- [CSV Injection](#preventing-csv-injection)
- [Cross-Site Request Forgery (CSRF)](#preventing-cross-site-request-forgery)
- [Server-Side Request Forgery (SSRF)](#server-side-request-forgery-ssrf)
- [Arbitrary File Uploads](#arbitrary-file-uploads)
- [Credential leaks](#credential-leaks)

By following the guidelines in this document your application will be more robust against these vulnerability classes and provide a solid foundation for developers to develop secure features for the application.

## Recommendations
### Open-Redirect
###### Description

Open-redirect vulnerabilities occur in web applications that use untrusted input for HTTP 30X location targets.
###### Why We Care

The attacker is able to use the vulnerable endpoint as a proxy for redirecting a victim to a malicious website - all using our applications and infrastructure. Allowing this to happen may lead to complaints from hosting providers, additions of Brightcove domains to security blacklists, and other adverse effects.
###### Example of Issue

A web application designed to redirect users to the home page after logging in accepts the destination URL as a GET variable, and doesn't perform any validation on it, allowing the attacker to redirect a victim to any URL.
###### How to Fix?

There are several methods that can be used to prevent this:
* Tokenize the URL
  * Instead of using the actual destination URL for the input, generate a unique token that identifies the URL indirectly. The token-to-URL relationship can be stored via a database record, and should be referenced when validating the redirect.
  * Ex: https://open-redirect.example.com/?url_token=1234567890abcdefgh
* Implement an intermediate jump page for all redirects; the page would clearly show the target URL, and include a long timeout (5s+) to allow the user enough time to visually validate the URL is the URL they expect and/or trust
  * Ex: Google and Facebook regularly use this for URLs in their ads and posts, respectively.

Regardless of the solution that's chosen, as a part of best practices for public url redirects, we should also validate that the final URL is an external URL; that includes no RFC-1918 IPs as well as no link-local IPs (169.254.0.0/16 range).

One important note regarding black/whitelisting: while **whitelists** can be used to successfully remediate this issue if you know the FQDN(s) that the endpoint should accept. However, **blacklists** are easy to circumvent, and should not be used as a solution for this.
###### Security Level

This attack is typically classified as a low risk vulnerability, mainly because:
1. It requires an external malicious component (such as a link to malware or a phishing website) to really be successful, from the attackers perspective
2. It almost always requires the user to take an action (like clicking a link) to exploit this type of vulnerability, as opposed to attacks that can silently affect Brightcove and its users
3. Almost all web browsers display a full URL for a link when it's hovered over by the user, which usually includes the malicious URL within it, increasing the chances of tipping off the user that something fishy is happening. 
###### References

- https://cwe.mitre.org/data/definitions/601.html
- https://portswigger.net/kb/issues/00500100_open-redirection-reflected
- https://cheatsheetseries.owasp.org/cheatsheets/Unvalidated_Redirects_and_Forwards_Cheat_Sheet.html

 ---
### Preventing Clickjacking
###### Description

Clickjacking, also known as a "UI redress attack", occurs when an attacker uses a transparent or opaque iframe to trick a user into clicking on a button or link on another page when they were intending to click on the the top level page.
###### Why We Care

With this attack, an attacker can trick the user into performing sensitive actions on a page that only the user has access to.
###### Example of Issue

Say that we have a “Pay out” button on an internal advertising payout tool, and the application does not implement any protection against Clickjacking. An attacker that knows the internal URL for the tool can now include a hidden iframe on his site `www.cute-and-funny-puppies.net`:

```html
<iframe src=”https://ads-awesome-payout-tool.unity3d.com” style=”opacity:100”></iframe>
```

When an admin that gets bored of approving payouts visits the attacker’s site to view some funny dog pictures, he simultaneously gets tricked into approving a payout for an attacker by clicking an element on the site that really clicks the payout button in the hidden iframe.
###### How to Fix?

Set the following HTTP response headers in your application:

    X-Frame-Options: DENY

    Content-Security-Policy: frame-ancestors 'self';
###### Security Level

This attack is usually a pretty low risk because most application don’t have that many single click sensitive actions and also because the attack has a social engineering component that requires the user to visit a site that the attacker controls.

However, in the some cases the risk can be high if there is a button such as “Make user admin” in the application.
###### References

- https://www.owasp.org/index.php/Clickjacking_Defense_Cheat_Sheet

 ---
### All the cool kids use HTTP Security Headers
###### Description

Browser vendors are trying to help solve security issues that are common in web applications. However, they need to take backward compatibility into account because the Internet is full of old websites. Therefore, some of the new security features in browsers are opt-in, using HTTP Security headers which can be set by an application to enable these features. You should do this for all your applications to take full advantage of modern browser security.
###### Why We Care

Setting HTTP Security headers mostly protects your users by turning on security features in their browser when visiting Brightcove applications. We want to protect our customers so that they can safely browse our applications, even in “hostile environments” such as airports or cafés, and this is a very cheap defense-in-depth security measure.  
###### Example of Issue

For example: Almost all web applications redirect from HTTP->HTTPS so that users don’t have to type https:// at the start of the url when they visit your site. However, if the user already had an active session on the site, the browser might send some sensitive information on that very first HTTP request before it gets redirected. This information can be sniffed by an attacker unless we take some precautions and tell it to always browse Brightcove applications over an encrypted connection.
###### How to Fix?

We have a whole page dedicated to setting HTTP Security Headers which can be found here:
[HTTP Header Security](./Coding%20Practive/HTTP-Header-Security.md)

Read through the recommendations, add your headers in the early phases of the project, and make sure to choose a modern front-end technology that is friendly to Content Security Policy.
###### Security Level

Adding security headers is basic security hygiene and you can see that large companies such as Facebook and Google are using these. Web-based organizations such as Mozilla also require all their applications to implement them. 
###### References

Mozilla has a great web security guideline reference here which includes a lot about security headers:

- https://infosec.mozilla.org/guidelines/web_security

---
### HTTP Request Smuggling (a.k.a. HTTP Desync Attacks)
###### Description

HTTP Request Smuggling is a technique used to "smuggle" malicious HTTP requests along with valid, authorized HTTP requests. This is facilitated by abusing the `Content-Length` and `Transfer-Encoding` HTTP headers, and taking advantage of discrepancies between technologies in handling non-compliant HTTP requests.
###### Why We Care

Allowing an unauthorized request into our network backend can create a very large impact depending on how much network access there is for the affected endpoint. Ensuring each HTTP request is uniformly processed will help prevent unauthorized access to Brightcove's backend services.
###### Example of Issue

Suppose an attacker sent an HTTP request like the one below to a load-balancer sitting upstream from a public web application: 
```
POST login.html HTTP/1.1
Host: www.example.com
Content-Length: 6
Transfer-Encoding: chunked

53
GET admin_console.html HTTP/1.1
Host: www.example.com
0
```

In the above example, the load-balancer software uses the `Transfer-Encoding` header, while the backend application uses the `Content-Length` header. The load-balancer transfers the next 53 bytes of data as a single request onto the backend. The backend application will see the `Content-Length` is set to `6`, end the first request after the `53`, and then treat the next set of data it receives as a separate request.

Other techniques include using `Transfer-Encoding` then the `Content-Length` headers, as well as using multiple `Transfer-Encoding` headers, the second being obfuscated in order to take advantage of discrepancies in how technologies tolerate invalid header characters.  

For a real world example, see a [recently disclosed vulnerability in Slack](https://hackerone.com/reports/737140) that allowed account takeovers using this technique. 
###### How to Fix?

* Prevent the use of backend connection reuse so that each request sent to the backend application by the front-end is in a separate layer 4 connection
  * This can sometimes increase overhead and performance of your application, so isn't recommended for applications with high-traffic use cases
* Use HTTP/2 for backend connections as it is not affected by this type of vulnerability
* Use the same web server or library across all servers that process HTTP requests for the given application
###### Security Level

HTTP Request Smuggling attacks typically introduce the same amount of risk as CSRF or SSRF vulnerabilities, so usually Medium to High.
###### References

- https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Transfer-Encoding
- https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Length
- https://cwe.mitre.org/data/definitions/444.html
- https://portswigger.net/web-security/request-smuggling
- http://projects.webappsec.org/w/page/13246928/HTTP%20Request%20Smuggling

---
### Preventing XSS
###### Description

XSS or Cross-Site Scripting is perhaps the most common web application vulnerability out there. The vulnerability allows an attacker to modify the front-end behaviour of your application by injecting Javascript or HTML into the application, altering the intended behaviour for malicious purposes.
###### Why We Care

XSS vulnerabilities can be serious as they can be used to steal sensitive information, such as credentials, session tokens or credit card data. It can be used to bypass firewalls and get access to internal networks - e.g. the Brightcove office network - because it allows an attacker’s code to run in the the victim’s browser which may very well be running on a computer in the office or that’s connected via VPN.

It is also commonly used to attack browsers directly by injecting browser exploit code or, as has been seen lately, to steal computing power to mine cryptocurrency by injecting currency mining code into an innocent website.
###### Example of Issue

There are two main types of XSS vulnerabilities: reflected and persistent. We will provide an example of both.

**Reflected**

Reflected XSS vulnerabilities are essentially dynamically-induced. They occur due to an injection of code via a dynamically-set variable.
E.g.:

```
https://www.application.brightcove.com/newuser?name=<script>alert(1)</script>
```

If this parameter ends up being treated as HTML by the application downstream, an alert box would pop up to prove we can inject Javascript.

**Persistent**

An example of a persistent XSS attack would occur when arbitrary data coming from the user is stored in a _persistent_ database and later included in a webpage displaying in a victim's browser.

E.g.: a custom CMS application was designed for a blog, but any HTML entered as part of a blog post is unsanitized and gets executed by the victim's browser.
###### How to Fix?

XSS prevention is a huge topic. But the main takeaways are as follows:

- Choose a front-end framework that support output encoding by default
- Sanitize all dynamic/user-supplied data - whether from a dynamic GET variable or a DBMS - before output to a web page
- Implement a Content Security Policy that disallows inline Javascript

The first and best way to fix this is to use a front-end framework that output encodes / escapes data by default making it hard for any developer to make the mistake of not handling malicious input.

The following front-end frameworks are good choices that will make it harder to introduce XSS vulnerabilities as they were designed with this in mind:

- https://angular.io/guide/security#xss
- https://reactjs.org/docs/introducing-jsx.html#jsx-prevents-injection-attacks

There are more frameworks that have good default XSS prevention, but these two that are in use already at Brightcove, and are supported and used by larger companies such as Google and Facebook.

Secondly, make sure that your application sanitizes any dynamic data it utilizes before outputting it to a web page, and optionally, when it's received by the user. This needs to be done on the server for it to be effective. Data can come from many sources including (but not limited to) direct input by a user or data coming from another application.

For example, if you expect a phone number to be entered, make sure that your program validates that only expected data such as numbers, dashes and maybe a + for country code is accepted by the application. Additionally, ensure that phone number is encoded and sanitized before being included as a field in a web page. 

If you aren't using a framework for sanitization - e.g. just vanilla JS or Node.js, etc - then it's recommended to first try using a commonly-used sanitization library/plugin, if one is available. There are many open-source, or otherwise free, libraries that can help with this: [Joi](https://github.com/sideway/joi) or [DOMPurify](https://github.com/cure53/DOMPurify) for NodeJS, [Bleach by Mozilla](https://github.com/mozilla/bleach) for Python, Ruby on Rails have Active Record Validations, and so on.

If not, and you need to perform your own sanitization, ensure that you are taking the following into account:

- Escape all untrusted input by default
  - For instances where you actually *do* require HTML to be included, carve out exceptions for each instance (e.g. output_html(raw_input, allow_untrusted=True) with allow_untrusted set to False by default)
- At a minimum, he main characters that you want to escape are:
  - `& --> &amp;`
  - `< --> &lt;`
  - `> --> &gt;`
  - `" --> &quot;`
  - `' --> &#x27;`
  - Note that if you are using untrusted data in other areas listed below **other than HTML elements**, then there are even more characters that must be escaped
    - Contact Security Engineering for assistance, or reference the OWASP XSS Prevention Cheat Sheet listed in the references below
- Make sure that you are sanitizing untrusted data when it's being placed into:
  - HTML elements (e.g. `<div>`, `<p>`)
  - HTML attributes (e.g. `div class="<UNTRUSTED_DATA>">`)
  - JavaScript data values (e.g. quotes string, quoted variable values)
  - CSS values (yes, you can actually execute JavaScript within certain CSS tags)
  - URL parameters for links
- Make sure you aren't placing untrusted input into JavaScript functions that still execute untrusted data _even if it's been encoded already_ (yes, these exist too)
  - One example of this is the `setInterval()` function
  - If you plan on using untrusted data within built-in JS functions, contact Security Engineering for assistance, or reference the OWASP XSS Prevention Cheat Sheet listed in the references below
  
Given the above requirements, it's **_highly recommended_** to use one of the first two options instead.

Finally, we can implement a safeguard mechanism against XSS issues that all modern browsers support. Content Security Policy is effective in limiting the impact of XSS vulnerabilities should they occur in your application, even after you encode output data and sanitize input.

By carefully designing a CSP we can tell the browser which scripts are allowed to run and which aren’t. We can limit where the application is allowed to load Javascript resources from, and we can disallow any inline Javascript. Malicious Javascript injected by exploiting an XSS vulnerability usually ends up being inline in the HTML document and not in an external .js file, so if we instruct the browser to not allow inline Javascript we can prevent a lot of the common XSS vulnerabilities.

An example policy that only allows script files from the domain the application lives on would look like this:

    Content-Security-Policy: "script-src 'self'"
###### Security Level

XSS vulnerabilities are very common, and constitute a Medium to High risk depending on context and application.
###### References

- https://www.owasp.org/index.php/XSS_(Cross_Site_Scripting)_Prevention_Cheat_Sheet
- https://csp.withgoogle.com/

---
### Preventing SQL Injection
###### Description

SQL injection is an old vulnerability class which allows an attacker to alter an application’s SQL queries to perform other actions than those intended by the developer. The problem occurs due to a lack of separation between data and code, allowing an attacker to input data to alter the logic of a query.

SQL injection vulnerabilities are still found even in modern web applications, leading to loss of data, authentication bypass, and server compromise.

Even applications that do not use traditional relational databases have been found to be vulnerable to SQL injection. If user input is used to build the database query, an attacker can potentially alter the query being made and bypass authentication, get unintended access to data, or perform remote code execution.
###### Why We Care

Protecting data from attackers is paramount for any data-driven company, and protecting our web applications from SQL injection is a very important part of preventing data from being stolen. Many of the breaches that have been seen over the years have been caused by SQL injection in web applications, leading either to full server compromise or access to all data in the database.
###### Example of Issue

An example of this would be an API that allows users to fetch a list of video playlists. The API uses MySQL or another DBMS as its backend, and of course has queries it runs to fetch the video playlist data. However, those queries are all generated from dynamic, user-sourced data without any sort of parameterization or sanatization.
###### How to Fix?
###### _Relational Databases (MySQL, PostgreSQL, ...)_

All SQL queries should be **parameterized queries**. The exact syntax varies based on technology stack used, but usually looks more or less like this:

```sql
SELECT * FROM users WHERE username = ‘?’, username
```

Limit the privileges of your database user. You should never run application SQL queries as the database admin or a root / admin user.

Use an Object Relational Mapper (ORM) that helps you build queries securely. This also helps ease the pain of writing raw SQL queries.
 
###### _noSQL Databases (MongoDB, CouchDB, ...)_

Almost the same principles apply for protecting applications against NoSQL injection: 

Sanitize user input that is used in queries. Check the type of the input (string, array, dict etc), and the expected format against a regex or a whitelist.

Use a library to help you out such as Mongoose (similar to an ORM).

Limit the privileges of your database user to the minimum access needed for your application.
###### Security Level

The severity of a SQL injection vulnerability ranges from high to critical depending on the application and context. 

For example, a SQL injection in your auth provider would be a critical vulnerability because it could lead to compromise of all user data and potentially bypass access controls.
###### References

- https://www.owasp.org/index.php/SQL_Injection_Prevention_Cheat_Sheet
- https://blog.websecurify.com/2014/08/hacking-nodejs-and-mongodb.html (Example noSQL attacks)

--- 
### Preventing CSV Injection
###### Description

A CSV command injection vulnerability allows an attacker to run arbitrary commands and macros within a user's CSV parsing software, e.g. Microsoft Excel.

The vulnerability works by injecting spreadsheet software functions into data points. The data points are then included as part of a generated CSV. The victim downloads the malicious CSV and opens it in theri environment, triggering the payload.
###### Why We Care

CSV injections put our users at risk by allowing arbitrary (though semi-sandboxed) code execution on whatever node opens the CSV file. Users should be able to trust that any files that are deliver to them are as safe as possible.
###### Example of Issue

The main example that's used for CSV injection is abuse of the `cmd` function.

Say we have a web-app that allows researchers to deliver anonymous questionaires publicly. Users are able to answer the questions, and then the researchers are able to download the results in multiple formats, including CSV.

If one of the takers of the questionaire enters a malformed answer of `=cmd|' /C notepad'!'A1'`, that will then become a payload that gets run once any user opens a report CSV containing that answer. Replace `notepad` with a command that exfiltrates a user's files to a third-party, or triggers malware to run, and it can lead to a huge risk for the user. 
###### How to Fix?

CSV injection can be fixed in a few ways, both of which involve any reserve characters that can be used to trigger spreadsheet software functions.

Those characters are: `=`, `+`, `-`,`@`

1. Remove the reserved characters completely from any output destined to be used within a CSV file
2. If these reserve characters must be preserved for business cases, ensure that all of them are escaped by proceeding each character with a backslash character (`\`)
###### Security Level

CSV injections aren't the most common vulnerabilities due to their limited scope. However, if they are triggered, they typically lead LOW to HIGH severities, depending on the difficulty of exploitation and number of affected users.
###### References

- https://owasp.org/www-community/attacks/CSV_Injection#:~:text=CSV%20Injection%2C%20also%20known%20as,the%20software%20as%20a%20formula.
- https://medium.com/@ismailtasdelen/csv-injection-payload-list-e8e1deca6da5
- https://www.whiteoaksecurity.com/2020-4-23-csv-injection-whats-the-risk/

---
### Preventing Cross-Site Request Forgery
###### Description

Cross-Site Request Forgery attacks allow an attacker to perform actions in an application in the context of an authenticated user. By abusing the way cookies work, if the application’s authentication is cookie-based (e.g. a session cookie is stored and sent with every request), an attacker can setup a website that makes requests to the application for instance via an iframe, an image, or by automatically posting some form data and the browser will send along the cookies of the user.

This attack is done across origins from website www.malicious.com to www.application.unity.com so the attacker won’t be able to read the response of the cross-origin request due to the Same-Origin Policy. However, this isn’t necessary for the attack to succeed if the application only requires a session cookie and don’t validate that the request initiated from the expected domain.

The attack can happen in the background on a malicious website while the user browses funny cat pictures and suspects nothing. Exploitable actions can be anything the application allows, such as changing a password, changing shipping address, deleting a user, or anything else that might be used to an attacker’s benefit.
###### Why We Care

Cross-Site Request Forgery is a well-known attack vector today, but many prominent sites such as Gmail suffered from this 10 years ago. The attack is usually targeted towards a single user account, but if that user account is an administrator of the application, the business consequences can be very serious.
###### Example of Issue

A lot of comsumer-grade routers are vulnerable to this. While they have authentication, a lot of the endpoints (e.g. changing the admin password) do not check the authentication token properly.

Another example would be Studio allowing an attacker to add arbitrary users to any account via a user creation API.
###### How to Fix?

There are several strategies to fixing this problem, and it depends a bit on the application’s architecture. Here are the main strategies that are most commonly used and accepted as a standard way of solving this problem:
###### _Authentication via HTTP header_

Some auth APIs use the `“Authentication: Bearer <token goes here>”`  header for authentication. Requests that require an authentication header to succeed are not vulnerable to CSRF attacks.
###### _CSRF Tokens_

This is the most common and accepted protection mechanism. It works by requiring every form submission and request that performs an action to include a random CSRF token in the submitted data. Since this token is set and stored by the server, and tied to a user’s session, an attacker won’t be able to access it or guess it. This prevents CSRF attacks from succeeding as the server should reject any requests that don’t have a valid token.

The first thing to make sure is that your application uses the HTTP GET and POST verbs appropriately. See http://guides.rubyonrails.org/security.html#csrf-countermeasures

Next you should look at your web application framework to see if they have support for CSRF tokens. Most frameworks, like Ruby on Rails as mentioned above, will either have built-in support for easily adding CSRF tokens, or libraries that help implement it. 

Here are two popular options for NodeJS and Golang:

- https://github.com/expressjs/csurf
- https://github.com/utrack/gin-csrf

###### _Samesite Cookies_

This is the third mechanism for protecting against CSRF. It is a new browser feature that allows marking cookies with a flag controlling whether requests initiated from a third-party site will include the cookies.

There are two modes: strict and lax.

Strict mode prevents the browser from sending the cookies cross-origin for any request. This can have negative consequences in situations where a logged-in user clicks a link leading back to an authenticated page in the application because the cookies won’t be sent along with the request, and the user needs to reauthenticate.

Lax mode prevents this situation and allows GET requests to send along the cookies. Requests using any other HTTP verbs won’t send along the cookies. If the application uses HTTP verbs appropriately this would normally be enough to prevent CSRF attacks because the application shouldn’t use a GET request to perform any action in the application.

- Set Lax mode: `Set-Cookie: CookieName=CookieValue; SameSite=Lax;`
- Set Strict mode: `Set-Cookie: CookieName=CookieValue; SameSite=Strict;`

Since this is a very new browser security feature, not all browsers support it yet and not all frameworks have APIs for setting cookies with this flag. Consider implementing it though, as it is simple, unobtrusive, and effective way of preventing CSRF attacks and browser support is likely to increase in the future.
###### Security Level

The example above shows how serious CSRF vulnerabilities can be. The impact of a CSRF vulnerability can range from low to critical depending on the application.

CSRF attacks are usually targeted and need to be tailored to a specific application, but there have also been CSRF vulnerabilities in commonly used libraries and frameworks that can be used by an attacker to target a multitude of sites at the same time.
###### References

- https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)_Prevention_Cheat_Sheet
- https://www.netsparker.com/blog/web-security/same-site-cookie-attribute-prevent-cross-site-request-forgery/

 ---
### Server-Side Request Forgery (SSRF)
###### Description

Server-Side Request Forgeries (SSRF), similar to CSRFs described above, abuse the trust given to remote data being sent across a network. With CSRFs, this abuse occurs in the trust granted to a client-side request; the client sends a request that the server then (mistakenly) assumes should be executed. With SSRFs, this same abuse occurs, but with requests coming from other servers within Brightcove's network.
###### Why We Care

Often, organizations will grant more trust to endpoints _within_ their network than external hosts, such as internet endpoints. This means that when an SSRF vulnerability is found, it's often as simple as the attacker sending regular HTTP requests to gain access to internal-only data, such as PII.

This has also become more of an issue with the usage of cloud computing. A lot of cloud computing companies grant trust to an individual computing instance that allows access to cloud APIs. An example of this would be AWS's metadata endpoint that's reachable from all EC2 instances: `169.254.169.254`

Since Brightcove integrates with customers' media and APIs, we have a lot of our own APIs that support making arbitrary network requests. The intention is to limit it to only legitimate customer content, but we've had SSRFs come up with these endpoints in the past for these services.
###### Example of Issue

An example of this type of issue would be an API that fetches videos from an arbitrary URL. The URL is supplied via a GET variable:
```
GET /video?url=http://my.video.com/video.mp4
```

If this variable data isn't sanitized properly, and attacker could supply any URL, including one that's internally-accessible:
```
GET /video?url=http://admin.internal.company.com/secret-data
```
###### How to Fix?

SSRFs can be tricky to fix since a lot of HTTP and network libraries allow the user to supply the IP/FQDN/URL in many forms. Specific instructions on fixing this will vary between languages, but in general:
- Don't allow URLs with RFC-1918 (private) IP addresses specified for the host
- Normalize URL components before evaluation (e.g. ensure the host component isn't a decimal-encoded IP address)
- Whitelist URLs, if possible
  - Don't bother trying to blacklist; there's too many protocols and URL schemes to account for for this to be effective
- If it's not possible to whitelist certain URLs, ensure that all HTTP redirects (HTTP 30x) are followed until a non-30x response is returned
  - This is to protect against URL-shortener services that don't properly blacklist RFC-1918 IPs from the long URLs
  - Ensure that you also limit the number of 30x redirects in order to prevent a DoS situation (100 is typically a good number)
- Limit the HTTP verbs/methods that can be used with your API
  - Ex: if your API just serves up static data read by other services, allow GET requests and generate an error for all others
- Use authentication for internal services whenever possible
  - This is especially important with databases, e.g. Redis, Kibana, etc
  - This practice falls in line with Zero-Trust Architecture, the primary security architecture framework employed by Brightcove Security Engineering
###### Security Level

Depending on the case, SSRFs essentially allow an open proxy for outside attackers to run arbitrary network requests on an organization's internal network. Since this is typically where sensitive data is stored, and internal networks commonly have less safeguards than external network zones, this often presents a Medium to High risk.
###### References

- https://portswigger.net/web-security/ssrf
- https://blog.detectify.com/2019/01/10/what-is-server-side-request-forgery-ssrf/
- https://owasp.org/www-community/attacks/Server_Side_Request_Forgery

 ---
### Arbitrary File Uploads
###### Description

Many web applications support features that require the user to upload a file from their client to a backend application/server. Acceptance of these arbitrary files presents several security concerns, especially regarding malware, XSS, and content-sniffing.
###### Why We Care

File uploads present a unique opportunity for attackers as they are able to write arbitrary files to a backend system. This often allows for greater access to internal networks, as well as increased vectors for subverting an application's logic.
###### Example of Issue

A public file upload service accepts files for upload and allows them to share them via a static link. Because the site does not properly handle these uploads, attackers are able to upload malware and JS files used alongside other XSS vulnerabilities.
###### How to Fix?

The way to securely accept files for upload essentially boils down to verifying the data being uploaded is exactly what you expect to be uploaded, e.g. don't accept or serve up Javascript files if your service is used for image sharing only.

Recommendations include:
- Implement a whitelist of file-types that your application is expected to accept
  - Use a MIME-type analyzer library; don't rely on file extensions as they can be forged
  - Don't bother with a blacklist as there's always some type of rare file-type that will get missing
  - Ensure that you don't whitelist a file-type that can be executed by the web server of accepting server application
    - E.g. don't whitelist `.php` files if your Apache instance if it has a mod_php installed
    - For a full list of common MIME types, see [this Mozilla Developer Network page](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Common_types)
- Serve up uploaded files from a domain singularly used for these uploads
  - e.g. if your webapp runs on app.domain.com, serve up files from files.domain.net
- Generate and overwrite filenames
  - Instead of keeping the client-supplied filename during persistent saving/service up of the file, generate a new filename based on unique, static details
  - `<sha256_hash>-<epoch_timestamp_of_upload>.<file_ext>` is a good option
  - This should be performed _before_ the file is saved to persistent storage
- Restrict the size of the file to a sane file size limit
  - The limit you choose depends on your application's use cases
- When serving up files to users, ensure that the `Content-Type` header is set to the appropriate value
  - e.g. serving up a `.txt` file should result in this header being set to `text/plain`
- Ensure this part of your application particularly includes well-detailed logging

###### Security Level

Unrestricted file uploads to webapps can create security issues ranging in severity from Low to High depending on what vector they exploit and how it affects the target application.
###### References

- https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload

 ---
### Credential Leaks
###### Description

Application development requires integration with a growing number of services. This requires developers to handle a number of secrets: Database credentials, api tokens, oauth secrets etc. These should be high-entropy, unique per service and per environment, leading to large number of secrets to manage. These often end up being stored in configuration files which are then accidentally checked into your favorite version control system on a publicly accessible cloud-hosted repositories, and in turn, into the hands of attackers.
###### Why We Care

Secrets are secret for a reason and often provide access to a lot of data, computing resources or privileged accounts. It is therefore crucial that we handle secrets well, and make the probability of accidental leaks as small as possible.
###### Example of Issue

Credential leaks to Github are a common, and troublesome issue within the industry. This can give malicious actors access to internal systems, data, APIs and more.

There are also other examples, such as accidentally hosting script files that contain credentials on a public web server.
###### How to Fix?

When developing an application, it is best practice to load secrets from environment variables. This has a few benefits: 

First, it stores the credentials in the running process’ address space. This means that other user accounts on the machine that aren’t privileged, won’t be able to access the credentials.

Secondly, it separates the secrets from your application's normal configuration files, which are often checked into version control systems. When developing the application, use a .env file for storing the secrets and load them into your environment before running the application locally.

For more internal information on how to securely store secrets, visit [our internal guide to secret data storage](https://confluence.brightcove.com/display/IS/Storing+Application+Secrets+Securely)

The Security Engineering team does utilize a tool for monitoring of secrets in source code repositories. But this mechanism is passive, and only supposed to be the last line of defense and to avoid credential leaks. It is up to the developer to be diligent when handling these sensitive pieces of data.
###### Security Level

Leaking credentials can be critical as they often grant a high level of access to Brightcove systems (especially third-party SaaS services) and can lead to compromise of data and systems.

