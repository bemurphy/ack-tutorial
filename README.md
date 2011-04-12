Welcome to this short tutorial on Ack, a better `grep(1)` for
programmers.

The goal of this tutorial is to show you Ack's power, especially for
searching code bases.

Installation
===

* OS X / Homebrew: `brew install ack`
* OS X / MacPorts: `sudo port install p5-app-ack`
* Debian: `sudo apt-get install ack-grep; sudo ln -s $(which ack-grep) /usr/local/bin/ack`
* Ubuntu: `sudo apt-get install ack-grep; sudo ln -s $(which ack-grep) /usr/local/bin/ack`

First searches
===

Say we're working on an application. For this example we'll
use the [Redis website](http://redis.io), which is [open
source](https://github.com/antirez/redis-io).

So -- I know there's a helper method called `related_topics_for`,
because it's throwing an exception. I'd like to see where it's defined.

    $ ack related_topics_for

That was a short and simple result. Things to note:

* Ack is ignoring everything in my `.git` directory, because it knows I
don't care about what's in there.

* Only 'known' file types are searched. We'll come back to this later.

* Ack uses colored output by default. It will highlight the file name,
the line matched and the match itself (this is as long as I'm not piping
its output to another program -- in that case, Ack's output looks a lot
like grep's). Usability wins.

Okay. I found where the method is defined, and I'm thinking the issue
may be related to some library I'm using. I'd like to see everything
that my code is loading. In Ruby, this means searching for the keyword
`require`.

    $ ack require

You'll see the result is much longer than our previous match. It goes
way beyond the height of my terminal. Fortunately, we can tell Ack to
use a pager:

    $ echo '--pager=less' > ~/.ackrc

If you're using `less(1)`, you'll need to tell it to allow color to be
displayed. In short, if you're using Bash:

    $ echo export LESS=-RFX >> ~/.bashrc
    $ source ~/.bashrc

Check out `man less` to know exactly what `-R`, `-F` and `-X` do, but
believe me it's a good general configuration for `less`.

Try our previous search again:

    $ ack require

It's much more usable now.

However, those search results do have some noise: the word "requires"
also appears in some template files. One easy way to filter those out
would be to use word boundaries. Remember: Ack is Perl, and its regular
expressions are too.

    $ ack '\brequire\b'

Now we see only those cases where "require" appears by itself,
surrounded either by spaces, commas, etc. But never "requires",
"required", etc. Since this use case is so common, Ack has a switch for
it:

    $ ack -w require

More switches
===

Now that we have more or less what we were looking for, it's a common
need to open those files. For this use case, there's a very handy switch
to only print the filenames that matched our search.

    $ ack -l -w require

A tip: it's common to run Ack a few times to filter what you really
need and then wanting to open the files that matched your very last
search. Assuming you're using Bash, you can do this:

    $ vim -p $(!! -l)

Hopefully we fixed the bug.

Now, say we would like to check if we're doing any browser sniffing (we
all know it's bad, so we'd like to see if we can remove that). In jQuery
it's common to do these checks using the object `$.browser`.

    $ ack $.browser

That will return no results, because in regular expressions `$` is the
end of the line -- so we're actually searching for things after the EOL,
which doesn't make sense. One solution would be to escape (to Ack, not
the shell) those characters:

    $ ack '\$\.browser'

But that's quite a bit of thought, and we may have some other control
characters that we don't notice (`[`, `-`, `{`). So another way to do it
would be:

    $ ack -Q $.browser

This switch makes Ack escape all necessary characters so that the string
we're passing is considered a literal and not a regular expression.

Ack by default returns lines it matches on.  If you want to see context
for the code in the results, you can use the -C flag, and provide the
number of lines of context to include from above and below the match:

    $ ack -C 1 require


To be continued...
