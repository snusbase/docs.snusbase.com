# Table of Contents

1. [Getting Started](#1-getting-started)
2. [Authentication](#2-authentication)
3. [Endpoints](#3-endpoints)
    - [3.1 Database Search API](#31-database-search-api)
    - [3.2 Database Statistics](#32-database-statistics)
    - [3.3 IP Whois API](#33-ip-whois-api)
    - [3.4 Hash Lookup API](#34-hash-lookup-api)
4. [Code Examples](#4-code-examples)
    - [4.1 JavaScript](#41-javascript)
    - [4.2 Python](#42-python)

# 1. Getting Started

Welcome to our API documentation! In this guide, you will find information on how to authenticate, use, and interact with our API endpoints.

If you'd rather not write your own code, or want some code to start, you can check out some of our examples here or use popular third party software like h8mail (old API).

# 2. Authentication

For regular subscribers of Snusbase.com, you'll need to include an 'Auth' header with your activation code for authentication. Activation codes generated within the past year should start with "sb", followed by 28 random characters.

**Normal API Users**

```
Auth: sb[...]
```

**Paid API Users**

```
Authorization: Bearer API_KEY_HERE
```

# 3. Endpoints

Our API provides the following endpoints:

## 3.1 Database Search API

This endpoint lets you search the Snusbase database for leaked information, like "email", "username", "lastip" (IP address), "password", "hash" (un-cracked/hashed password) and "name".

**Required Parameters:**

* terms (array of strings)
* types (array of strings)

**Optional Parameters:**

* wildcard (boolean: true/false)


**Curl**

```
curl --request POST --url https://api.snusbase.com/data/search \
--header "Auth: ACTIVATION_CODE" --header "Content-Type: application/json" \
--data '{"terms":["example@gmail.com"], "types":["email"], "wildcard": false}'
```

**Request**

```
POST https://api.snusbase.com/data/search
Content-Type: application/json
Auth: API_KEY_HERE
{
  "terms": [ "example@gmail.com" ],
  "types": [ "email" ],
  "wildcard": false
}
```

**Example Response**

```
{
  "took": 291,
  "size": 1038,
  "results": {
    "1669_VERIFICATIONS_IO_789M_MARKETING_022019": [
      {
        "email": "example@gmail.com",
        "lastip": "70.111.255.206",
        "name": "Jason Bourne",
        "phone": "310-833-3021",
        "city": "San Pedro",
        "state": "CA",
        "zip": "90731",
        "gender": "female"
      }
    ],
    "1667_ONLINETRADE_RU_3M_SHOPPING_092022": [
      {
        "email": "example@gmail.com",
        "name": "Сергей"
      }
    ], /* (...) */
  }
}
```

## 3.2 Database Statistics

Get the current databases imported in our main (non-combolist) search engine, as well as the fields/columns that were leaked.

**Curl**

```
curl https://api.snusbase.com/data/stats
```

**Request**

```
GET https://api.snusbase.com/data/stats
```

**Example Response**

```
{
  "rows": 16165424269,
  "tables": {
    "0001_IMGUR_COM_1753K_SOCIAL_092013": [
      "password",
      "email"
    ],
    "0002_BIT_LY_8514K_SOCIAL_052014": [
      "username",
      "email",
      "hash"
    ],
    "0003_KICKSTARTER_COM_5M_FINANCIAL_022014": [
      "hash",
      "email",
      "salt"
    ], /* (...) */
  }
}
```

## 3.3 IP Whois API

**Required Parameters:**

* terms (array of strings)

**Curl**

```
curl --request POST --url https://api.snusbase.com/tools/ip-whois \
--header "Auth: ACTIVATION_CODE" --header "Content-Type: application/json" \
--data '{"terms":["12.34.56.78", "127.0.0.1"]}'
```

**Request**

```
POST https://api.snusbase.com/tools/ip-whois
Content-Type: application/json
Auth: API_KEY_HERE
{
  "terms": [ "12.34.56.78", "127.0.0.1" ]
}
```

**Example Response**

```
{
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

## 3.4 Hash Lookup API

Use this endpoint to search our cracked password hash database for corresponding plaintext passwords or vice versa. It supports the search types "hash" and "password".
Required Parameters:

* terms (array of strings)

**Optional Parameters:**

* types (array of strings)
* wildcard (boolean: true/false)


**Curl**

```
curl --request POST --url https://api.snusbase.com/tools/hash-lookup \
--header "Auth: ACTIVATION_CODE" --header "Content-Type: application/json" \
--data '{"terms":["482c811da5d5b4bc6d497ffa98491e38"]}'
```

**Request**

```
POST https://api.snusbase.com/tools/hash-lookup
Content-Type: application/json
Auth: API_KEY_HERE
{
  "terms":["482c811da5d5b4bc6d497ffa98491e38"]
}
```

**Example Response**

```
{
  "found":true,
  "password":"password123",
  "term":"482c811da5d5b4bc6d497ffa98491e38"
}
```

# 4. Code Examples

Here are some examples on how to use our API:

## 4.1 JavaScript

Very simple code example showing the usage of Snusbase's API with JavaScript.

**Requirements**

If you're using a NodeJS version older than NodeJS 18, you'll need to run:

```
> npm install node-fetch
```

And then add the following line to the very top of the file:

```
const fetch = (...args) => import('node-fetch').then(({default: fetch}) => fetch(...args));
```

**Code**

```
const snusbaseAuth = 'API_KEY_HERE'
const snusbaseAPI = 'https://api.snusbase.com/'

const sendRequest = async function (url, body = false) {
  const options = {
    method: (body) ? 'POST' : 'GET',
    headers: { 
      'Auth': snusbaseAuth,
      'Content-Type': 'application/json',
    },
    body: (body) ? JSON.stringify(body) : null
  }
  const response = await fetch(snusbaseAPI + url, options);
  return await response.json();
}

// Search Snusbase
sendRequest('data/search', {
  terms: ["example@gmail.com"],
  types: ["email"], // can be username, email, lastip, hash, password and/or name
  wildcard: false,
}).then(response => console.log(response))

// Get Snusbase Statistics
sendRequest('data/stats').then(response => console.log(response))

// Get IP Whois Information
sendRequest('tools/ip-whois', {
  terms: ["12.34.56.78"],
}).then(response => console.log(response))

// Search Hash Lookup
sendRequest('tools/hash-lookup', {
  terms: ["5f4dcc3b5aa765d61d8327deb882cf99"],
  types: ["hash"], // can be hash and/or password
}).then(response => console.log(response))
```

**Usage**

To use the code, add it to a file named "snusbase.js" and run this command in your terminal:

```
> node snusbase.js
```

## 4.2 Python

Very simple code example showing the usage of Snusbase's API with Python.

**Code**

```
import json
import requests

snusbase_auth = 'API_KEY_HERE'
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

# Search Snusbase
search_response = send_request('data/search', {
    'terms': ["example@gmail.com"],
    'types': ["email"],  # can be username, email, lastip, hash, password and/or name
    'wildcard': False,
})
print(search_response)

# Get Snusbase Statistics
stats_response = send_request('data/stats')
print(stats_response)

# Get IP Whois Information
ip_whois_response = send_request('tools/ip-whois', {
    'terms': ["12.34.56.78"],
})
print(ip_whois_response)

# Search Hash Lookup
hash_lookup_response = send_request('tools/hash-lookup', {
    'terms': ["5f4dcc3b5aa765d61d8327deb882cf99"],
    'types': ["hash"],  # can be hash and/or password
})
print(hash_lookup_response)
```

**Usage**

To use the code, add it to a file named "snusbase.py" and run this command in your terminal:

```
> python3 snusbase.py
```
