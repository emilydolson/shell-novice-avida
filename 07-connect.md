---
layout: page
title: The Unix Shell
subtitle: Connecting to Remote Computers
minutes: 10
---
> ## Learning Objectives {.objectives}
>
> *   Connect to a remote computer
> *   Copy files to and from a remote computer

Using the shell often lets you use your personal
computer more efficiently, but the main reason
we're talking about it today is that most high-performance
computational resources (including MSU's HPCC) are
only accessible through the shell. If you want to do
research with Avida, you want to be running it on
something other than your laptop.

So how do we actually connect to the HPCC? We're going
to use a command called `ssh`, which stands for
"secure shell." It securely connects your shell to a 
shell on the computer that you tell it to connect to:

~~~ {.bash}
$ ssh nelle@hpcc.msu.edu
~~~
~~~ {.output}
Password:
~~~

The computer you're connecting to will ask you for your
password, to make sure you're really who you say you are.
After you enter the password for your account on the computer
you're connecting to, you will usually be given some sort of
welcome screen. For the MSU HPCC, it will look like this:

Congratulations! You're in!

You can navigate the file system on the remote computer
just like you've been navigating the shell on your computer.
The only difference is that there's no graphical user
interface to fall back on if you get stuck.

Every computing cluster is a little different. For the rest
of this lesson, we're going to assume you're using the MSU HPCC.
Other systems will probably be similar, but your mileage may vary.

When you first log in to the HPCC, you will be on what's called
the "gateway node." This is a not-very-powerful computer that's just
suppposed to handle getting people logged in. From here, you can
use `ssh` again to sign into one of the "development nodes" listed
on the welcome screen. Try to choose one with "low" usage.

~~~ {.bash}
$ ssh dev-intel14
~~~
~~~ {.output}
Logging into dev-intel14
===
This development node is intended for compiling, debugging, testing, and short interactive jobs. Jobs that need to run longer than two hours should be submitted to the queue. Long-running jobs on development nodes will be killed without warning; jobs using excessive resources may be killed without warning.

Please see the following wiki page for further details:
http://wiki.hpcc.msu.edu/x/nQBe
===
~~~

> ## Make sure to sign into a dev node! {.callout}
>
> If you ever get a weird error when you try to run a
> program on the HPCC, the most likely reason is that
> you forgot to sign into a dev node. Sign into one and
> try again.
>

TODO: transfering files
