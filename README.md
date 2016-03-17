Prattle API & Analytics Dashboard v0.1: Developer Guide
=================================================

# API Routes

The API offers several url calls to return processed data from the database. See the following tables for detail. Parameters are preceded by a colon (`:`).

## Bearer Tokens

All API calls must carry a bearer token or cookie from the server. To get a token, you must submit a POST request with `application/x-www-form-urlencoded` data form to the url `http://portal.prattle.co/auth/local/`. The POST request should include two parameters: `email` and `password`, where `email` is the account email for the portal and `password` is the account password for the portal.

An example call from the command line would be `curl -X POST -d "email=YOUR@EMAIL.COM&password=YOURPASSWORD" http://portal.prattle.co/auth/local`. This will return the bearer token in the form of JSON. Use the `"token"` key to yield the token from the JSON.

This process is streamlined for all Prattle R and Python wrappers.

## Base URL

Action | url | Notes
-----|------|-------
Base url | `http://portal.prattle.co/api` |

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

To come: calls based on speakers.

# Angular Data Injections Available (Services)

__A note on async callbacks:__ when injecting data in nested views, you may need to have a promise resolution. Therefore, the surefire way to inject is in the format that follows:

```javascript
var banks = Banks.getList().then(function(data){
	// reveal banks to scope
	$scope.banks = data;
	});
```

## Bank Data

Inject the `Bank` service into the controller:

```javascript
northAmericanDangerZone.controller('DashboardController', ['$scope', function($scope, Bank) {
	$scope.message = 'Hello!';
}]);
```

The Banks service has several data injection calls.

### Banks.getScores(bank, startDate, endDate)
Returns documents from bank between two dates.

parameter | specification
-----------|--------------
`bank` | Three-letter bank code. Example: 'boj'.
`startDate` | Beginning of desired time period for query. Format: YYYY-MM-DD, YYYY-MM, or YYYY. Dates specified without days or months are passed as the first month or day (01).
`endDate` | End of desired time period for query. Format: YYYY-MM-DD, YYYY-MM, or YYYY. Dates specified without days or months are passed as the first month or day (01). If left blank, will be treated as current date.

#### Examples:

- Banks.getScores('boj', '2012-01-05', '2012-01-10');
	- Will return all docs between 2012-01-05 and 2012-01-10.
- Banks.getScores('boj', '2012-01-15');
	- Will return all docs between 2012-01-15 and the current day.
- Banks.getScores('boj', '2012-02', '2015);
	- Will return all docs between 2012-02-01 and 2015-12-31.
- Banks.getScores('boj');
	- Will return all docs from beginning of time for bank..

### Banks.getList()
Returns list of all available banks.

### Banks.getInfo(bank)
Returns info on specific three-letter-code bank.

parameter | specification
-----------|--------------
`bank` | Three-letter bank code. Example: 'boj'.

### Banks.getRecentDocs(bank, numberOfDocs)
Returns `numberOfDocs` most recent documents (counting backwards from current timestamp) for `bank`.

parameter | specification
-----------|--------------
`bank` | Three-letter bank code. Example: 'boj'.
`numberOfDocs` | Integer of the number of documents you would like to request.

## Document Data

### Documents.getInfo(documentId)

parameter | specification
-----------|--------------
`documentId` | Mongo Hash ID of document.

### Documents.getRecent(numberOfDocs)
Returns `numberOfDocs` recent documents from all documents, regardless of bank.

parameter | specification
-----------|--------------
`numberOfDocs` | Integer of the number of documents you would like to request.

### Documents.fromSpeaker(speakerName, numberOfDocs)

parameter | specification
-----------|--------------
`speakerName` | String with speaker's full name. Need to better map out how to do this. Will likely be a mapping of the person's name or short-name to an integer, which then is used to query the DB.)
`numberOfDocs` | Integer of the number of documents you would like to request. Returned counting backwards from current timestamp.


## Speakers

### Speakers.getInfo(speakerName)

parameter | specification
-----------|--------------
`speakerName` | String with speaker's full name. Need to better map out how to do this. Will likely be a mapping of the person's name or short-name to an integer, which then is used to query the DB.)

## User Authentication

### Auth.getCurrentUser()

Returns details on current user. Data map:

```javascript
var UserSchema = new Schema({
  name: String,
  email: { type: String, lowercase: true },
  role: {
	type: String,
	default: 'user'
  },
  hashedPassword: String,
  provider: String,
  salt: String,
  company: String,
  contact: {
	first: String,
	last: String,
	phone: Number
  },
  access: {
	banks: []
  },
  subscription_start: Date,
  subscription_end: Date,
  api_key: String
  });
```


# Auth Creds

User login using email and password. Server validates login credentials. If successful, returns JSON web token, signed by app secret, for 5-hour session. Can use this token for anything the user is authorized to do. This includes using the token as a bearer token for CURL requests and pRattle package use.

# Other factories

## Loess

Helper tools to run LOESS.

### Loess.run(data, bandwidth)

Takes data formed as an array of arrays, e.g. `[ [x1,y1], [x2,y2], ... , [xi, yi] ]`. Specify bandwidth as a decimal (usually within the range `0.1` to `0.5`). Returns an array of arrays with LOESS values, matched to original x-values.

Call with `Loess.run(data, bandwidth)` after injecting the factory `Loess`.

## Classify

Helper tools to classify a document as Hawkish, Neutral, or Dovish.

### Classify.detect(score)

Takes `score` as a number. Returns `Hawkish`, `Neutral`, or `Dovish`.

### Classify.color(score)

Takes `score` as a number. Returns the color property `alert` if Hawkish, nothing if Neutral (defaults to the baby blue), and `secondary` (dark blue) if Dovish.
