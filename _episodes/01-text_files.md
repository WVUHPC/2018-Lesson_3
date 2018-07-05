---
title: "Processing Text Files with grep and awk"
teaching: 15
exercises: 15
questions:
- "How to use grep and awk to extra useful information from large text files?"
objectives:
- "Using grep and regular expressions to extract data"
keypoints:
- "Both grep a awk are powerful tools available from the command line"
---

Download the data for the lesson. There you will find a file called "output.log". That is an actual output file from a Molecular Dynamics code.

Lets start for getting an idea about the size number of lines of that file:

~~~
$ ls -al output.log
~~~
{: .language-bash}

~~~
-rw-r--r--  1 guilleaf  staff  4264619 Jul  4 22:19 output.log
~~~
{: .output}

The file is 4.1MB. That is a big file for a human to read. Lets see how many lines we can count on it

~~~
$ wc -l output.log
~~~
{: .language-bash}

~~~
92624 output.log
~~~
{: .output}

To get an idea, a normal book page has 30 lines. The text in that output file is equivalent to a book of around 3000 pages. Again, do not pretend to read such output like a book.

To get an idea about the contents of the file we can use less to navigate on the file from the first line.

~~~
$ less output.log
~~~
{: .language-bash}

We are interested on information about the "Time step", in "less" we can find a text content using the "/" command. Inside less enter "/" and the text "Time step" without the  quotes. To search for the next occurrence enter "n"

Leave "less" with the command "q".

The command "grep" can help us to extract the lines from "output.log" that contains the words "Time step" in exactly that case.

~~~
$ grep "Time step" output.log
~~~
{: .language-bash}

~~~
 EBS =         -4651.878461 Time step =     1 SCF step =    1
 EBS =         -3402.809950 Time step =     1 SCF step =    2
 EBS =         -4033.543868 Time step =     1 SCF step =    3
 EBS =         -4157.520527 Time step =     1 SCF step =    4
 EBS =         -3953.007514 Time step =     1 SCF step =    5
 EBS =         -3858.170888 Time step =     1 SCF step =    6
 EBS =         -3823.829702 Time step =     1 SCF step =    7
 EBS =         -3855.938308 Time step =     1 SCF step =    8
 EBS =         -3878.765498 Time step =     1 SCF step =    9
 EBS =         -3831.114003 Time step =     1 SCF step =   10
 EBS =         -3836.165174 Time step =     1 SCF step =   11
 EBS =         -3836.606761 Time step =     1 SCF step =   12
 EBS =         -3840.290377 Time step =     1 SCF step =   13
 EBS =         -3840.380671 Time step =     1 SCF step =   14
 EBS =         -3838.858239 Time step =     1 SCF step =   15
 EBS =         -3838.697136 Time step =     1 SCF step =   16
 EBS =         -3839.093080 Time step =     1 SCF step =   17
 EBS =         -3839.391689 Time step =     1 SCF step =   18
 EBS =         -3839.434217 Time step =     1 SCF step =   19
 EBS =         -3839.663931 Time step =     1 SCF step =   20
 EBS =         -3839.919566 Time step =     1 SCF step =   21
 EBS =         -3840.187570 Time step =     1 SCF step =   22
 EBS =         -3840.160399 Time step =     1 SCF step =   23
 EBS =         -3840.057741 Time step =     1 SCF step =   24
 EBS =         -3840.073968 Time step =     1 SCF step =   25
 EBS =         -3840.056858 Time step =     1 SCF step =   26 SCF success!
   Time step =      1 SCF step =  26 etot/atom =    -275.487042
 EBS =         -3840.090954 Time step =     2 SCF step =    1
 EBS =         -3840.100909 Time step =     2 SCF step =    2 SCF success!
   Time step =      2 SCF step =   2 etot/atom =    -275.487077
 EBS =         -3840.200857 Time step =     3 SCF step =    1
 EBS =         -3840.188603 Time step =     3 SCF step =    2
 EBS =         -3840.114068 Time step =     3 SCF step =    3
 EBS =         -3840.231523 Time step =     3 SCF step =    4 SCF success!
   Time step =      3 SCF step =   4 etot/atom =    -275.487182
 EBS =         -3840.394095 Time step =     4 SCF step =    1
 EBS =         -3840.358688 Time step =     4 SCF step =    2
 EBS =         -3840.158114 Time step =     4 SCF step =    3
 EBS =         -3840.304757 Time step =     4 SCF step =    4 SCF success!
   Time step =      4 SCF step =   4 etot/atom =    -275.487356
 EBS =         -3840.519359 Time step =     5 SCF step =    1
 EBS =         -3840.466959 Time step =     5 SCF step =    2
 EBS =         -3840.172580 Time step =     5 SCF step =    3
 EBS =         -3840.379226 Time step =     5 SCF step =    4 SCF success!
 ...
~~~
{: .output}

Notice that grep is capturing all places where the words "Time step" is found.
Lets imagine that we are interested only in capturing the lines that start with "Time step". The following command instruct grep to do that:


~~~
$ grep "^[ ]*Time step" output.log
~~~
{: .language-bash}

~~~
Time step =      1 SCF step =  26 etot/atom =    -275.487042
Time step =      2 SCF step =   2 etot/atom =    -275.487077
Time step =      3 SCF step =   4 etot/atom =    -275.487182
Time step =      4 SCF step =   4 etot/atom =    -275.487356
Time step =      5 SCF step =   4 etot/atom =    -275.487590
Time step =      6 SCF step =   4 etot/atom =    -275.487877
Time step =      7 SCF step =   4 etot/atom =    -275.488204
Time step =      8 SCF step =   4 etot/atom =    -275.488563
Time step =      9 SCF step =   4 etot/atom =    -275.488951
Time step =     10 SCF step =   4 etot/atom =    -275.489364
Time step =     11 SCF step =   4 etot/atom =    -275.489792
...
~~~
{: .output}

The "^" means to capture only the expression that starts with "Time step", excluding lines where those words appear in the middle. "[ ]\*" means to consider any number of spaces between the beginning of the line and the words we are looking for.

The lines that we are capturing are part of an important block of data that contains information about energetics for that particular time step. With "-A" and "-B" arguments, we can get some lines "Before" and "After" the line captured by our expression, see:

~~~
$ grep "^[ ]*Time step" -A 13 -B 1 output.log
~~~
{: .language-bash}

~~~
---------- T H E  T O T A L  E N E R G Y -----------
 Time step =      1 SCF step =  26 etot/atom =    -275.487042

           ebs =    -3840.056858
     uii - uee =   -31418.040456
     etotxc_1c =     1807.827145
        uxcdcc =      116.338043
          ETOT =   -33333.932126
     Etot/atom =     -275.487042
 Atomic Energy =   -32720.456910
     CohesiveE =     -613.475216

 Cohesive Energy per atom  =       -5.070043
-----------------------------------------------------

--
...
~~~
{: .output}



{% include links.md %}
