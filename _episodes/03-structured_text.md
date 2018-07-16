---
title: "Structurated Text (XML and JSON)"
teaching: 15
exercises: 15
questions:
- "How to extract data from XML and JSON files?"
objectives:
- "Using python to navigate XML text and read and write JSON files"
keypoints:
- "XML is simpler than SGML, but JSON is much simpler than XML. JSON has a much smaller grammar and maps more directly onto the data structures used in modern programming languages.
Both format are commonly used as exchange format for transferring data between applications."
---

# Structured File Formats (XML and JSON)

Let retake the case we were analyzing in our previous episode:

~~~
import re
rf = open('OUTCAR')
data = rf.read()
kp=re.findall('k-point([\d\s]*):([\d\s.]*)band[\s\w.]*occupation([\s\d:.\-]*)\n\n', data)

ret=[]
for ikp in kp:
    entry={}
    entry['number']= int(ikp[0])
    entry['position']= [float(x) for x in ikp[1].split()]
    entry['values']= np.array(ikp[2].split()[:48], dtype=float).reshape(-1,3).tolist()
    ret.append(entry)
~~~
{: .source}

It is time for us to save the data that we parse in something that allow us to recover later.

If we have just one table and a very plain structure like rows and columns we can just use a 'csv' file or even a simple table with numbers in columns.
But in more complex scenarios, a single table is not enough. If you can organized the data hierarchically, like a tree or a set of keys and values, where values can be arrays, it is convenient to use a structured format instead of a human-readable but not computer readable file format.

We will present two options for storing data, using well know and standard format for structured data, JSON and XML.

## JSON

JSON (JavaScript Object Notation) is a lightweight data-interchange format. It is easy for humans to read and write. It is easy for machines to parse and generate.

JSON is a good choice, for data that will be used on the web, data that has a hierarchical structure and the number of numerical values is reasonable. For storing large amounts of numerical values, using a text-based format comes with a huge computational and storage overhead and binary formats are the solution.

### JSON in Python

The JSON module in python offers convenient functions to convert simple variables such as list and dictionaries into strings that could be stored in text files such that their contents could be easily retrieved.

Right now, the variable `ret` is a list of dictionaries where each of them contains either single numbers, lists or lists of lists. We can store that in a JSON file such that we can recover that information easily.

Try first executing something like this

~~~
import json
json.dumps(ret)
~~~
{: .source}

Not easy to read for a human but that long string can be easily understood by a computer to recover the information you stored in it. Try this version for something more human-readable.

~~~
import json
json.dumps(ret, sort_keys=True, indent=4, separators=(',', ': '))
~~~
{: .source}

Lets now store ret into a file and read it again to test we can recover the file.

~~~
wf = open('k-points.json','w')
json.dump(ret, wf, sort_keys=True, indent=4, separators=(',', ': '))
wf.close()
~~~
{: .source}

Now lets test recovering the data from the file.

~~~
rf2=open('k-points.json')
json.load(rf2)
~~~
{: .source}

Finally, lets summarize all that we learn with this example.
The whole script will be listed here:

~~~
#!/usr/bin/env python

import re
import numpy as np
import json

rf = open('OUTCAR')
data = rf.read()

# Parsing of the data
kp=re.findall('k-point([\d\s]*):([\d\s.]*)band[\s\w.]*occupation([\s\d:.\-]*)\n\n', data)

# Giving structure to the data
ret=[]
for ikp in kp:
    entry={}
    entry['number']= int(ikp[0])
    entry['position']= [float(x) for x in ikp[1].split()]
    entry['values']= np.array(ikp[2].split()[:48], dtype=float).reshape(-1,3).tolist()
    ret.append(entry)

# Storing the results into a JSON file
wf = open('k-points.json','w')
dp = json.dump(ret, wf, sort_keys=True, indent=4, separators=(',', ': '))
wf.close()
~~~
{: .source}

### JSON in R

You can get support for JSON file-format using several packages like `rjson` or `jsonlite`. We will demonstrate the use of `jsonlite`

On folder `2018-Data-HandsOn/3.Data/3.json_xml`, there is a file called `crystals.json`. Lets import its contents as a dataframe in R

~~~
> library('jsonlite')
> data<-fromJSON("crystals.json")
> data[1,]
  band_gap energy_pa formula
1     6.69  -6.71326    F3La
> head(data)
  band_gap energy_pa formula
1    6.690 -6.713260    F3La
2    2.512 -3.860909  ClNaO2
3       NA        NA     AgO
4    8.281 -6.581713    CeF3
5    7.276 -5.755375 H2LiO3P
6    2.011 -4.378286  BaS3Te
>
> nrow(data)
[1] 471857
~~~
{: .language-r}

## XML

XML stands for Extensible Markup Language (XML). Similar to HTML it contains markup tags. XML is a file format which shares both the file format and the data on the World Wide Web, intranets, and elsewhere using standard ASCII text.

Being more verbose than JSON, the big advantage for XML is being widely adopted by the computer industry and many programming languages and data applications offer support  for it.

### XML in R

Using XML in R involves the use of the `XML` library in R. The success reading a XML depends in a great deal of how correct is the XML data that you are trying to parse. For things like Webpages with missing tags and improper nodes. The simple function that we present here will not work.

On folder `2018-Data-HandsOn/3.Data/3.json_xml` there is a file `crystals_10k.xml`, corresponding to the first 10K entries on the same JSON file above. As the natural structure in R is a dataframe, the objective will be convert the XML file into that.

When everything goes smoothly the process is very simple:

~~~
library(XML)
df<-xmlToDataFrame("crystals_10k.xml")
~~~
{: .language-r}

The alternative is far more evolved, having to deal with each individual node, and eventually attributes and values as shown in the example below:

~~~
> data<-xmlParse('crystals_10k.xml')
> rootnode<-xmlRoot(data)
> rootnode[1]
$CRYSTAL
<CRYSTAL>
  <FORMULA>F3La</FORMULA>
  <ENERGY_PA>-6.71325961083333</ENERGY_PA>
  <BAND_GAP>6.69</BAND_GAP>
</CRYSTAL>

attr(,"class")
[1] "XMLInternalNodeList" "XMLNodeList"
> xmlChildren(rootnode[[1]])
$FORMULA
<FORMULA>F3La</FORMULA>

$ENERGY_PA
<ENERGY_PA>-6.71325961083333</ENERGY_PA>

$BAND_GAP
<BAND_GAP>6.69</BAND_GAP>

attr(,"class")
[1] "XMLInternalNodeList" "XMLNodeList"
> one<-xmlChildren(rootnode[[1]])
> one
$FORMULA
<FORMULA>F3La</FORMULA>

$ENERGY_PA
<ENERGY_PA>-6.71325961083333</ENERGY_PA>

$BAND_GAP
<BAND_GAP>6.69</BAND_GAP>

attr(,"class")
[1] "XMLInternalNodeList" "XMLNodeList"
> one$FORMULA
<FORMULA>F3La</FORMULA>
> xmlChildren(one$FORMULA)
$text
F3La

attr(,"class")
[1] "XMLInternalNodeList" "XMLNodeList"
> xmlChildren(one$FORMULA)$text
F3La
~~~
{: .language-r}


{% include links.md %}
