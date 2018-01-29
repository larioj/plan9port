Notes
=====

Link Acme
---------
```bash
mkdir -p /Applications/Acme.app/Contents/MacOS
ln -s \
  $PLAN9/mac/Acme.app/Contents/MacOS/Acme \
  /Applications/Acme.app/Contents/MacOS/Acme
```

Fix Man Links
-------------
When I click on a man link like 9p(1) the paths to the sources
at the end of page are incorrect. For example it shows
`/src/cmd/9p.c` instead of `/usr/local/plan9/src/cmd/9p.c`.
I would like to fix this to show the path of the real sources.

Looking at the plumb rules (`9p read plumb/rules`) we see that
man pages are being gerenerated by the rule:

```text
type	is	text
data	matches	'([a-zA-Z¡-￿0-9_\-./]+)\(([1-8])\)'
plumb	start	rc -c 'man '$2' '$1' >[2=1] | nobs | plumb -i -d edit -a ''action=showdata filename=/man/'$1'('$2')'''
```

* Which man is this?
	* `which man` in my system is the unix man
* Can we man plan9 pages with the system man?
	* `man 9p` does give the plan9 page
	* `man 1 9p` works too
	* `9 man 9p` succeeds but gives a lot of pages
	* `9 man 1 9p` gives the right number of pages

It appears that 9 man actually generates the correct links!
Let's change the plumb rule to use `9 man` instead.

```text
type	is	text
data	matches	'([a-zA-Z¡-￿0-9_\-./]+)\(([1-8])\)'
plumb	start	rc -c '9 man '$2' '$1' >[2=1] | nobs | plumb -i -d edit -a ''action=showdata filename=/man/'$1'('$2')'''
```

```bash
9p read plumb/rules > /tmp/current.plumbing
# Edit /tmp/current.plumbing
cat /tmp/current.plumbing | 9p write plumb/rules
```

Let's test it out: left click on 9p(1) works! It shows the
correct paths to the sources.

Plumber
-------
Let's learn about the plumber plumber(4).

* What does `9p ls plumb` show?
* What does `9p read plumb/send` show?
	* We get a permision denied error

Resolve $HOME and $PLAN9 During Plumbing
----------------------------------------
We want to be able to left click open $HOME/.profile.

Lets look at the rules `9p read plumb/rules`.

We can also look at the default rules in plumb/basic file.

We can add a rule that matches $HOME and one
that matches $PLAN9, and then sends the resulting
file to edit. This works maybe even better than
the more general method of resolving the variables.

Let's create a script that for a set of variables appends
rules to a file that matches paths beggning with those
variables.

```bash
RULES=$HOME/.config/plumber/plumbing

for VARIABLE in '$HOME' '$PLAN9'; do
	VALUE=$(eval "echo $VARIABLE")

cat >> $RULES <<EOF

# existing files tagged by line number:columnumber
# or linenumber.columnumber, twice, go to editor
type is text
data matches '\\$VARIABLE/([.a-zA-Z¡-￿0-9_/\-]*[a-zA-Z¡-￿0-9_/\-])':\$twocolonaddr,\$twocolonaddr
arg isfile     $VALUE/\$1
data set       \$file
attr add       addr=\$2-#1+#\$3,\$4-#1+#\$5
plumb to edit
plumb client \$editor

# existing files tagged by line number:columnumber
# or linenumber.columnumber, twice, go to editor
type is text
data matches '\\$VARIABLE/([.a-zA-Z¡-￿0-9_/\-]*[a-zA-Z¡-￿0-9_/\-])':\$twocolonaddr
arg isfile     $VALUE/\$1
data set       \$file
attr add       addr=\$2-#1+#\$3
plumb to edit
plumb client \$editor

# existing files, possibly tagged by line number, go to editor
type is text
data matches '\\$VARIABLE/([.a-zA-Z¡-￿0-9_/\-]*[a-zA-Z¡-￿0-9_/\-])('\$addr')?'
arg isfile	$VALUE/\$1
data set	\$file
attr add	addr=\$3
plumb to edit
plumb client \$editor

# the variable itself
data matches '\\$VARIABLE'
arg isfile	$VALUE
data set	\$file
plumb to edit
plumb client \$editor

EOF
done
```

Fast Builds
-----------
At the moment I'm not sure how to build all sources quickly.
I could use the $PLAN9/INSTALL script, but that takes a long time.
I attempted `mk --all` in the src directory but that also took
a really long time even though no files had been edited. I want
to figure out how to build only some components.

Lets look at the man page mk(1).


Interactive Edit: TODO
----------------
We want to be able to write

```bash
9p read plumb/rules > /tmp/current.plumbing
ied /tmp/current.plumbing
cat /tmp/current.plumbing | 9p write plumb/rules
```

The `ied` command should open an acme window,
then pause execution util that window is closed.
This allows the user to make changes to the file
and then resume execution of the script.
