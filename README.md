Just some handy scripts.
=========================

rdat, r, tadr
---------------

This is a sort of generalization of `ggplotcmd` below. 

Have you ever wished you could use tidyr and related functions in shell pipelines, without having to load data into data frames within R first? Now you can! *Note: this version works only with tab-seperated tabular files, and produces tab-separated output. This may be adjusted in future (more featurefull) versions.*

Suppose we've got `data.txt` and `data_noheader.txt`, which are identical except the latter doesn't have a header row.

```
bash$ cat data.txt
name	score	homework
Joe	87	hw1
Jim	92	hw2
Kim	95	hw1
Kat	76	hw1
Len	21	hw2
Bob	68	hw2
bash$ cat data_noheader.txt
Joe	87	hw1
Jim	92	hw2
Kim	95	hw1
Kat	76	hw1
Len	21	hw2
Bob	68	hw2
```

Let's first work with `data.txt`, removing Kim's entry with `grep` (to illustrate working with existing pipelines):

```
bash$ cat data.txt | grep -v Kim
name	score	homework
Joe	87	hw1
Jim	92	hw2
Kat	76	hw1
Len	21	hw2
Bob	68	hw2
```

Now, suppose we want to compute the means of the `score` column, grouped by the `homework` column. First, we need a utility that "encodes" the data; this is `rdat`; from this point on in the pipeline the data is encoded inside an R data frame for usage by the `r` script (not plain text), so at the end we'll need to unencode it by passing the data through `tadr` (`rdat` spelled in reverse). 

Here's the pipeline, using `group_by()` and `summarize()` (from the `tidyr` package, which is imported by the `r` script, along with `dplyr` and `ggplot2` in this version).

```
bash$ cat data.txt | grep -v Kim | rdat | r 'group_by(homework)' | r 'summarize(mean_score = mean(score))' | tadr
 homework mean_score
      hw1   81.50000
      hw2   60.33333
```

In effect, `rdat` encodes the data as an R object and sends it out on standard out (using some black magic with `save()` and `load()`).
This is read into the `r` script and sent to the given bit of code with `%>%` from the `magrittr` package; the `r` script similary outputs its' results as encoded data. The `tadr` function reads encoded, but prints the results as plain text for display or further processing. (Currently only supports printing of data frame and vector data.) 

Be warned that `rdat` assumes that the input does have a header line (and no rownames). If the input does not have a header line,
`rdat` optional arguments with column names to use:

```
bash$ cat data_noheader.txt | grep -v Kim | rdat name score homework | r 'group_by(homework)' | r 'summarize(mean_score = mean(score))' | tadr
 homework mean_score
      hw1   81.50000
      hw2   60.33333
```

If fewer column names are given than columns, they will be used for the first columns, and the standard "V" column names will be used for the rest:

```
bash$ cat data_noheader.txt | grep -v Kim | rdat name score | tadr
 name score  V3
  Joe    87 hw1
  Jim    92 hw2
  Kat    76 hw1
  Len    21 hw2
  Bob    68 hw2
```

Thus, to work with a file without a header line, but just to use the standard V1 etc. column names, you can just use 'V1' as the first column name:

```
bash$ cat data_noheader.txt | grep -v Kim | rdat V1 | tadr
  V1 V2  V3
 Joe 87 hw1
 Jim 92 hw2
 Kat 76 hw1
 Len 21 hw2
 Bob 68 hw2
 ```

fasta_subset
-------------

Given a list of IDs to keep, extracts them from a fasta file.

More documentation to come here (maybe), see fasta_subset -h for details. #



ggplotcmd
---------

A quick command-line interface to ggplot2.

Requires: Rscript, library ggplot2. Also, currently it's set up specifically for OSX, you'll need to edit
a line or two for whatever you like to use to display PDFs.


Usage:
```
ggplotcmd file.txt 'stat_bin(aes(x = V1))' F
```
or, you can read on stdin (my favorite). Also, you can add layers etc and soforth:
```
cat data.txt | ggplotcmd - 'stat_bin(aes(x = V1)) + scale_x_continuous(limits = c(0,100))' F
```

If your data has header names, use the T flag:

Usage:
```
cat data.txt | ggplotcmd - 'stat_bin(aes(x = colname)) + scale_x_continuous(limits = c(0,100))' T
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
outerjoin file1.txt file2.txt file3.txt
cat file1.txt | outerjoin - file2.txt file3.txt
```

This script also uses fifo() to allow the use of redirect subshells (subprocesses), eg
```
cat file1.txt | outerjoin - <(cat file2.txt | grep 'whathaveyou') file3.txt
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
