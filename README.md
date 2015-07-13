# vertica-python

[![PyPI version](https://badge.fury.io/py/vertica-python.png)](http://badge.fury.io/py/vertica-python)

This is fork of https://github.com/uber/vertica-python with Python 3.4 compatibility only!

if you find some bug or an issue that is not related to compatibility, please, submit it to original repository.


## Run unit tests
    # install nose if you don't have it
    pip install -r requirements_test.txt

    # you will need to have access to a vertica database.
    # connection info is in tests/basic_tests.py

    # run tests
    nosetests


## Usage


**Create connection**

```python
from vertica_python import connect

conn_info = {'host': '127.0.0.1', 
             'port': 5433, 
             'user': 'some_user', 
             'password': 'some_password', 
             'database': 'a_database'}

# simple connection, with manual close
connection = vertica_python.connect(**conn_info)
# do things
connection.close()

# using with for auto connection closing after usage
with vertica_python.connect(**conn_info) as connection:
    # do things
```


**Stream query results**:

```python
cur = connection.cursor()
cur.execute("SELECT * FROM a_table LIMIT 2")
for row in cur.iterate():
    print(row)
# {'id': 1, 'value': 'something'}
# {'id': 2, 'value': 'something_else'}
```
Streaming is recommended if you want to further process each row, save the results in a non-list/dict format (e.g. Pandas DataFrame), or save the results in a file.


**In-memory results as list**:

```python
cur = connection.cursor()
cur.execute("SELECT * FROM a_table LIMIT 2")
cur.fetchall()
# [ [1, 'something'], [2, 'something_else'] ]
```


**In-memory results as dictionary**:

```python
cur = connection.cursor('dict')
cur.execute("SELECT * FROM a_table LIMIT 2")
cur.fetchall()
# [ {'id': 1, 'value': 'something'}, {'id': 2, 'value': 'something_else'} ]
connection.close()
```


**Query using named parameters**:

```python
# Using named parameter bindings requires psycopg2>=2.5.1 which is not includes with the base vertica_python requirements.

cur = connection.cursor()
cur.execute("SELECT * FROM a_table WHERE a = :propA b = :propB", {'propA': 1, 'propB': 'stringValue'})
cur.fetchall()
# [ [1, 'something'], [2, 'something_else'] ]
```

**Insert and commits** :

```python
cur = connection.cursor()

# inline commit
cur.execute("INSERT INTO a_table (a, b) VALUES (1, 'aa'); commit;")

# commit in execution
cur.execute("INSERT INTO a_table (a, b) VALUES (1, 'aa')")
cur.execute("INSERT INTO a_table (a, b) VALUES (2, 'bb')")
cur.execute("commit;")

# connection.commit()
cur.execute("INSERT INTO a_table (a, b) VALUES (1, 'aa')")
connection.commit()
```


**Copy** :

```python
cur = connection.cursor()
cur.copy("COPY test_copy (id, name) from stdin DELIMITER ',' ",  "1,foo\n2,bar")
```


## License

MIT License, please see `LICENSE` for details.


## Acknowledgements

Many thanks go to the contributors to the Ruby Vertica gem (https://github.com/sprsquish/vertica), since they did all of the wrestling with Vertica's protocol and have kept the gem updated. They are:

 * [Matt Bauer](http://github.com/mattbauer)
 * [Jeff Smick](http://github.com/sprsquish)
 * [Willem van Bergen](http://github.com/wvanbergen)
 * [Camilo Lopez](http://github.com/camilo)
