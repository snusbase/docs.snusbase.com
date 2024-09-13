# Snusbase API Docs

## Table of Contents

1. [Introduction](#introduction)
2. [Authentication](#authentication)
3. [API Endpoints](#api-endpoints)
   - [Database Search](#database-search)
   - [Database Statistics](#database-statistics)
   - [IP WHOIS Lookup](#ip-whois-lookup)
   - [Hash Lookup](#hash-lookup)
4. [Example Search Queries](#example-search-queries)
   - [Multiple Terms and Types](#multiple-terms-and-types)
   - [Search Specific Tables](#search-specific-tables)
   - [Group by Specific Column](#group-by-specific-column)
   - [Search with Wildcards](#search-with-wildcards)
5. [Code Examples](#code-examples)
   - [JavaScript](#javascript)
   - [Python](#python)
6. [Error Handling](#error-handling)
7. [Rate Limiting](#rate-limiting)
8. [Important Information](#important-information)

## Introduction

Welcome to the Snusbase API documentation. This guide provides comprehensive information on how to authenticate and interact with our API endpoints.

## Authentication

All API requests require authentication using an activation code. Include your activation code in the `Auth` header of each request:

```
Auth: sb[...]
```

Activation codes generated after September 2021 start with `sb` followed by 28 random characters.

## API Endpoints

### Database Search

Search the Snusbase database for leaked information.

- **Endpoint:** `https://api.snusbase.com/data/search`
- **Method:** `POST`
- **Headers:**
  - `Content-Type: application/json`
  - `Auth: YOUR_API_KEY_HERE`

#### Parameters

| Parameter  | Type                | Required | Description                                                                                                                                                         |
|------------|---------------------|----------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `terms`    | Array of strings    | Yes      | Search terms.                                                                                                                                                       |
| `types`    | Array of strings    | Yes      | Types of data to search. Possible values: `"email"`, `"username"`, `"lastip"`, `"password"`, `"hash"`, `"name"`, `"_domain"`.                                      |
| `wildcard` | Boolean             | No       | Enable wildcard search (`true` or `false`).                                                                                                                         |
| `group_by` | Boolean or string   | No       | Group results. Defaults to `"db"`. Set to `false` to disable grouping.                                                                                            |
| `tables`   | Array of strings    | No       | Limit search to specific tables.                                                                                                                                    |

#### Request Example

```http
POST https://api.snusbase.com/data/search
Content-Type: application/json
Auth: YOUR_API_KEY_HERE

{
  "terms": ["example@gmail.com"],
  "types": ["email"]
}
```

#### Response Example

```json
{
  "took": 173,
  "size": 1176,
  "results": {
    "2012_BREACHED_TO_212K_HACKING_112022": [
      {
        "username": "example",
        "email": "example@gmail.com",
        "lastip": "2a0b:f4c2:1::1",
        "hash": "4b437d0eb94655ab691a7d38caf69d6e",
        "salt": "bdyYGKsW",
        "uid": "71998",
        "created": "1660914400",
        "regip": "2a0b:f4c2:1::1"
      }
      /* Other results */
    ]
  }
}
```

### Database Statistics

Retrieve information about the current databases in the main search engine.

- **Endpoint:** `https://api.snusbase.com/data/stats`
- **Method:** `GET`
- **Headers:**
  - `Auth: YOUR_API_KEY_HERE`

#### Request Example

```http
GET https://api.snusbase.com/data/stats
Auth: YOUR_API_KEY_HERE
```

#### Response Example

```json
{
  "rows": 16720220425,
  "tables": {
    "0001_STEALERLOGS_NA_106M_MALWARE_2023": [
      "host",
      "username",
      "password",
      "_domain",
      "email"
    ]
    /* Other tables */
  },
  "features": {
    "view_more": [
      "0032_CANADA_CA_2M_BUSINESS_2011",
      /* ... */
    ],
    "hash_lookup": [
      "0001_HASHES_ORG_1913M_2021",
      /* ... */
    ],
    "combo_lookup": [
      "0001_PEMIBLANC_COMBOLIST_245M",
      /* ... */
    ]
  }
}
```

### IP WHOIS Lookup

Retrieve WHOIS information for IP addresses.

- **Endpoint:** `https://api.snusbase.com/tools/ip-whois`
- **Method:** `POST`
- **Headers:**
  - `Content-Type: application/json`
  - `Auth: YOUR_API_KEY_HERE`

#### Parameters

| Parameter | Type             | Required | Description                  |
|-----------|------------------|----------|------------------------------|
| `terms`   | Array of strings | Yes      | IP addresses to look up.     |

#### Request Example

```http
POST https://api.snusbase.com/tools/ip-whois
Content-Type: application/json
Auth: YOUR_API_KEY_HERE

{
  "terms": ["12.34.56.78", "127.0.0.1"]
}
```

#### Response Example

```json
{
  "took": 108,
  "size": 1,
  "results": {
    "12.34.56.78": {
      "as": "AS7018 AT&T Services, Inc.",
      "city": "Atlanta",
      "country": "United States",
      "countryCode": "US",
      "isp": "AT&T Services, Inc.",
      "lat": 33.7173,
      "lon": -84.4783,
      "org": "AT&T Services, Inc.",
      "region": "GA",
      "regionName": "Georgia",
      "status": "success",
      "timezone": "America/New_York",
      "zip": "30311"
    }
  },
  "errors": {
    "127.0.0.1": "reserved range"
  }
}
```

### Hash Lookup

Search for corresponding plaintext passwords or vice versa in the cracked password hash database.

- **Endpoint:** `https://api.snusbase.com/tools/hash-lookup`
- **Method:** `POST`
- **Headers:**
  - `Content-Type: application/json`
  - `Auth: YOUR_API_KEY_HERE`

#### Parameters

| Parameter  | Type                | Required | Description                                                                                                                                                         |
|------------|---------------------|----------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `terms`    | Array of strings    | Yes      | Hashes or passwords to look up.                                                                                                                                     |
| `types`    | Array of strings    | Yes      | Types of lookup. Possible values: `"hash"`, `"password"`.                                                                                                           |
| `wildcard` | Boolean             | No       | Enable wildcard search.                                                                                                                                             |
| `group_by` | Boolean or string   | No       | Group results. Defaults to `"db"`. Set to `"false"` to disable grouping.                                                                                            |
| `tables`   | Array of strings    | No       | Limit search to specific tables.                                                                                                                                    |

#### Request Example

```http
POST https://api.snusbase.com/tools/hash-lookup
Content-Type: application/json
Auth: YOUR_API_KEY_HERE

{
  "terms": ["482c811da5d5b4bc6d497ffa98491e38"],
  "types": ["hash"]
}
```

#### Response Example

```json
{
  "took": 57,
  "size": 1,
  "results": {
    "0001_HASHES_ORG_1913M_2021": [
      {
        "hash": "482c811da5d5b4bc6d497ffa98491e38",
        "password": "password123"
      }
    ]
  }
}
```

## Example Search Queries

### Multiple Terms and Types

When using multiple types, each type will be applied to all terms, so you don't need to match them in order.

#### Request Example

```http
POST https://api.snusbase.com/data/search
Content-Type: application/json
Auth: YOUR_API_KEY_HERE

{
  "terms": ["example1", "example2"],
  "types": ["username", "email", "lastip", "hash", "password", "name"]
}
```

### Search Specific Tables

You can specify one or multiple tables. This is less efficient for searches involving more than ~100 tables, as it will display all available columns rather than the default 30 or so. This is used for the "View More" feature on our websites.

#### Request Example

```http
POST https://api.snusbase.com/data/search
Content-Type: application/json
Auth: YOUR_API_KEY_HERE

{
  "terms": ["example"],
  "types": ["username"],
  "tables": ["0001_STEALERLOGS_NA_106M_MALWARE_2023"]
}
```

### Group by Specific Column

Supported columns to group by include:

- `"username"`, `"email"`, `"lastip"`, `"hash"`, `"salt"`, `"password"`, `"name"`, `"_domain"`, `"id"`, `"uid"`, `"phone"`, `"domain"`, `"date"`, `"created"`, `"host"`, `"followers"`, `"updated"`, `"address"`, `"birthdate"`, `"other"`, `"city"`, `"state"`, `"country"`, `"zip"`, `"unparsed"`, `"gender"`, `"company"`, `"language"`, `"url"`, `"job"`, `"regip"`.

If a result doesn't have the column you want to group by, it will be appended to the `NO_{GROUP_BY}` object. For example, if you group by `"email"` but a result doesn't have an email column, it can be found in `NO_EMAIL`.

By default, the value of `group_by` is `"db"`, but you can turn this off by setting `group_by` to `"false"`.

#### Request Example

```http
POST https://api.snusbase.com/data/search
Content-Type: application/json
Auth: YOUR_API_KEY_HERE

{
  "terms": ["example"],
  "types": ["username"],
  "group_by": "_domain"
}
```

### Search with Wildcards

Our wildcard characters are `%` (any number of any characters, including none) and `_` (exactly one character). Wildcard characters cannot be the first character in a search term. If you need domain searches, use the `"_domain"` search type.

You can escape wildcard characters by prepending them with a backslash (`\`).

#### Request Example

```http
POST https://api.snusbase.com/data/search
Content-Type: application/json
Auth: YOUR_API_KEY_HERE

{
  "terms": ["exa____@%.tld"],
  "types": ["email"],
  "wildcard": true
}
```

## Code Examples

### JavaScript

```javascript
const snusbaseAuth = 'YOUR_API_KEY_HERE';
const snusbaseAPI = 'https://api.snusbase.com/';

const sendRequest = async (url, body = null) => {
  const options = {
    method: body ? 'POST' : 'GET',
    headers: {
      'Auth': snusbaseAuth,
      'Content-Type': 'application/json',
    },
    body: body ? JSON.stringify(body) : null,
  };
  const response = await fetch(snusbaseAPI + url, options);
  return await response.json();
};

// Example: Search Snusbase
sendRequest('data/search', {
  terms: ['example@gmail.com'],
  types: ['email'],
}).then(response => console.log(response));

// Example: Get Database Statistics
sendRequest('data/stats').then(response => console.log(response));

// Example: IP WHOIS Lookup
sendRequest('tools/ip-whois', {
  terms: ['12.34.56.78'],
}).then(response => console.log(response));

// Example: Hash Lookup
sendRequest('tools/hash-lookup', {
  terms: ['482c811da5d5b4bc6d497ffa98491e38'],
  types: ['hash'],
}).then(response => console.log(response));
```

### Python

```python
import json
import requests

snusbase_auth = 'YOUR_API_KEY_HERE'
snusbase_api = 'https://api.snusbase.com/'

def send_request(url, body=None):
    headers = {
        'Auth': snusbase_auth,
        'Content-Type': 'application/json',
    }
    method = 'POST' if body else 'GET'
    response = requests.request(method, snusbase_api + url, headers=headers, json=body)
    return response.json()

# Example: Search Snusbase
search_response = send_request('data/search', {
    'terms': ['example@gmail.com'],
    'types': ['email'],
    'wildcard': False,
})
print(search_response)

# Example: Get Database Statistics
stats_response = send_request('data/stats')
print(stats_response)

# Example: IP WHOIS Lookup
ip_whois_response = send_request('tools/ip-whois', {
    'terms': ['12.34.56.78'],
})
print(ip_whois_response)

# Example: Hash Lookup
hash_lookup_response = send_request('tools/hash-lookup', {
    'terms': ['482c811da5d5b4bc6d497ffa98491e38'],
    'types': ['hash'],
})
print(hash_lookup_response)
```

## Error Handling

The API uses standard HTTP status codes to indicate the success or failure of requests.

| Status Code | Meaning                                         |
|-------------|-------------------------------------------------|
| **200 OK**  | Successful request.                             |
| **400 Bad Request** | Invalid input or missing required parameters. |
| **401 Unauthorized** | Invalid or missing API key.            |
| **429 Too Many Requests** | Rate limit exceeded.              |

Detailed error messages are provided in the `errors` array in the response body. These messages are safe to display to end users in your application.

## Rate Limiting

To ensure fair usage and prevent abuse, the API implements rate limiting. Current limits are:

- **`/data/search`**: 2,048 requests every 12 hours.
- **All other endpoints**: 512 requests per minute.

If you exceed these limits, you'll receive a **429 Too Many Requests** response. The response headers include information about your remaining quota and reset time via `X-RateLimit`, `X-RateLimit-Remaining`, and `X-RateLimit-Reset`. These limits are subject to change, so it's advisable to implement these headers in your application logic.

## Important Information

- **API Key Security**: Keep your API key confidential and never expose it in client-side code. You are responsible for your API key, and misuse may result in termination per our [Terms of Service](https://docs.snusbase.com/terms-of-service).

- **Data Sanitization**: Always sanitize outputs from our API before rendering them in your application. We don't modify much of the data we import, so expect HTML and potential exploits aimed at the third-party websites we import to still be present in some datasets.

- **Data Types**: Most results will have column values output as strings. However, in some cases—especially with older datasets—you may encounter inconsistent typing, such as integers for the `"id"` and `"uid"` columns.
