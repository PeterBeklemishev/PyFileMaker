Note: This documentation is now also in PyFileMaker/__init__.py. Available as a Docstring for the module.

#PyFileMaker

Old version of documentation and code is available at http://code.google.com/p/pyfilemaker

PyFileMaker module is designed for both script and interactive use.
Any command used during interactive session is possible to type in a script.

-------------------------------------------------------------------------------
###Short introduction

####Setting of database and layout

In the beginnig you have to set the server, database and layout::

```python
  >> from PyFileMaker import FMServer

  >> fm = FMServer('http://login:password@filemaker.server.com')
  >> fm.
  [and press Tab to see available methods and variables]
  >> help fm.getDbNames
  [displays help for the method]

  >> fm.getDbNames()
  ['dbname','anoterdatabase']
  >> fm.setDb('dbname')

  >> fm.getLayoutNames()
  ['layoutname','anotherlayout']
  >> fm.setLayout('layoutname')
```

You can also type directly:

```python
  >> fm = FMServer('http://login:password@filemaker.server.com','dbname','layoutname')
```

####List fieldnames

Get the list of fields from the active layout::

```python
  >> fm.doView()
  ['column1', 'column2']
```

####Find records

To search records::

```python
  >> fm.doFind(column1='abc')
  <FMProResult instance with 2 records>
  [column1='abcdef'
  column2'='some data'
  RECORDID=1,
  column1='abc'
  column2='another data'
  RECORDID=2]
```

You've got list of 2 records, usually you need to work only with one record:

```python
  >> a = fm.doFind(column1='abc')
  >> len(a)
  2
  >> r = a[0]
  >> r.
  [press Tab for available layout variables or for completition of variable name]

  >> r.column1
  'abcdef'
  >> r['column1']
  'abcdef'
  >> print r.column1
  abcdef
  >> r.column.related
  'content'

  >> fm.doFind( column1='abc', column__related='abc', LOP='OR', SKIP=1, MAX=1)
```

Get latest record if documentID field is autoincremented during insertion in FileMaker:

```python
  >> fm.doFind( SORT={'documentID':'<'}, MAX=1)
```

Or more low level access using dict - for operators, and non-ascii fields:

```python
  >> fm.doFind( {'documentID.op':'lt', '-max':1})
```

Any combination of attributes is allowed...

BTW For query empty record:

```python
  >> fm.doFind( column1='==')
```

####Editing records

It's enought when you change some variables inside of previosly returned record::

```python
  >> r = fm.doFindAny()[0]
  >> r.column = 'NEWVALUE'
  >> fm.doEdit(r)
```

This will update only changed column 'column' in table.
You can use changed old record in other functions too - doDup, doNew, doDelete.
doFind(r) will find record with the same RECORDID, but MODID is updated.

New records can be specified as arguments of doNew() like::

```python
  fm.doNew(column1='newvalue', column2='old')
  fm.doNew({'column':'newvalue','column2':'old'})
```

#### Find multiple records by passing list of arguments

This command allows to find all records with attributes passed as keys and values passed as values in the list.

##### Creating OR query

All query arguments are treated as separate arguments and each of it will include more results if any.

1. Result of this call will return all objects with id in [1,2,3,4]. If there is no object with id = 4 only three results will be returned.

```python
  fm.doFindQuery({'id': [1, 2, 3, 4],})
```

2. We can query by multiple keys. This call will try to find all elements with id in [1,2,3,4] or color in ['red', 'blue'] or gender 'm'.

```python
  fm.doFindQuery({'id': [1, 2, 3, 4], 'color': ['red', 'blue'], 'gender': 'm'})
```

3. We can create exclude query as well, by putting '!' in front of our key.
This call will try to find all elements with id in [1,2,3,4] or color in ['red', 'blue'] and NOT gender 'm'.

```python
  fm.doFindQuery({'id': [1, 2, 3, 4], 'color': ['red', 'blue'], '!gender': 'm'})
```

##### Creating AND query

By using the syntax as showed below we can create AND query and force FM to take in consideration all arguments 

1. Find all entries with gender equal 'm' and color 'red'.
```python
  fm.doFindQuery({
    'subquery_1': {
      'color': 'red',
      'gender': 'm'
    }
  })
```
Note: Key name `subquery_x` is just a convention and it's not included to the XML query itself. It's used only to distinguish the AND groups.

##### Combine AND and OR

Both syntaxes can be combined to create complex queries

1. Find all entries with gender equal 'm' and color 'red' or gender equal 'f' and color 'blue'.
```python
  fm.doFindQuery({
    'subquery_1': {
        'color': 'red',
        'gender': 'm'
    },
    'subquery_2': {
        'color': 'blue',
        'gender': 'f'
    },
  })
```

#### Execute scripts

FileMaker API allow to execute scripts. We can execute two types of them with or without parameters. We can send parameter as string separated by special character set from FM side.


```python
# execute script without parameters
resp = fm.doScript('script_name')

# execute script with parameters
parameters = "first|second|third"
resp = fm.doScript('script_name', params=parameters)
```

#### Retrieve files

We can retrieve files from FM server as binary stream. To do this we need to point to the container field of the object and query for it's content.
Each object that contains container field contains pre-generated by FM query to retrieve its content. We can easily fin it by displaying all fields for that given object.


```python
container_field = fm.doFind(id="10")
container_field
  MODID = '10'
  RECORDID = '9791'
  creation_timestamp = '2014-06-25 16:25:05'
  id = '37'
  number = '199'
  email = 'some@email.co.uk'
  title = 'Rambo II'
  from_ts = '2014-06-25 00:00:00'
  ooh_zip = '/fmi/xml/cnt/data.zip?-db=DATABASE&-lay=layout&-field=ooh_zip&-recid=114'
  status = 'Complete'
```

In that case we need the `ooh_zip` field.

```python
file_name, file_extension, file_binary = fm_server.getFile(container.ooh_zip)
```

#### Execute script after another command

Method doScriptAfter is taking arguments:

 1. FM method
 2. Method arguments as kwargs(dict formatted)
 3. Script name
 4. Script arguments (optional)

```python
resp = fm.doScriptAfter(fm_server.doNew, {
      'first_name': 'Foo',
      'last_name': 'Bar',
      'ip': user_ip,
      'action': 'Download'
    },
  'script_name',
  'script_params'
)
```

#### Parse response to JSON like dictionary/list

This can parse each response to JSON like response.
  
```python
  >> import json

  >>resp = fm.doFind( column1='==')
  >>json.dumps(fm.toJSON(resp))
```

####Templates

The structure of returned data is suitable for use with Cheetah Templates.
It is really easy to write a template::

```python
  import Cheetah.Template
  t = Cheetah.Template.Template('''
  Document Template
  ~~~~~~~~~~~~~~~~-
  DocumentID: $documentID
  DocumentType: $DocumentType.documentType

  Item descriptions:
  #for $l in $DocumentLine
   - $l.description
  #end for      
  ''', searchList=[r[0]])
```

####Debugging connection

Best way howto debug what's wrong::

```python
  >> fm._debug = True
```

then check printed url request by external tools (like curl, xmlstarlet):

```
``$ curl 'http://test:test@filemaker.server.com:80/fmi/xml/fmresultset.xml?-db=test&-layout=test&-findall' | xmlstarlet fo``
```

####Error reporting

In case something is not running the way it should, please report an Issue on the the project website.
New contributions to the code are welcomed. 
