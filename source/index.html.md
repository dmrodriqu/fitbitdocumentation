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

* self.filepath -> str
* self.parsedTable -> pd.DataFrame

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

* Table.openJsonFile(self): 

Opens and reads the JSON file at Table.filepath.

* Table.parseJson(self): 

Parses a JSON file to a pandas DataFrame

* Table.createDataFrame(self): 

Creates a Pandas DataFrame from a dictionary

* Table.parseDataColumn(self): 

Iterates over dictionaries in a Pandas DataFrame

* Table.concatAndTransposeData(self): 

Formats all dictionaries present in columns to a pandas DataFrame

* Table.parseNameSpace(self, regex, testString): 

takes a regex string and searches for matches.


Parameters| Data Type|Description
--------- | -------  |------- 
regex     |  str  |A regex string for pattern matching| 
testString|  str  |The string to find matches in| 

* Table.namespaceBruteSearch(self): 

Extracts all available column headers in the namespace column.

* Table.getAllFrames(self): 

Creates a human readable DataFrame

* Table.addFromBruteSearch(self): 

adds column headers from namespaces to each record

* Table.renameCol(self): 

Creates an appropriate name for each column







## findOmissions.py

findOmissions.py creates a tree of which no node is more than two steps away from the root node.

Two classes are contained in findOmissions MainData and SubData:


### MainData

MainData has several instance variables containing information pertinent to the overview of the study:

* self.df -> pd.Dataframe

The current DataFrame to be processed

* self.surveyParticipants -> list

A unique list of survey participants

* self.arrayOfSubsetObjects -> list

A list containing the SubData objects detailing each patient’s participation in the study

* self.patientsToContact -> list

- - deprecated - -


Methods:

* _getJsonPath(self)

gets the path of the JSON file in the current directory

* _stringCleaning(self, stringToClean)

removes parenthesis

Parameters| Data Type|Description
--------- | ------- | ------- 
stringToClean  | Str| String containing extraneous parentheses.

* _convertTime(self, unixTime)

converts epoch to datetime

Parameters| Data Type|Description
--------- | ------- | ------- 
unixTime  | Str     | Time expressed in the number of seconds past Thursday, 1 January 1970, minus the number of leap seconds that have taken place since then 


* _newTable(self)

removes parenthesis in table





* _getStartDates(self, table)

converts epochs in table.


Parameters| Data Type|Description
--------- | -------  |------- 
table     |   pd.DataFrame    |A pandas DataFrame containing a parsed JSON file| 


Values | Data Type |Description
--------- | ------- | ------- 
table |  pd.DataFrame   | A pandas DataFrame with UnixTime objects converted to DateTime


* _construct(self)

removes parenthesis and converts epochs in old table, sets instance variable of self.df to current DataFrame

* _getCondition(): 

 - - deprecated - -

* createTraversal(self)

traverses over DataFrame and creates SubData objects for each participant ID


> To set structure survey data:

```python
structure =  MainData()
structure.createTraversal()
```


### SubData

SubData structures surveys, scoring, dates to complete, and each individuals enrollment date.

SubData’s instance variables:

* self.participantID -> str

ID of the participant

* self.df -> pd.DataFrame

Data respective of the individual participant

* self.uniqueSurveys -> list

Each survey of the participant

* self.enrollmentDate -> [dateTime]

Date of enrollment for the participant

* self.datesToCompleteSurveys -> [dateTime]

Date to complete each survey (index respective of the unique survey list)

* self.contactPatient -> bool

Default false, True if to contact patient



Methods:

* getQuestionDate(self, question, requested = requested) -> [dateTime]

Retrieves a list of the dates for each task presented

Parameters| Data Type |Description
--------- | ------- | ------- 
question  |   Str   |Survey or Task (‘psqi’, ‘vas’, ‘sibdq’, ‘sleep’)| 
requested |   Str   | time requested, default = requested, (‘completed’, ‘requested’)


* calculateCompletionDates(self) -> list

Returns the dates of when individuals should receive survey tasks

* calculateDateRanges(self) -> (DateTime, DateTime)

Returns a tuple of DateTimes

Values | Data Type |Description
--------- | ------- | ------- 
datesOfCompletion  |   DateTime   | First possible date of completion of survey tasks
addWindow |   DateTime  | Last possible date of completion of survey tasks


* findOmissions(self, question)

Parameters| Data Type |Description
--------- | ------- | ------- 
question  |   Str   |Survey or Task (‘psqi’, ‘vas’, ‘sibdq’, ‘sleep’)| 
requested |   Str   | time requested, default = requested, (‘completed’, ‘requested’)

Values | Data Type |Description
--------- | ------- | ------- 
participantID  |   Str   | The ID of the Participant
completionDates |   list  | Dates of task completion
completionAfterDates  |   list   | Dates of task completion after date range 
nonCompletionDates |   list   | Last possible dates of completion for surveys not completed



```python

```
