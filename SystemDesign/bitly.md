# Design Bit.ly

## Understanding the problem

### Functional Requirements

* Functional requirements are the features that the system must have, to satisfy the needs of the user.

* It is advisable to ask the interviewer some clarifying questions to gain a better understanding of the system.

* Zero in on the top 3-4 features.

**Core Requirements**

1. Users should be able to submit a long URL and receive a shortened version
  * Optionally, users should be able to specify a custom alias for their shortened URL.
  * Optionally, users should be able to specify an expiration date for their shortened URL.


2. Users should be able to access the original URL by using the shortened URL.

**Below the line (out of scope):**
* User authentication and account management.
* Analytics on link clicks (e.g., click counts, geographic data).


### Non-Functional Requirements

* Non-functional requirements refer to specifications about how a system operates, rather than what tasks it performs.

* These requirements are critical as they define system attributes like scalability, latency, security and availability, and are often framed as specific benchmarks

**Core non-functional requirements**

1. The system should ensure uniqueness for the short codes (no two long URLs can map to the same short URL)
2. The redirection should occur with minimal delay (< 100ms)
3. The system should be reliable and available 99.99% of the time (availability > consistency)
4. The system should scale to support 1B shortened URLs and 100M DAU

**Below the line (out of scope):**
* Data consistency in real-time analytics.
* Advanced security features like spam detection and malicious URL filtering.

For this problem, an important consideration is the imbalance between read and write operations. The read-to-write ratio is heavily skewed towards reads, as there is frequency users access to shortened URLS but creation is comparatively rare.

### Defining the core entities

Establish the key entities so we can guide our thought progress and lay a solid foundation as we progress.

In this example, the core entities are straightforward:
1. **Original URL**: The original long URL the user wants to shorten.
2. **Short URL**: The shortened URL that the user receives and can share.
3. **User**: Represents the user who created the shortened URL.

### The API

This is the first point of reference for the high-level design

Go one-by-one through the core requirements and define the APIs that are necessary to satisfy them. These map usually 1:1 to the functional requirements.

9/10 times this will use a REST API and focus on choosing the right HTTP method or verb to use.

* **C**reate --> POST
* **R**ead   --> GET
* **U**pdate --> PUT
* **D**elete --> DELETE

To shorten a URL we'll need a POST endpoint that takes in a the long URL and optionally a custom alias and expiration data, and returns the the shortened URL. We POST because we are creating a new entry in our database.

```
// Shorten a URL
POST /urls
{
  "long_url": "https://www.example.com/some/very/long/url",
  "custom_alias": "optional_custom_alias",
  "expiration_date": "optional_expiration_date"
}
->
{
  "short_url": "http://short.ly/abc123"
}

```

For redirection we'll need a GET endpoint that takes int he shortcode and redirects to the original long URL.

```
// Redirect to Original URL
GET /{short_code}
-> HTTP 302 Redirect to the original long URL

```

### High-level Design

Start by going one-by-one through the functional requirements and designing a single system to satisfy them.

**1. Users should be able to submit a long URL and receive a shortened version**

We need to consider how we're going to generate a short URL. We can outline the core components to make this happen

1. Client: Users interact with the system through an application
2. Primary Server: the primary server receives requests from the client and handles all business logic like short url creation and validation
3. Database: Stores the mapping of short codes to long urls, as well as user-generated aliases and expiration dates
