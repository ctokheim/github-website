---
layout: post
title:  "Usability of Python scripts"
author: Collin Tokheim
date:   2014-08-26 16:22:14
categories: python
tag: [python]
---

All too often scripts can be all but unusuable by either another person or even the same person who wrote it a couple months earlier. I'll discuess some tricks I use that I've found makes usage considerably easier. Some common themes:

* don't rely on sys.argv
* argparse (CLI parsing)
* use descriptions (so we don't forget what things do)

## Handling arguments

Command line arguments are usually passed through one of two mechanisms: 

* positional arguments 
* use of options

### Positional arguments

Input for positional arguments immediately follow the script like as follows.

{% highlight bash %}
$ python myscript.py bar.txt foo.txt
{% endhighlight %}

In python, the arguments `bar.txt` and `foo.txt` can be accessed using the default `sys` module.

{% highlight python linenos %}
import sys

script_name = sys.argv[0]
first_argument = sys.argv[1]
second_argument = sys.argv[2]
{% endhighlight %}

Using `sys.argv`, the first element is the script name while subsequent elements are the passed in positional arguments.
However, you can immediately realize that if you had two arguments (or more) then it's not clear which should go first. If the
writer of the script didn't provide extensive print help then anybody else would need to decipher their source code
to figure out the argument order. Yikes! there's got to be a better way.

### Argparse

Argparse provides an easy interface for parsing command line arguments and displaying help information to the user.
It also even does error checking on user input! Let's take the previous example and increase its usability.

{% highlight python linenos %}
import argparse


def parse_arguments():
    info = 'Description of script can go here!'
    parser = argparse.ArgumentParser(description=info)
    help_str = 'Tab separated text file as input'
    parser.add_argument('-i', '--input',
                        type=str, required=True,
                        help=help_str)
    help_str = 'Tab separated text file as output'
    parser.add_argument('-o', '--output',
                        type=str, required=True,
                        help=help_str)
    args = parser.parse_args()  # parse CLI options
    opts = vars(args)  # make CLI options a dictionary
    return opts


def main(opts):
    # get arguments from dictionary
    first_argument = opts['input']
    second_argument = opts['output']
    print first_argument
    print second_argument


if __name__ == '__main__':
    opts = parse_arguments()
    main(opts)
{% endhighlight %}

Using `argparse` gives you a help description print out for free!

{% highlight bash %}
$ python myscript.py --help
usage: myscript.py [-h] -i INPUT -o OUTPUT

Description of script can go here!

optional arguments:
  -h, --help            show this help message and exit
  -i INPUT, --input INPUT
                        Tab separated text file as input
  -o OUTPUT, --output OUTPUT
                        Tab separated text file as output
{% endhighlight %}

For this dummy script, I did not craft a useful help message for the user.
Although in this case, only the help_str and info strings need to be modified
so that it will print out to the user.

Argparse also catches scenarios where the user doesn't provide a required argument (`required=True`).

{% highlight bash %}
$ python myscript.py
usage: myscript.py [-h] -i INPUT -o OUTPUT
myscript.py: error: argument -i/--input is required
{% endhighlight %}

Now if the correct arguments are provided, then the program will run as expected.

{% highlight bash %}
$ python myscript.py -i myinput.txt -o myoutput.txt
myinput.txt
myoutput.txt
{% endhighlight %}

## Conclusion

If somebody else is going to use your script or even yourself a month from now, do yourself a favor and 
just use argparse. It provides an easy way to give users help information. It also does error checking
for required arguments and argument type (no putting an int where a string is expected). In fact, if
you specify an option as type int then you will get an int object back from argparse rather than a string.
