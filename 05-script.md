---
layout: page
title: The Unix Shell
subtitle: Shell Scripts
minutes: 15
---
> ## Learning Objectives {.objectives}
>
> *   Write a shell script that runs a command or series of commands for a fixed set of files.
> *   Run a shell script from the command line.
> *   Write a shell script that operates on a set of files defined by the user on the command line.
> *   Create pipelines that include user-written shell scripts.

We are finally ready to see what makes the shell such a powerful programming environment.
We are going to take the commands we repeat frequently and save them in files
so that we can re-run all those operations again later by typing a single command.
For historical reasons,
a bunch of commands saved in a file is usually called a **shell script**,
but make no mistake:
these are actually small programs.

Let's start by going back to `~/avida/cbuild/work/data` and putting the following line in the file `middle.sh`:

~~~ {.bash}
$ cd ~/avida/cbuild/work/data
$ nano middle.sh
~~~
~~~
head -22 average.dat | tail -3
~~~

This is a variation on the pipe we constructed earlier:
it selects lines 19-22 of the file `average.dat`.
Remember, we are *not* running it as a command just yet:
we are putting the commands in a file.

Once we have saved the file,
we can ask the shell to execute the commands it contains.
Our shell is called `bash`, so we run the following command:

~~~ {.bash}
$ bash middle.sh
~~~
~~~ {.output}
0 97 389 0 0 0 100 97 0 1 1 0 0 0 0 0 
100 94.0267 386.707 0.242776 0 0 98.76 94.0267 0 0.04 0 0 7 1.71205 0 0 
200 94.226 388.904 0.243294 0 0 100.59 95.4251 0 0.034398 0.02457 0 14.8722 1.14454 0 0 
~~~

Sure enough,
our script's output is exactly what we would get if we ran that pipeline directly.

> ## Text vs. Whatever {.callout}
>
> We usually call programs like Microsoft Word or LibreOffice Writer "text
> editors", but we need to be a bit more careful when it comes to
> programming. By default, Microsoft Word uses `.docx` files to store not
> only text, but also formatting information about fonts, headings, and so
> on. This extra information isn't stored as characters, and doesn't mean
> anything to tools like `head`: they expect input files to contain
> nothing but the letters, digits, and punctuation on a standard computer
> keyboard. When editing programs, therefore, you must either use a plain
> text editor, or be careful to save files as plain text.

What if we want to select lines from an arbitrary file?
We could edit `middle.sh` each time to change the filename,
but that would probably take longer than just retyping the command.
Instead,
let's edit `middle.sh` and replace `average.dat` with a special variable called `$1`:

~~~ {.bash}
$ cat middle.sh
~~~
~~~ {.output}
head -22 "$1" | tail -3
~~~

Inside a shell script,
`$1` means "the first filename (or other parameter) on the command line".
We can now run our script like this:

~~~ {.bash}
$ bash middle.sh average.dat
~~~
~~~ {.output}
0 97 389 0 0 0 100 97 0 1 1 0 0 0 0 0 
100 94.0267 386.707 0.242776 0 0 98.76 94.0267 0 0.04 0 0 7 1.71205 0 0 
200 94.226 388.904 0.243294 0 0 100.59 95.4251 0 0.034398 0.02457 0 14.8722 1.14454 0 0 
~~~

or on a different file like this:

~~~ {.bash}
$ bash middle.sh count.dat
~~~
~~~ {.output}
0 30 1 1 1 0 0 0 1 0 1 1 1 1 0 0 
100 2190 75 51 7 0 0 0 3 1 0 28 39 75 0 0 
200 12060 407 311 38 0 0 0 14 9 10 150 235 407 0 0 
~~~


> ## Double-Quotes Around Arguments {.callout}
>
> We put the `$1` inside of double-quotes in case the filename happens to contain any spaces.
> The shell uses whitespace to separate arguments,
> so we have to be careful when using arguments that might have whitespace in them.
> If we left out these quotes, and `$1` expanded to a filename like
> `methyl butane.pdb`,
> the command in the script would effectively be:
>
>     head -15 methyl butane.pdb | tail -5
>
> This would call `head` on two separate files, `methyl` and `butane.pdb`,
> which is probably not what we intended.


We still need to edit `middle.sh` each time we want to adjust the range of lines,
though.
Let's fix that by using the special variables `$2` and `$3`:

~~~ {.bash}
$ cat middle.sh
~~~
~~~ {.output}
head "$2" "$1" | tail "$3"
~~~
~~~ {.bash}
$ bash middle.sh resource.dat -9 -3
~~~
~~~ {.output}
0
100
200
~~~

We didn't add any resources to the environment, so there are no columns
besides update!

This works,
but it may take the next person who reads `middle.sh` a moment to figure out what it does.
We can improve our script by adding some **comments** at the top:

~~~ {.bash}
$ cat middle.sh
~~~
~~~ {.output}
# Select lines from the middle of a file.
# Usage: middle.sh filename -end_line -num_lines
head "$2" "$1" | tail "$3"
~~~

A comment starts with a `#` character and runs to the end of the line.
The computer ignores comments,
but they're invaluable for helping people understand and use scripts.

What if we want to process many files in a single pipeline?
For example, if we want to sort our `.dat` files by length, we would type:

~~~ {.bash}
$ wc -l *.dat | sort -n
~~~

because `wc -l` lists the number of lines in the files
(recall that wc stands for 'word count', adding the -l flag means 'count lines' instead)
and `sort -n` sorts things numerically.
We could put this in a file,
but then it would only ever sort a list of `.dat` files in the current directory.
If we want to be able to get a sorted list of other kinds of files,
we need a way to get all those names into the script.
We can't use `$1`, `$2`, and so on
because we don't know how many files there are.
Instead, we use the special variable `$@`,
which means,
"All of the command-line parameters to the shell script."
We also should put `$@` inside double-quotes
to handle the case of parameters containing spaces
(`"$@"` is equivalent to `"$1"` `"$2"` ...)
Here's an example:

~~~ {.bash}
$ cat sorted.sh
~~~
~~~ {.output}
wc -l "$@" | sort -n
~~~
~~~ {.bash}
$ bash sorted.sh *.dat *.spop
~~~
~~~ {.output}
   12 resource.dat
   13 time.dat
   21 tasks.dat
   25 average.dat
   25 count.dat
   25 dominant.dat
   60 grid_task.0.dat
   60 grid_task.100.dat
   60 grid_task.200.dat
   60 grid_task.300.dat
   60 grid_task.400.dat
   60 grid_task.500.dat
   26 detail-0.spop
   4713 detail-500.spop

~~~

> ## Why Isn't It Doing Anything? {.callout}
>
> What happens if a script is supposed to process a bunch of files, but we
> don't give it any filenames? For example, what if we type:
>
>     $ bash sorted.sh
>
> but don't say `*.dat` (or anything else)? In this case, `$@` expands to
> nothing at all, so the pipeline inside the script is effectively:
>
>     wc -l | sort -n
>
> Since it doesn't have any filenames, `wc` assumes it is supposed to
> process standard input, so it just sits there and waits for us to give
> it some data interactively. From the outside, though, all we see is it
> sitting there: the script doesn't appear to do anything.

We have two more things to do before we're finished with our simple shell scripts.
If you look at a script like:

~~~
wc -l "$@" | sort -n
~~~

you can probably puzzle out what it does.
On the other hand,
if you look at this script:

~~~
# List files sorted by number of lines.
wc -l "$@" | sort -n
~~~

you don't have to puzzle it out --- the comment at the top tells you what it does.
A line or two of documentation like this make it much easier for other people
(including your future self)
to re-use your work.
The only caveat is that each time you modify the script,
you should check that the comment is still accurate:
an explanation that sends the reader in the wrong direction is worse than none at all.

Second,
suppose we have just run a series of commands that did something useful --- for example,
that created a graph we'd like to use in a paper.
We'd like to be able to re-create the graph later if we need to,
so we want to save the commands in a file.
Instead of typing them in again
(and potentially getting them wrong)
we can do this:

~~~ {.bash}
$ history | tail -4 > redo-figure-3.sh
~~~

The file `redo-figure-3.sh` now contains:

~~~
297 bash goostats -r NENE01729B.txt stats-NENE01729B.txt
298 bash goodiff stats-NENE01729B.txt /data/validated/01729.txt > 01729-differences.txt
299 cut -d ',' -f 2-3 01729-differences.txt > 01729-time-series.txt
300 ygraph --format scatter --color bw --borders none 01729-time-series.txt figure-3.png
~~~

After a moment's work in an editor to remove the serial numbers on the commands,
we have a completely accurate record of how we created that figure.

> ## Unnumbering {.callout}
>
> Nelle could also use `colrm` (short for "column removal") to remove the
> serial numbers on her previous commands.
> Its parameters are the range of characters to strip from its input:
>
> ~~~
> $ history | tail -5
>   173  cd /tmp
>   174  ls
>   175  mkdir bakup
>   176  mv bakup backup
>   177  history | tail -5
> $ history | tail -5 | colrm 1 7
> cd /tmp
> ls
> mkdir bakup
> mv bakup backup
> history | tail -5
> history | tail -5 | colrm 1 7
> ~~~

In practice, most people develop shell scripts by running commands at the shell prompt a few times
to make sure they're doing the right thing,
then saving them in a file for re-use.
This style of work allows people to recycle
what they discover about their data and their workflow with one call to `history`
and a bit of editing to clean up the output
and save it as a shell script.

In addition to being generally cool, shell scripts let you submit jobs to the
HPCC. If your job is longer than 2 hours (which Avida runs usually are) or
you want to run more than one copy (which you usually do with Avida), this
is critical.

A submission script is basically just a bash script with some fancy comments
at the top to tell the scheduler how to run it:

~~~
#!/bin/bash -login
#PBS -l walltime=4:00:00
#PBS -l mem=4096mb
#PBS -N complex_features
#PBS -t 1-5
#PBS -M [your email address]

cd complex_features_experiment/results/all_rewarded
mkdir ${PBS_ARRAYID}
cd ${PBS_ARRAYID}

cp -r ../configs/* .

./avida -set RANDOM_SEED ${PBS_ARRAY_ID} -set COPY_MUT_PROB 0.0025 
~~~

TODO: More details on submitting

> ## Variables in shell scripts {.challenge}
>
> In the molecules directory, you have a shell script called `script.sh` containing the 
> following commands:
>
> ~~~
> head $2 $1
> tail $3 $1
> ~~~
> 
> While you are in the molecules directory, you type the following command:
>
> ~~~
> bash script.sh '*.pdb' -1 -1
> ~~~
> 
> Which of the following outputs would you expect to see?
>
> 1. All of the lines between the first and the last lines of each file ending in `*.pdb`
>    in the molecules directory 
> 2. The first and the last line of each file ending in `*.pdb` in the molecules directory
> 3. The first and the last line of each file in the molecules directory
> 4. An error because of the quotes around `*.pdb`

> ## List unique species {.challenge}
> 
> Leah has several hundred data files, each of which is formatted like this:
> 
> ~~~
> 2013-11-05,deer,5
> 2013-11-05,rabbit,22
> 2013-11-05,raccoon,7
> 2013-11-06,rabbit,19
> 2013-11-06,deer,2
> 2013-11-06,fox,1
> 2013-11-07,rabbit,18
> 2013-11-07,bear,1
> ~~~
> 
> Write a shell script called `species.sh` that takes any number of
> filenames as command-line parameters, and uses `cut`, `sort`, and
> `uniq` to print a list of the unique species appearing in each of
> those files separately.

> ## Find the longest file with a given extension {.challenge}
> 
> Write a shell script called `longest.sh` that takes the name of a
> directory and a filename extension as its parameters, and prints
> out the name of the file with the most lines in that directory
> with that extension. For example:
> 
> ~~~
> $ bash longest.sh /tmp/data pdb
> ~~~
> 
> would print the name of the `.pdb` file in `/tmp/data` that has
> the most lines.

> ## Why record commands in the history before running them? {.challenge}
> 
> If you run the command:
> 
> ~~~
> history | tail -5 > recent.sh
> ~~~
> 
> the last command in the file is the `history` command itself, i.e.,
> the shell has added `history` to the command log before actually
> running it. In fact, the shell *always* adds commands to the log
> before running them. Why do you think it does this?

> ## Script reading comprehension {.challenge}
> 
> Joel's `data` directory contains three files: `fructose.dat`,
> `glucose.dat`, and `sucrose.dat`. Explain what a script called
> `example.sh` would do when run as `bash example.sh *.dat` if it
> contained the following lines:
> 
> ~~~
> # Script 1
> echo *.*
> ~~~
> 
> ~~~
> # Script 2
> for filename in $1 $2 $3
> do
>     cat $filename
> done
> ~~~
> 
> ~~~
> # Script 3
> echo $@.dat
> ~~~
