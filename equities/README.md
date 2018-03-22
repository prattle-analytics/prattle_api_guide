Prattle Equities API & Analytics Dashboard v0.1: Developer Guide
=================================================

This guide describes the utilization of the API for Prattle Earnings Call data, as well as give detail on how to organize a data 
harvesting system to keep new data from Prattle organized and up to date. Any data harvest system will consist of two parts: establishing 
an archive based of our historical record files, and then utilizing our api to regularly query for new releases. This guide will also give
documentation on our API.

# Archive

Prattle maintains a series of flat files for our various data products. If you need access, please reach out to your Prattle representative.
The archive files will also be accompanied by a codebook.

# API

The API offers several url calls to return processed data from the database. See the following tables for detail. Parameters are preceded by a colon `:`.

## Bearer Tokens

All API calls must carry a bearer token or cookie from the server. To get a token, you must submit a POST request with `application/x-www-form-urlencoded` data form to the url `https://equities.prattle.co/auth/local/`. The POST request should include two parameters: `email` and `password`, where `email` is the account email for the portal and `password` is the account password for the portal.

An example call from the command line would be `curl -X POST -d "email=YOUR@EMAIL.COM&password=YOURPASSWORD" https://portal.prattle.co/auth/local`. This will return the bearer token in the form of JSON. Use the `"token"` key to yield the token from the JSON.

## Bearer Tokens in Python

If you plan on using the requests package, you need to request authorization, store the result, and add it to the header in the next `get` request. For example:

```python
import requests
import json

url = 'https://equities.prattle.co/auth/local/'
email = 'your account id'
password = 'your password'


hdr = {'Accept': 'application/json'}

r = requests.post(url,
			data={
				'email': email,
				'password': password
			},
			headers=hdr
		)
auth = r.json()
```
You would reuse the `auth` object and add it to the previously defined header, i.e. `hdr['Authorization'] = 'Bearer ' + auth['token']`.

## Base URL

Action | url | Notes
-----|------|-------
Base url | `https://equities.prattle.co/api/events` | Earnings calls are available under `events`.