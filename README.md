# Goodreads Part I: CSV Files

We will use three main programming languages in this class: python, shell, and sql.
In this first part of the lab we will do some basic data exploration with each of these languages.

We will use data on user "interactions" from the <https://goodreads.com> website,
which will be explained in more detail below.

## Part 0: Setup

Log in to the VPN and lambda server.
Get access to the lab material files by running the commands
```
$ git clone https://github.com/mikeizbicki/lab-goodreads
$ cd lab-goodreads
$ ls
goodreads_interactions.csv  img  README.md
```

Recall that the `$` symbol is called the command prompt and indicates that everything after this symbol is a *shell command*.
(Shell commands are also sometimes called *terminal commands*.)
The lines that do not start with a `$` are the output from running the shell commands.
Sometimes, the output of a command is "uninteresting", in which case the output is not shown.
(For example, the `git clone` command above is likely to have a lot of output that is not shown.)
This is a standard practice when writing documentation.
You will see it throughout this class in both material that I write and external references.

You should follow along this tutorial by typing these commands into your terminal and verifying that you get similar output.
For example, if you get different output when running `ls`,
then something has gone wrong.

We are now ready to start working with python to analyze the `goodreads_interactions.csv` file.

## Part 1: Python and Pandas

Pandas is a popular python library for data analysis.
We will start by using pandas to get a quick overview of the csv file.

Start python in interactive mode by running the command
```
$ python3
```
You should get output that looks roughly like
```
Python 3.6.9 (default, Nov 25 2022, 14:10:45)
[GCC 8.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>>
```
The first three lines are basic information about your version of python,
and the last line, beginning with `>>>` is the python prompt.
Whenever in a tutorial you see code that starts with `>>>`,
you know that code should be entered on a terminal in python interactive mode.

Next, run the following two commands.
```
>>> import pandas as pd
>>> df = pd.read_csv('goodreads_interactions.csv')
```
The `pd.read_csv` function loads the csv file into a pandas [`DataFrame`](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.html) object.
The pandas `DataFrame` is a python implementation of the [R data frame](https://www.rdocumentation.org/packages/base/versions/3.6.2/topics/data.frame);
it is a standard representation for tabular data like you might find in an excel spreadsheet.

We can calculate the size of our dataset using the built-in `len` function.
```
>>> len(df)
9999
```
As you can see, this is a relatively small dataset.
<!--
It is common practice to start working on a small sample before moving to the full dataset because the smaller dataset is faster and easier to work with.
-->
After exploring this smaller dataset, we will switch to a larger version of this dataset that contains millions of rows.
We will see that some techniques that work for the smaller dataset don't work for the larger one.

Next, we can manually inspect the data by just printing the `df` object directly
```
>>> df
      user_id  book_id  is_read  rating  is_reviewed
0           0      948        1       5            0
1           0      947        1       5            1
2           0      946        1       5            0
3           0      945        1       5            0
4           0      944        1       5            0
5           0      943        1       5            0
6           0      942        1       5            0
7           0      941        1       5            0
8           0      940        1       5            0
9           0      939        1       5            1
...
```
We can see that this CSV file has 5 columns:

- The `user_id` and `book_id` columns contain unique identifiers for each user and book on the <https://goodreads.com> website.

- `is_read` is set to `0` if the user has not read the book, and set to `1` if they have.

- `rating` is set to `0` if the user has not rated the book, otherwise it is the number of stars between 1-5 that the user assigned the book.

- `is_reviewed` is set to `0` if the user has not left a detailed text review of the book, and `1` if they have.

### Part 1.a: Querying with Pandas

We will now look at two interesting queries that we might want to run on this dataset.

The first is the "count distinct" query,
where we want to count the total number of distinct entries in a particular column.
In pandas, we can calculate the total number of distinct `user_id`s by using the `.nunique()` method like so:
```
>>> df['user_id'].nunique()
15
```

> **Exercise:**
>
> Write a command that counts the total number distinct values in the `rating` column.
>
> **Note:**
>
> There will be several exercise problems like this one throughout the lab.
> These exercises are designed to help you check your own understanding of the material,
> and you do not have to turn in your answers.
> Nevertheless, you should not move on to the next portion of the lab until you understand how to complete the exercise.
> I recommend you work with a partner to complete these exercises and check your work.

The second interesting query we'll consider is the "group count" query,
where we want to count the total number of times each of these distinct values appears.
For example, to see the total number of books each user has interacted with, we can run the command
```
>>> df.groupby('user_id').size()
user_id
0      949
1      136
2      204
3      309
4      188
5     5479
6      117
7      142
8      242
9      280
10     124
11     514
12     123
13     471
14     721
```

> **Exercise:**
>
> Write a command that counts the total number of times each rating has been given.

There are many more things we can do with a pandas `DataFrame`,
but we won't explore those for now.
Instead, we'll run similar queries on the larger version of this dataset.

### Part 1.b: Loading the Larger Dataset

The full dataset is stored in the `/data-fast/goodreads/` folder of the lambda server in a file called `goodreads_interactions.csv.gz`.
The `.gz` extension means the file has been compressed using the [gzip algorithm](https://en.wikipedia.org/wiki/Gzip).
Compressing large datasets is standard practice to save disk space,
and pandas is able to automatically detect this compression and load the dataset for us.
Load the file into python with the command
```
>>> df = pd.read_csv('/data-fast/goodreads/goodreads_interactions.csv.gz')
```

> **Recall:**
>
> The path used in the command above is called an *absolute path* because it starts with a leading `/` character.
> The path we used to access our smaller dataset (`goodreads_interactions.csv`) is called a *relative path* because it does not start with a leading `/`.
> Absolute paths always refer to the same file no matter what your current working directory is.
> The relative path refers to whatever file is in your current folder,
> and this file will change if you use the `cd` command to change your current working folder.

After a minute or so, you should get a large [python traceback](https://realpython.com/python-traceback/) that ends with 
```
MemoryError
```
Oh no!

The problem is that pandas is trying to load the entire csv file into memory at once,
and so the process runs out of memory and crashes.
Your user accounts are allowed to use up to 16GB of memory on the lambda server,
and so you're encountering this `MemoryError` exception because python is trying to allocate more than 16GB for the data frame.

A fundamental principle of working with big data is that we need algorithms with only $O(1)$ memory usage.
<!--
Many computers today have more than 16GB of RAM available,
and so could load the full `goodreads_interactions.csv.gz` into a pandas data frame.
-->
Later in this class we will be working with datasets that are terabytes or petabytes in size,
and so this $O(1)$ memory usage requirement will become even more important.
Unfortunately, pandas is not designed to use constant space memory,
and so it is not suitable for large scale data analysis problems.

<!--
<img src=img/pandas-bad1.jpg width=300px />
-->

Python has many other libraries for providing dataframe-like functionality that are more efficient.
Two popular ones are [dask](https://docs.dask.org/en/stable/) and [polars](https://realpython.com/polars-python/).
Unfortunately, they are more awkward to use and have fewer features.
For this reason, most big data analysis is not done in just python.
Instead, it is much more common to use the POSIX shell and SQL,
occasionally combining these tools with python.
Tools like Jupyter Notebooks do not combine well with these other languages.
They are therefore not widely used for big data projects,
and we won't be using them in this class.

We'll now turn to reviewing the basics of POSIX and SQL.
You should leave the interactive python session by typing `CTRL-D` in the terminal.
(For Mac users, ensure that you are hitting the `CTRL` key and not the `Command` key.)

## Part 2: The POSIX Shell

The POSIX shell is one of the oldest and most widely used programming environments.
It is especially useful for large scale data science applications.
<!--
There are many "dialects" of the shell, but bash is the most popular.
We will be writing many bash scripts throughout this class.

<img src=img/bash.jpeg />
-->

The shell has many built-in utilities useful for working with data sets.
These utilities are all designed to work efficiently on large scale files.
Most of these programs use $O(1)$ memory and $O(n)$ compute.

The first example we'll see is the `wc` command.
`wc` stands for "word count" and can be used to count the number of lines in a file if we pass the `-l` command line argument.
Run the following command to count the number of lines in our example csv file:
```
$ wc -l goodreads_interactions.csv
10000
```
The `head` command prints the first 10 lines of a file,
and it is useful for manually inspecting the csv file to see its structure:
```
$ head goodreads_interactions.csv
user_id,book_id,is_read,rating,is_reviewed
0,948,1,5,0
0,947,1,5,1
0,946,1,5,0
0,945,1,5,0
0,944,1,5,0
0,943,1,5,0
0,942,1,5,0
0,941,1,5,0
0,940,1,5,0
```
With both of these commands, we can see the same information that we extracted from the pandas data frame.

> **Note:**
> 
> The `wc -l` command outputs `10000`, but the `len(df)` command returned `9999`.
> The output of the `head` command explains this discrepancy.
> The first line of the csv file is the column headers, and not an actual data point.
> So we need to subtract 1 from the output of `wc -l` to get the total number of data points.

> **Historical Aside:**
>
> We will explore many powerful features of the POSIX shell in this class,
> but one of the widely acknowledged downsides of the shell is the arcane command names.
> For example, the two-letter name `wc` is awkward to modern programmers,
> especially those coming from a python background where self-explanatory function names are common.
> `wc` stands for "word count", and the program was originally written in the early 1970s when computer memory was so limited that programs had to have very short names.
> For backwards compatibility reasons, the name of the program can't be changed at this point.
> Python's [benevolent dictator for life (BFDL)](https://en.wikipedia.org/wiki/Benevolent_dictator_for_life) [Guido van Rossum](https://en.wikipedia.org/wiki/Guido_van_Rossum) began development of the python programming language in the 1980s in order to overcome limitations like this with shell scripting and make programming more user friendly.
> By the 1980s, computers had larger memory and could store longer program names.

<!--
> The `wc` program in use on most linux systems today is part of the GNU `coreutils` package.
> You can find the source code at <https://github.com/coreutils/coreutils/blob/master/src/wc.c>.
> It is a 981 line long C program.
-->

Next we will see how to apply the `wc` and `head` programs to the full dataset.
Run the command
```
$ head /data-fast/goodreads/goodreads_interactions.csv.gz
```
You should notice a large amount of gibberish output.
This is because the file `goodreads_interactions.csv.gz` has been compressed,
and `head` does not natively understand how to read compressed files.

The command `zcat` is used to read compressed files.
Try it by running the command
```
$ zcat /data-fast/goodreads/goodreads_interactions.csv.gz
```
You should notice this program prints a LOT of text,
and this text should look like the contents of a csv file.
It will probably take `zcat` a few hours to finishing printing all of this text.
(For various complicated reasons, printing is slow.)
So you should press `CTRL-C` to stop the program.
In general, `CTRL-C` can be used to stop any long-running shell command.

Now what we need to do is to use the output of the `zcat` command as the input to the `head` command in order to view just the first lines of our compressed file.
We do this in the shell using the `|` operator (pronounced as the *pipe operator*).
The pipe takes the output of the first command and connects it to the input of the second command.
For example:
```
$ zcat /data-fast/goodreads/goodreads_interactions.csv.gz | head
```
takes the output of `zcat` (which we saw prints the full contents of the decompressed file to the screen) and connects it to the input of `head` (which we've seen prints only the first 10 lines of the input).
You should get output that looks like
```
user_id,book_id,is_read,rating,is_reviewed
0,948,1,5,0
0,947,1,5,1
0,946,1,5,0
0,945,1,5,0
0,944,1,5,0
0,943,1,5,0
0,942,1,5,0
0,941,1,5,0
0,940,1,5,0
```
and the command should complete instantaneously.
These tools are all designed to be extremely efficient.
They had to be in order to work on 1970s computers!

The [Unix philosophy](https://en.wikipedia.org/wiki/Unix_philosophy) is that all programs "should do one thing well" and "work together".
Using pipes to combine programs like this is therefore an extremely common practice.

> **Historical Aside:**
>
> The Unix philosophy has been widely adopted by modern programming languages in various forms.
> For example, the R programming language has two pipe-like operators `|>` and `%>%`.
> These operators are inspired by the Unix pipe and have the same *semantics* of connecting the output of one program to the input of another.
> Unlike Unix pipes, however, R pipes are typically not efficient and so cannot be used on large datasets.
> The difference between the efficiency of R and Unix pipes is entirely cultural.
> Unix programs are typically built by computer scientists who care deeply about performance of their code,
> whereas R programs are normally built by statisticians who have less understanding of how to write efficient code.

In order to count the number of lines in the full dataset, we will pipe the results of the `zcat` command to the `wc -l` command:
```
$ zcat /data-fast/goodreads/goodreads_interactions.csv.gz | wc -l
```
This command should take about a minute to run.
(The `head` command finished instantly because it did not have to loop over the entire file.
The `wc -l` command, however, does have to loop over the entire file.)
You should get that there are over 228 million data points.

<!--
228648343 /data-fast/goodreads/goodreads_interactions.csv
-->

> **Exercise:**
> 
> The goodreads dataset contains many other files.
> In the next portion of the lab, we will use the file `goodreads_reviews_dedup.json.gz` to extract user reviews.
> Each line of this file contains a single user review.
> Combine the `zcat` and `wc` commands to count the total number of user reviews.
> Recall that you'll need to use an absolute path to access the `goodreads_reviews_dedup.json.gz` file.

One disadvantage of the POSIX shell is that complicated queries (like the "count distinct" and "group count" queries we did above in pandas) can be difficult or impossible to write.
Normally we combine the shell with other tools (like python or SQL) to accomplish these tasks.
In a later homework for this class, we will see how to write the count distinct and group by queries in the shell.
But now we turn to reviewing SQL.

## Part 3: SQL

We will now turn our attention to SQL and how to combine SQL with the shell.
SQL is a standard language for creating databases and extracting information from the databases.
There are many "dialects" of SQL.
For this lab, we will use SQLite.
SQLite is the most widely deployed piece of software in the world, [with over 1 trillion (1e12) active databases in use](https://sqlite.org/mostdeployed.html).
Most phones contain 100s of SQLite databases,
and even many kitchen appliances contain SQLite databases.
One of the main reasons for its popularity is that SQLite is easy to combine with other programming languages like the POSIX shell.

<!--
> **Historical Aside:**
>
> Many open source projects have adopted a "code of conduct" describing how contributors to the project should behave.
> SQLite is unique in that [its code of conduct is the Rule of St Benedict](https://sqlite.org/codeofethics.html)

> **Historical Aside:**
>
> There used to be a `sqlite4`, but [it is now depricated](https://sqlite.org/src4/doc/trunk/www/index.wiki).
> `sqlite3` is the latest (and probably last) version of the software.
-->

As before, we'll start by analyzing the smaller `goodreads_interactions.csv` file.
In the shell, run the command
```
$ sqlite3 goodreads.db
```
This creates and opens the file `goodreads.db` as a sql database.
You will see output that looks something like
```
SQLite version 3.44.2 2023-11-24 11:41:44
Enter ".help" for usage hints.
sqlite>
```
The prompt `sqlite>` lets us know that we are now interacting with the `sqlite3` program and not the shell.

We will start by loading the uncompressed csv.
`sqlite3` supports this with the `.import` command:
```
sqlite> .import goodreads_interactions.csv interactions --csv
```
The first parameter of the `.import` command is a file to load,
the second is the name of a SQL table to store the data in,
and the `--csv` indicates that the file is in CSV format.
Verify the command was successful by running the command
```
sqlite> .schema
```
which should give you output that looks similar to
```
CREATE TABLE IF NOT EXISTS "interactions"(
"user_id" TEXT, "book_id" TEXT, "is_read" TEXT, "rating" TEXT,
 "is_reviewed" TEXT);
```
We will discuss how to interpret this output in more detail later in the class.
But basically this is telling us that there is a single table in the database called `interactions` with 5 columns (`user_id`, `book_id`, `is_read`, `rating`, and `is_reviewed`) that correspond to the 5 columns of the csv file we've already seen.

We can count the total number of data points with the following SQL command:
```
sqlite> select count(*) from interactions;
9999
```
And we can manually inspect the first 10 rows with the command:
```
sqlite> select * from interactions limit 10;
0|948|1|5|0
0|947|1|5|1
0|946|1|5|0
0|945|1|5|0
0|944|1|5|0
0|943|1|5|0
0|942|1|5|0
0|941|1|5|0
0|940|1|5|0
0|939|1|5|1
```
By default, the output of `select` queries is designed to be more machine-friendly than human-friendly.
You can change the output to be the more human-friendly markdown format with SQLite's `.mode` command.
```
sqlite> .mode markdown
sqlite> select * from interactions limit 10;
| user_id | book_id | is_read | rating | is_reviewed |
|---------|---------|---------|--------|-------------|
| 0       | 948     | 1       | 5      | 0           |
| 0       | 947     | 1       | 5      | 1           |
| 0       | 946     | 1       | 5      | 0           |
| 0       | 945     | 1       | 5      | 0           |
| 0       | 944     | 1       | 5      | 0           |
| 0       | 943     | 1       | 5      | 0           |
| 0       | 942     | 1       | 5      | 0           |
| 0       | 941     | 1       | 5      | 0           |
| 0       | 940     | 1       | 5      | 0           |
| 0       | 939     | 1       | 5      | 1           |
```

> **Note:**
>
> Commands starting with a `.` like `.import` and `.mode` are specific to `sqlite3` and will not work with other database engines.
> All other commands (like `select`) are generic SQL commands that will work with other databases.
> We will explore this distinction in more detail later in the semester when we work with the Postgres database.
> Postgres has a lot more features than SQLite, but is harder to setup.

Now leave the `sqlite3` environment by pressing `CTRL-D`.
Run
```
$ ls
```
and observe that the file `goodreads.db` has been added to your current directory.

We can directly run queries on this database from the shell by passing those queries to the `sqlite3` command as a command line argument.
For example, we can count the number of rows in the table `interactions` by running
```
$ sqlite3 goodreads.db 'select count(*) from interactions;'
```

We are now ready to load our large, compressed dataset into a SQLite database.
This will allow us to finally by able to run our "select distinct" and "group count" queries on the full dataset.

Like with the `head` and `wc` programs, `sqlite3` does not natively understand compression.
Therefore, we must combine it with the `zcat` program to access compressed files.
To connect `sqlite3` to `zcat`, we will use the special file `/dev/stdin`.
The details are complicated, but you can think of this file as containing whatever the contents of the pipe operator are.
Combining all of this together, the following incantation loads the compressed dataset into a new database called `goodreads2.db`.
```
$ zcat /data-fast/goodreads/goodreads_interactions.csv.gz | sqlite3 goodreads2.db '.import /dev/stdin interactions --csv'
```
It takes a long time (about 3 minutes for me) to run,
but now we can use this new database file to run a variety of queries using only $O(1)$ memory.
<!--
After the command finishes, leave `sqlite3` by pressing `^D` on an empty line.

> **Note:**
>
> The up-carat symbol `^` represents the `CTRL` key on your keyboard,
> and so `^D` means pressing the `CTRL` and `D` keys at the same time.
> (For Max users, ensure that you are pressing `CTRL` and not `Command`.)
>
> For a fun rabbit hole, see the [Wikipedia article on carat notation](https://en.wikipedia.org/wiki/Caret_notation) and the related articles.

Run the `ls` command.
You should notice a new file by the name of `goodreads.db` that was created by the `sqlite3` command.
This is a rather large file
```
$ du -h goodreads.db
```
-->

<!--
$ /usr/bin/time -v /home/mizbicki/tmp/sqlite-autoconf-3440200/sqlite3 temp.db <<< '.import /data-fast/goodreads/goodreads_interactions.csv interactions'
-->

Load the new database in interactive mode with the command
```
$ sqlite3 goodreads2.db
```
You can now count the number of rows in the table with the following command:
```
sqlite> select count(*) from interactions;
228648342
```
Notice that this result ran much faster than our previous `wc -l` command and gives us the same answer.
These database files have special structure inside of them that allows certain queries to run faster.
CSV files (and JSON files) are just text files with no special structure.
We often call files without special structure *flat* files.

We can now finally run our more complicated queries on this dataset.
The following SQLite commands compute the total number of distinct users that have interacted with books,
and the total number of ratings that have been assigned.
```
sqlite> .mode markdown
sqlite> select count(distinct user_id) from interactions;
| count(distinct user_id) |
|-------------------------|
| 876145                  |
sqlite> select rating, count(*) from interactions group by rating;
| rating | count(*)  |
|--------|-----------|
| 0      | 124096793 |
| 1      | 2050529   |
| 2      | 6189946   |
| 3      | 23307457  |
| 4      | 37497451  |
| 5      | 35506166  |
```
Each of these commands takes 1-2 minutes for me.
Some SQL commands are fast by default (like `count(*)`),
and some are not.
We will talk in detail throughout this semester about why some commands are fast and some slow,
and how to speed up these slow commands so that they return results instantly.
Websites (like <https://goodreads.com>) need to be able to return results instantly or they will lose users.
[Google even applies a penalty to a website's ranking if it is slower than 200ms](https://developers.google.com/speed/docs/insights/Server).
By the end of this course, you'll be able to compute these queries (and many more complicated ones) in under 200ms.

## Submission

Modify the two `select` queries above in order to calculate:

1. The total number of distinct `book_id`s in the CSV file.

2. The total number of books that have been reviewed or not reviewed.  (That is, do a "group by" query on the `is_reviewed` column.)

Paste your queries and their results into sakai.
