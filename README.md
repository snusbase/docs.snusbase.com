# Snusbase API Docs

## Table of Contents

1. [Introduction](#introduction)
2. [Authentication](#authentication)
3. [API Endpoints](#api-endpoints)
   - [Database Statistics](#database-statistics)
   - [Database Search](#database-search)
   - [Combo Lookup](#combo-lookup)
   - [Hash Lookup](#hash-lookup)
   - [IP WHOIS Lookup](#ip-whois-lookup)
   - [Domain WHOIS Lookup](#domain-whois-lookup)
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

### Database Statistics

Retrieve information about the current databases in the main search engine. This endpoint does not require authentication.

- **Endpoint:** `https://api.snusbase.com/data/stats`
- **Method:** `GET`

#### Request Example

```http
GET https://api.snusbase.com/data/stats
```

#### Response Example

```json
{
  "rows": 18109138413,
  "tables": {
    "0001_STEALERLOGS_NA_121M_MALWARE_2023": [
      "email", "username", "password", "host", "_domain"
    ],
    /* Other tables */
  },
  "features": {
    "view_more": [
      "0005_ZING_VN_51M_ENTERTAINMENT_052015",
      /* ... */
    ],
    "combo_lookup": [
      "0001_PEMIBLANC_COMBOLIST_245M_2018",
      /* ... */
    ]
  }
}
```

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
  "took": 31.714,
  "size": 1233,
  "results": {
    "2123_BREACHFORUMS_BF_323K_HACKING_012026": [
      {
        "username": "avvd",
        "email": "example@gmail.com",
        "lastip": "127.0.0.9",
        "hash": "$argon2i$v=19$m=65536,t=4,p=1$NVphazc1SUg3YUVxNFV3Nw$EmshtvIzcwhG8lnDTsl0XvzCyg7h8k+Qr3tgTQihvZI",
        "salt": "Wlwirmmh",
        "uid": "331153",
        "created": "1729340505",
        "updated": "1729341708"
      }
    ],
    /* Other results.. */
  }
}
```

### Combo Lookup

Search the combolist database for username/password combinations from large credential dumps.

- **Endpoint:** `https://api.snusbase.com/tools/combo-lookup`
- **Method:** `POST`
- **Headers:**
  - `Content-Type: application/json`
  - `Auth: YOUR_API_KEY_HERE`

#### Parameters

| Parameter  | Type                | Required | Description                                                                                                                |
|------------|---------------------|----------|----------------------------------------------------------------------------------------------------------------------------|
| `terms`    | Array of strings    | Yes      | Search terms (usernames or passwords).                                                                                     |
| `types`    | Array of strings    | Yes      | Types of data to search. Possible values: `"username"`, `"password"`.                                                      |
| `wildcard` | Boolean             | No       | Enable wildcard search (`true` or `false`).                                                                                |
| `group_by` | Boolean or string   | No       | Group results. Defaults to `"db"`. Set to `false` to disable grouping.                                                     |

#### Request Example

```http
POST https://api.snusbase.com/tools/combo-lookup
Content-Type: application/json
Auth: YOUR_API_KEY_HERE

{
  "terms": ["example@gmail.com"],
  "types": ["username"]
}
```

#### Response Example

```json
{
  "took": 2.905,
  "size": 1194,
  "results": {
    "COLLECTION1_1212M_COMBOLIST_2019": [
      {
        "username": "example@gmail.com",
        "password": "0981122847"
      },
      {
        "username": "example@gmail.com",
        "password": "123456"
      },
      /* Other results.. */
    ],
    /* Other combolists */
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
| `group_by` | Boolean or string   | No       | Group results. Defaults to `"db"`. Set to `false` to disable grouping.                                                                                            |

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
  "took": 0.102,
  "size": 1,
  "results": {
    "HASHES": [
      {
        "hash": "482c811da5d5b4bc6d497ffa98491e38",
        "password": "password123"
      }
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

| Parameter  | Type              | Required | Description                                                                                                                                                                                                          |
|------------|-------------------|----------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `terms`    | Array of strings  | Yes      | IP addresses to look up.                                                                                                                                                                                              |
| `group_by` | Boolean or string | No       | When omitted, results are an object keyed by the IP. Set to any field name (e.g. `"country"`, `"isp"`, `"as"`) to get arrays of records grouped by that value, with a `NO_{FIELD}` bucket for records missing it. Set to `false` for a flat array. |

#### Request Example

```http
POST https://api.snusbase.com/tools/ip-whois
Content-Type: application/json
Auth: YOUR_API_KEY_HERE

{
  "terms": ["12.34.56.78"]
}
```

#### Response Example

```json
{
  "took": 6.4,
  "size": 1,
  "results": {
    "12.34.56.78": {
      "query": "12.34.56.78",
      "continent": "North America",
      "continentCode": "NA",
      "country": "United States",
      "countryCode": "US",
      "region": "OH",
      "regionName": "Ohio",
      "city": "Columbus",
      "zip": "43215",
      "lat": 39.9612,
      "lon": -82.9988,
      "timezone": "America/New_York",
      "isp": "AT&T Enterprises, LLC",
      "org": "AT&T Enterprises, LLC",
      "as": "AS7018 AT&T Enterprises, LLC",
      "asname": "ATT-INTERNET4",
      "mobile": false,
      "proxy": false,
      "hosting": false
    }
  }
}
```

### Domain WHOIS Lookup

Retrieve WHOIS/RDAP information for domain names. Works with gTLDs, ccTLDs, and IDN domains.

- **Endpoint:** `https://api.snusbase.com/tools/domain-whois`
- **Method:** `POST`
- **Headers:**
  - `Content-Type: application/json`
  - `Auth: YOUR_API_KEY_HERE`

#### Parameters

| Parameter  | Type              | Required | Description                                                                                                                                                                          |
|------------|-------------------|----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `terms`    | Array of strings  | Yes      | Domain names to look up (max 100 per request).                                                                                                                                                                                                      |
| `group_by` | Boolean or string | No       | When omitted, results are an object keyed by the input. Set to any field path (e.g. `"tld"`, `"registrar.name"`, `"registrant.org"`, `"meta.source"`) to get arrays grouped by that value, with a `NO_{FIELD}` bucket for records missing it. Dotted paths walk into nested objects. Set to `false` for a flat array. |

#### Request Example

```http
POST https://api.snusbase.com/tools/domain-whois
Content-Type: application/json
Auth: YOUR_API_KEY_HERE

{
  "terms": ["example.org", "presence.sh", "not8489291real.com"]
}
```

#### Response Example

```json
{
  "took": 264.706,
  "size": 2,
  "results": {
    "example.org": {
      "domain": "example.org",
      "tld": "org",
      "date": {
        "created": "1995-08-31T04:00:00Z",
        "expires": "2026-08-30T04:00:00Z",
        "updated": "2026-01-16T15:54:51.136Z"
      },
      "nameserver": ["katelyn.ns.cloudflare.com", "mitch.ns.cloudflare.com"],
      "registrar": {
        "name": "icann",
        "handle": "376"
      },
      "status": ["serverdeleteprohibited", "servertransferprohibited", "serverupdateprohibited"],
      "contact": {
        "abuse": {
          "email": "cbath@pir.org"
        }
      },
      "meta": {
        "checked": "2026-06-28T14:48:30Z",
        "updated": "2026-06-20T16:47:39Z",
        "seen_czds": "2026-06-20T16:47:39Z",
        "sources": ["czds", "rdap", "dns"]
      },
      "dns": {
        "security": { "signed": false },
        "records": [
          { "type": "A", "name": "example.org", "ttl": 113, "values": ["104.20.26.136", "172.66.157.237"] },
          { "type": "AAAA", "name": "example.org", "ttl": 129, "values": ["2606:4700:10::6814:1a88", "2606:4700:10::ac42:9ded"] },
          { "type": "MX", "name": "example.org", "ttl": 300, "values": [{ "priority": 0, "server": "." }] },
          { "type": "TXT", "name": "example.org", "ttl": 300, "values": ["_uyh0vgv5ukyofwj3ddwspnni01vmlyy", "v=spf1 -all"] },
          { "type": "A", "name": "www.example.org", "ttl": 300, "values": ["104.20.26.136", "172.66.157.237"] },
          { "type": "AAAA", "name": "www.example.org", "ttl": 300, "values": ["2606:4700:10::6814:1a88", "2606:4700:10::ac42:9ded"] }
        ]
      }
    },
    "presence.sh": {
      "domain": "presence.sh",
      "tld": "sh",
      "date": {
        "created": "2026-01-09T15:22:13Z",
        "expires": "2027-01-09T15:22:13Z",
        "updated": "2026-01-14T15:22:58Z"
      },
      "nameserver": ["colin.ns.cloudflare.com", "dayana.ns.cloudflare.com"],
      "registrar": {
        "name": "immaterialism limited",
        "handle": "802672"
      },
      "status": ["ok"],
      "contact": {
        "owner": {
          "organization": "njalla okta llc",
          "address": "kn"
        }
      },
      "meta": {
        "checked": "2026-06-28T14:48:17Z",
        "updated": "2026-06-23T10:10:10Z",
        "sources": ["whois", "dns"]
      },
      "dns": {
        "security": { "signed": false },
        "records": [
          { "type": "A", "name": "presence.sh", "ttl": 300, "values": ["104.21.46.130", "172.67.139.8"] },
          { "type": "AAAA", "name": "presence.sh", "ttl": 300, "values": ["2606:4700:3033::6815:2e82", "2606:4700:3033::ac43:8b08"] }
        ]
      }
    }
  },
  "errors": ["not8489291real.com: Likely not registered"]
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

You can specify one or multiple tables. This will display all available columns rather than the default set. This is used for the "View More" feature on our websites.

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

If a result doesn't have the column you want to group by, it will be appended to the `NO_{GROUP_BY}` object. For example, if you group by `"email"` but a result doesn't have an email column, it can be found in `NO_EMAIL`.

By default, the value of `group_by` is `"db"`, but you can turn this off by setting `group_by` to `false`.

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

Our wildcard characters are `%` (any number of any characters, including none) and `?` (exactly one character). Wildcard characters cannot be the first character in a search term. If you need domain searches, use the `"_domain"` search type.

You can escape wildcard characters by prepending them with a backslash (`\`).

#### Request Example

```http
POST https://api.snusbase.com/data/search
Content-Type: application/json
Auth: YOUR_API_KEY_HERE

{
  "terms": ["exa????@%.tld"],
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

// Example: Get Database Statistics
sendRequest('data/stats').then(response => console.log(response));

// Example: Search Snusbase
sendRequest('data/search', {
  terms: ['example@gmail.com'],
  types: ['email'],
}).then(response => console.log(response));

// Example: Combo Lookup
sendRequest('tools/combo-lookup', {
  terms: ['example@gmail.com'],
  types: ['username'],
}).then(response => console.log(response));

// Example: Hash Lookup
sendRequest('tools/hash-lookup', {
  terms: ['482c811da5d5b4bc6d497ffa98491e38'],
  types: ['hash'],
}).then(response => console.log(response));

// Example: IP WHOIS Lookup
sendRequest('tools/ip-whois', {
  terms: ['12.34.56.78'],
}).then(response => console.log(response));

// Example: Domain WHOIS Lookup
sendRequest('tools/domain-whois', {
  terms: ['google.com', 'bbc.co.uk'],
}).then(response => console.log(response));
```

### Python

```python
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

# Example: Get Database Statistics
stats_response = send_request('data/stats')
print(stats_response)

# Example: Search Snusbase
search_response = send_request('data/search', {
    'terms': ['example@gmail.com'],
    'types': ['email'],
})
print(search_response)

# Example: Combo Lookup
combo_lookup_response = send_request('tools/combo-lookup', {
    'terms': ['example@gmail.com'],
    'types': ['username'],
})
print(combo_lookup_response)

# Example: Hash Lookup
hash_lookup_response = send_request('tools/hash-lookup', {
    'terms': ['482c811da5d5b4bc6d497ffa98491e38'],
    'types': ['hash'],
})
print(hash_lookup_response)

# Example: IP WHOIS Lookup
ip_whois_response = send_request('tools/ip-whois', {
    'terms': ['12.34.56.78'],
})
print(ip_whois_response)

# Example: Domain WHOIS Lookup
domain_whois_response = send_request('tools/domain-whois', {
    'terms': ['google.com', 'bbc.co.uk'],
})
print(domain_whois_response)
```

## Error Handling

The API uses standard HTTP status codes to indicate the success or failure of requests.

| Status Code | Meaning                                         |
|-------------|-------------------------------------------------|
| **200 OK**  | Successful request.                             |
| **400 Bad Request** | Invalid input or missing required parameters. |
| **401 Unauthorized** | Invalid, expired, or missing API key.  |
| **429 Too Many Requests** | Rate limit exceeded.              |

Detailed error messages are provided in the `errors` array in the response body. These messages are safe to display to end users in your application.

## Rate Limiting

To ensure fair usage and prevent abuse, the API implements rate limiting. Current limits are:

- **`/data/search`**: 2,048 requests every 12 hours.
- **`/tools/combo-lookup`**: 4,096 requests every 12 hours.
- **All other endpoints**: 256 requests every 2 minutes.

If you exceed these limits, you'll receive a **429 Too Many Requests** response. The following response headers are included on all authenticated requests:

| Header                    | Description                                    |
|---------------------------|------------------------------------------------|
| `X-Rate-Limit`           | Maximum number of requests allowed in the current window. |
| `X-Rate-Limit-Remaining` | Number of requests remaining in the current window.       |
| `X-Rate-Limit-Reset`     | Seconds until the current rate limit window resets.       |

These limits are subject to change, so it's advisable to implement these headers in your application logic.

## Important Information

- **API Key Security**: Keep your API key confidential and never expose it in client-side code. You are responsible for your API key, and misuse may result in termination per our [Terms of Service](https://docs.snusbase.com/terms-of-service).

- **Data Sanitization**: Always sanitize outputs from our API before rendering them in your application. We don't modify much of the data we import, so expect HTML and potential exploits aimed at the third-party websites we import to still be present in some datasets.
