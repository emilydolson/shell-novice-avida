---
layout: page
title: The Unix Shell
subtitle: Pipes and Filters
minutes: 15
---
> ## Learning Objectives {.objectives}
>
> *   Redirect a command's output to a file.
> *   Process a file instead of keyboard input using redirection.
> *   Construct command pipelines with two or more stages.
> *   Explain what usually happens if a program or pipeline isn't given any input to process.
> *   Explain Unix's "small pieces, loosely joined" philosophy.

Now that we know a few basic commands,
we can finally look at the shell's most powerful feature:
the ease with which it lets us combine existing programs in new ways.
This can be powerful for quickly looking at the output of Avida.
Let's go back to `~/avida/cbuild/work/data` to practice on the output we
generated before.

~~~ {.bash}
$ cd ~/avida/cbuild/work/data
$ ls
~~~
~~~ {.output}
average.dat    detail-500.spop  grid_task.100.dat  grid_task.400.dat  tasks.dat
count.dat      dominant.dat     grid_task.200.dat  grid_task.500.dat  time.dat
detail-0.spop  grid_task.0.dat  grid_task.300.dat  resource.dat
~~~

Let's run the command `wc *.dat`.
`wc` is the "word count" command:
it counts the number of lines, words, and characters in files.
The `*` in `*.dat` matches zero or more characters,
so the shell turns `*.dat` into a complete list of `.dat` files:

~~~ {.bash}
$ wc *.dat
~~~
~~~ {.output}
   25   182   967 average.dat
   25   210   960 count.dat
   25   204  1007 dominant.dat
   60  3600 10859 grid_task.0.dat
   60  3600 10785 grid_task.100.dat
   60  3600 10453 grid_task.200.dat
   60  3600  9787 grid_task.300.dat
   60  3600  8832 grid_task.400.dat
   60  3600  7631 grid_task.500.dat
   12    40   215 resource.dat
   21   126   439 tasks.dat
   13    48   259 time.dat
  481 22410 62194 tota
~~~

> ## Wildcards {.callout}
> 
> `*` is a **wildcard**. It matches zero or more
> characters, so `*.pdb` matches `ethane.pdb`, `propane.pdb`, and every
> file that ends with '.pdb'. On the other hand, `p*.pdb` only matches 
> `pentane.pdb` and `propane.pdb`, because the 'p' at the front only 
> matches filenames that begin with the letter 'p'.
> 
> `?` is also a wildcard, but it only matches a single character. This
> means that `p?.pdb` matches `pi.pdb` or `p5.pdb`, but not `propane.pdb`.
> We can use any number of wildcards at a time: for example, `p*.p?*`
> matches anything that starts with a 'p' and ends with '.', 'p', and at
> least one more character (since the '?' has to match one character, and
> the final '\*' can match any number of characters). Thus, `p*.p?*` would
> match `preferred.practice`, and even `p.pi` (since the first '\*' can
> match no characters at all), but not `quality.practice` (doesn't start
> with 'p') or `preferred.p` (there isn't at least one character after the
> '.p').
> 
> When the shell sees a wildcard, it expands the wildcard to create a
> list of matching filenames *before* running the command that was
> asked for. As an exception, if a wildcard expression does not match
> any file, Bash will pass the expression as a parameter to the command
> as it is. For example typing `ls *.pdf` in the molecules directory
> (which contains only files with names ending with `.pdb`) results in
> an error message that there is no file called `*.pdf`.
> However, generally commands like `wc` and `ls` see the lists of
> file names matching these expressions, but not the wildcards
> themselves. It is the shell, not the other programs, that deals with
> expanding wildcards, and this is another example of orthogonal design.

If we run `wc -l` instead of just `wc`,
the output shows only the number of lines per file:

~~~ {.bash}
$ wc -l *.dat
~~~
~~~ {.output}
   25 average.dat
   25 count.dat
   25 dominant.dat
   60 grid_task.0.dat
   60 grid_task.100.dat
   60 grid_task.200.dat
   60 grid_task.300.dat
   60 grid_task.400.dat
   60 grid_task.500.dat
   12 resource.dat
   21 tasks.dat
   13 time.dat
  481 total
~~~

We can also use `-w` to get only the number of words,
or `-c` to get only the number of characters.

Which of these files is shortest?
It's an easy question to answer when there are only six files,
but what if there were 6000?
Our first step toward a solution is to run the command:

~~~ {.bash}
$ wc -l *.dat > lengths.txt
~~~

The greater than symbol, `>`, tells the shell to **redirect** the command's output
to a file instead of printing it to the screen.
The shell will create the file if it doesn't exist,
or overwrite the contents of that file if it does.
(This is why there is no screen output:
everything that `wc` would have printed has gone into the file `lengths.txt` instead.)
`ls lengths.txt` confirms that the file exists:

~~~ {.bash}
$ ls lengths.txt
~~~
~~~ {.output}
lengths.txt
~~~

We can now send the content of `lengths.txt` to the screen using `cat lengths.txt`.
`cat` stands for "concatenate":
it prints the contents of files one after another.
There's only one file in this case,
so `cat` just shows us what it contains:

~~~ {.bash}
$ cat lengths.txt
~~~
~~~ {.output}
   25 average.dat
   25 count.dat
   25 dominant.dat
   60 grid_task.0.dat
   60 grid_task.100.dat
   60 grid_task.200.dat
   60 grid_task.300.dat
   60 grid_task.400.dat
   60 grid_task.500.dat
   12 resource.dat
   21 tasks.dat
   13 time.dat
  481 total
~~~

Now let's use the `sort` command to sort its contents.
We will also use the -n flag to specify that the sort is 
numerical instead of alphabetical.
This does *not* change the file;
instead, it sends the sorted result to the screen:

~~~ {.bash}
$ sort -n lengths.txt
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
  481 total
~~~

We can put the sorted list of lines in another temporary file called `sorted-lengths.txt`
by putting `> sorted-lengths.txt` after the command,
just as we used `> lengths.txt` to put the output of `wc` into `lengths.txt`.
Once we've done that,
we can run another command called `head` to get the first few lines in `sorted-lengths.txt`:

~~~ {.bash}
$ sort -n lengths.txt > sorted-lengths.txt
$ head -1 sorted-lengths.txt
~~~
~~~ {.output}
  12 resource.dat
~~~

Using the parameter `-1` with `head` tells it that
we only want the first line of the file;
`-20` would get the first 20,
and so on.
Since `sorted-lengths.txt` contains the lengths of our files ordered from least to greatest,
the output of `head` must be the file with the fewest lines.

If you think this is confusing,
you're in good company:
even once you understand what `wc`, `sort`, and `head` do,
all those intermediate files make it hard to follow what's going on.
We can make it easier to understand by running `sort` and `head` together:

~~~ {.bash}
$ sort -n lengths.txt | head -1
~~~
~~~ {.output}
  12 resource.dat
~~~

The vertical bar between the two commands is called a **pipe**.
It tells the shell that we want to use
the output of the command on the left
as the input to the command on the right.
The computer might create a temporary file if it needs to,
or copy data from one program to the other in memory,
or something else entirely;
we don't have to know or care.

We can use another pipe to send the output of `wc` directly to `sort`,
which then sends its output to `head`:

~~~ {.bash}
$ wc -l *.dat | sort -n | head -1
~~~
~~~ {.output}
  12 resource.dat
~~~

This is exactly like a mathematician nesting functions like *log(3x)*
and saying "the log of three times *x*".
In our case,
the calculation is "head of sort of line count of `*.dat`".

Here's what actually happens behind the scenes when we create a pipe.
When a computer runs a program --- any program --- it creates a **process**
in memory to hold the program's software and its current state.
Every process has an input channel called **standard input**.
(By this point, you may be surprised that the name is so memorable, but don't worry:
most Unix programmers call it "stdin".
Every process also has a default output channel called **standard output**
(or "stdout").

The shell is actually just another program.
Under normal circumstances,
whatever we type on the keyboard is sent to the shell on its standard input,
and whatever it produces on standard output is displayed on our screen.
When we tell the shell to run a program,
it creates a new process
and temporarily sends whatever we type on our keyboard to that process's standard input,
and whatever the process sends to standard output to the screen.

Here's what happens when we run `wc -l *.dat > lengths.txt`.
The shell starts by telling the computer to create a new process to run the `wc` program.
Since we've provided some filenames as parameters,
`wc` reads from them instead of from standard input.
And since we've used `>` to redirect output to a file,
the shell connects the process's standard output to that file.

If we run `wc -l *.dat | sort -n` instead,
the shell creates two processes
(one for each process in the pipe)
so that `wc` and `sort` run simultaneously.
The standard output of `wc` is fed directly to the standard input of `sort`;
since there's no redirection with `>`,
`sort`'s output goes to the screen.
And if we run `wc -l *.pdb | sort -n | head -1`,
we get three processes with data flowing from the files,
through `wc` to `sort`,
and from `sort` through `head` to the screen.

![Redirects and Pipes](fig/redirects-and-pipes.png)

This simple idea is why Unix has been so successful.
Instead of creating enormous programs that try to do many different things,
Unix programmers focus on creating lots of simple tools that each do one job well,
and that work well with each other.
This programming model is called "pipes and filters".
We've already seen pipes;
a **filter** is a program like `wc` or `sort`
that transforms a stream of input into a stream of output.
Almost all of the standard Unix tools can work this way:
unless told to do otherwise,
they read from standard input,
do something with what they've read,
and write to standard output.

The key is that any program that reads lines of text from standard input
and writes lines of text to standard output
can be combined with every other program that behaves this way as well.
You can *and should* write your programs this way
so that you and other people can put those programs into pipes to multiply their power.

> ## Redirecting Input {.callout}
> 
> As well as using `>` to redirect a program's output, we can use `<` to
> redirect its input, i.e., to read from a file instead of from standard
> input. For example, instead of writing `wc ammonia.pdb`, we could write
> `wc < ammonia.pdb`. In the first case, `wc` gets a command line
> parameter telling it what file to open. In the second, `wc` doesn't have
> any command line parameters, so it reads from standard input, but we
> have told the shell to send the contents of `ammonia.pdb` to `wc`'s
> standard input.

> ## What does `sort -n` do? {.challenge}
>
> If we run `sort` on this file:
> 
> ~~~
> 10
> 2
> 19
> 22
> 6
> ~~~
> 
> the output is:
> 
> ~~~
> 10
> 19
> 2
> 22
> 6
> ~~~
> 
> If we run `sort -n` on the same input, we get this instead:
> 
> ~~~
> 2
> 6
> 10
> 19
> 22
> ~~~
> 
> Explain why `-n` has this effect.

> ## What does `<` mean? {.challenge}
>
> What is the difference between:
> 
> ~~~
> wc -l < mydata.dat
> ~~~
> 
> and:
> 
> ~~~
> wc -l mydata.dat
> ~~~

> ## What does `>>` mean? {.challenge}
>
> What is the difference between:
>
> ~~~
> echo hello > testfile01.txt
> ~~~
>
> and:
>
> ~~~
> echo hello >> testfile02.txt
> ~~~
>
> Hint: Try executing each command twice in a row and then examining the output files.

> ## Piping commands together {.challenge}
>
> In our current directory, we want to find the 3 files which have the least number of 
> lines. Which command listed below would work?
>
> 1. `wc -l * > sort -n > head -3`
> 2. `wc -l * | sort -n | head 1-3`
> 3. `wc -l * | head -3 | sort -n`
> 4. `wc -l * | sort -n | head -3`

> ## Why does `uniq` only remove adjacent duplicates? {.challenge}
>
> The command `uniq` removes adjacent duplicated lines from its input.
> For example, if a file `salmon.txt` contains:
> 
> ~~~
> coho
> coho
> steelhead
> coho
> steelhead
> steelhead
> ~~~
> 
> then `uniq salmon.txt` produces:
> 
> ~~~
> coho
> steelhead
> coho
> steelhead
> ~~~
> 
> Why do you think `uniq` only removes *adjacent* duplicated lines?
> (Hint: think about very large data sets.) What other command could
> you combine with it in a pipe to remove all duplicated lines?

> ## Pipe reading comprehension {.challenge}
>
> A file called `animals.txt` contains the following data:
> 
> ~~~
> 2012-11-05,deer
> 2012-11-05,rabbit
> 2012-11-05,raccoon
> 2012-11-06,rabbit
> 2012-11-06,deer
> 2012-11-06,fox
> 2012-11-07,rabbit
> 2012-11-07,bear
> ~~~
> 
> What text passes through each of the pipes and the final redirect in the pipeline below?
> 
> ~~~
> cat animals.txt | head -5 | tail -3 | sort -r > final.txt
> ~~~

> ## Pipe construction {.challenge}
>
> The command:
> 
> ~~~
> $ cut -d , -f 2 animals.txt
> ~~~
> 
> produces the following output:
> 
> ~~~
> deer
> rabbit
> raccoon
> rabbit
> deer
> fox
> rabbit
> bear
> ~~~
> 
> What other command(s) could be added to this in a pipeline to find
> out what animals the file contains (without any duplicates in their
> names)?
