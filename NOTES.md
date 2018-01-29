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
Looking at the plumb rules (`9p read plumb/rules`) we see that
they are being gerenerated by the rule:

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

Fast Builds
-----------
At the moment I'm not sure how to build all sources quickly.
I could use the INSTALL script, but that takes a long time.
I attempted `mk --all` in the src directory but that also took
a really long time even though no files had been edited. I want
to figure out how to build only some components.




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

