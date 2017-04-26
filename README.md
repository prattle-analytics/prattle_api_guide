Prattle API & Analytics Dashboard v0.1: Developer Guide
=================================================

# API Routes

The API offers several url calls to return processed data from the database. See the following tables for detail. Parameters are preceded by a colon (`:`).

## Bearer Tokens

All API calls must carry a bearer token or cookie from the server. To get a token, you must submit a POST request with `application/x-www-form-urlencoded` data form to the url `https://portal.prattle.co/auth/local/`. The POST request should include two parameters: `email` and `password`, where `email` is the account email for the portal and `password` is the account password for the portal.

An example call from the command line would be `curl -X POST -d "email=YOUR@EMAIL.COM&password=YOURPASSWORD" https://portal.prattle.co/auth/local`. This will return the bearer token in the form of JSON. Use the `"token"` key to yield the token from the JSON.

This process is streamlined for all Prattle R and Python wrappers.

## Base URL

Action | url | Notes
-----|------|-------
Base url | `https://portal.prattle.co/api` |

## Bank API

Action | url | Notes
-----|------|-------
Get list of banks available | `/banks` |
Get information on a bank by id | `/banks/:id` | `:id` is three-letter code.

## Document API

Action | url | Notes
-----|------|-------
Get document by id | `/documents/:id` |
Get all documents from bank | `/documents/bank/:bank` | `:bank` is three-letter code. This is used for the R package.
Get all documents from bank between two dates | `/documents/bank/:bank/:lowDate/:highDate` |  `:bank` is three-letter code. `:lowDate` is the beginning of your desired time period. `:highDate` is the end of your desired time period. Acceptable date formats: `YYYY-MM-DD`, `YYYY-MM`, `YYYY`.
Get all documents from bank since date | `/documents/bank/:bank/:lowDate` |  `:bank` is three-letter code. `:lowDate` is the beginning of your desired time period. Acceptable date formats: `YYYY-MM-DD`, `YYYY-MM`, `YYYY`.
Get a number of recent documents from bank | `/documents/recent/bank/:bank/:numberOfDocs` | `:bank` is three-letter code. `:numberOfDocs` is an integer value.
Gets a number of recent documents from across all banks | `documents/recent/:numberOfDocs` |  `:numberOfDocs` is an integer value.

## Python example with requests

```python
import requests
import json

def getBankData(bank):
    # reqests lib documentation: http://docs.python-requests.org/en/latest/user/quickstart/
    # get json token
    auth_info = {'email': '<your email>', 'password': '<your account password>'}
    r = requests.post("https://portal.prattle.co/auth/local/", params=auth_info)

    # convert to json
    auth_token = json.loads(r.text)

    # get token by key
    auth_token = auth_token['token']

    # get bank info from api using bearer token
    url = 'https://portal.prattle.co/api/documents/bank/{0}'.format(bank)
    bearer_token = 'Bearer ' + auth_token
    headers = {'Authorization': bearer_token}
    print(headers)
    r = requests.get(url, headers=headers)

    # return bank data as json
    bank_data = json.loads(r.text)
    return bank_data

data = getBankData('frc')
print data
```

## R wrapper

Our R package, `pRattle`, [is available on github here.](https://github.com/prattle-analytics/pRattle)

THE CODE AND ALL OTHER INFORMATION OF PRATTLE ANALYTICS, LLC WHICH MAY BE PROVIDED OR MADE AVAILABLE TO YOU IS PROVIDED “AS/IS” WITHOUT ANY WARRANTY OR REPRESENTATION OF ANY KIND.  PRATTLE ANALYTICS, LLC EXPRESSLY DISCLAIMS ALL OTHER WARRANTIES OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING: WARRANTIES OF MERCHANTABILITY, SUITABILITY, INTEGRATION, CURRENTNESS, ACCURACY, AND FITNESS FOR A PARTICULAR OR GENERAL PURPOSE. PRATTLE ANALYTICS, LLC SHALL HAVE NO LIABILITY TO YOU AS A RESULT OF ITS PROVIDING THE CODE, WHETHER FOR LOST PROFITS, LOSS OF DATA, WORK STOPPAGE, CONSEQUENTIAL, EXEMPLARY, SPECIAL, INDIRECT, INCIDENTAL OR PUNITIVE DAMAGES, EVEN IF IT HAS BEEN ADVISED OF THE POSSIBILITY OF SUCH DAMAGES.
