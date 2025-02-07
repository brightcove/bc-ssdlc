# Server-Side Request Forgery (SSRF) Prevention

## Overview

> Server-side request forgery (also known as SSRF) is a web security vulnerability that allows an attacker to induce the server-side application to make HTTP requests to an arbitrary domain of the attacker's choosing.
>
> In a typical SSRF attack, the attacker might cause the server to make a connection to internal-only services within the organization's infrastructure. In other cases, they may be able to force the server to connect to arbitrary external systems, potentially leaking sensitive data such as authorization credentials.
>
> -- PortSwigger, [Server-side request forget (SSRF)](https://portswigger.net/web-security/ssrf)

Server-Side Request Forgeries (SSRF) - similar to CSRF vulnerabilities - abuse the trust given to remote data being sent across a network. With CSRFs, this abuse occurs in the trust granted to a client-side request; the client sends a request that the server then (mistakenly) assumes should be executed. With SSRFs, this same abuse occurs, but with requests coming from other servers within Brightcove's network.

Often, organizations will grant more trust to endpoints _within_ their network than external hosts, such as internet endpoints. This means that when an SSRF vulnerability is found, it's often as simple as the attacker sending regular HTTP requests to gain access to internal-only data, such as PII.

This has also become more of an issue with the usage of cloud computing. A lot of cloud computing companies grant trust to an individual computing instance that allows access to cloud APIs. An example of this would be AWS's metadata endpoint that's reachable from all EC2 instances: `169.254.169.254`

Since Brightcove integrates with customers' media and APIs, we have a lot of our own APIs that support making arbitrary network requests. The intention is to limit it to only legitimate customer content, but we've had SSRFs come up with these endpoints in the past for these services.

### Example of Issue

An example of this type of issue would be an API that fetches videos from an arbitrary URL. The URL is supplied via a GET variable:

```HTTP
GET /video?url=http://my.video.com/video.mp4
```

If this variable data isn't sanitized properly, and attacker could supply any URL, including one that's internally-accessible:

```HTTP
GET /video?url=http://admin.internal.company.com/secret-data
```

## Best Practices

SSRFs can be tricky to fix since a lot of HTTP and network libraries allow the user to supply the IP/FQDN/URL in many forms. In general, the best defense against SSRFs is to validate user submitted URLs with an allow-list, if your use case will allow for them.

Otherwise, if arbitrary URLs are expected:

- Validate the submitted URL string with a regular expression or URL parsing library to ensure it fits the expected URL format.
  - If possible, use a popular, well-known (and optimally, audited) third-party library that performs validation for you
    - Some examples are:
      - NPM: [ssrf-req-filter](https://www.npmjs.com/package/ssrf-req-filter) , [request-filtering-agent](https://www.npmjs.com/package/request-filtering-agent) , [got-ssrf](https://www.npmjs.com/package/got-ssrf)
      - Golang: [ssrf](https://pkg.go.dev/code.dny.dev/ssrf)
- Don't bother trying to blacklist URLs; there's too many protocols, URL schemes, and format exceptions to account for for this to be effective
- Use host, cluster, or VPC egress firewalls to block access to internal resources.
- Normalize URL components before evaluation (e.g. ensure the host component isn't a decimal-encoded IP address)
- Limit the HTTP verbs/methods that can be used with your API
  - Ex: if your API just serves up static data read by other services, allow GET requests and generate an error for all others
- Validate the URL protocol against an allow-list (e.g., only expected web URLs? Only allow http and https)
- Validate user submitted authentication tokens using a regular expression (e.g., [a-zA-Z0-9]{20})
- Block access to internal resources
  - See [Block Access to Internal Resources](#block-access-to-internal-resources) for additional details
- Use authentication for internal services whenever possible
  - This is especially important with databases, e.g. Redis, Kibana, etc
  - This practice falls in line with Zero-Trust Architecture, the primary security architecture framework employed by Brightcove Security Engineering
- Return only the information needed by the frontend; don't return the raw HTTP response from the destination web server
- Ensure that the HTTP client is not passing internal credentials to external resources
- Prevent the HTTP client from following redirections (HTTP 301, 302, etc.)
  - This is to protect against an attacker utilizing a web server (or abusing a link-shortener service) to perform an HTTP redirect to a private IP (e.g. `Location: http://169.254.169.254/metadata/v1/user-data`)

## Block Access to Internal Resources

When you need to make server-side requests based on a user-submitted URL - such as with webhooks - prevention centers around rejecting requests aimed at internal resources. There are a couple of checks that need to be performed on a submitted URL before making the server-side request.

### Payloads For Testing

For a list of payloads to code for, see [this document in PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Request%20Forgery/README.md).

### Step 1. Check for submission of non-public IP address

Determine if the submitted URL is a non-public IP address using a block-list. If the submission uses a domain name, resolve the domain's IP addresses (A and AAAA records) and then perform this check on each of them. URLs containing local, APIPA, or Private IP addresses (note that these can be provided in multiple forms; see [Payloads for Testing](#payloads-for-testing) above) should be rejected.

#### Example CIDR Ranges to Block

The CIDR ranges listed below are a sample - but _not_ an exhaustive list - of IP addresses to validate against.

Local Address Ranges:

- 127.0.0.0 – 127.255.255.255

Private IP Address Ranges:

- 10.0.0.0 - 10.255.255.255
- 172.16.0.0 - 172.31.255.255
- 192.168.0.0 - 192.168.255.255

APIPA Address Range:

- 169.254.0.1 - 169.254.255.254

#### Code Examples

<details>
  <summary>Golang Example</summary>
  
```go
func validateIPs(ips []net.IP) (bool, error) {
    if len(ips) == 0 {
        return false, errors.New("IP not found")
    }

    for _, ip := range ips {
        if ip.To16() == nil && ip.To4() == nil {
        log.Errorf("IP: %v is not valid", ip)
        return false, errors.New("IP is not valid")
        }
        // IsPrivate reports whether ip is a private address, according to
        // RFC 1918 (IPv4 addresses) and RFC 4193 (IPv6 addresses).
        if ip.IsPrivate() {
        log.Errorf("IP address: %v is a private address", ip)
        return false, errors.New("IP address is a private address")
        }
        // checks Local Address Range of 127.0.0.0 - 127.255.255.255
        if ip.IsLoopback() {
        log.Errorf("IP address: %v is a local address", ip)
        return false, errors.New("IP address is a local address")
        }
        // checks APIPA Address Range of 169.254.0.0 - 169.254.255.255
        if ip.IsLinkLocalUnicast() {
        log.Errorf("IP address: %v is a link-local unicast address", ip)
        return false, errors.New("IP address is a link-local unicast address")
        }
    }

    return true, nil
    }
```

</details>

<details>
  <summary>Gitlab's Ruby Example</summary>
  
```ruby
# Source: https://gitlab.com/gitlab-org/gitlab-foss/-/blob/eabd80f72f4f7d8e19b26526aa1f44c43d78e8b3/lib/gitlab/url_blocker.rb#L214-L240
def validate_localhost(addrs_info)
    local_ips = ["::", "0.0.0.0"]
    local_ips.concat(Socket.ip_address_list.map(&:ip_address))

    return if (local_ips & addrs_info.map(&:ip_address)).empty?

    raise BlockedUrlError, "Requests to localhost are not allowed"
end

def validate_loopback(addrs_info)
    return unless addrs_info.any? { |addr| addr.ipv4_loopback? || addr.ipv6_loopback? }

    raise BlockedUrlError, "Requests to loopback addresses are not allowed"
end

def validate_local_network(addrs_info)
    return unless addrs_info.any? { |addr| addr.ipv4_private? || addr.ipv6_sitelocal? || addr.ipv6_unique_local? }

    raise BlockedUrlError, "Requests to the local network are not allowed"
end

def validate_link_local(addrs_info)
    netmask = IPAddr.new('169.254.0.0/16')
    return unless addrs_info.any? { |addr| addr.ipv6_linklocal? || netmask.include?(addr.ip_address) }

    raise BlockedUrlError, "Requests to the link local network are not allowed"
end
```

</details>

### Step 2. Prevent secondary name resolution

If the URL submitted uses a domain name, you need to protect against _*DNS rebinding attacks*_ by preserving the DNS resolution done in Step 1.

It is common for an HTTP client library to perform its own DNS resolution when passed a URL. If an attacker changes the DNS record to a local, APIPA, or Private IP address between the resolution in Step 1 and the DNS resolution performed by the HTTP client, this validation performed in Step 1 can be bypassed. This is called a DNS Rebinding Attack.

The video below explains how SSRF vulnerabilities can be combined with DNS Rebinding to bypass IP checks.

[![GitLab DNS Rebinding SSRF](https://img.youtube.com/vi/R5WB8h7hkrU/0.jpg)](https://www.youtube.com/watch?v=R5WB8h7hkrU)

There are a couple of ways prevent this.

#### Option 1

The first is the method used by Gitlab in the video above (patch commit found [here](https://gitlab.com/gitlab-org/gitlab-foss/-/blob/eabd80f72f4f7d8e19b26526aa1f44c43d78e8b3/lib/gitlab/url_blocker.rb#L22)). The solution is to replace the URL's domain with a validated IP address from Step 1. Pass this URL to your HTTP client to prevent a secondary DNS resolution.

For example, change the user submitted URL:

`https://www.mywebsite.com/validate`

to the following URL:

`https://55.26.115.78/validate`

Simplified code example of how Gitlab validates a submitted URI and transforms it. The `protected_uri_with_hostname` returned is used by an HTTP client.

_*Important note:*_ This is effective, but you can run into issues if the destination web server is using virtual hosts. Without a domain to parse, the request will fail.

<details>
  <summary>Gitlab's Ruby Example</summary>
  
```ruby
# Source: https://gitlab.com/gitlab-org/gitlab-foss/-/blob/eabd80f72f4f7d8e19b26526aa1f44c43d78e8b3/lib/gitlab/url_blocker.rb#L22
require 'ipaddress'

# Expects Addressable::URI
def validate_uri(uri)
    address_info = get_address_info(uri)
    ip_address = address_info.first&.ip_address
    # Replace domain with resolved IP address
    protected_uri_with_hostname = enforce_uri_hostname(ip_address, uri)
    protected_uri_with_hostname
end

def enforce_uri_hostname(ip_address, uri)
    return [uri, nil] unless ip_address && ip_address != uri.hostname
    new_uri = uri.dup
    new_uri.hostname = ip_address
    [new_uri, uri.hostname]
end

def get_address_info(uri)
    Addrinfo.getaddrinfo(uri.hostname, get_port(uri), nil, :STREAM).map do |addr|
        addr.ipv6_v4mapped? ? addr.ipv6_to_ipv4 : addr
    end
rescue SocketError
    raise BlockedUrlError, "Host cannot be resolved or invalid"
rescue ArgumentError => error
    raise unless error.message.include?('hostname too long')
    raise BlockedUrlError, "Host is too long (maximum is 1024 characters)"
end
```

</details>

#### Option 2

The second way to remediate DNS Rebinding Attacks is to override the destination IP address within the transport configuration of the HTTP library in use. Changing it to the validated IP address from Step 1 will ensure the request goes to the validated destination.

<details>
  <summary>Golang Example</summary>
  
```go
func sendGetRequest(webIP, host, scheme, path) (string, error) {
    dialer := &net.Dialer{
        Timeout:   10 * time.Second,
        KeepAlive: 10 * time.Second,
    }

    // provide a custom Transport.DialContext function
    // to force requests to use a specific destination IP address
    http.DefaultTransport.(*http.Transport).DialContext = func(ctx context.Context, network, addr string) (net.Conn, error) {
        port := ":80"
        if scheme == "https" {
            port = ":443"
        }

        addr = webIP + port
        return dialer.DialContext(ctx, network, addr)
    }

    webURL := scheme + "://" + host + path
    resp, err := http.Get(webURL)
    if err != nil {
        return "", fmt.Errorf("error when doing a GET request to publisher webURL [%s]: %v", webURL, err)
    }
}
```

</details>

<details>
  <summary>C# .NET Example</summary>
  
```c#
HttpWebRequest request = (HttpWebRequest)WebRequest.Create("https://1.2.3.4");
request.Host = "www.example.com";
var response = request.GetResponse();
```

</details>

## Full Code Examples

<details>
  <summary>Golang Example</summary>
  
```go
func executeWebhook(webUrl string) {
    u, err := url.Parse(webUrl)
    if err != nil {
        log.Errorf("error at parse url %v, err: %v", webUrl, err)
    } else {
        host := u.Hostname()
        ip, err := getIPAdress(host)
        if err == nil {
          res = h.sendGetRequest(ip.String(), host, u.Scheme, u.Path, 'WebHook')
        }
    }
}
 
// getIPAdress returns IP address from domain name if IP address is not in a restricted range
func getIPAdress(host string) (net.IP, error) {
  ips, err := net.LookupIP(host)
  if err != nil {
    log.Errorf("Error at IP lookUP: %v, err: %v", host, err)
    return nil, err
  }
 
  if valid, err := validateIPs(ips); !valid {
    return nil, err
  }
 
  return findIpv4(ips)
}
 
func validateIPs(ips []net.IP) (bool, error) {
  if len(ips) == 0 {
    return false, errors.New("IP not found")
  }
 
  for _, ip := range ips {
    if ip.To16() == nil && ip.To4() == nil {
      log.Errorf("IP: %v is not valid", ip)
      return false, errors.New("IP is not valid")
    }
    // IsPrivate reports whether ip is a private address, according to
    // RFC 1918 (IPv4 addresses) and RFC 4193 (IPv6 addresses).
    if ip.IsPrivate() {
      log.Errorf("IP address: %v is a private address", ip)
      return false, errors.New("IP address is a private address")
    }
    // checks Local Address Range of 127.0.0.0 - 127.255.255.255
    if ip.IsLoopback() {
      log.Errorf("IP address: %v is a local address", ip)
      return false, errors.New("IP address is a local address")
    }
    // checks APIPA Address Range of 169.254.0.0 - 169.254.255.255
    if ip.IsLinkLocalUnicast() {
      log.Errorf("IP address: %v is a link-local unicast address", ip)
      return false, errors.New("IP address is a link-local unicast address")
    }
  }
 
  return true, nil
}
 
// findIpv4 returns the first Ipv4 out of ips
func findIpv4(ips []net.IP) (net.IP, error) {
  for _, ip := range ips {
    if ipv4 := ip.To4(); ipv4 != nil {
      return ipv4, nil
    }
  }
  return nil, errors.New("no IPv4 found")
}
 
func sendGetRequest(webIP, host, scheme, path) (string, error) {
    dialer := &net.Dialer{
        Timeout:   10 * time.Second,
        KeepAlive: 10 * time.Second,
    }
 
    // provide a custom Transport.DialContext function
    // to force requests to use a specific destination IP address
    http.DefaultTransport.(*http.Transport).DialContext = func(ctx context.Context, network, addr string) (net.Conn, error) {
        port := ":80"
        if scheme == "https" {
            port = ":443"
        }
 
        addr = webIP + port
        return dialer.DialContext(ctx, network, addr)
    }
 
    webURL := scheme + "://" + host + path
    resp, err := http.Get(webURL)
    if err != nil {
        return "", fmt.Errorf("error when doing a GET request to publisher webURL [%s]: %v", webURL, err)
    }
}
```

</details>

<details>
  <summary>Gitlab's Ruby Example</summary>
  
```ruby
# Source: https://gitlab.com/gitlab-org/gitlab-foss/-/blob/eabd80f72f4f7d8e19b26526aa1f44c43d78e8b3/lib/gitlab/url_blocker.rb
require 'ipaddress'
 
# Expects Addressable::URI
def validate_uri(uri)
    address_info = get_address_info(uri)
    ip_address = address_info.first&.ip_address
    # Replace domain with resolved IP address    
    protected_uri_with_hostname = enforce_uri_hostname(ip_address, uri)
    # Verify that the resolved IP address is not localhost, loopback, private, or link local
    validate_localhost(address_info)
    validate_loopback(address_info)
    validate_local_network(address_info)
    validate_link_local(address_info)
    protected_uri_with_hostname
end  
 
def validate_localhost(addrs_info)
    local_ips = ["::", "0.0.0.0"]
    local_ips.concat(Socket.ip_address_list.map(&:ip_address))
 
    return if (local_ips & addrs_info.map(&:ip_address)).empty?
 
    raise BlockedUrlError, "Requests to localhost are not allowed"
end
 
def validate_loopback(addrs_info)
    return unless addrs_info.any? { |addr| addr.ipv4_loopback? || addr.ipv6_loopback? }
 
    raise BlockedUrlError, "Requests to loopback addresses are not allowed"
end
 
def validate_local_network(addrs_info)
    return unless addrs_info.any? { |addr| addr.ipv4_private? || addr.ipv6_sitelocal? || addr.ipv6_unique_local? }
 
    raise BlockedUrlError, "Requests to the local network are not allowed"
end
 
def validate_link_local(addrs_info)
    netmask = IPAddr.new('169.254.0.0/16')
    return unless addrs_info.any? { |addr| addr.ipv6_linklocal? || netmask.include?(addr.ip_address) }
 
    raise BlockedUrlError, "Requests to the link local network are not allowed"
end  def enforce_uri_hostname(ip_address, uri)
    return [uri, nil] unless ip_address && ip_address != uri.hostname
    new_uri = uri.dup
    new_uri.hostname = ip_address
    [new_uri, uri.hostname]
end
 
def get_address_info(uri)
    Addrinfo.getaddrinfo(uri.hostname, get_port(uri), nil, :STREAM).map do |addr|
        addr.ipv6_v4mapped? ? addr.ipv6_to_ipv4 : addr
    end
rescue SocketError
    raise BlockedUrlError, "Host cannot be resolved or invalid"
rescue ArgumentError => error
    raise unless error.message.include?('hostname too long')
    raise BlockedUrlError, "Host is too long (maximum is 1024 characters)"
end
```

</details>

## Resources

- [PortSwigger SSRF Explanation and examples](https://portswigger.net/web-security/ssrf)
- [OWASP SSRF Explanation](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [OWASP SSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)
- [SSRF + DNS Rebinding Example](https://www.youtube.com/watch?v=R5WB8h7hkrU) (Video)
- [Gitlab SSRF + DNS Rebinding Fix](https://gitlab.com/gitlab-org/gitlab-foss/-/blob/eabd80f72f4f7d8e19b26526aa1f44c43d78e8b3/lib/gitlab/url_blocker.rb#L22)
