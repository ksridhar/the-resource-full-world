# RFC 8288

Here are practical examples of Web Linking in HTTP headers, compliant with **RFC 8288**.

RFC 8288 defines the `Link` header field, which allows HTTP responses to point to other related resources. The general syntax is:

`Link: <URI-Reference>; param1=value1; param2="value2"`

-----

## Set 1

### 1\. Pagination

This is one of the most common uses, typically seen in APIs (like GitHub's) to help clients navigate through lists of resources.

**Scenario:** A client requests page 2 of a list of users. The server provides links to the next, previous, first, and last pages.

```http
Link: <https://api.example.com/users?page=3>; rel="next",
      <https://api.example.com/users?page=1>; rel="prev",
      <https://api.example.com/users?page=1>; rel="first",
      <https://api.example.com/users?page=50>; rel="last"
```

  * **`rel="next"`**: Indicates the URL for the next page of results.
  * **Comma Separation**: Multiple links can be combined in a single header line using commas.

### 2\. Stylesheets and Preloading

Web browsers use these links to discover assets early, often before parsing the full HTML, which improves page load performance.

**Scenario:** The server tells the browser to load a CSS file and preload a font.

```http
Link: <https://example.com/style.css>; rel="stylesheet",
      <https://example.com/font.woff2>; rel="preload"; as="font"; crossorigin
```

  * **`rel="preload"`**: Tells the browser to fetch the resource immediately.
  * **`as="font"`**: An extension attribute (not strictly core RFC 8288 but standard in usage) helping the browser prioritize the fetch.

### 3\. Canonical and Alternates

This is crucial for SEO and handling multiple formats or languages of the same resource.

**Scenario:** You are serving a PDF version of a report, but you want to point to the "main" HTML version (canonical) and a French translation.

```http
Link: <https://example.com/report>; rel="canonical",
      <https://example.com/report.fr.pdf>; rel="alternate"; hreflang="fr",
      <https://example.com/report-feed.xml>; rel="alternate"; type="application/rss+xml"
```

  * **`hreflang="fr"`**: Hints that the linked resource is in French.
  * **`type="application/rss+xml"`**: Hints at the media type of the linked resource so the client doesn't have to fetch it to find out.

### 4\. Changing the Context (The "Anchor" Parameter)

By default, a link describes a relationship between the *requested* URL and the *linked* URL. The `anchor` parameter changes this, allowing the response to describe a relationship between two *other* URIs.

**Scenario:** You request `current-data.json`. The server wants to tell you that `image-A.jpg` describes `image-B.jpg`, even though neither is the requested JSON file.

```http
Link: <https://example.com/image-B.jpg>; rel="describedby"; anchor="https://example.com/image-A.jpg"
```

  * **Usage**: This asserts that `image-A.jpg` is described by `image-B.jpg`.

### 5\. Authorship and Licensing

Used to provide metadata about the content without cluttering the response body.

**Scenario:** An image is served, and the headers provide the license and author info.

```http
Link: <https://creativecommons.org/licenses/by/4.0/>; rel="license",
      <https://example.com/profile/jane-doe>; rel="author"
```

### 6\. Versioning

Useful for document management systems to point to historical or newer versions of a document.

**Scenario:** A user requests an old version of a contract. The server points to the latest version and the predecessor.

```http
Link: <https://example.com/docs/contract-v2>; rel="latest-version",
      <https://example.com/docs/contract-v1>; rel="predecessor-version"
```

-----

### Summary of Key Syntax Rules (RFC 8288)

1.  **Angle Brackets**: The Target URI must always be enclosed in `< >`.
2.  **Relation Type (`rel`)**: This is mandatory. It describes *how* the link relates to the context.
3.  **Quotes**: Parameter values (like `rel="next"`) are usually quoted, though technically optional for simple tokens. It is best practice to quote them.
4.  **Case Sensitivity**: The `rel` parameter value is case-insensitive.

**Would you like me to write a snippet of code (Python/Node.js) to parse these headers into a dictionary?**


## Set 2

Here are examples using some of the more obscure or "rarely used" link relations from the IANA registry. These are fully compliant with RFC 8288 but serve niche use cases compared to standard pagination or stylesheets.

### 1\. Censorship and Legal Blocking (`blocked-by`)

This relation is used when a resource is unavailable due to legal demands or censorship. It links to the entity that demanded the block.

**Scenario:** A user tries to access a webpage that has been taken down due to a court order. The server responds with a 451 (Unavailable For Legal Reasons) status and links to the legal authority or order.

```http
HTTP/1.1 451 Unavailable For Legal Reasons
Link: <https://www.example-court.gov/orders/case-12345>; rel="blocked-by"
```

  * **Usage:** Machine-readable way to indicate *who* or *what* is responsible for the content being inaccessible.

### 2\. Document Conversion (`converted-from`)

This is useful in document processing pipelines to trace the lineage of a file.

**Scenario:** You request a PDF report. The server serves the PDF but uses the Link header to indicate that this file was generated from a specific DOCX file.

```http
HTTP/1.1 200 OK
Content-Type: application/pdf
Link: <https://example.com/reports/source/quarterly.docx>; rel="converted-from"
```

  * **Difference from `alternate`:** `alternate` implies the two resources are equivalents (just different formats). `converted-from` implies a directional lineage (A came from B).

### 3\. Payment and Tipping (`payment`)

This relation points to a resource where a user can pay or donate money related to the content.

**Scenario:** A blog post or a digital asset is served. The author includes a link to their payment processor or "tip jar" in the headers so that browser extensions or wallets can auto-discover it.

```http
HTTP/1.1 200 OK
Link: <https://payment-provider.com/pay/jdoe>; rel="payment"; title="Support the author"
```

  * **Utility:** Allows decoupled payment discovery without scraping HTML for "Donate" buttons.

### 4\. Time-Based Versioning (`memento` and `timegate`)

Used heavily in Web Archiving (like the Wayback Machine). This follows the "Memento" framework (RFC 7089) but utilizes standard Web Linking.

**Scenario:** You are viewing an archived version of a page from 2015. The response links to the original live resource and the "TimeGate" (a router for finding other versions).

```http
HTTP/1.1 200 OK
Date: Tue, 20 Oct 2015 10:00:00 GMT
Link: <http://example.com/page>; rel="original",
      <http://archives.example.org/timegate/http://example.com/page>; rel="timegate",
      <http://archives.example.org/web/20120101/http://example.com/page>; rel="memento"; datetime="Sun, 01 Jan 2012 00:00:00 GMT"
```

  * **`rel="original"`**: Points to the resource as it exists on the live web.
  * **`datetime`**: An extension attribute specific to the Memento protocol, typically compliant with RFC 8288's parameter syntax.

### 5\. Privacy Policies (`privacy-policy`)

While links to privacy policies are common in HTML footers, putting them in the HTTP header allows user agents (browsers/crawlers) to find the policy without parsing the DOM.

**Scenario:** An API endpoint returns JSON data. Since there is no "footer" in JSON, the server puts the link to the data handling policy in the header.

```http
HTTP/1.1 200 OK
Content-Type: application/json
Link: <https://example.com/legal/privacy-v2.html>; rel="privacy-policy"
```

### 6\. Geolocation Feeds (`geofeed`)

Used by ISPs and content providers to declare the geolocation of their IP prefixes.

**Scenario:** A server responding to a network diagnostic tool links to a CSV file that maps IP ranges to geographical locations (RFC 8805).

```http
HTTP/1.1 200 OK
Link: <https://example.com/geo/feed.csv>; rel="geofeed"
```

### 7\. Link Inverse (`describes` vs `describedby`)

We often use `describedby` (e.g., "This image is `describedby` this metadata.json"). The `describes` relation is the rare inverse.

**Scenario:** You are requesting a metadata file (e.g., an XML description). The server links back to the image file that this metadata is describing.

```http
HTTP/1.1 200 OK
Content-Type: application/xml
Link: <https://example.com/images/photo-123.jpg>; rel="describes"
```

* **Meaning:** "The content you are currently reading (`metadata.xml`) describes the resource at the target URI (`photo-123.jpg`)."


