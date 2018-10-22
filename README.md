# SYNOPSIS

mgrep \[options\] \[file ...\]

mgrep: grep for known patterns in files

A pattern follows the **recipe**:

    <name>?/<pattern>/<flags>?

# OPTIONS

- **--pattern|-p &lt;pattern>**

    Add the pattern to the list of known patterns. Can be specified multiple times.
    Each pattern follows the recipe.

- **--file-with-patterns|-f &lt;file>**

    Load a set of patterns from a file. Each line that isn't empty and that doesn't
    begin with a '#' is assumed to match the recipe, and is added to the list of
    known patterns.

    Each file can be absolute or relative. If no such file can be found a file with
    that name is searched for in `$HOME/.mgrep/$file` and in `/etc/mgrep/$file`.

    Can be specified multiple times.

- **--list-files-with-patterns**

    Show files fould in default locations and exit

- **-v|--invert-match**

    Invert match. Like grep's -v.

- **--prepend-names**

    Prefix each matched line with the name of the recipe it matches

- **--show-name|-s &lt;name>**

    Only show lines for pattern named &lt;name>

- **--all|-a**

    Print every line whether it matches or not (useful with -n).

- **--count|-c**

    Instead of printing every line that matches (or not with -v), print a count of
    matches pr. recipe. Prints most frequent matches first.

- **--reverse-count|-r**

    Reverses the count, so most frequent matches are printed last. Implies --count.

- **--count-no-matches**

    Count also that patterns that don't match at all as 0.

- **--examples**

    Show the first matched line as an example line along with each summary count.
    Implies --count.

- **--help|-h|-?**

    Print a brief help message and exits.

- **--man**

    Prints the manual page and exits.

# DESCRIPTION

`mgrep` is a tool well-suited for investigating e.g. log files. Or other kinds
of files where you want to grep "away" different things. It started because I
was tired of log command lines similar to this:

    cat sample.log | grep -v something | grep -v something_else | grep -v foo

It is useful for looking for (and counting) patterns of interest or - perhaps
more interestingly - removing known noise from log files so only the unexpected
remains.

So with `sample.log` file like:

    01:00 Foo
    01:05 Bar
    01:07 Baz
    01:10 Foo
    01:15 Bar
    01:20 Foo
    01:30 Foo
    01:45 Unexpected

We can do this:

    > mgrep -v -p /Foo/ -p 'barbaz/Ba[rz]/' sample.log
    01:45 Unexpected

And count the pattern hits:

    > mgrep -c -p /Foo/ -p 'barbaz/Ba[rz]/' sample.log
         4: Foo
         3: barbaz
         1: <other>

And especially if you have many patterns and many matches, it can help to see a
single real-world example of each pattern as it actually matched:

    > mgrep --examples -p /Foo/ -p 'barbaz/Ba[rz]/' sample.log
    (removed output for brevity)

## Patterns

A pattern follows:

    <name>?/<pattern>/<flags>?

`<name>` cannot contain the '/' char, but anything else goes.
`<flags>` may contain any of the `/[msix]/` chars from [perlre](https://metacpan.org/pod/perlre).

## Example Patterns

For the example above, this could be a pattern file for sample.log, e.g. in
`~/.mgrep/sample-patterns`:

    # This first one doesn't have a name,
    # so the regex itself is used as the name
    /Foo/

    # This next one has a name: barbaz and is case-insensitive
    barbaz/BA[RZ]/i

Now you can `mgrep -f sample-patterns sample.log`.
