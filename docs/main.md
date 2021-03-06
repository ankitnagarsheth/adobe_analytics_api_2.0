# aanalytics2

The aanalytics2 python library stands for Adobe Analytics API 2.0 support.
It is set to wrap the different endpoints from the documentation.
You can find the swagger documentation [here](https://adobedocs.github.io/analytics-2.0-apis/).

The different section will quickly explain the methods available in the differennt part of this API.

## Core components

The methods available directly from the module are the following:

### createConfigFile

This methods allows you to create JSON file that will host the different information required for your connection to Adobe IO and to retrieve the token.
It usually is like this.

```python
import aanalytics2 as api2
api2.createConfigFile()
```

As you can see, it takes no argument and the output of the file will look like this :

```javaScript
{
    'org_id': '<orgID>',
    'api_key': "<APIkey>",
    'tech_id': "<something>@techacct.adobe.com",
    'secret': "<YourSecret>",
    'pathToKey': '<path/to/your/privatekey.key>',
}
```

You update the information from the Adobe IO account that you have setup.

### importConfigFile

As you have created your JSON config file, you will need to import it before realizing any request to the Analytics API.
The method takes the file name as an argument, or the path where your file exist.

```python
import aanalytics2 as api2
api2.importConfigFile('config.json')
```

or

```python
import aanalytics2 as api2
myfilePath = './myCredential/config.json'
api2.importConfigFile(myfilePath)
```

### retrieveToken

This method is not mandatory because each of your requests will be taken care by a wrapping function that will generate a token if you don't have one.
However, at some point, you may want to generate a token for a reason X or Y, so this function is available for that use-case.
This function takes no argument as long as you have imported the config file.
It returns the token.

```python
import aanalytics2 as api2
api2.importConfigFile('config.json')

token = api2.retrieveToken()
```

This method also caches the time limit of the token usage in a "private" variable "_date_limite"
It can be accessed like this:

```python
api2._date_limite
```

you can review when this token expire through the time module:

```python
import time
time.ctime(api2._date_limit)

## returns something like : 'Mon May 17 18:08:42 2023'
```

### getCompanyId

Once the previous steps are completed, you can start using the methods attached directly to Analytics API.
The first method is the _getCompanyId_, that will return you the different company ID attached to your account.
You will extract the *globalCompanyId* that you want to use to create your instance of the Analytics class.
This method takes an optional parameter (infos) that is used to directly fetched the predefined element.

```python
import aanalytics2 as api2
api2.importConfigFile('myconfig.json')
cids = api2.getCompanyId() ## as you can see, no need to call the retrieveToken method.
cid = cids[0]['globalCompanyId'] ## using the first one
mycompany = api2.Analytics(cid)
```

If you already know which ID you want to select, you can pass the reference directly in the "infos" parameter.

```python
import aanalytics2 as api2
api2.importConfigFile('myconfig.json')
cid = api2.getCompanyId(infos=0) ## selecting the first ID directly in the request.
mycompany = api2.Analytics(cid)
```

## Analytics class

Adobe Analytics API 2.0 requires you to send the companyID you have selected in the header of each request you do in that company.
There used to be a method in this API to update the header accordingly.
I changed this logic to create a class that encapsulate this choice directly.
Therefore it is possible to use 2 instances of the class at the same time and therefore 2 companyID can be used almost at the same time.

As seen in the previous example, you instantiate the class by passing the companyID.

```python
import aanalytics2 as api2
#...
mycompany = api2.Analytics(cid)
```

You instance will have all of the Analytics endpoint wrapped.
At any point in time, you can use the docstring that have been set in this module by using the help function.

```python
help(mycompany.getSegments)
## returns:
##getSegments(name: str = None, tagNames: str = None, inclType: str = 'all', rsids_list: list = None, sidFilter: list = None, extended_info: bool = False, format: str = 'df', save: bool = False, verbose: bool = False, **kwargs) -> object method of aanalytics2.Analytics instance
#    Retrieve the list of segments. Returns a data frame.
#    Arguments:
#        name : OPTIONAL : Filter to only include segments that contains the name (str)
#        tagNames : OPTIONAL : Filter list to only include segments that contains one of the tags (string delimited with comma, can be list as well)
#...
```

One of the specific functionality for this class, is that you can update the token with a new one by using the _refreshToken_ method.
It takes one argument (the token string) and requires you to run the retrieveToken method beforehand.

```python
import aanalytics2 as api2
#...
token = api2.retrieveToken()
mycompany.refreshToken(token)

```

With all of my API wrappers, I ususall separate the methody by the GET (fetching information), the CREATE methods (posting information), the DELETE methods, the UPDATE mehthods.
This API is no exception.

### The get methods

There are several get methods.

* getReportSuites : Get the list of reportSuites.Returns a data frame.
  Arguments:
  * txt : OPTIONAL : returns the reportSuites that matches a speific text field
  * rsid_list : OPTIONAL : returns the reportSuites that matches the list of rsids set
  * limit : OPTIONAL : How many reportSuite retrieves per serverCall
  * save : OPTIONAL : if set to True, it will save the list in a file. (Default False)

* getSegments: Retrieve the list of segments. Returns a data frame.
    Arguments:
  * name : OPTIONAL : Filter to only include segments that contains the name (str)
  * tagNames : OPTIONAL : Filter list to only include segments that contains one of the tags (string delimited with comma, can be list as well)
  inclType : OPTIONAL : type of segments to be retrieved.(str) Possible values: 
    * all : Default value (all segments possibles)
    * shared : shared segments
    * template : template segments
    * deleted : deleted segments
    * internal : internal segments
    * curatedItem : curated segments
  * rsid_list : OPTIONAL : Filter list to only include segments tied to specified RSID list (list)
  * sidFilter : OPTIONAL : Filter list to only include segments in the specified list (list)
  * extended_info : OPTIONAL : additional segment metadata fields to include on response (bool : default False)
  if set to true, returns reportSuiteName, ownerFullName, modified, tags, compatibility, definition
  * format : OPTIONAL : defined the format returned by the query. (Default df)
    possibe values :
    * "df" : default value that return a dataframe
    * "raw": return a list of value. More or less what is return from server.
  * save : OPTIONAL : If set to True, it will save the info in a csv file (bool : default False)
  * verbose : OPTIONAL : If set to True, print some information
  Possible kwargs:
  * limit : number of segments retrieved by request. default 500: Limited to 1000 by the AnalyticsAPI.

* getDimensions: Retrieve the list of dimensions from a specific reportSuite.Shrink columns to simplify output. Returns the data frame of available dimensions.
  Arguments:
  * rsid : REQUIRED : Report Suite ID from which you want the dimensions
  * tags : OPTIONAL : If you would like to have additional information, such as tags. (bool : default False)
  * save : OPTIONAL : If set to True, it will save the info in a csv file (bool : default False)
  Possible kwargs:
  * full : Boolean : Doesn't shrink the number of columns if set to true
  example : getDimensions(rsid,full=True)

* getMetrics: Retrieve the list of metrics from a specific reportSuite. Shrink columns to simplify output. Returns the data frame of available metrics.
  Arguments:
  * rsid : REQUIRED : Report Suite ID from which you want the dimensions
  * tags : OPTIONAL : If you would like to have additional information, such as tags. (bool : default False)
  * save : OPTIONAL : If set to True, it will save the info in a csv file (bool : default False)
  Possible kwargs:
  * full : Boolean : Doesn't shrink the number of columns if set to true
  example : getMetrics(rsid,full=True)

* getUsers: Retrieve the list of users for a login company.Returns a data frame.
  Arguments:
  * save : OPTIONAL : Save the data in a file (bool : default False).
  Possible kwargs:
  * limit : OPTIONAL : Nummber of results per requests. Default 100.

* getDateRanges: Get the list of date ranges available for the user.
  Arguments:
  * extended_info : OPTIONAL : additional segment metadata fields to include on response
        additional infos: reportSuiteName, ownerFullName, modified, tags, compatibility, definition
  * save : OPTIONAL : If set to True, it will save the info in a csv file (Default False)
  Possible kwargs:
  * limit : number of segments retrieved by request. default 500: Limited to 1000 by the Analytics API.
  * full : Boolean : Doesn't shrink the number of columns if set to true

Example of getSegments usage:

```python
mysegments = mycompany.getSegments(extended_info=True,save=True)
```

Example of getDimensions usage:

```python
mydims = mycompany.getDimensions('rsid')
```

### Get report

The get report from Adobe Analytics, I will recommend you to watch the [video](https://youtu.be/j1kI3peSXhY) that explains how you can generate the JSON file for requesting the report through API.
This getReport methods is a bit special because it is mostly what users would have liked to have from this API, so it is a separate part.

First of all, you need to understand that this API is a replicate of Adobe Analytics Workspace interface.
There is no additional feautres or way to bypass the limitation of Analytics Workspace from this API.
It means that the (low traffic) will still appear and also the filters are different requests.
Also, there is a limit of 120 requests per minute, and a limit threshold of 12 requests for 6 seconds.
Therefore, requesting large amount of data is not the use-case for the API 2.0.

When you have generated your request through the UI interface you can directly saved it in a json file.
You can then either directly use it with the aanalytics2 module or rework it in python.

Before going with examples, I will explain the method and its arguments:

* getReport : Retrieve data from a JSON request.Returns an object containing meta info and dataframe.
  Arguments:
  * json_request: REQUIRED : JSON statement that contains your request for Analytics API 2.0.
  The argument can be :
    * a dictionary : It will be used as it is.
    * a string that is a dictionary : It will be transformed to a dictionary / JSON.
    * a path to a JSON file that contains the statement (must end with ".json").
  * n_result : OPTIONAL : Number of result that you would like to retrieve. (default 1000)
    if you want to have all possible data, use "inf".
  * item_id : OPTIONAL : Boolean to define if you want to return the item id for sub requests (default False)
  * save : OPTIONAL : If you would like to save the data within a CSV file. (default False)
  * verbose : OPTIONAL : If you want to have comment display (default False)

As you can see, you can use the getReport with different variable types (string, path or dictionary).

The object that is being returned is hosting the information about your request. In case you need to have the context during your usage of the data.
Example of the object being returned :

```python
{'dimension': 'variables/eVarX',
    'filters': {'globalFilters': ['2019-11-01T00:00:00.000/2019-12-01T00:00:00.000'],
    'metricsFilters': {}},##if any filters have been applied.
    'rsid': 'my.rsid',
    'metrics': ['metrics/visits'],
    'data': #pandas DataFrame.
}
```

As you can see it returns the actual data of your request in the "data" key, as a dataframe.
Therefore, you can use the data by doing the extraction of the data.
Example:

```python
myreport = mycompany.getReport('myReport.json')
mydata = myreport['data']
```

### The create methods

Adobe Analytics API 2.0 currently, limiting number of creation.
You can only create Calculated Metrics and Segments so far.

* createSegment:
  Arguments:
  * segmentJSON : REQUIRED : the dictionary that represents the JSON statement for the segment.

* createCalculatedMetrics:
  Arguments:
  * metricJSON : REQUIRED : the dictionary that represents the JSON statement for the  Calculated Metrics.
    Required in the dictionary :  name, definition, rsid
