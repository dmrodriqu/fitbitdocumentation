---
title: Fitbit Data Parsers

language_tabs:
  - python

toc_footers:


search: true
---

# Introduction

The Fitbit Data Parsers take JSON data provided by the study-hub and parse it into two formats. 

1. A human readable format that allows for use in either Microsoft Excel for clinical trial staff or querying in other languages.
2. Structuring data so that it can be readily processed for future analyses.

# Application Structure

The parser structure manages several functions so that it automatically parses data with minimal interaction on the user’s part.


A JSON file is first parsed by Parser.py. 

findOmissions.py performs several functions. It structures the data into a tree and contains several methods to find omissions in data.

printOmissions.py allows for the output of data a user defines to the terminal.

## Parser.py

The Parser module contains one class, Table(), initialized with the filepath of the JSON and contains two instance variables


Values                    |    Data Type     |Description
------------------------  | ---------------- | ------------------------------------------- 
filepath 	          |   Str            | File path of the JSON file
parsedTable               |   pd.DataFrame   | The resulting DataFrame of the parsed JSON file



```python
Table(str)
Table.filepath -> str # The filepath of the JSON file
Table.parsedTable -> pd.DataFrame # The resulting parsed table
```

> To parse a JSON file:

```python
from Parser import Table
fpath = ~/documents/V2/someJsonFile.json
table = Table(fpath)
table.parsedTable()
```

Table() contains multiple methods to achieve the above:


Function                | Input Data Type  | Return Data Type |Description
---------------------   |----------------  | ---------------- | ------------------------------------------- 
openJsonFile	        |                  |file          | Opens and reads the JSON file at Table.filepath.
parseJsonFile	        |                  |pd.DataFrame  | Parses a JSON file to a pandas DataFrame
createDataFrame	        |  pd.Series       |pd.DataFrame  | Creates a Pandas DataFrame from a dictionary
parseDataColumn         |  pd.DataFrame    |[pd.DataFrame]| Creates a DataFrame from dictionaries in a Pandas DataFrame
concatAndTransposeData	|                  |pd.DataFrame   | Concatenates a list of dataframes
parseNameSpace  	|  str, str        |pd.DataFrame   | takes a regex string and searches for matches within the namespace column
namespaceBruteSearch  	|                  |list           | formats the namespace column using a reformatted regex string and self.parseNameSpace
getAllFrames     	|                  |pd.DataFrame   | Creates a human readable DataFrame
addFromBruteSearch 	|                  |pd.DataFrame   | concatenates reformatted namespace column
renameCol        	|                  |pd.DataFrame   | renames the Columns



* Table.createDataFrame(self, series): 

Parameters| Data Type|Description
--------- | -------  |------- 
series    |  pd.Series  |A regex string for pattern matching 

Values | Data Type|Description
--------- | -------  |------- 
          |  pd.DataFrame  |Creates a Pandas DataFrame from a dictionary



* Table.parseDataColumn(self, dataToClean): 

Parameters| Data Type|Description
--------- | -------  |------- 
dataToClean |   pd.DataFrame  |A regex string for pattern matching 

Values| Data Type|Description
--------- | -------  |------- 
          |  pd.DataFrame  | The resulting DataFrame from a list of dictionaries


* Table.parseNameSpace(self, regex, testString): 


Parameters| Data Type|Description
--------- | -------  |------- 
regex     |  str  |A regex string for pattern matching
testString|  str  |The string to find matches in

Values| Data Type|Description
--------- | -------  |------- 
regex     |  str  |A regex string for pattern matching
testString|  str  |The string to find matches in






## findOmissions.py

findOmissions.py creates a tree of which no node is more than two steps away from the root node.

Two classes are contained in findOmissions MainData and SubData:


### MainData

MainData has several instance variables containing information pertinent to the overview of the study:



Values                    |    Data Type     |Description
------------------------  | ---------------- | ------------------------------------------- 
df	               |   pd.DataFrame            | The current DataFrame to be processed
surveyParticipants     |   list   | A unique list of survey participants
arrayOfSubsetObjects   |   list   | A list containing the SubData objects detailing each patient’s participation in the study
patientsToContact      |   list   | deprecated


Methods:




Function                | Input Data Type  | Return Data Type |Description
---------------------   |----------------  | ---------------- | ------------------------------------------- 
_getJsonPath	        |                  |file          | Opens and reads the JSON file at Table.filepath.
_stringCleaning	        |  string          |string       | removes parenthesis
_convertTime            |  string          |DateTime  | converts epoch to datetime
_newTable               |  pd.DataFrame    |pd.DataFrame| removes parenthesis in table
_getStartDates	        |  pd.DataFrame    |pd.DataFrame   | Returns time requested and time completed for each survey with ID as the pKey
_construct       	|                  |               | sets self.df using the return value from _getStartDates and sets self.surveyIDs as unique IDs
_getCondition   	|                  |               | deprecated
createTraversal    	|                  |self.arrayOfSubsetObjects | Creates a SubData class for every participant ID


* _stringCleaning(self, stringToClean)

Parameters| Data Type|Description
--------- | ------- | ------- 
stringToClean  | Str| String containing extraneous parentheses.



* _convertTime(self, unixTime)


Parameters| Data Type|Description
--------- | ------- | ------- 
unixTime  | Str     | Time expressed in the number of seconds past Thursday, 1 January 1970, minus the number of leap seconds that have taken place since then 



* _getStartDates(self, table) -> table

converts epochs in table.


Parameters| Data Type|Description
--------- | -------  |------- 
table     |   pd.DataFrame    |A pandas DataFrame containing a parsed JSON file 

Values | Data Type |Description
--------- | ------- | ------- 
table |  pd.DataFrame   | A pandas DataFrame with UnixTime objects converted to DateTime



* createTraversal(self)

traverses over DataFrame and creates SubData objects for each participant ID


> To set structure survey data:

```python
From findOmissions import MainData
structure =  MainData()
structure.createTraversal()
```


### SubData

SubData structures surveys, scoring, dates to complete, and each individuals enrollment date.

SubData’s instance variables:

Values                    |    Data Type     |Description
------------------------  | ---------------- | ------------------------------------------- 
participantID	          |   Str            | ID of the Participant
df	                  |   pd.DataFrame   | Data respective of the individual participant
uniqueSurveys	          |   pd.DataFrame   | Surveys of the participant
enrollmentDate	          |   [DateTime]     | Date of Enrollment for the participant
datesToCompleteSurveys	  |   [DateTime]     | Last possible date to complete survey respective of self.uniqueSurveys
self.contactPatient	  |   Bool           | True if staff to contact patient



Methods:

* getQuestionDate(self, question, requested = requested) -> [dateTime]

Retrieves a list of the dates for each task presented

Parameters| Data Type |Description
--------- | ------- | ------- 
question  |   Str   |Survey or Task (‘psqi’, ‘vas’, ‘sibdq’, ‘sleep’)| 
requested |   Str   | time requested, default = requested, (‘completed’, ‘requested’)


* calculateCompletionDates(self) -> list

Returns the dates of when individuals should receive survey tasks

* calculateDateRanges(self) -> (datesOfCompletion, addWindow)

Returns a tuple of DateTimes

Values | Data Type |Description
--------- | ------- | ------- 
datesOfCompletion  |   DateTime   | First possible date of completion of survey tasks
addWindow |   DateTime  | Last possible date of completion of survey tasks


* findOmissions(self, question) -> [participantID, completionDates, completionAfterDates, nonCompletionDates]

Parameters| Data Type |Description
--------- | ------- | ------- 
question  |   Str   |Survey or Task (‘psqi’, ‘vas’, ‘sibdq’, ‘sleep’) 
requested |   Str   | time requested, default = requested, (‘completed’, ‘requested’)

Values | Data Type |Description
--------- | ------- | ------- 
participantID  |   Str   | The ID of the Participant
completionDates |   list  | Dates of task completion
completionAfterDates  |   list   | Dates of task completion after date range 
nonCompletionDates |   list   | Last possible dates of completion for surveys not completed

# Usage

As each individual’s raw surveys and enrollment date is accessible separately from others, surveys can be scored and accessed systematically.

Surveys and their dates are 0 indexed and are accessed separately. Should one want to correlate a survey with a date or tuples of dates, their indices are equivalent.




> An example of calculating PSQIs

```python
from findOmissions import MainData

data = MainData() # creating the MainData Object
data.createTraversal() # structuring the data
for subject in data.getSubsetData():
    subject.getUniqueSurveys()
    print subject.participantID #prints an ID for the current participant
    subject.getSurveySeries('PSQI')
    print map (subject.scorePSQI, subject.uniqueSurveys) # prints an array of the PSQI scores
```

# The Data Dashboard

The dashboard pulls all longitudinal data according to two lines of code:

```python
import results
result = Results('#questionsToQuery')
result.output(values = False) # gives dates, defaults to true to give values
```

questions to query:
* PSQI
* SIBDQ
* WongBaker
* SubjectGlobalAssesmentVAS
* SleepVas

The results query also does a rudimentary search. If the query is a substring in the question above,
it will return a result. Careful use of this function is warranted, and it should be advised against
unless the two questions have distinctly different characters.


The instance variables in the Results Class:

Values                    |    Data Type     |Description
------------------------  | ---------------- | ------------------------------------------- 
IDs          |   list            | List of all IDs in the study
Dates	                  |  np.array   | dates of participant tasks
Values	          |   np.array   | values of participatn tasks
survey	          |   string     | string of task to retrieve results for


* _getData(self) -> self.Dates, self.Values
Retrieves and structures data. Structures all data according to participant task date and value.

* insertDateGaps(self, dates) -> dic(self.IDs,self.Dates)
Iterates over the current dictionary of survey dates. If there are a pair of dates with greater than
a datetime difference of 1 day, it inserts the integer value of the number of days in "SKIPPED" strings.

* output(self, date = True, values = False) -> "survey.csv"

Performs all of the functions as above to format a CSV file as the output, either for date times or values.


