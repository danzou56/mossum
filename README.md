Mossum
======

What is it?
-----------

Mossum is a tool for summarizing results from Stanford's
[Moss](http://theory.stanford.edu/~aiken/moss/). The tool generates a graph for
(multiple) results from Moss, which can help in identifying groups of students
that have shared solutions.

The tool can also generate a report, which shows which solutions are similar
between all pairs of students. When submitting multiple parts of an assignment
to Moss, this can help in identifying which students have multiple similar
solutions.


Installation
------------

The script can be installed via `pip`:

```
pip3 install git+https://github.com/hjalti/mossum@master
```

Please note that this project is only developed and tested using Python 3.

Basic usage
-----------

After installation, the script can be called as follows

```
mossum MossURL1 MossURL2 ... MossURLn
```

If no URLs are specified as parameters, the URLs are read from the
standard input. In this example, mossum generates *n* PNG images, that
show the relationship between the submissions in each result page. The name of
each image is obtained from the comment used when submitting to Moss (the `-c`
flag).

By default mossum only shows a link between two submissions if the match Moss
finds
* accounts for more than 90% of either submission, and
* accounts for more than one line of either submission.

The percentage and number of lines can be changed using the `--min-percent`
(`-p`) option and `--min-lines` (`-l`) option, respectively.

For example, when called with

```
mossum -p 95 -l 30 MossURL1 MossURL2 ... MossURLn
```

mossum only shows a link between submission if either match accounts for more
than 95% of the submission or more than 30 lines are matched.



More options
------------

### Transformation

When submitting to Moss, the whole path of all files is shown. When
summarizing, we can extract relevant information from the path. This can also
be useful if assignments are submitted in multiple files. In which case, we can
extract relevant information from the path (e.g., student ID), when
summarizing.

The transformation option takes a regular expression as a parameter. If the
regular expression contains groups, the transformed submission names are formed
from the strings matched inside all groups, joined by an underscore. If the
regular expression contains no groups, only the part of path that matches the
regular expression is used as the transformed name.

For example, if the submissions are stored in the following directory structure

```
<assignment_name>/<assignment_id>/<student_id>/<assignment_file>
```

for example,

```
assignment1/1337/fred24/a1_part1.cpp
assignment1/1337/fred24/a1_part2.cpp
assignment1/1337/fred24/a1_part3.cpp
assignment1/1337/sarah37/a1_p1.cpp
assignment1/1337/sarah37/a1_part2.cpp
assignment1/1337/sarah37/a13.cpp
```

the student ID can be extracted using the regular expression `.*/(.+)/.*` and
if we want the name to contain the assignment name and the student ID, we can
use the regular expression `(.*)/.*/(.*)/.*`.

By extracting only the students' IDs from the path, the output might look something like this.

![Example](example1.png?raw=true "Example")


### Anonymize

The summarization can be anonymized using the `--anonymize` or `-a` flag. When
using this, all names are substituted with a random name. This is useful for
frightening students.

### Report

When assignments are split into parts and each part is submitted to Moss
separately, it can be hard to get an overview of which students have many
submissions in common. If the `--report` or `-r` flag is specified, mossum
generates a *report*. The report generated shows which matches each pair of
students have in common. The report is sorted, such that the pair of students
with the most matches in common come first.

Calling

```
mossum -r -t "*./(.+)/.*" MossURL1 MossURL2 ... MossURLn
```

would yield a report with entries similar to

```
fred24 and sarah37:
part1: http://moss.stanford.edu/results/???/match??.html
part2: http://moss.stanford.edu/results/???/match??.html
part3: http://moss.stanford.edu/results/???/match??.html
```

The reports are stored in `<filename>.txt`, where `<filename>` is either
specified by the `--output` or `-o` flag or obtained from the name of the
assignments (extracted from the Moss pages).

### Format

The format of the image generated by mossum can be changed using the `--format`
or `-f` option. Mossum uses Graphviz to generate graphs. All output formats
supported by Graphviz are therefore supported by mossum. See [Graphviz output
formats](http://www.graphviz.org/doc/info/output.html).


### Labels

Edges are labeled, by default, with the percentage of code matched (the higher
percentage is used) and the number of lines matched. Additionally, hyperlinks
to the Moss page of the match are added to edge labels. However, hyperlinks
only work if the specified format is `svg` or `xlib`.

For example, calling

```
mossum -f svg -t "*./(.+)/.*" -o assignment MossURL1 MossURL2 ... MossURLn
```

will generate a SVG image file. If this file is opened in a browser, the
matches found by Moss, which represent each edge, can be opened by clicking the
label displayed on each edge.

If you are using X, you can specify the format `xlib`. In this case mossum
displays the graph in an Xlib window, and the graph is therefore not saved to
a file.

Edge labels can be hidden with the `--hide-labels` flag.

### Verbosity

It may be difficult to know whether Mossum is running. If the `--verbose` or 
`-v` flag is used, Mossum will let you know what it's currently doing.

## Filters

### Node filters

It is possible to to add or remove a name from the collection of assignments
using filters. There are four filter options, all of which take a list of names
as arguments. The following filters are available.

* `--filter   N1 N2 ... Nn`: Only show connections between the specified names.
No other connections are shown.

* `--filteri  N1 N2 ... Nn`: Only show connections including at least one of
specified names. No other connections are shown.

* `--filterx  N1 N2 ... Nn`: Do not show connections between the specified
names. All other connections are shown.

* `--filterxi N1 N2 ... Nn`: Do not show connections where one of the specified
names is involved. All other connections are shown.

### Edge filters

It is also possible to filter out nodes that are connected by more than one
edge using the `--min-edges` option. This is only applicable to merged results.
This can be useful, e.g., in assignments with multiple problems. Submissions to
each problem can then submitted to Moss separately. Merging these results can
lead to a lot of noise. For example, like this.

![Example 2](example2_all.png?raw=true "Example")

Applying `--min-edges 2` highlights students sharing more than one solution.

![Example 2](example2_pruned.png?raw=true "Example")
