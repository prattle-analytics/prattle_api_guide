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

All API calls must carry a bearer token or cookie from the server. To get a token, you must submit a POST request with `application/x-www-form-urlencoded` data form to the url `https://api-prattle.liquidnet.com/auth/local`. The POST request should include two parameters: `email` and `password`, where `email` is the account email for the portal and `password` is the account password for the portal.

An example call from the command line would be `curl -X POST -d "email=YOUR@EMAIL.COM&password=YOURPASSWORD" https://portal.prattle.co/auth/local`. This will return the bearer token in the form of JSON. Use the `"token"` key to yield the token from the JSON.

In order to return results, you must add in the bearer token when you send `GET` requests with your query. In curl, this is accomplished by adding `-H "Authorization: Bearer token_you_received"`. A quick curl example would look like this:
```
curl "https://equities.prattle.co/api/events/?symbol=AAPL&between=2016|2017" -H "Authorization: Bearer token_you_received"
```

## Bearer Tokens in Python

If you plan on using the requests package, you need to request authorization, store the result, and add it to the header in the next `GET` request. For example:

```python
import requests
import json

url = 'https://api-prattle.liquidnet.com/auth/local'
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
Base url | `https://equities.prattle.co/api/events`

## Events API documentation

Action | Query string | Notes
-----|------|-------
Get all earnings events from an organization. | `symbol={symbol}` | `{symbol}` is organization's stock exchange ticker symbol, in all caps. for example `symbol=AAPL`.
Get all events from all organizations. | `symbol=` | When the previous query is passed without querying for a specific ticker, the query returns the previous 12 months of events from _active_ companies.
Get all events from an organization after (or before) certain date. | `after={date}` | `{date}` is the start date of your desired time period. Acceptable date formats: `YYYY-MM-DD`, `YYYY-MM`, and `YYYY`. `before={date}` can be used as well.
Get all events from an organization between certain dates. | `between={lowdate}\|{highdate}` | `{lowDate}` is the beginning of your desired time period. `{highDate}` is the end of your desired time period. Separated by a "|" pipe. Acceptable date formats: `YYYY-MM-DD`, `YYYY-MM`, and `YYYY`.
Get events of a particular type. | `type={type}` | `{type}` is a variable with acceptable values of `ecall`, `pr`, `speech`, `pr,speech`,  and `any`. When omitted, the default is `ecall`.

Query strings are preceded by a `?` in the url, and are joined with an `&`. A full examples of the url to pass would be `full example: https://equities.prattle.co/api/events/?symbol=AAPL&between="2016"|"2017"` for a time slice (quote marks around the date are optional) of all earnings calls from Apple in 2016.

### Earnings Call, Speech, and Press Release Sample JSON

Here is a sample of the api's JSON response for an earnings call:
```json
{
      "id": "132397",
      "symbol": "AAPL",
      "datetime_utc": "2016-10-25T21:00:00.000Z",
      "fiscal_quarter": 4,
      "fiscal_year": 2016,
      "oos": true,
      "score": -1.5601837336579774,
      "score_car": -4.53
}
```

Query results for press releases, and speeches are similarly structured.

## Query 10-K and 10-Qs

Under the 0.1 version of the API, results for regulatory filings can be queried by directly requesting a companies filing results utilizing an alternate identifier (not the primary ticker symbol). The filings are retrieved with the query `kq?factset_entity_id={factset_entity_id}`. No other query methods work in coordination with this query, meaning that the entire set of filings is retrieved each time the query is executed.

The field `{factset_entity_id}` can be retrieved by querying for the events of any given company. Programmatically, a user would first, retrieve results from a company, and cache the `{factset_entity_id}` from that set of results, and then use it to query for the regulatory filing information. The `{factset_entity_id}` to `{primary_symbol}` mapping changes infrequently, and can be cached for long term use.<sup>[1](#myfootnote1)</sup>


### 10-K/10-Q Sample JSON

A sample query for Apple would be structured thusly: `https://equities.prattle.co/api/events/kq?factset_entity_id=000C7F-E`. The result is a JSON array, with each element of the array being a JSON object with five key-value pairs. For example, from our Apple query:

```json
[{"date": "2019-05-01T20:44:57",
  "factset_entity_id": "000C7F-E",
  "score": 0.0,
  "type": "10-Q",
  "url": "https://www.sec.gov/Archives/edgar/data/000320193000032019319000066/0000320193-19-000066-index.htm"},
  ...
  {"date": "1999-02-08T05:00:00",
  "factset_entity_id": "000C7F-E",
  "score": 0.0,
  "type": "10-Q",
  "url": "https://www.sec.gov/Archives/edgar/data/000320193000032019399000002/0000320193-99-000002-index.htm"}]
```



### Example request in Python
To pass the url from the above `AAPL` example, you can add the following code to the first python example.
```python
# This is a pretty print package, just to make the output appear nicer.
# You can replace the pprint with a regular print.
import pprint

base_url = 'https://equities.prattle.co/api/events'
query_url = base_url + '/?symbol=AAPL&between="2016"|"2017"'

r = requests.get(query_url, headers=hdr)

# loop over the objects from the query for printing
for ob in json.loads(r.text):
    pprint.pprint(ob)
```

## Events with full component scores API Reference
Earnings calls are separated into components defined by individuals speaking, and Prattle supplies the scores for these.
These can be viewed, in aggregate form, on a page of an specific call. The set of component scores for a call can be queried via api.
This also provides a variety of addtional data points about the call.

Action | url | Notes
-----|------|-------
Queries for full events, including component scores, must contain an id. For multiple results, programmatically loop over a returned list of events from the base /events/ endpoint which can return a (more concise) list. | `/events/full?id={id}` | `{id}` is the event's id, for example `https://equities.prattle.co/api/events/full?id=260496`

### Sample JSON from a call.

```json
{  
   "access":true,
   "eps":null,
   "revenue":null,
   "mean_eps_estimate":null,
   "mean_rev_estimate":null,
   "id":"260496",
   "oos":true,
   "score":0.0656755641102791,
   "datetime_utc":"2018-02-01T22:00:00.000Z",
   "prattle_score_datetime_utc":"2018-02-02T00:33:10.000Z",
   "factset_upload_datetime_utc":"2018-02-02T00:17:35.000Z",
   "score_valid_datetime_utc":"2018-02-02T00:33:10.000Z",
   "score_rounded":0.07,
   "factset_entity_id":"000C7F-E",
   "fiscal_quarter":1,
   "fiscal_year":2018,
   "symbol":"AAPL",
   "company":"Apple Inc.",
   "prev":"257286",
   "next":null,
   "evt_pctl":54,
   "qa":[  
      {  
         "name":"Timothy Donald Cook",
         "term_count":95,
         "factset_speaker_id":"05F520-E",
         "comp_id":54227090,
         "id":"05F520-E",
         "title":"Chief Executive Officer & Director",
         "speaker_type":"corprep",
         "factset_entity_affiliation_id":"000C7F-E",
         "company":"Apple, Inc.",
         "speaker_score":-0.01,
         "weight":20.01
      },
      {  
         "name":"Luca Maestri",
         "term_count":11,
         "factset_speaker_id":"07F5D0-E",
         "comp_id":54227092,
         "id":"07F5D0-E",
         "title":"Senior Vice President & Chief Financial Officer",
         "speaker_type":"corprep",
         "factset_entity_affiliation_id":"000C7F-E",
         "company":"Apple, Inc.",
         "speaker_score":0,
         "weight":12.17
      },
      ...
         {  
         "name":"Nancy Paxton",
         "term_count":14,
         "factset_speaker_id":"07VGY3-E",
         "comp_id":54227041,
         "id":"07VGY3-E",
         "title":"Senior Director, Investor Relations and Treasury",
         "speaker_type":"corprep",
         "factset_entity_affiliation_id":"000C7F-E",
         "company":"Apple, Inc.",
         "speaker_score":0.01,
         "weight":3.2
      }
   ]
}
```

<a name="myfootnote1">1</a>: This method will be updated in future iterations of the API, to streamline the query process.
