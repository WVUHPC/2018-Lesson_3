---
title: "Working with Compressed Files"
teaching: 45
exercises: 75
questions:
- "How to read data from compressed files without uncompressing them explicitly?"
objectives:
- "Using python we can read the contents of compressed files without explicit uncompress them."
keypoints:
- "Reading the contents of compressed files is easy with Python. We can use the zipfile, bz2, zlib, gzip and tarfile modules to achieve this. For the case of a zip file, we can using the ZipFile constructor. Get the list of files inside it using infolist() and open each of those files using open(). After that you can read from it just like a normal file."
---
FIXME

{% include links.md %}
