The `Referer` (sic) HTTP request header allows a client to identify the address (URL) of the web page from which the requested resource was linked.

It essentially tells the server: **"I am here because I clicked a link on *that* page."**

### 1\. The Name (A Famous Typo)

The most notable characteristic of this header is its spelling. It is a misspelling of the word "referrer." The error was made in the original HTTP proposal; by the time it was noticed, it was too late to change it without breaking compatibility.

  * **Standard Header:** `Referer` (RFC-compliant)
  * **Correct English:** Referrer
  * **DOM JavaScript Property:** `document.referrer` (Correctly spelled)

### 2\. Syntax

```http
Referer: <absolute-URI>
```

  * **Example:** `Referer: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referer`
  * It must be an absolute URI (e.g., it must contain `https://`).
  * It generally does not contain the `#fragment` or `user:password` information for privacy and security reasons.

### 3\. Common Use Cases

Server-side applications use this header for three main purposes:

1.  **Analytics & Logging:** Determining where traffic is coming from (e.g., "50% of our visitors came from Google, 30% from Twitter").
2.  **Hotlinking Protection:** Preventing other sites from using your bandwidth.
      * *Example:* An image server checks the `Referer`. If the request comes from `my-site.com`, it serves the image. If it comes from `thief-site.com`, it returns a 403 Forbidden or a placeholder image.
3.  **Back Buttons / Navigation:** Some simple applications use it to implement a "Back" button functionality (though this is unreliable).

### 4\. Privacy and Security

The `Referer` header can leak sensitive information. For example, if a user is on a password reset page `example.com/reset-password?token=SECRET123` and clicks a link to an external site, that secret token could be sent in the `Referer` header to the external site.

To control this, modern browsers and servers use the **`Referrer-Policy`** header.

#### The `Referrer-Policy` Header

This response header controls how much information the browser includes in the `Referer` header for outgoing links.

  * `no-referrer`: Never send the header.
  * `origin`: Send only the domain (e.g., `https://example.com`), stripping the specific page path.
  * `strict-origin-when-cross-origin` (Default in most modern browsers):
      * Same-origin (internal links): Send full path.
      * Cross-origin (external links): Send only the domain.
      * Downgrade (HTTPS -\> HTTP): Send nothing.

### 5\. Example Flow

**Scenario:** User is on `https://google.com` and clicks a link to `https://wikipedia.org`.

**Request sent to Wikipedia:**

```http
GET / HTTP/1.1
Host: wikipedia.org
Referer: https://google.com/
```

**Scenario:** User is on `https://bank.com/account/12345` and clicks a link to `https://evil.com`.

  * *If Policy is `strict-origin-when-cross-origin`:*
    `Referer: https://bank.com/` (Path and account number are hidden).
