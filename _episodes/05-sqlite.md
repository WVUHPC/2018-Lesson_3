---
title: "Creating simple Databases with SQLite"
teaching: 15
exercises: 15
questions:
- "How to use SQLite?"
objectives:
- "Learn how to read and write data on a SQLite database file"
keypoints:
- "SQLite is a C library that provides a lightweight disk-based database that doesn’t require a separate server process and allows accessing the database using a nonstandard variant of the SQL query language. "
---

We will store the data on `crystals.json` inside a SQLite3 file-base database:

~~~
import json
import sqlite3
~~~
{: .source}

~~~
data=json.load(open("crystals.json"))
~~~
{: .source}

Creating a database
~~~
db = sqlite3.connect(':memory:')
db=sqlite3.connect("crystals.db")
~~~
{: .source}

Creating the Table for the 3 values, one string and two real numbers.

~~~
cursor=db.cursor()
cursor.execute('''CREATE TABLE crystals(id INTEGER PRIMARY KEY, formula TEXT, band_gap real, energy_pa real)''')
db.commit()
~~~
{: .source}

Inserting data:

~~~
for i in data:
    cursor.execute('''INSERT INTO crystals(formula,band_gap,energy_pa) VALUES(?,?,?)''',(i['formula'], i['band_gap'], i['energy_pa']))

db.commit()
~~~
{: .source}

With the code above all the entries in the original data in json are now stored in a database.

~~~
cursor.execute('''SELECT formula, band_gap, energy_pa FROM crystals''')
~~~
{: .source}

~~~
cursor.fetchone()
cursor.fetchall()
~~~
{: .source}

All the original values can be recovered:

~~~
cursor.execute('''SELECT formula, band_gap, energy_pa FROM crystals''')
all_rows = cursor.fetchall()
all_rows
for row in all_rows:
    print("{0} : {1} {2}".format(row[0],row[1],row[2]))
~~~
{: .source}    



{% include links.md %}
