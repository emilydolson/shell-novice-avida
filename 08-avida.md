---
layout: page
title: The Unix Shell
subtitle: Running Avida
minutes: 15
---
> ## Learning Objectives {.objectives}
>
> *   Install Avida
> *   Run Avida
> *   Change Avida configurations


The whole reason that we wanted to log onto the HPCC in the
first place was so we could run Avida. In order to do that,
we need to install Avida! The first step is to download the code.
Avida is open source, and publicly available on github. So
the easiest way to get the code is to clone the repository

~~~ {.bash}
$ git clone https://github.com/devosoft/avida.git
~~~

Now that we have the code, we need to compile it. We can
do that by going into the directory that was just created
and running the `build_avida` script. This might take a
few minutes...

~~~ {.bash}
$ cd avida
$ ./build_avida
~~~

Ta-da! Now we have avida on our HPCC account. There's a lot
going on in the avida directory:

~~~ {.bash}
$ ls
~~~
~~~ {.output}
apps        Avida.xcworkspace  build.xml  CMakeLists.txt  libs    run_tests
avida-core  build_avida        cbuild     documentation   README
~~~

All of the code is in directories inside `avida/avida-core/source`:

~~~ {.bash}
$ ls avida-core/source
~~~
~~~ {.output}
actions  cpu      environment  output    systematics  util    viewer-core
analyze  data     Jamfile      platform  targets      utils
core     drivers  main         script    tools        viewer
~~~

`avida/avida-core` also contains the `tests` directory. Theoretically, 
every feature in Avida shouldhave its own test, and this is where 
they all live. These tests help us make sure that we haven't accidentally 
broken anything when we added a new feature. If you ever add features to 
Avida, you should add tests for them too! Having a set of tests is a good 
idea for any piece of code that you write. To run all of the tests, you
can use the `run_tests` script in the `avida` directory:

~~~ {.bash}
$ ./run_tests
~~~

Another useful feature in the main Avida directory is the `documentation`
directory. The [github wiki](https://github.com/devosoft/avida/wiki) is
generally more up-to-date, but the included documentation can be a good
quick reference. Even the github wiki doesn't document all of Avida's
features, though. This is something we are constantly working to improve (and
you should feel free to help if you notice something missing!).

Anyway, let's get to the main event. The avida executable file is located
in `avida/cbuild/work`, as are all of the necessary configuration files:

~~~ {.bash}
$ cd cbuild/work
$ ls
~~~
~~~ {.output}
analyze.cfg        default-heads-sex.org  instset-heads.cfg
avida              default-transsmt.org   instset-heads-sex.cfg
avida.cfg          environment.cfg        instset-transsmt.cfg
default-heads.org  events.cfg
~~~

The avida executable is the file called `avida`. Let's try running it!

~~~ {.bash}
$ ./avida
~~~

Avida is off and running! You should see four columns of scrolling
numbers. The first column lists the current update. Updates are the
underlying unit of time in Avida. Every update, each organism executes
an average of 30 instructions. The next column shows the current average
generation across the population. The third column shows the current
average fitness. Fitness in Avida is measured by dividing the speed
at which organisms can execute the instructions in their genomes (merit)
by the length of their genome (gestation time). As a result, it is an
approximate measure of the number of offspring we expect that organism to
produce. The last column in the output shows the number of organsisms in 
the population. By default, the population is started off with a single 
organism that is capable of copying itself. Over the first few hundred updates,
its descendants fill up the environment (a 60x60 grid, by default). After
that, the population size generally stays relatively constant at around 3600
(the number that can fit in the environment), unless you do something to it.

By default, Avida will run for 100,000 updates. We don't have time for that
right now, so let's stop it. We can do that by holding down the `ctrl` and `c`
keys simultaneously. This will stop most programs, including Avida.

While it was running, Avida probably outputted some data about the first few
hundred updates. Let's take a look:

~~~ {.bash}
$ cd data
$ ls
~~~
~~~ {.output}
average.dat  detail-0.spop    dominant.dat  tasks.dat
count.dat    detail-500.spop  resource.dat  time.dat
~~~

These are the default data files the Avida produces. Let's start with 
`time.dat`:

~~~ {.bash}
$ cat time.dat
~~~
~~~ {.output}
# Avida time data
# Sat Oct 10 22:28:05 2015
#  1: update
#  2: avida time
#  3: average generation
#  4: num_executed?

0 0 0 30 
100 1.03086 6.77551 1470 
200 2.0754 14.8343 10800 
300 3.1134 22.6788 29610 
400 4.14478 30.4458 57060 
500 5.17496 38.2065 92040
~~~

The four numbered lines tell us what is in each of the columns. For instance, 
the first column indicates which update that row of data was from. The 
numbers are all round because, by default, data is outputted every hundered 
updates. The other `.dat` files have a similar format.

There's another kind of output file here: the `detail-*.spop` files.
These files are snapshots of the population taken at the specified number of
updates. They contain the complete genotypes, counts, lineages, and more,
for all organisms in the population.

There are many more data files that you can have Avida output if you want - 
these are just the default set. We can control when which data files get output
(among many many other things) by modifying the `events.cfg` file. So that we 
don't lose the default events file, we'll make our changes to a copy.

~~~ {.bash}
$ cd ..
$ cp events.cfg events_test.cfg
$ nano events_test.cfg
~~~

Notice that all of the non-comment lines in this file start with `u`, followed
by one or more numbers. The `u` indicates that the numbers represent updates
at which the specified action is to be performed. Lines that just have a
single number indicate that an action is only supposed to happen once, at the
specified update. Lines containing three numbers separated by colons
(e.g. x:y:z) indicate that an action is to be performed every y updates between
update x and update z. `end` is a special word that means an event should
happen whenever the run ends. `begin` is a special word that means that an
event should happen when the run is started.

For instance, this line:
~~~ {.output}
u 0:100:end PrintAverageData
~~~

will add a line to `average.dat` every 100 updates from the beggining of the run
to the end of the run.

Most of the actions in the default `events.cfg` file just print out data.
There are two exceptions to this:

The first event in the file is:
~~~ {.output}
u begin Inject default-heads.org
~~~

This tells Avida to add one organism of the type specified in the file
`default-heads.org` to the population at the begginign of the run. 
This is the seed organism, from which
all others will be descended. If we look inside the `default-heads.org` file,
we can see that it contains the entire sequence of assembly code that makes up
the organism's genome. In this case, it's mosly filler instructions (`nop-C`),
with a loop that copies itself. For more information, take a look at
[this page](https://github.com/devosoft/avida/wiki/Default-Ancestor-Guided-Tour).

The last event in the file is:
~~~ {.output}
u 100000 Exit   
~~~

This tells Avida to stop running when it reaches the 100,000th update.

For the sake of trying out some things quickly, let's tell Avida to only run
for 500 updates:
~~~ {.output}
u 500 Exit   
~~~

Let's also add another event, to print a different type of data file:
~~~ {.output}
u 0:100:end DumpTaskGrid
~~~

There are tons of actions that we could add. For instance, we could periodically
kill off a bunch of organisms. A (mostly) complete list is available 
[here](https://github.com/devosoft/avida/wiki/List-of-actions).

Let's empty the data directory, and try running Avida again. We can tell it
to use the new events file we made with the -s flag:

~~~ {.bash}
$ rm -r data
$ ./avida -set EVENT_FILE events_test.cfg
~~~

This time, it should stop after 500 updates. If we go back to the data
directory, we'll see the new files we printed out:

~~~ {.bash}
$ cd data
$ ls
~~~
~~~ {.output}
average.dat      dominant.dat       grid_task.300.dat  tasks.dat
count.dat        grid_task.0.dat    grid_task.400.dat  time.dat
detail-0.spop    grid_task.100.dat  grid_task.500.dat
detail-500.spop  grid_task.200.dat  resource.dat
~~~

The `grid_task.*.dat` files are an example of spatial data files in Avida.
They represent a snapshot of the update they were outputted at. They contain
a grid of numbers representing the tasks that the organism in that grid cell
at that point in time could perform. If you look at them all, you can see the
population expanding out from original organism.

Now it's time to take a look at the main configuration file, `avida.cfg`.
This is where all of the options for how to run Avida are set. It can be
pretty overwhelming. Don't worry - you don't need to understand what all of
the configuration options do. For any given experiment, we leave the vast
majority of these set to their defaults.

Notice that the file to use as the events file is one of the config options:
~~~ {.output}
EVENT_FILE events.cfg 
~~~

When we ran Avida with the `-set EVENT_FILE events_test.cfg` flag, we were 
actually telling Avida to set this configuration option to the specified
value before starting the run. `-set` works on any option in `avida.cfg`.
Often, for the purposes of record-keeping, we prefer to set options on the
command-line like this, rather than actually changing the config file.
That way, it's obvious from looking at the command we executed what conditions
each replicate was run with. 

> ## Playing with config settings {.challenge}
>
> Choose a config setting and run Avida with it set to a different value.
>
> Here are some good choices:
> - WORLD_X
> - WORLD_Y
> - WORLD_GEOMETRY 
> - BIRTH_METHOD (try 4, 8, or 11) (if you use 11, set DISPERSAL_RATE to something greater than 0)

There are two more configuration files. The first is the instruction set. 
This file tells Avida which instructions are allowed to be used in genomes.
The default is `instset-heads.cfg`.

If we take a look at it, we can see that that's pretty much all it is:

~~~ {.bash}
$ head instset-heads.cfg
~~~
~~~ {.output}
INSTSET heads_default:hw_type=0

# No-ops
INST nop-A         # a
INST nop-B         # b
INST nop-C         # c

# Flow control operations
INST if-n-equ      # d
INST if-less       # e
~~~

Everything preceeded with a `#` is a comment. The letter comments next to each
instruction indicate what letter will be used to represent that instruction
when printing out organismss genotypes. Letters are assigned in sequence to
the instructions listed in the file, so the comment is just a reference for 
the user.

There are a few other commonly used instruction sets provided with Avida.
One is `instset-heads-sex.cfg`, which is used in set-ups in which recombination
between organisms is allowed. If we look at `instset-heads-sex.cfg`, it's
pretty similar to `instset-head.cfg`. We can use a program called `diff` to
see what the differences are:

~~~ {.bash}
$ diff instset-heads.cfg instset-heads-sex.cfg
~~~
~~~ {.output}
1c1
< INSTSET heads_default:hw_type=0
---
> INSTSET heads_sex:hw_type=0
35c35
< INST h-divide      # x
---
> INST divide-sex    # x
39c39
< INST h-search      # z
\ No newline at end of file
---
> INST h-search      # z
~~~

This tells us that there are three differences in the files:
* Line 1 changed because the instruction sets are labeled different things 
(`heads_default` vs. `heads_sex`)
* Line 35 changed because `instset-heads-sex.cfg` replaces the normal divide
command with one that uses sexual recombination
* Line 39 changed because one file has an empty line at the bottom and the other
does not (this doesn't matter)

Lastly, there's the environment file. This is probably the most flexible of the
configuration files. It's where you can specify reactions and resources.

TODO: More on environment