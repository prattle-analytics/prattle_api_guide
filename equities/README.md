Prattle Equities API & Analytics Dashboard v0.1: Developer Guide
=================================================

This guide describes the utilization of the API for Prattle Earnings Call data, as well as give detail on how to organize a data 
harvesting system to keep new data from Prattle organized and up to date. Any data harvest system will consist of two parts: establishing 
an archive based of our historical record files, and then utilizing our api to regularly query for new releases. This guide will also give
documentation on our API.

A general note: our systems are geared towards displaying _active_ companies. Companies that are no longer public and actively traded we 
generally referred to as a "dead ticker". This is most evident in the portal. Currently dead tickers are not available via API and are only available
in the archival flat files. These companies will be available to query in future deployments of the API that are, at the moment, in development (as of 3/23/18).

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
      data={'email': email, 'password': password},
	headers=hdr
)
auth = r.json()

# for use in further queries add the token to the header
hdr['Authorization'] = 'Bearer ' + auth['token']
```
You would reuse the `auth` object and add it to the previously defined header.

## Base URL

Action | url 
-----|------
Base url | `https://equities.prattle.co/api/` 

## Event Sample JSON
Here is a sample of the api's JSON response for an earnings call:
```json
{
      id: "132397",
      symbol: "AAPL",
      datetime_utc: "2016-10-25T21:00:00.000Z",
      fiscal_quarter: 4,
      fiscal_year: 2016,
      oos: true,
      score: -1.5601837336579774,
      score_car: -4.53
}
```

## Events API documentation
Action | url | Notes
-----|------|-------
Get all events from all organizations. | `/events/` | Events, passed without parameters, returns the previous 12 months of events from _active_ companies.	
Get all events from an organization. | `/events/?symbol={symbol}` | `{symbol}` is organization's stock exchange ticker symbol, in all caps. for example `symbol=AAPL`.
Get all events from an organization after (or before) certain date. | `/events/?symbol={symbol}&after={date}	` | `{symbol}` is organization's stock exchange code, in all caps. `{date}` is the start date of your desired time period. Acceptable date formats: `YYYY-MM-DD`, `YYYY-MM`, and `YYYY`.
Get all events from an organization between certain dates. | `/events/?symbol={symbol}&between={lowdate}|{highdate}` | `{symbol}` is organization's stock exchange code, in all caps. `{lowDate}` is the beginning of your desired time period. `{highDate}` is the end of your desired time period. Seperated by a "|" pipe. Acceptable date formats: `YYYY-MM-DD`, `YYYY-MM`, and `YYYY`.

A full examples of the url to pass would be `full example: https://equities.prattle.co/api/events/?symbol=AAPL&between="2016"|"2017"` for a time slice (quote marks around the date are optional).

# Sample request in Python
To pass the url from the above `AAPL` example, you can add the following code to the first python example.
```python
# This is a pretty print package, just to make the output appear nicer. You can replace the pprint with a regular print.
import pprint

query_url = base_url + 'events/?symbol=AAPL&between="2016"|"2017"'

r = requests.get(query_url, headers=hdr)

# loop over the objects from the query for printing
for ob in json.loads(r.text):
    pprint.pprint(ob)
```

