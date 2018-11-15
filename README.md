# Man in the Middle

_Firefox Extension._  

---

Allow user to block or redirect requests, modify headers and response body, inject JavaScript and CSS into pages.

---

**[Get Man in the Middle](https://addons.mozilla.org/en-US/firefox/addon/man-in-the-middle/)** on Firefox Add-ons.  
**[Get help](#rules)** writing rules.


## Screenshots

![Block or redirect requests](https://addons.cdn.mozilla.net/user-media/previews/full/211/211150.png?modified=1542280030)
_Use Blocking Rules to block or redirect requests._

![Modify request and response headers](https://addons.cdn.mozilla.net/user-media/previews/full/211/211152.png?modified=1542280030)
_Use Header Rules to modify request and response headers._

![Use JavaScript to modify request headers](https://addons.cdn.mozilla.net/user-media/previews/full/211/211153.png?modified=1542280030)
_Headers can be modified using JavaScript._

![Use JavaScript to modify response body](https://addons.cdn.mozilla.net/user-media/previews/full/211/211154.png?modified=1542280030)
_Use Response Rules to modify network responses._

![Inject JavaScript and CSS into pages](https://addons.cdn.mozilla.net/user-media/previews/full/211/211151.png?modified=1542293585)
_Use Content Scripts to inject JavaScript and CSS codes into pages._

![Man in the Middle dark theme](https://addons.cdn.mozilla.net/user-media/previews/full/211/211160.png?modified=1542300620)
_Content Scripts can even be injected to the extension's pages._


## Rules

### Blocking Rules
Rules to block or redirect requests.  
- [URL filters (Required)](#url-filters);
- [Method](#method);
- [Redirect URL](#redirect-url);
- [Origin URL filters](#origin-url-filters).

### Header Rules
Rules to modify request and response headers.
- [Text headers (Required)](#text-headers);
- [Text type](#text-type);
- [Header type](#header-type);
- [URL filters (Required)](#url-filters);
- [Method](#method);
- [Origin URL filters](#origin-url-filters).

### Response Rules
Rules to modify network responses.
- [Text response (Required)](#text-response);
- [Text type](#text-type);
- [URL filters (Required)](#url-filters);
- [Method](#method);
- [Origin URL filters](#origin-url-filters).

### Content Scripts
Rules to inject JavaScript and CSS into pages.
- [Code (Required)](#code);
- [Script type](#script-type);
- [URL filters](#url-filters);
- [DOM event](#dom-event);
- [Origin URL filters](#origin-url-filters).


## Properties

### URL filters
Filter `request URL`s or `document URL`s.
- Format: [RegExp pattern](#regexp-pattern) or [String filter](#string-filter).
- Separator: `line break`, i.e, `'\n'`, `'\r'` or `'\r\n'`.
- A `URL` is satisfied if it matches at least one of the filters.
  - A `URL` matches a filter if it matches the `RegExp pattern` or includes the `String filter`.
- Rules: [Blocking Rules](#blocking-rules), [Header Rules](#header-rules), [Response Rules](#response-rules) and [Content Scripts](#content-scripts).

### Method
Filters `request method`s.
- Value can be one of the HTTP request methods, i.e, `'GET'`, `'POST'`, `'HEAD'`, etc.
- A `request method` is satisfied if it equals to the `method`.
- Rules: [Blocking Rules](#blocking-rules), [Header Rules](#header-rules) and [Response Rules](#response-rules).

### Redirect URL
A `URL` to redirect `request`s to.
- If not set, matched requests are blocked.
- Parameters `'$n'` (`1 <= <int>n <= 100`), in a `redirect URL` are replaced with capture groups from `RegExp pattern` [URL filter](#url-filters).
- Examples:
  ````
  Force HTTPS for all network requests.
  URL filter:   /^http:(.*)/
  Redirect URL: https:$1
  ````
- Rules: [Blocking Rules](#blocking-rules).

### Origin URL filters
Filter `document URL`s.
- Format: [RegExp pattern](#regexp-pattern) or [String filter](#string-filter).
- Separator: comma `','`.
- A `document URL` is satisfied if one of the following is satisfied:
  - No `filter` is set (default);
  - The `document URL` matches one of the filters.
    - A `document URL` matches a filter if it matches the `RegExp pattern` or includes the `String filter`.
- Rules: [Blocking Rules](#blocking-rules), [Header Rules](#header-rules), [Response Rules](#response-rules) and [Content Scripts](#content-scripts).

### Text headers
To modify request or response headers.
- Format: `Plaintext` or [Restricted JavaScript](#restricted-javascript).
- Type `Plaintext`:  
  _`Pair`s of headers._
  - Separator: `line break`, i.e, `'\n'`, `'\r'` or `'\r\n'`.
  - A `Pair` is of the format: `name: value`.
    - If `name` is empty, the header is omitted.
    - If `value` is empty, the header with the name `name` is removed if it exists, or the header is omitted.
    - If a header with the name `name` exists, the header is modified. If there're more than one existing, the first is modified.
    - If no header with the name `name` exists, a new header is added.
  - Examples:
    ````
    This overrides the default Accept header
    Accept: *
    ````
    ````
    This removes Referer header if it exists
    Referer:
    ````
    ````
    This adds new headers to the request
    Test-0: On
    Test-1: Off
    ````
- Type [Restricted JavaScript](#restricted-javascript):  
  _Returns request or response headers._
  - The code must `return` an array of objects, each objects has two properties: `'name'` and `'value'`.
  - Depending on [Header type](#header-type),
    the code will be passed an argument `requestHeaders` or `responseHeaders`, which is the list of the existing headers.
  - Examples:
    ```` JavaScript
    // Header type: Request headers
    const acceptHeader = requestHeaders.find(({name}) => (
        name.toLowerCase() === 'accept'
    ));
    // Accept: *
    acceptHeader && acceptHeader.value = '*';
    return requestHeaders; 
    ````
    ```` JavaScript
    // Header type: Request headers
    const refererHeaderIndex = requestHeaders.findIndex(({name}) => (
        name.toLowerCase() === 'referer'
    ));
    // Remove Referer header
    if (refererHeaderIndex !== -1) {
        requestHeaders.splice(refererHeaderIndex, 1);
    }
    return requestHeaders;
    ````
    ```` JavaScript
    // Header type: Response headers
    responseHeaders.push({
        name: 'Set-Cookie',
        value: 'Firefox-Extension=Man in the Middle; HttpOnly',
    });
    return responseHeaders;
    ````
- Rule: [Header Rules](#header-rules).

### Text type
`'Plaintext'` or`'JavaScript'`.
- Rule: [Header Rules](#header-rules) and [Response Rules](#response-rules).

### Header type
`'Request headers'` or `'Response headers'`.
- Rule: [Header Rules](#header-rules).

### Text response
To modify network responses.
- Format: `Plaintext` or [Restricted JavaScript](#restricted-javascript).
- Type `Plaintext`:  
  _Any text as response body._
- Type [Restricted JavaScript](#restricted-javascript):  
  _Returns response body._
  - The code must `return` a string which is the response body.
  - The code will be passed an argument `responseBody`, which is the response from the server.
  - Examples:
    ```` JavaScript
    // Site: http://internetbadguys.com/
    return `<!DOCTYPE html>
    <html>
    <head>
        <meta charset="utf-8">
    </head>
    <body>
    <h1>Bad guys are ${(
        responseBody.includes('phish.opendns.com/?url=') ? 'blocked' : 'coming'
    )}!</h1>
    </body>
    </html>`;
    ````
- Rule: [Response Rules](#response-rules).

### Code
`JavaScript` or `CSS` code to be injected.
- Rule: [Content Scripts](#content-scripts).

### Script type
`'JavaScript'` or `'CSS'`.
- Rule: [Content Scripts](#content-scripts).

### DOM event
A stage of the `DOM` loading on which the code is injected.
- Can be one of the following values:
  - `Loading`;
  - `Loaded`;
  - `Completed`.
- Rule: [Content Scripts](#content-scripts).


## Formats

### RegExp pattern
Begins with a slash `'/'` and ends with a slash `'/'`.
- The characters inside the two slashes must form a valid RegExp, otherwise, the pattern is treated as a [String filter](#string-filter).
- Examples:
  ````
  /./
  /faceb(\w{2})k\.[\w]+/
  ````
- Properties: [URL filters](#url-filters) and [Origin URL filters](#origin-url-filters).

### String filter
A string that is not a [RegExp pattern](#regexp-pattern).
- Examples:
  ````
  http
  facebook.com
  /invalid { RegExp/
  ````
- Properties: [URL filters](#url-filters) and [Origin URL filters](#origin-url-filters).


### Restricted JavaScript
A JavaScript function body that will be executed inside a sandbox.
- The code may access only built-in objects and some APIs, which are:
  - `Object`, `Array`, `String`, `RegExp`, `JSON`, `Map`, `Set`, `Promise`, ...built-in objects;
  - `isFinite`, `isNaN`, `parseInt`, `parseFloat`;
  - `encodeURI`, `encodeURIComponent`, `decodeURI`, `decodeURIComponent`;
  - `crypto`, `performance`, `atob`, `btoa`, `fetch` and `XMLHttpRequest`.
- The function is `async`, hence, `await` can be used to perform asynchronous tasks.
- The code should always `return` a value.
- Properties: [Text headers](#text-headers) and [Text response](#text-response).



## Others

- If you have questions or need help, feel free to message me at:
  [Facebook/dangkyokhoang](https://www.facebook.com/dangkyokhoang).
- If you have feature requests, suggestions, or you've found bugs, raise issues at:
  [Man-in-the-Middle/issues](https://github.com/dangkyokhoang/Man-in-the-Middle/issues).
