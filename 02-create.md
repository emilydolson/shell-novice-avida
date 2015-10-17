---
layout: page
title: The Unix Shell
subtitle: Creating Things
minutes: 15
---
> ## Learning Objectives {.objectives}
>
> *   Create a directory hierarchy that matches a given diagram.
> *   Create files in that hierarchy using an editor or by copying and renaming existing files.
> *   Display the contents of a directory using the command line.
> *   Delete specified files and/or directories.

We now know how to explore files and directories,
but how do we create them in the first place?
Let's go back to Nelle's home directory,
`/Users/nelle`,
and use `ls -F` to see what it contains:

~~~ {.bash}
$ pwd
~~~
~~~ {.output}
/mnt/home/nelle
~~~
~~~ {.bash}
$ ls -F
~~~
~~~ {.output}
avida/    notes.txt
~~~

We're going to repeat a classic Avida experiment on the [origin of complex
features](http://myxo.css.msu.edu/papers/nature2003/Nature03_Complex.pdf).
Let's set up a directory structure to keep track of everything.


~~~ {.bash}
$ mkdir complex_features_experiment
~~~

As you might (or might not) guess from its name,
`mkdir` means "make directory".
Since `thesis` is a relative path
(i.e., doesn't have a leading slash),
the new directory is created in the current working directory:

~~~ {.bash}
$ ls -F
~~~
~~~ {.output}
avida/    notes.txt    complex_features_experiment/
~~~

However, there's nothing in it yet:

~~~ {.bash}
$ ls -F complex_features_experiment
~~~

Let's change our working directory to `complex_features_experiment` using `cd`,
then run a text editor called Nano to create a file called `notes.txt`:

~~~ {.bash}
$ cd complex_features_experiment
$ nano notes.txt
~~~

> ## Which Editor? {.callout}
>
> When we say, "`nano` is a text editor," we really do mean "text": it can
> only work with plain character data, not tables, images, or any other
> human-friendly media. We use it in examples because almost anyone can
> drive it anywhere without training, but please use something more
> powerful for real work. On Unix systems (such as Linux and Mac OS X),
> many programmers use [Emacs](http://www.gnu.org/software/emacs/) or
> [Vim](http://www.vim.org/) (both of which are completely unintuitive,
> even by Unix standards), or a graphical editor such as
> [Gedit](http://projects.gnome.org/gedit/). On Windows, you may wish to
> use [Notepad++](http://notepad-plus-plus.org/).
>
> No matter what editor you use, you will need to know where it searches
> for and saves files. If you start it from the shell, it will (probably)
> use your current working directory as its default location. If you use
> your computer's start menu, it may want to save files in your desktop or
> documents directory instead. You can change this by navigating to
> another directory the first time you "Save As..."

Let's type in a few lines of text,
then use Control-O to write our data to disk:

![Nano in action](fig/nano-screenshot.png)

Once our file is saved,
we can use Control-X to quit the editor and return to the shell.
(Unix documentation often uses the shorthand `^A` to mean "control-A".)
`nano` doesn't leave any output on the screen after it exits,
but `ls` now shows that we have created a file called `draft.txt`:

~~~ {.bash}
$ ls
~~~
~~~ {.output}
notes.txt
~~~

Let's tidy up by running `rm draft.txt`:

~~~ {.bash}
$ rm notes.txt
~~~

This command removes files ("rm" is short for "remove").
If we run `ls` again,
its output is empty once more,
which tells us that our file is gone:

~~~ {.bash}
$ ls
~~~

> ## Deleting Is Forever {.callout}
>
> The Unix shell doesn't have a trash bin that we can recover deleted
> files from (though most graphical interfaces to Unix do).  Instead,
> when we delete files, they are unhooked from the file system so that
> their storage space on disk can be recycled. Tools for finding and
> recovering deleted files do exist, but there's no guarantee they'll
> work in any particular situation, since the computer may recycle the
> file's disk space right away.

Let's re-create that file
and then move up one directory to `/mnt/home/nelle` using `cd ..`:

~~~ {.bash}
$ pwd
~~~
~~~ {.output}
/mnt/home/nelle/complex_features_experiment
~~~
~~~ {.bash}
$ nano notes.txt
$ ls
~~~
~~~ {.output}
notes.txt
~~~
~~~ {.bash}
$ cd ..
~~~

If we try to remove the entire `thesis` directory using `rm thesis`,
we get an error message:

~~~ {.bash}
$ rm complex_features_experiment
~~~
~~~ {.error}
rm: cannot remove `complex_features_experiment': Is a directory
~~~

This happens because `rm` only works on files, not directories.
The right command is `rmdir`,
which is short for "remove directory".
It doesn't work yet either, though,
because the directory we're trying to remove isn't empty:

~~~ {.bash}
$ rmdir complex_features_experiment
~~~
~~~ {.error}
rmdir: failed to remove `thesis': Directory not empty
~~~

This little safety feature can save you a lot of grief,
particularly if you are a bad typist.
To really get rid of `thesis` we must first delete the file `draft.txt`:

~~~ {.bash}
$ rm complex_features_experiment/notes.txt
~~~

The directory is now empty, so `rmdir` can delete it:

~~~ {.bash}
$ rmdir complex_features_experiment
~~~

> ## With Great Power Comes Great Responsibility {.callout}
>
> Removing the files in a directory just so that we can remove the
> directory quickly becomes tedious. Instead, we can use `rm` with the
> `-r` flag (which stands for "recursive"):
>
> ~~~
> $ rm -r thesis
> ~~~
>
> This removes everything in the directory, then the directory itself. If
> the directory contains sub-directories, `rm -r` does the same thing to
> them, and so on. It's very handy, but can do a lot of damage if used
> without care.

Let's create that directory and file one more time.
(Note that this time we're running `nano` with the path 
`complex_features_experiment/notes.txt`,
rather than going into the `complex_features_experiment` directory and 
running `nano` on `notes.txt` there.)

~~~ {.bash}
$ pwd
~~~
~~~ {.output}
/mnt/home/nelle
~~~
~~~ {.bash}
$ mkdir complex_features_experiment
~~~
~~~ {.bash}
$ nano complex_features_experiment/notees.txt
$ ls complex_features_experiment
~~~
~~~ {.output}
notes.txt
~~~

`notes.txt` isn't a particularly informative name,
so let's change the file's name using `mv`,
which is short for "move":

~~~ {.bash}
$ mv complex_features_experiment/notes.txt complex_features_experiment/experiments.txt
~~~

The first parameter tells `mv` what we're "moving",
while the second is where it's to go.
In this case,
we're moving `complex_features_experiment/notes.txt` to 
`complex_features_experiment/experiments.txt`
which has the same effect as renaming the file.
Sure enough,
`ls` shows us that `complex_features_experiment` now contains one file 
called `experiments.txt`:

~~~ {.bash}
$ ls complex_features_experiments
~~~
~~~ {.output}
experiments.txt
~~~

Just for the sake of inconsistency,
`mv` also works on directories --- there is no separate `mvdir` command.

Let's move `experiments.txt` into the current working directory.
We use `mv` once again,
but this time we'll just use the name of a directory as the second parameter
to tell `mv` that we want to keep the filename,
but put the file somewhere new.
(This is why the command is called "move".)
In this case,
the directory name we use is the special directory name `.` that we mentioned earlier.

~~~ {.bash}
$ mv complex_features_experiment/experiments.txt .
~~~

The effect is to move the file from the directory it was in to the current working directory.
`ls` now shows us that `complex_features_experiment` is empty:

~~~ {.bash}
$ ls complex_features_experiment
~~~

Further,
`ls` with a filename or directory name as a parameter only lists that file or directory.
We can use this to see that `quotes.txt` is still in our current directory:

~~~ {.bash}
$ ls experiments.txt
~~~
~~~ {.output}
experiments.txt
~~~

The `cp` command works very much like `mv`,
except it copies a file instead of moving it.
We can check that it did the right thing using `ls`
with two paths as parameters --- like most Unix commands,
`ls` can be given thousands of paths at once:

~~~ {.bash}
$ cp experiments.txt complex_features_experiment/planned_experiments.txt
$ ls 
~~~
~~~ {.output}
experiments.txt complex_features_experiment/planned_experiments.txt
~~~

To prove that we made a copy,
let's delete the `experiments.txt` file in the current directory
and then run that same `ls` again.

~~~ {.bash}
$ rm experiments.txt
$ ls experiments.txt complex_features_experiment/planned_experiments.txt
~~~
~~~ {.error}
ls: cannot access experiments.txt: No such file or directory
complex_features_experiment/planned_experiments.txt
~~~
This time it tells us that it can't find `experiments.txt` in the current 
directory, but it does find the copy in `complex_features_experiment` 
that we didn't delete.

> ## Another Useful Abbreviation {.callout}
>
> The shell interprets the character `~` (tilde) at the start of a path to
> mean "the current user's home directory". For example, if Nelle's home
> directory is `/Users/nelle`, then `~/data` is equivalent to
> `/Users/nelle/data`. This only works if it is the first character in the
> path: `here/there/~/elsewhere` is *not* `/Users/nelle/elsewhere`.

To get ready for the next step, let's make a few more directories:

~~~ {.bash}
$ cd complex_features_experiment
$ mkdir configs
$ mkdir results
$ mkdir results/all_rewarded
$ mkdir results/equ_rewarded
~~~

Now let's copy the Avida config files into our configs directory, so that
we always know what configs we used to run this experiment:

~~~ {.bash}
$ cp ~/avida/cbuild/work/*.cfg configs
$ cp ~/avida/cbuild/work/*.org configus
$ cp ~/avida/cbuild/work/avida configus
~~~

> ## Renaming files {.challenge}
>
> Suppose that you created a `.txt` file in your current directory to contain a list of the
> statistical tests you will need to do to analyze your data, and named it: `statstics.txt`
>
> After creating and saving this file you realize you misspelled the filename! You want to
> correct the mistake, which of the following commands could you use to do so?
>
> 1. `cp statstics.txt statistics.txt`
> 2. `mv statstics.txt statistics.txt`
> 3. `mv statstics.txt .`
> 4. `cp statstics.txt .`

> ## Moving and Copying {.challenge}
>
> What is the output of the closing `ls` command in the sequence shown below?
>
> ~~~
> $ pwd
> /Users/jamie/data
> $ ls
> proteins.dat
> $ mkdir recombine
> $ mv proteins.dat recombine
> $ cp recombine/proteins.dat ../proteins-saved.dat
> $ ls
> ~~~
>
> 1.   `proteins-saved.dat recombine`
> 2.   `recombine`
> 3.   `proteins.dat recombine`
> 4.   `proteins-saved.dat`

> ## Organizing Directories and Files {.challenge}
>
> Jamie is working on a project and she sees that her files aren't very well
> organized:
>
> ~~~
> $ ls -F
> analyzed/  fructose.dat    raw/   sucrose.dat
> ~~~
>
> The `fructose.dat` and `sucrose.dat` files contain output from her data
> analysis. What command(s) covered in this lesson does she need to run so that the commands below will
> produce the output shown?
>
> ~~~
> $ ls -F
> analyzed/   raw/
> $ ls analyzed
> fructose.dat    sucrose.dat
> ~~~

> ## Copy with Multiple Filenames {.challenge}
>
> What does `cp` do when given several filenames and a directory name, as in:
>
> ~~~
> $ mkdir backup
> $ cp thesis/citations.txt thesis/quotations.txt backup
> ~~~
>
> What does `cp` do when given three or more filenames, as in:
>
> ~~~
> $ ls -F
> intro.txt    methods.txt    survey.txt
> $ cp intro.txt methods.txt survey.txt
> ~~~

> ## Listing Recursively and By Time {.challenge}
>
> The command `ls -R` lists the contents of directories recursively,
> i.e., lists their sub-directories, sub-sub-directories, and so on
> in alphabetical order at each level.
> The command `ls -t` lists things by time of last change,
> with most recently changed files or directories first.
> In what order does `ls -R -t` display things?
