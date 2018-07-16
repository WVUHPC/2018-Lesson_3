---
title: "Using Regular Expressions with Python"
teaching: 15
exercises: 15
questions:
- "How to use Regular Expressions in python?"
objectives:
- "Learn how regular expressions can be used with python to extract data"
keypoints:
- "Python supports basically the same regular expression syntax as Perl. However, the syntax changes between Perl and Python and the way of using regular expressions is substantially different."
---

# Regular Expressions with python

Consider the following challenge.
We have the output from a simulation with some data that we would like to process, the problem now is that the data is not on a single like, so a simple grep will not work. The data we want to parse looks like this:

~~~
...
     14       7.7300      0.00000
     15       7.9145      0.00000
     16       8.7421      0.00000

 k-point 115 :       0.4444    0.3636    0.4286
  band No.  band energies     occupation
      1      -6.7076      1.00000
      2      -6.6256      1.00000
      3      -3.8932      1.00000
      4      -3.8031      1.00000
      5       0.1344      1.00000
      6       0.4871      1.00000
      7       0.7520      1.00000
      8       1.1131      1.00000
      9       3.2272      1.00000
     10       3.3574      1.00000
     11       7.4689      0.00000
     12       7.4905      0.00000
     13       7.7325      0.00000
     14       7.9343      0.00000
     15       8.3742      0.00000
     16       8.7648      0.00000

 k-point 116 :       0.0000    0.4545    0.4286
  band No.  band energies     occupation
      1      -7.0118      1.00000
      2      -6.8668      1.00000
      3      -4.4179      1.00000
...
~~~
{: .source}

This is quite complex set of data and we would like to take the different elements in such a way that we can manipulate them later on.

There are several ways of solving this problem, for example, knowing that each block of data starts with "k-point"
 and spans 17 rows. Such task could be done using just grep

~~~
grep -A 17 k-point OUTCAR
~~~
{: .source}

The argument "-A 17" will tell grep to show 17 rows after each occurrence of the line k-point.
We extract the line of information that we need but still is just a piece of text that is not so simple to manipulate.

With python we can achieve this task with just 4 lines, using the so called regular expressions, a way to explain a computer that we want extract text with some format by indicating the kind of data that we expect on the text.

This following script will extract the pieces still as text but, we will work on the conversion to text later.

~~~
import re
rf = open('OUTCAR')
data = rf.read()
kp = re.findall('k-point([\d\s]*):([\d\s.]*)band[\.\s\w]*occupation([\s\d:\-\.]*)\n\n', data)
~~~
{: .source}

The most cryptic part of this small script is understanding the meaning of all those symbols used as arguments for the findall function.
Lets start with a simpler version of the findall line an we will understand it piece by piece.

Using IPython lets start with executing the first 3 lines and we will explore the findall function step by step

~~~
import re
rf = open('OUTCAR')
data = rf.read()
~~~
{: .source}

Now, we start exploring this line

~~~
re.findall('k-point[\d\s]*:', data)
~~~
{: .source}

The output will look like this:

~~~
['k-point  1 :',
 'k-point  2 :',
 'k-point  3 :',
 'k-point  4 :',
~~~
{: .source}

The line in findall can be read like this: Search for text that start with `k-point` followed by 0 or more (that is the meaning of `*`) groups of characters (what is enclose by `[` and `]`) that can be either numbers `\d` or characters that looks like spaces `\s`

Now, if we just need the number, we can enclose the information that findall will return by enclosing it in parenthesis, like this:

~~~
re.findall('k-point([\d\s]*):', data)
~~~
{: .source}

At this point could be interesting to show how we can convert the list of strings returned by findall into actual numbers.
This could be done like this:

~~~
[int(x) for x in re.findall('k-point([\d\s]*):', data)]
~~~
{: .source}

Now lets move forward and get the next piece of information, the three numbers after colon, the numbers before the word 'band'

~~~
re.findall('k-point([\d\s]*):([\d\s.]*)band', data)
~~~
{: .source}

As you can see we are getting more information this time

~~~
[('   1 ', '       0.0000    0.0000    0.0000\n  '),
 ('   2 ', '       0.1111    0.0000    0.0000\n  '),
 ('   3 ', '       0.2222    0.0000    0.0000\n  '),
 ('   4 ', '       0.3333    0.0000    0.0000\n  '),
 ('   5 ', '       0.4444    0.0000    0.0000\n  '),
 ('   6 ', '       0.0000    0.0909    0.0000\n  '),
 ('   7 ', '       0.1111    0.0909    0.0000\n  '),
 ('   8 ', '       0.2222    0.0909    0.0000\n  '),
...
~~~
{: .source}

The output is a list of tuples, each tuple consisting of two strings. There is just one extra character on the regular expression, dot `.` is added to cover the existence of that character in the 3 numbers after colon. In regular expressions "dot" is used to match any character except a newline. Inside the `[]` special characters lose their special meaning, so `dot` here means just a `.`.

Now we can go to our final version of the `findall` function

~~~
re.findall('k-point([\d\s]*):([\d\s.]*)band[\s\w.]*occupation([\s\d:.\-]*)\n\n', data)
~~~
{: .source}

The meaning of all this cryptic code become far more clear now, the only notice here is that the character minus `-` needs still to be escaped like `\-` because it has a meaning for ranges inside `[]`. We close the regular expression with a double `\n\n`, indicating that each block is separated by a double newline.

Out final task is to convert the output from `findall` into actual numbers such that we can manipulate them for whatever purpose we need.

Lets do first a more simple exercise by storing correctly the k-point number and the three numbers after colon, they are the positions but their actual meaning is not important here.

~~~
ret=[]
for ikp in kp:
    entry={}
    entry['number']= int(ikp[0])
    entry['position']= [float(x) for x in ikp[1].split()]
    entry['values']= len(ikp[2].split())
    ret.append(entry)
~~~
{: .source}

What we are doing here is creating a list called \textbf{ret}
and for each element in our list kp we will create a python dictionary, converting the elements from the tuple into numbers, the first one will be integer, the second one is a set of three floating point numbers and for the third one we will just split the string into words and count the elements.

~~~
[{'number': 1, 'position': [0.0, 0.0, 0.0], 'values': 48},
 {'number': 2, 'position': [0.1111, 0.0, 0.0], 'values': 48},
 {'number': 3, 'position': [0.2222, 0.0, 0.0], 'values': 48},
 {'number': 4, 'position': [0.3333, 0.0, 0.0], 'values': 48},
 {'number': 5, 'position': [0.4444, 0.0, 0.0], 'values': 48},
 {'number': 6, 'position': [0.0, 0.0909, 0.0], 'values': 48},
...
~~~
{: .source}

For the position we use a list comprehension, a syntactic construct available in some programming languages for creating a list based on existing lists.

The conversion of values is a bit more elaborated. First, notice that the final element contain 49 elements due to a final string with several dashes. We would like to extract the numbers that are really relevant the floating point numbers. Lets consider just the final element from kp

~~~
In [44]: kp[-1]
Out[44]:
(' 120 ',
 '       0.4444    0.4545    0.4286\n  ',
 ' \n      1      -6.6226      1.00000\n      2      -6.5937      1.00000\n      3      -3.7536      1.00000\n      4      -3.7218      1.00000\n      5      -0.0498      1.00000\n      6       0.0803      1.00000\n      7       0.5565      1.00000\n      8       0.6896      1.00000\n      9       3.3146      1.00000\n     10       3.3603      1.00000\n     11       7.6585      0.00000\n     12       7.6740      0.00000\n     13       7.9721      0.00000\n     14       8.0356      0.00000\n     15       8.8014      0.00000\n     16       8.9382      0.00000\n\n\n--------------------------------------------------------------------------------------------------------\n')
~~~
{: .source}

Using Numpy we can easily get the information converted ready easily. Consider this line

~~~
import numpy
np.array(kp[-1][2].split()[:48], dtype=float).reshape(-1,3)
~~~
{: .source}

This line can be readed like this. Take the last element in kp (kp[-1]). Now take the third element of the tuple (`kp[-1][2]`). Split the string in words and make a list with the first 48 words encountered

~~~
kp[-1][2].split()[:48]
~~~
{: .source}

The final step is to convert those 48 strings into numbers as floating point numbers and reshape the whole array in 3 columns.

The final version of this part of the script will look like this:

~~~
ret=[]
for ikp in kp:
    entry={}
    entry['number']= int(ikp[0])
    entry['position']= [float(x) for x in ikp[1].split()]
    entry['values']= np.array(ikp[2].split()[:48], dtype=float).reshape(-1,3)
    ret.append(entry)
~~~
{: .source}

For reasons that will become clearer later we would like to keep everything as simple lists of numbers rather than numpy arrays. So we will serialize the numpy array into a list of lists

~~~
ret=[]
for ikp in kp:
    entry={}
    entry['number']= int(ikp[0])
    entry['position']= [float(x) for x in ikp[1].split()]
    entry['values']= np.array(ikp[2].split()[:48], dtype=float).reshape(-1,3).tolist()
    ret.append(entry)
~~~
{: .source}

> ## Exercises
>
> Using the file `output.log` from the previous episode. We need to extract the lines related to the forces, the look like this:
>
> ~~~
>   The grand total force (eV/A):
>   iatom =    1 ftot      =   0.363965E-01  0.227546E-01  0.189086E-01
>   iatom =    2 ftot      =   0.109832E+00  0.248904E+00  0.105642E+00
>   iatom =    3 ftot      =  -0.188313E-01 -0.577637E-01  0.156602E+00
>   iatom =    4 ftot      =  -0.973435E-01 -0.742613E-01 -0.481847E-01
>   iatom =    5 ftot      =   0.438460E-01 -0.165809E-04 -0.486130E-01
>   iatom =    6 ftot      =  -0.456146E-01 -0.246738E-01  0.288479E-01
>   iatom =    7 ftot      =  -0.150027E+00 -0.123757E+00 -0.515015E-01
>   ...
>   iatom =  114 ftot      =   0.114213E-01 -0.181273E-01  0.937052E-02
>   iatom =  115 ftot      =  -0.129298E-01 -0.184219E-01  0.203681E-01
>   iatom =  116 ftot      =  -0.257657E-01 -0.650361E-02  0.716669E-02
>   iatom =  117 ftot      =  -0.243455E-01  0.995426E-01 -0.324035E-02
>   iatom =  118 ftot      =  -0.294742E-02  0.437461E-01  0.122048E+00
>   iatom =  119 ftot      =   0.788438E-01 -0.137172E-01 -0.267298E-01
>   iatom =  120 ftot      =   0.135066E+00 -0.216788E-01 -0.425860E-01
>   iatom =  121 ftot      =  -0.503320E-01  0.130771E+00  0.534540E-01
>   
>   Cartesian Forces:  Max =       0.25029829    RMS =       0.04998891
> ~~~
> {: .source}
>
> 1. Use regular expressions to capture the blocks of data following the pattern
>
> 2. Extract the values of forces as a numpy array
>
> 3. Determine which atom has the largest and smallest magnitude of force.
>
>> ## Solution for the regex
>>
>> One option could be:
>>
>> ~~~
>> with open('output.log') as rf:
>>     data=rf.read()
>> re.findall("The grand total force \(eV/A\):[=\w\-+\.\d\s]*\n\s*\n",data)
>> ~~~
>> {: .source}
> {: .solution}
{: .challenge}



{% include links.md %}
