# Snusbase API Documentation

## Table of Contents

1. [Introduction](#1-introduction)
2. [Authentication](#2-authentication)
3. [API Endpoints](#3-api-endpoints)
   - [3.1 Database Search](#31-database-search)
   - [3.2 Database Statistics](#32-database-statistics)
   - [3.3 IP Whois Lookup](#33-ip-whois-lookup)
   - [3.4 Hash Lookup](#34-hash-lookup)
4. [Example Search Queries](#4-example-search-queries)
   - [4.1 Multiple Terms and Types](#41-multiple-terms-and-types)
   - [4.2 Search Specific Tables](#42-search-specific-tables)
   - [4.3 Group by Specific Column](#43-group-by-specific-column)
   - [4.4 Search With Wildcard](#44-search-with-wildcard)
5. [Code Examples](#5-code-examples)
   - [5.1 JavaScript](#51-javascript)
   - [5.2 Python](#52-python)
6. [Error Handling](#6-error-handling)
7. [Rate Limiting](#7-rate-limiting)
8. [Important Information](#8-important-information)

## 1. Introduction

Welcome to the Snusbase API documentation. This guide provides comprehensive information on how to authenticate, use, and interact with our API endpoints.

## 2. Authentication

All API requests require authentication using an activation code. Include your activation code in the `Auth` header of each request.

```
Auth: sb[...]
```

Activation codes generated after September 2021 should start with "sb" followed by 28 random characters.

## 3. API Endpoints

### 3.1 Database Search

Search the Snusbase database for leaked information.

**Endpoint:** `https://api.snusbase.com/data/search`

**Parameters:**
- `terms` (array of strings, required): Search terms
- `types` (array of strings, required): Types of data to search (e.g., "email", "username", "lastip", "password", "hash", "name", "_domain")
- `wildcard` (boolean, optional): Enable wildcard search
- `group_by` (boolean or string, optional): Group results, defaults to "db", can be turned off with "false"
- `tables` (array of strings, optional): Limit search to specific tables

**Request**

```shell
POST https://api.snusbase.com/data/search
Content-Type: application/json
Auth: API_KEY_HERE
{
  "terms": ["example@gmail.com"],
  "types": ["email"]
}
```

**Example Response:**
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
      },
      /* Other results */
    ]
  }
}
```

### 3.2 Database Statistics

Retrieve information about the current databases in the main search engine.

**Endpoint:** `https://api.snusbase.com/data/stats`

**Request**

```shell
GET https://api.snusbase.com/data/stats
```

**Example Response:**
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
    ],
    /* Other tables */
  },
  "features": {
    "view_more": ["0032_CANADA_CA_2M_BUSINESS_2011", /* ... */],
    "hash_lookup": ["0001_HASHES_ORG_1913M_2021", /* ... */],
    "combo_lookup": ["0001_PEMIBLANC_COMBOLIST_245M", /* ... */]
  }
}
```

### 3.3 IP Whois Lookup

Retrieve WHOIS information for IP addresses.

**Endpoint:** `https://api.snusbase.com/tools/ip-whois`

**Parameters:**
- `terms` (array of strings, required): IP addresses to look up

**Request**

```shell
POST https://api.snusbase.com/tools/ip-whois
Content-Type: application/json
Auth: API_KEY_HERE
{
  "terms": ["12.34.56.78", "127.0.0.1"]
}
```

**Example Response:**
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

### 3.4 Hash Lookup

Search for corresponding plaintext passwords or vice versa in the cracked password hash database.

**Endpoint:** `https://api.snusbase.com/tools/hash-lookup`

**Parameters:**
- `terms` (array of strings, required): Hashes or passwords to look up
- `types` (array of strings, required): Types of lookup ("hash" or "password")
- `wildcard` (boolean, optional): Enable wildcard search
- `group_by` (boolean or string, optional): Group results, defaults to "db", can be turned off with "false"
- `tables` (array of strings, optional): Limit search to specific tables

**Request**

```shell
POST https://api.snusbase.com/tools/hash-lookup
Content-Type: application/json
Auth: API_KEY_HERE
{
  "terms":["482c811da5d5b4bc6d497ffa98491e38"],
  "types": ["hash"]
}
```

**Example Response:**
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

## 4. Example Search Queries

### 4.1 Multiple Terms and Types

When using multiple types, each type will be applied to all terms, so you don't need to match them in order.

**URL**
```
POST https://api.snusbase.com/data/search
```

**Body**
```json
{
  "terms": ["example1", "example2"],
  "types": ["username", "email", "lastip", "hash", "password", "name"]
}
```

### 4.2 Search Specific Tables

You can specify one or multiple tables. This is less efficient for searches involving more than ~100 tables as it will display all available columns rather than the default 30 or so. This is what we use for the "View More" feature on our websites.

**URL**
```
POST https://api.snusbase.com/data/search
```

**Body**
```json
{
  "terms": ["example"],
  "types": ["username"],
  "tables": ["0001_STEALERLOGS_NA_106M_MALWARE_2023"]
}
```

### 4.3 Group by Specific Column

Supported columns to group by are "username", "email", "lastip", "hash", "salt", "password", "name", "_domain", "id", "uid", "phone", "domain", "date", "created", "host", "followers", "updated", "address", "birthdate", "other", "city", "state", "country", "zip", "unparsed", "gender", "company", "language", "url", "job", and "regip".

If a result doesn't have the column you want to group by, it will be appended to the NO_{GROUP_BY} object. That means if you group by email, but a result doesn't have an email column, it can be found in "NO_EMAIL".

By default the value of "group_by" is "db", but you can turn this off by setting "group_by" to "false".

**URL**
```
POST https://api.snusbase.com/data/search
```

**Body**
```json
{
  "terms": ["example"],
  "types": ["username"],
  "group_by": "_domain"
}
```

### 4.4 Search With Wildcard

Our wildcard characters are "%", meaning any amount of any characters or none at all, and "_" representing exactly one character. As of now wildcard characters cannot be the first character in a search term. If you need domain searches, use the "_domain" search type.

You can escape wildcard characters by prepending them with a backslash ("\\").

**URL**
```
POST https://api.snusbase.com/data/search
```

**Body**
```json
{
  "terms": ["exa____@%.tld"],
  "types": ["email"],
  "wildcard": true
}
```

## 5. Code Examples

### 5.1 JavaScript

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
    body: body ? JSON.stringify(body) : null
  };
  const response = await fetch(snusbaseAPI + url, options);
  return await response.json();
};

// Example: Search Snusbase
sendRequest('data/search', {
  terms: ["example@gmail.com"],
  types: ["email"]
}).then(response => console.log(response));

// Example: Get Database Statistics
sendRequest('data/stats').then(response => console.log(response));

// Example: IP Whois Lookup
sendRequest('tools/ip-whois', {
  terms: ["12.34.56.78"],
}).then(response => console.log(response));

// Example: Hash Lookup
sendRequest('tools/hash-lookup', {
  terms: ["482c811da5d5b4bc6d497ffa98491e38"],
  types: ["hash"],
}).then(response => console.log(response));
```

### 5.2 Python

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
    data = json.dumps(body) if body else None
    response = requests.request(method, snusbase_api + url, headers=headers, data=data)
    return response.json()

# Example: Search Snusbase
search_response = send_request('data/search', {
    'terms': ["example@gmail.com"],
    'types': ["email"],
    'wildcard': False,
})
print(search_response)

# Example: Get Database Statistics
stats_response = send_request('data/stats')
print(stats_response)

# Example: IP Whois Lookup
ip_whois_response = send_request('tools/ip-whois', {
    'terms': ["12.34.56.78"],
})
print(ip_whois_response)

# Example: Hash Lookup
hash_lookup_response = send_request('tools/hash-lookup', {
    'terms': ["482c811da5d5b4bc6d497ffa98491e38"],
    'types': ["hash"],
})
print(hash_lookup_response)
```

## 6. Error Handling

The API uses standard HTTP status codes to indicate the success or failure of requests. Common status codes include:

- 200 OK: Successful request
- 400 Bad Request: Invalid input or missing required parameters
- 401 Unauthorized: Invalid or missing API key
- 429 Too Many Requests: Rate limit exceeded
- 500 Internal Server Error: Unexpected server error

Detailed error messages are also provided in the form of an errors array in the response body. These don't leak any information and are safe to display to your end users in your application.

## 7. Rate Limiting

To ensure fair usage and to prevent abuse, the API implements rate limiting. Current limits are:

- `/data/search`: 2048 every 12 hours
- All Others: 512 per minute

If you exceed these limits, you'll receive a 429 Too Many Requests response. The response headers include information about your remaining quota and reset time in the form of `x-ratelimit`, `x-ratelimit-remaining` and `x-ratelimit-reset`. These limits are not binding unless you have a separate signed agreement with us, and are subject to change at any time, so implement these headers in your application logic if applicable.

## 8. Important Information

- Keep your API key confidential and never expose it in client-side code. You are responsible for your API key, and abuse may result in termination per our [terms of service](https://docs.snusbase.com/terms-of-service).
- Always escape outputs from our API before rendering them in your application. We don't modify much of the data we import, so expect HTML and potential exploits aimed at the third-party website databases we import to still be present in some of our datasets.
- Most results will have column values output as strings, but in some cases, especially with older datasets, you may see inconsistent typing such as integers for the "id" and "uid" columns, or very rarely other types.
