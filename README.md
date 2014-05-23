Just some handy scripts. 
=========================

ggplot
-------

A quick command-line interface to ggplot2. 

Requires: Rscript, library ggplot2. Also, currently it's set up specifically for OSX, you'll need to edit
a line or two for whatever you like to use to display PDFs. 


Usage:
```
ggplotcmd file.txt 'stat_bin(aes(x = V1)) + scale_x_continuous(limits = c(0,100))' F
```
or, you can read on stdin (my favorite):
```
cat data.txt | ggplotcmd - 'stat_bin(aes(x = V1)) + scale_x_continuous(limits = c(0,100))' F
```

If your data has header names, use the T flag:

Usage: 
```
cat data.txt | ggplotcmd 'stat_bin(aes(x = colname)) + scale_x_continuous(limits = c(0,100))' T
```

outerjoin
----------

A quick utility for joining files by their first column.

Requires: Rscript, plyr library

This script performs a full outer join on a number of files using the first column of each as a common key.
The files may have a variable number of columns, but each file must have the same number of columns represeted in each row.
At least two files must be listed as an option, if you use '-' as a file name, the script will read one of them from standard input.

Example usages: 

```
outerjoin file1 file2
cat file1 | outerjoin - file2
```

This script also uses fifo() to allow the use of redirect subshells (subprocesses), eg 
```
cat file1 | outerjoin - <(cat file2 | grep 'whathaveyou') file3
```

The files need NOT be sorted, and rows will appear in the order of their first appearance in files
(that is, the IDs in file1 will be listed first, then the unique ids in file2, then the remaining unique ids in file 3, etc.)

Finally, if one or more of the files is simply an ID list, an additional column will be added with the file name ('-' in the
case of std input, or the name of the file descriptor in the case of subprocess redirects), this will indicate the presense of the ID in the file.
That is to say, if you join two files:
testa.txt:

```
ID1
ID4
ID2
```

and testb.txt
```
ID6
ID4
ID1
```
You'll get
```
ID1   testa.txt   testb.txt
ID4   testa.txt   testb.txt
ID2   testa.txt   NA
ID6   NA  testb.txt
```
