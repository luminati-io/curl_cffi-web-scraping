# Using curl\_cffi for Web Scraping in Python

[![Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.com/) 

This guide explains how to use curl\_cffi to enhance a web scraping script in Python by mimicking real browser TLS fingerprints.

- [What Is `curl_cffi`?](#what-is-curl_cffi)
- [How It Works](#how-it-works)
- [How to Use `curl_cffi` for Web Scraping](#how-to-use-curl_cffi-for-web-scraping)
  - [Step #1: Project Setup](#step-1-project-setup)
  - [Step #2: Install `curl_cffi`](#step-2-install-curl_cffi)
  - [Step #3: Connect to the Target Page](#step-3-connect-to-the-target-page)
  - [Step #4: Add the Data Scraping Logic](#step-4-add-the-data-scraping-logic)
  - [Step #5: Put It All Together](#step-5-put-it-all-together)
- [`curl_cffi`: Advanced Usage](#curl_cffi-advanced-usage)
  - [Browser Impersonation Selection](#browser-impersonation-selection)
  - [Session Management](#session-management)
  - [Proxy Integration](#proxy-integration)
  - [Async API](#async-api)
  - [WebSockets Connection](#websockets-connection)
- [`curl_cffi` vs Requests vs AIOHTTP vs HTTPX for Web Scraping](#curl_cffi-vs-requests-vs-aiohttp-vs-httpx-for-web-scraping)
- [`curl_cffi` Alternatives for Web Scraping](#curl_cffi-alternatives-for-web-scraping)

## What Is `curl_cffi`?

[`curl_cffi`](https://github.com/lexiforest/curl_cffi) provides Python bindings for the `curl-impersonate` fork via CFFI and thus can impersonate browser TLS/JA3/HTTP2 fingerprints. This helps bypassing anti-bot blocks based on [TLS fingerprinting](/blog/web-data/tls-fingerprinting).

Here are some of its features:

- Support for JA3/TLS and HTTP2 fingerprint impersonation, including recent browsers and custom fingerprints
- Much faster than `requests` and `httpx`, on par with `aiohttp`
- Mimics the `requests` API
- Support for `asyncio` to perform asynchronous HTTP requests
- Support for proxy rotation on each request
- Support for HTTP/2.0 and `WebSocket`

## How It Works

When you send an HTTPS request, a [TLS handshake](https://www.cloudflare.com/learning/ssl/what-happens-in-a-tls-handshake/) takes place, generating a unique TLS fingerprint. Because HTTP clients operate differently from web browsers, their fingerprints can reveal automation, potentially activating anti-bot defenses.

cURL Impersonate, that `curl_cffi` is based on, customizes cURL to replicate authentic browser TLS fingerprints:

- **TLS library tweaks**: Rely on the libraries for TLS connection used by browsers instead of that of cURL.
- **Configuration changes**: Adjust TLS extensions and SSL options to mimic browsers.
- **HTTP/2 customization**: Match browser handshake settings.
- **Non-default cURL flags**: Set `--ciphers`, `--curves`, and custom headers for accuracy.

This makes the requests resemble those from a real browser, aiding in bypassing bot detection.

## How to Use `curl_cffi` for Web Scraping

Let's try to scrape the “Keyboard” page from Walmart:  

![The Walmart “Keyboard” product page](https://github.com/luminati-io/curl_cffi-web-scraping/blob/main/Images/s_5A13EDF6E0EA32867C0C89DFE864B4C8FA81CE91CC8CE80729F465B232BE7073_1737992236291_image.png)

If you try to access this page using any HTTP client, you will receive the following error page:  

![Note the response from the server](https://github.com/luminati-io/curl_cffi-web-scraping/blob/main/Images/s_5A13EDF6E0EA32867C0C89DFE864B4C8FA81CE91CC8CE80729F465B232BE7073_1737992185267_image.png)

You will get this bot detection page even if you set the `User-Agent` to simulate a real browser because of TLS fingerprinting. This is where `curl_cffi` comes in handy.

### Step #1: Project Setup

Make sure that you have Python 3+ installed on your machine. Then, create a directory for your `curl_cffi` scraping project:

```
mkdir curl-cfii-scraper
```

Navigate into that directory and set up a [virtual environment](https://docs.python.org/3/library/venv.html):

```
cd curl-cfii-scraper
python -m venv env
```

Open the project folder in your preferred Python IDE and create a `scraper.py` file in that folder.

In your IDE’s terminal, activate the virtual environment. On Linux or macOS, use:

```bash
./env/bin/activate
```

On Windows, launch:

```bash
env/Scripts/activate
```

### Step #2: Install `curl_cffi`

In an activated virtual environment, install the HTTP client:

```
pip install curl-cffi
```

### Step #3: Connect to the Target Page

Import `requests` from `curl_cffi`:

```python
from curl_cffi import requests
```

This object exposes a high-level Requests-like API. You can use it to perform a GET HTTP request to the target page:

```python
response = requests.get("https://www.walmart.com/search?q=keyboard", impersonate="chrome")
```

The `impersonate="chrome"` argument tells `curl_cffi` to make the HTTP request look like it is coming from the latest version of Chrome. This will make Walmart treat the automated request as a regular browser request and return the standard web page.

You can access the HTML content of the target page with:

```python
html = response.text
```

If you print `html`, you will see:

```html
<!DOCTYPE html>
<html lang="en-US">
   <head>
      <meta charSet="utf-8"/>
      <meta property="fb:app_id" content="105223049547814"/>
      <meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=1, interactive-widget=resizes-content"/>
      <link rel="dns-prefetch" href="https://tap.walmart.com "/>
      <link rel="preload" fetchpriority="high" crossorigin="anonymous" href="https://i5.walmartimages.com/dfw/63fd9f59-a78c/fcfae9b6-2f69-4f89-beed-f0eeb4237946/v1/BogleWeb_subset-Bold.woff2" as="font" type="font/woff2"/>
      <link rel="preload" fetchpriority="high" crossorigin="anonymous" href="https://i5.walmartimages.com/dfw/63fd9f59-a78c/fcfae9b6-2f69-4f89-beed-f0eeb4237946/v1/BogleWeb_subset-Regular.woff2" as="font" type="font/woff2"/>
      <link rel="preconnect" href="https://beacon.walmart.com"/>
      <link rel="preconnect" href="https://b.wal.co"/>
      <title>Electronics - Walmart.com</title>
      <!-- omitted for brevity ... -->
```

### Step #4: Add the Data Scraping Logic

To perform web scraping, you will also need a library for HTML parsing like [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/):

```bash
pip install beautifulsoup4
```

Import it in `scraper.py`:

```python
from bs4 import BeautifulSoup
```

Use it to parse the HTML of the page:

```python
soup = BeautifulSoup(response.text, "html.parser")
```

[`"html.parser"`](https://docs.python.org/3/library/html.parser.html) is the default HTML parser from Python’s standard library used by BeautifulSoup for parsing the HTML string. It contains methods to select HTML elements on the page and extract data from them.

The next example illustrates how to scrape just the page title. You can select it through a [CSS selector](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_selectors) using the `find()` method and then access its text with the `text` attribute:

```python
title_element = soup.find("title")
title = title_element.text
```

Print the page title:

```python
print(title)
```

### Step #5: Put It All Together

This is your final `curl_cffi` web scraping script:

```python
from curl_cffi import requests
from bs4 import BeautifulSoup

# Send a GET request to the Walmart search page for "keyboard"
response = requests.get("https://www.walmart.com/search?q=keyboard", impersonate="chrome")

# Extract the HTML from the page
html = response.text

# Parse the response content with BeautifulSoup
soup = BeautifulSoup(response.text, "html.parser")

# Find the title tag using a CSS selector and print it
title_element = soup.find("title")
# Extract data from it
title = title_element.text

# More complex scraping logic...

# Print the scraped data
print(title)
```

Launch it:

```bash
python3 scraper.py
```

On Windows:

```bash
python scraper.py
```

The result will be:

```
Electronics - Walmart.com
```

If you remove the `impersonate="chrome"` argument, you will get instead:

```
Robot or human?
```

## `curl_cffi` : Advanced Usage

### Browser Impersonation Selection

`curl_cffi` supports impersonating several browsers using unique labels that you can pass to the `impersonate` argument:

```python
response = requests.get("<YOUR_URL>", impersonate="<BROWSER_LABEL>")
```

You can use the following labels:

- `chrome99`, `chrome100`, `chrome101`, `chrome104`, `chrome107`, `chrome110`, `chrome116`, `chrome119`, `chrome120`, `chrome123`, `chrome124`, `chrome131`
- `chrome99_android`, `chrome131_android`
- `edge99`, `edge101`
- `safari15_3`, `safari15_5`, `safari17_0`, `safari17_2_ios`, `safari18_0`, `safari18_0_ios`

Here are some recommendations:

1.  To always impersonate the latest browser versions, you can simply use `chrome`, `safari` and `safari_ios`.
2.  [Firefox is currently not available](https://github.com/lexiforest/curl_cffi/issues/59) because it uses NSS, while other browsers use boringssl, and curl can only be linked to one TLS library at the same time.
3.  Browser versions are added only when their fingerprints change. If a version is skipped, you can still impersonate it by using the headers of the previous version.
4.  For non-browser targets, use `ja3`, `akamai`, and similar arguments to specify your own custom TLS fingerprints. Please refer to the [documentation on impersonation](https://curl-cffi.readthedocs.io/en/latest/impersonate.html) for details.

### Session Management

`curl-cfii` can use [`Session`](https://requests.readthedocs.io/en/latest/user/advanced/#session-objects) objects to persist certain parameters across multiple requests, such as cookies, headers, or other session-specific data.

Here is a code example:

```python
# Create a new session
session = requests.Session()

# This endpoint sets a cookie on the server
session.get("https://httpbin.io/cookies/set/userId/5", impersonate="chrome")

# Print the session's cookies to confirm they are being stored
print(session.cookies)
```

The output of the above script will be:

```
<Cookies[<Cookie userId=5 for httpbin.org />]>
```

The result proves that the session is maintaining state across requests, such as storing cookies defined by the server.

### Proxy Integration

Just like the `requests` library, `curl_cffi` supports proxy integration through a `proxies` object:

```python
# Define your proxy URL
proxy = "YOUR_PROXY_URL"

# Create a dictionary of proxies for HTTP and HTTPS
proxies = {"http": proxy, "https": proxy}

# Make a request using a proxy and browser impersonation
response = requests.get("<YOUR_URL>", impersonate="chrome", proxies=proxies)
```

### Async API

`curl_cffi` supports peforming async requests through `asyncio` via the [`AsyncSession`](https://curl-cffi.readthedocs.io/en/latest/api.html#curl_cffi.requests.AsyncSession) object:

```python
from curl_cffi.requests import AsyncSession
import asyncio

# Define an async function to execute the asynchronous code
async def fetch_data():
    async with AsyncSession() as session:
        # Perform the asynchronous GET request
        response = await session.get("https://httpbin.org/anything", impersonate="chrome")
        # Print the response text
        print(response.text)

# Run the async function
asyncio.run(fetch_data())
```

Using `AsyncSession` makes it easier to handle multiple asynchronous requests efficiently.

### WebScokets Connection

`curl_cffi` also supports `WebSocket`s through the [`WebSocket`](https://curl-cffi.readthedocs.io/en/latest/api.html#curl_cffi.requests.Session.ws_connect) class:

```python
from curl_cffi.requests import WebSocket


# Define a callback function to handle incoming messages
def on_message(ws, message):
    print(message)

# Initialize the WebSocket connection with the callback
ws = WebSocket(on_message=on_message)

# Connect to a sample WebSocket server and listen for messages
ws.run_forever("wss://api.gemini.com/v1/marketdata/BTCUSD")
```

This is especially useful for scraping real-time data from sites or APIs that use `WebSocket` to populate data dynamically.

Instead of scraping rendered pages, you can directly target the `WebSocket` channel for efficient data retrieval.

> **Note**:\
> You can use `WebSocket`s asynchronously thanks to the [`AsyncWebSocket`](https://curl-cffi.readthedocs.io/en/latest/api.html#curl_cffi.requests.AsyncSession.ws_connect) class.

## curl\_cffi vs Requests vs AIOHTTP vs HTTPX for Web Scraping

The following table compares `curl_cffi` with other popular Python HTTP clients for web scraping:

| **Feature** | **curl\_cffi** | **Requests** | **AIOHTTP** | **HTTPX** |
| --- | --- | --- | --- | --- |
| **Sync API** | ✔️  | ✔️  | ❌   | ✔️  |
| **Async API** | ✔️  | ❌   | ✔️  | ✔️  |
| **Support for** `**WebSocket**`s | ✔️  | ❌   | ✔️  | ❌   |
| **Connection pooling** | ✔️  | ✔️  | ✔️  | ✔️  |
| **Support for HTTP/2** | ✔️  | ❌   | ❌   | ✔️  |
| `**User-Agent**` **customization** | ✔️  | ✔️  | ✔️  | ✔️  |
| **TLS fingerprint spoofing** | ✔️  | ❌   | ❌   | ❌   |
| **Speed** | High | Medium | High | Medium |
| **Retry mechanism** | ❌   | Available via `HTTPAdapter`s | Available only via a third-party library | Available via built-in `Transport`s |
| **Proxy integration** | ✔️  | ✔️  | ✔️  | ✔️  |
| **Cookie handling** | ✔️  | ✔️  | ✔️  | ✔️  |

## `curl_cffi` Alternatives for Web Scraping

`curl_cffi` requires a manual approach to web scraping, where most of the code must be written by hand. While effective for simple static websites, it can be challenging when dealing with dynamic or highly secure sites.

Bright Data provides several `curl_cffi` alternatives:

- [Scraping Browser API](https://brightdata.com/products/scraping-browser): Fully managed cloud browser instances seamlessly integrated with Puppeteer, Selenium, and Playwright. These browsers come with built-in CAPTCHA solving and automated proxy rotation.
- [Web Scraper APIs](https://brightdata.com/products/web-scraper): Pre-configured endpoints provide fresh, structured data from over 100 popular domains.
- [No-Code Scraper](https://brightdata.com/products/web-scraper/no-code): A user-friendly, on-demand data collection service that requires no coding.
- [Datasets](https://brightdata.com/products/datasets): Browse pre-built datasets from various websites or tailor data collections to meet your specific needs.

Create a free Bright Data account today to test our proxies and scraping solutions!
