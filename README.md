## The FormAssembly OAuth2 API

### Introduction:

FormAssembly exposes an [OAuth2](http://oauth.net/2/) authenticated API.  Using this API, you can write applications which will interact with the accounts of users who grant your application permission.  OAuth interactions can be complicated, so if you're not familiar with OAuth2, please read a document such as [this one from Google](https://developers.google.com/accounts/docs/OAuth2) explaining how OAuth2 works in detail.

The key point of OAuth2 is that it allows a 'user' of FormAssembly, let's call them Adam, to authorize you (a 'client') to take actions in FormAssembly as if you were Adam.  As an example, you could write an application which when authorized by Adam, could download and display a list of all of Adam's forms in FormAssembly along with their associated number of responses, drop-out rate, etc.  This data could then be used by you to generate custom metrics for Adam, or stored for later use, etc.

### Behavior Outline:

1. Redirect the user to the authorization url:
https://app.formassembly.com/oauth/login?type=web&client_id=CLIENT_ID&redirect_uri=RETURN_URL&response_type=code where:
    + CLIENT_ID is the unique client id generated by FormAssembly for your client application
    + RETURN_URL is the url your user ('Adam') will be returned to when they complete the authorization step on FormAssembly.  FormAssembly will append a parameter 'code' to this url when returning the user to the url. So that a redirect_uri=https%3A%2F%2Fwww.google.com (note the value is URL-encoded) will result in the user ('Adam') being taken to: https://www.google.com?code=XXXXXXXXXX on authorizing FormAssembly.

2. Catch the users' authorization code:
    * When the user ('Adam') returns to your site after step 1, the parameter 'code' will be in the url.

3. Use that code to generate a token request on your server, by POSTing the following parameters to the url:
    
    HTTP POST:

    + "grant_type=authorization_code" – how you're authentication we're trying
    + "type=web_server" – what style of authentication steps we're using
    + "client_id=CLIENT_ID" – your unique client application id issued by FormAssembly
    + "client_secret=CLIENT_SECRET" – your unique client application secret password issued by FormAssembly
    + "redirect_uri=RETURN_URL" – the same RETURN_URL used in step 1
    + "code=CODE" – the code parameter appended to the RETURN_URL from step 1.

   (Note: Do *not* direct the user to this URL.  Make the request server-side using your preferred server-side HTTP library, such as cURL or HTTPLib.)

The response to this POST will be a JSON array like so:
 
    {"access_token":"XXXXXXXXXXXXXX",
      "expires_in":XXXXX,
      "scope":XXXX,
      "refresh_token":"XXXXXXXXXXXX"}

Store the access_token value for later use.


With the access token, you can make requests to any of FormAssembly's endpoints and receive data back as if the user ( 'Adam' ) was making the request.  For example:

https://app.formassembly.com/forms/index.json?access_token=ACCESS_TOKEN

(ACCESS_TOKEN is the token received in step 3.)

will result in a JSON formatted list of the forms in the user ('Adam')'s account:

    {"Forms":[{
    "Form":{
    "id":"XXXX", "version_id":"XXXX", "name":"XXXXXXXX",
    "category":"XXXX", "subcategory":"XXXX", "is_template":"XXXX",
    "display_status":"XXXX", "moderation_status":"XXXX", "expired":"XXXX",
    "use_ssl":"XXXX", "user_id":"XXXX",
    "created":"XXXXXXXX", "modified":"XXXXXXXX",
    "Aggregate_metadata":{
    "id":"XXXX", "response_count":"XXXX", "submitted_count":"XXXX",
    "saved_count":"XXXX", "unread_count":"XXXX", "dropout_rate":"XXXX",
    "average_completion_time":"XXXX", "is_uptodate":"XXXX"
        }}
    }]}

For an explanation of each field, please see [Object Reference](#object-reference).

Several output formats are accepted, see [End Points](#end-points) for more details.


### End Points:

#### Formats:

FormAssembly supports returning data in two main formats:

  + [json](http://en.wikipedia.org/wiki/Json): a lightweight data exchange format supported by newer languages.  Directly parsable by Javascript.
  + [xml](http://en.wikipedia.org/wiki/XML):  an industry standard data exchange format, parsable by almost all languages.
And some end-points support additional including:
  + [plist](http://en.wikipedia.org/wiki/Plist): used to provide Apple consumable data for Objective-C applications
  + [csv](http://en.wikipedia.org/wiki/Comma-separated_values): used as a standard record data exchange format.
  + [zip](http://en.wikipedia.org/wiki/ZIP_(file_format\)): a binary data container format.

#### Forms: [Returned Fields Reference]

##### Index:
+ https://app.formassembly.com/api_v1/forms/index.json
+ https://app.formassembly.com/api_v1/forms/index.xml

Returns a list of the forms in the user's account, along with associated metadata.

##### Admin Index: (Enterprise plan only)
 + https://app.formassembly.com/admin/api_v1/forms/index.json
 + https://app.formassembly.com/admin/api_v1/forms/index.xml

Returns a list of all forms in the FormAssembly instance.  Only accessible if using FormAssembly Enterprise and access_token from admin level user.

Code Examples:

 + PHP: https://github.com/veerwest/formassembly-api/blob/master/php/fa_api.php
 + Python (command line): https://github.com/veerwest/formassembly-api/blob/master/python/fa_api.py
 + Bash (command line): https://github.com/veerwest/formassembly-api/blob/master/curl/fa_api.sh
 + Salesforce: https://github.com/drewbuschhorn/sfdc-oauth-playground/tree/oauth2_drew


#### Responses: [Returned Fields Reference]

##### Export:
+ https://app.formassembly.com/api_v1/responses/export/#FORMID#.csv
+ https://app.formassembly.com/api_v1/responses/export/#FORMID#.json
+ https://app.formassembly.com/api_v1/responses/export/#FORMID#.xml
+ https://app.formassembly.com/api_v1/responses/export/#FORMID#.zip

additional parameters:
+ date_from: start date for export range
+ date_to: end date for export range
+ filter: if set to 'all', export will include both completed and incomplete responses
+ response_ids: set of comma delimited response ids to retrieve

examples:
+ https://app.formassembly.com/api_v1/responses/export/1.csv?date_form=01/01/2012&date_to=01/01/2013&filter=all
+ would retrieve all responses (including incompletes) created between January 1, 2012 and January 1, 2013.
+ https://app.formassembly.com/api_v1/responses/export/1.xml?response_ids=10,11,12
+ would retrieve only responses with id's: 10,11,12 

#### Object Reference:

##### Form:

***
"id":"XXXX"

Unique integer value identifying the form within the FormAssembly instance.  Every form has a single unique id in the form of an integer.  Can be used to construct a valid form url: http://app.formassembly.com/forms/view/XXXX

Example value: 1 

***
"version_id":"XXXX"

Unique integer id identifying the current version ( revision ) of the form.

Example value: 1

***
"name":"XXXXXXXX"

HTML encoded string representing the form's name as displayed in the user's FormAssembly form index list.  Not to be confused with the Form Title, found in the form's XML definition.

Example: "Mine &amp; Yours Form"

***
"category":"XXXX",

String representing one of the system wide default form organizational categories.  Can be an empty string for uncategorized forms. 

Example: "Contact Forms"

***
"subcategory":"XXXX",

String representing one of the user created organizational categories.  Can be an empty string for uncategorized forms.

Example: "IBM Contact Forms"

***
"is_template":"XXXX",

Integer specifying whether or not the form is shared as a template publicly.

Values:

 + 0  – Not a template
 + >1 – Is a template (Contact Support for more details)

Example: 0 

***
"display_status":"XXXX",

Integer specifying whether or not the form is active or archived. Values:

 + 0 – Archived
 + 2 – Active

Example: 2

***
"moderation_status":"XXXX",

Integer specifying whether or not the form is moderated (under review for suspicious content). Values:

  + 0 – Not checked
  + 2 – Reviewed and approved
  + 3 - Reviewed and denied

Example: 0

***
"use_ssl":"XXXX",

Integer representing if the form must be displayed over HTTPS.  If true, form URL must contain https:// .

***
 + "created":"XXXXXXXX"
 + "modified":"XXXXXXXX"
 + "expired":"XXXX"

String timestamps representing the date the form was created, the last time it or any of its settings were modified, and the date it was marked for deletion if any.  

Examples:

 + "created":"1983-01-01 23:59:59"
 + "modified":"2012-06-17 17:50:12"
 + "expired":null


### Self-register your app on an existing Enterprise instance

    1. Login to your Enterprise account as an admin
    2. Navigate to the Admin Tab
    3. On Application Settings page, navigate to the User Settings Tab
    4. Click register a new application
    5. Pick a name for your new application
