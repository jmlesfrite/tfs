TFS Python Library (TFS API Python client)
==========================================

[![dohq-tfs build status](https://travis-ci.org/devopshq/tfs.svg)](https://travis-ci.org/devopshq/tfs) [![dohq-tfs code quality](https://api.codacy.com/project/badge/Grade/a533e2d46b9b471893b4991e89649212)](https://www.codacy.com/app/tim55667757/tfs/dashboard) [![dohq-tfs code coverage](https://api.codacy.com/project/badge/Coverage/a533e2d46b9b471893b4991e89649212)](https://www.codacy.com/app/tim55667757/tfs/dashboard) [![dohq-tfs on PyPI](https://img.shields.io/pypi/v/dohq-tfs.svg)](https://pypi.python.org/pypi/dohq-tfs) [![dohq-tfs license](https://img.shields.io/pypi/l/vspheretools.svg)](https://github.com/devopshq/tfs/blob/master/LICENSE)

*Index:*
- [Introduction](#introduction)
- [Quickstart](#quickstart)
    - [Installation](#installation)
    - [Create connection](#create-connection)
        - [Timeout connection](#timeout-connection)
    - [Workitem](#workitem)
    - [Run Saved Queries](#run-saved-queries)
    - [Run WIQL](#run-wiql)
    - [Changesets](#changesets)
    - [Project and Team](#project--team)
- [Guide](#guide)
    - [Compability](#compability)
    - [Development](#development)
        - [Tests](#tests)
        - [TODO](#todo)

# Introduction

TFS Python Library is a TFS API Python client that can work with TFS workflow and workitems.

This tool allows:
1. Get WorkItems (WI).
2. Set WI fields.
3. Run WI search queries.
4. Work with TFVC changesets.
5. Work with TFS Projects.

## Installation
```
pip install dohq-tfs
```

## Create connection
```python
from tfs import TFSAPI

user="username"
password="password"

# Use DefaultCollection
client = TFSAPI("https://tfs.tfs.ru/tfs/", user=user, password=password)

# Use CustomCollection
client = TFSAPI("https://tfs.tfs.ru/tfs/", project="Development", user=user, password=password)

# Set path to ProjectName in project parameter
client = TFSAPI("https://tfs.tfs.ru/tfs/", project="Development/ProjectName", user=user, password=password)

workitem = client.get_workitem(100) # Test connection with Workitem id
```

## Timeout connection
You can set CONNECT and READ timeouts ([read more](http://docs.python-requests.org/en/master/user/advanced/#timeouts))
```python
from tfs import TFSAPI
client = TFSAPI("https://tfs.tfs.ru/tfs/", user=user, password=password, connect_timeout=30, read_timeout=None)

```

## Workitem
```python
# For single Workitem
workitem = client.get_workitem(100)

# For multiple
workitem = client.get_workitems([100,101,102]) # list
workitem = client.get_workitems("100,101,102") # string separated with comma

# Get all fields
print(workitem.field_names)

# Case insensetive. Remove space in field name
print(workitem['assignedTo']) 

# Update field
workitem['state'] = 'Complete' 

# Add comment
print(workitem.history)
workitem['History'] = "Omg, it is goos issue!"
print(workitem.history)

# Workitem Parent Workitem
parent = workitem.parent
if parent: # Parent is None if Workitem hasn't Parent link
    print("Workitem with id={} have parent={}".format(workitem.id, parent.id))

# Workitem Childs Workitem
childs = workitem.childs
if childs: # Child is empty list if Workitem hasn't Child link
    print("Workitem with id={} have Childs={}".format(workitem.id, ",".join([x.id for x in childs])))
```

## Run Saved Queries
You can run Saved Queries and get Workitems
```python
# Set path to ProjectName in project parameter
client = TFSAPI("https://tfs.tfs.ru/tfs/", project="Development/ProjectName", user=user, password=password)

# Run New query 1 in Shared Queries folder
quiery = client.run_query('Shared Queries/New query 1')

# result content raw data
result = quiery.result
print(quiery.columns)
print(quiery.column_names)

# Get all found workitems
workitems = quiery.workitems
```

## Run WIQL
You can run [Work Item Query Language](https://msdn.microsoft.com/en-us/library/bb130198(v=vs.90).aspx)
```python
# Set path to ProjectName in project parameter
client = TFSAPI("https://tfs.tfs.ru/tfs/", project="Development/ProjectName", user=user, password=password)

# Run custom query
### NOTE: Fields in SELECT really ignored, wiql return Work Items with all fields
query = """SELECT
    [System.Id],
    [System.WorkItemType],
    [System.Title],
    [System.ChangedDate]
FROM workitems
WHERE
    [System.WorkItemType] = 'Bug'
ORDER BY [System.ChangedDate]"""

wiql = client.run_wiql(query)

# Get founded Work Item ids
ids = wiql.workitem_ids
print("Found WI with ids={}".format(",".join(ids)))

# Get RAW query data - python dict
raw = wiql.result

# Get all found workitems
workitems = wiql.workitems
print(workitems[0]['Title'])
```

## Changesets
```python
# Get changesets from 1000 to 1002
changesets = client.get_changesets(from_=1000, to_=1002)

# Get changesets and related Workitems
changesets = client.get_changesets(top=1)
linked_workitems = changesets[0].workitems
```

## Project & Team
```python
# Get all project
all_projects = client.get_projects()

# Get project
project_name = client.get_project("MyProjectName")

# Get project team
project_team = project_name.team
```


## Guide
### Compability
- TFS 2015 
- TFS 2017

## Development
### Tests
We use HTTPPrety. For GET-response locate you response.json to folder by URL. E.g:
- http://tfs.tfs.ru/tfs/Development/_apis/wit/workitems?ids=anyid&anyflag => **tests/resources/tfs/Development/_apis/wit/workitems/response.json**
- http://tfs.tfs.ru/tfs/Development/_apis/tfvc/changesets/10/workItems => **tests/resources/tfs/Development/_apis/tfvc/changesets/10/workItems/response.json**

### TODO
- Implemented Resources-API (like https://github.com/pycontribs/jira)
