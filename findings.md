# OWASP Juice Shop Security Assessment — Findings

## Test 1: Application Discovery with Burp Suite

**Target:** `http://127.0.0.1:3000`  
**Tool:** Burp Suite Community Edition

**Observation:**

Burp Suite successfully intercepted and recorded HTTP traffic between its built-in browser and the locally hosted OWASP Juice Shop application. A product API request returned HTTP status `200 OK`, confirming that the application API was reachable and responding normally.

**Security Relevance:**

Reviewing normal application traffic helps an assessor understand available endpoints, request methods, response data, and the application attack surface before conducting controlled testing.

## Test 2: Security Response Header Review

**Tool:** curl  
**Command:** `curl -I http://127.0.0.1:3000`

**Positive Security Headers Observed:**

- `X-Content-Type-Options: nosniff`
- `X-Frame-Options: SAMEORIGIN`

These headers provide browser-side protection against MIME-type confusion and some clickjacking scenarios.

**Observations for Further Review:**

- `Access-Control-Allow-Origin: *` permits cross-origin requests. Its actual security impact depends on whether sensitive data or authenticated actions are exposed by the application.
- No `Content-Security-Policy` header was observed. A Content Security Policy can provide additional protection against certain client-side injection attacks.

**Conclusion:**

The application implements some security headers, but the response-header configuration provides opportunities for further hardening and controlled testing within this authorized local lab.

## Test 3: Publicly Accessible Directory Listing

**Discovery Method:**

The application’s public `robots.txt` file included the entry:

`Disallow: /ftp`

The route was then checked using:

`curl -I http://127.0.0.1:3000/ftp/`

**Result:**

The `/ftp/` route returned `HTTP/1.1 200 OK` without authentication or access restriction. The directory-listing page was accessible in the browser, and evidence was captured in `screenshots/ftp-directory-listing.png`.

**Finding: Public Directory Listing Exposure**

**Severity:** Medium

**Description:**

A publicly accessible directory listing was available at `/ftp/`. Although the path was listed in `robots.txt` as disallowed for search engines, the endpoint was not protected by authentication or access-control rules.

**Security Impact:**

Directory listings can reveal file names, backup files, documents, configuration files, or other content that helps an attacker understand the application structure. Even when individual files are not sensitive, exposing them unnecessarily increases the application attack surface.

**Recommendation:**

- Disable directory listing on the web server.
- Store non-public files outside the web-accessible directory.
- Require authentication and authorization for files that must remain accessible.
- Do not rely on `robots.txt` as an access-control mechanism.

**Evidence:**

- `robots.txt` contained `Disallow: /ftp`
- The `/ftp/` endpoint returned `200 OK`
- Screenshot: `ftp directory listing.png`
