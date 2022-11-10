# addheader - add headers to files
This repository contains a single command to manage a header section, 
e.g. copyright, for a source code tree.

Using UNIX glob patterns, addheader modifies an entire tree of
source code at once. The program replaces existing headers with
an updated version, and places the header after any shell magic
at the top of the file.

As of version 0.3.0, Jupyter notebooks can also be handled.
See Usage -> Adding headers to Jupyter Notebooks.

## Installation

_addheader_ is written in Python and can be simply installed from the PyPI package:

`pip install addheader`

## Usage

Use the command `addheader`. Invoking`addheader -h` shows a detailed help message
for the command arguments and options. Below are some examples and comments on usage.

### Basic usage

If you have the header file in "copyright.txt", and your source tree is a Python
package located at "./mypackage",
then you would invoke the program like this:
```shell
adddheader mypackage --text copyright.txt
```
By default, the header will not be added to "__init__.py" files.

### Additional actions

If you want to see which files would be changed without modifying them, add
`-n` or `--dry-run` to the command line arguments.
If this argument is given, any arguments related to modifying or removing headers will be ignored.

If you want to remove existing headers instead of adding or updating them,
add `-r` or `--remove` to the command line arguments.

### Specifying file patterns

You can customize the files that
are modified with the `-p` or `--pattern` argument, which takes a UNIX glob-style pattern and can be
repeated as many times as you like. To help exclude files, if the '~' is the first letter of the pattern,
then the rest of the pattern is used to exclude (not include) files. So, for example, if you provide the
following source code tree:
```
mypackage
   __init__.py
   foo.py
   bar.py
   tests/
       __init__.py
       test_foo.py
       test_bar.py
```
The following commands would match the following lists of files:

* `addheader mypackage -t header.txt -p *.py`  
mypackage/{__init__.py, foo.py, bar.py}, mypackage/tests/{__init__.py, test_foo.py, test_bar.py}
* `addheader mypackage -t header.txt -p *.py -p ~__init__.py`  
mypackage/{foo.py, bar.py}, mypackage/tests/{test_foo.py, test_bar.py}
* `addheader mypackage -t header.txt -p *.py -p ~__init__.py -p ~test_*.py`  
mypackage/{foo.py, bar.py}

### Header delimiters

The header itself is, by default, delimited by a line of 78 '#' characters. While _detecting_ an existing
header, the program will look for any separator of 10 or more '#' characters.
For example, if you have a file that looks like this:
```python
##########
my header with 10
hashes above and below
##########
hello
```

and a header text file containing simply "Hello, world!", then the modified
header will be:
```python
##############################################################################
# Hello, world!
##############################################################################
hello
```

The comment character and separator character, as well as the width of the
separator, can be modified with command-line options. For example, to add
a C/C++ style comment as a header, use these options:

```shell
addheader mypackage --comment "//" --sep "=" --sep-len 40 -t myheader.txt
```

This will insert a header that looks like this:
```
//========================================
// my text goes here
//========================================
```

Keep in mind that subsequent operations on files with this header, including
`--remove`, will need the same `--comment` and `--sep`
arguments so that the header can be properly identified. For example,
running `addheader mypackage --remove` after the above command will not
remove anything, and `addheader mypackage -t myheader.txt` will insert a 
second header (using the default comment character  and separator). To avoid
passing command-line arguments every time, see the "Configuration" section.

### Adding headers to Jupyter notebooks

Starting in version 0.3.0, you can add headers to Jupyter Notebooks as well.
To enable this, add a `-j` or `--jupyter` argument to the command-line, or
similarly add a `jupyter: {suffix}` argument in the configuration file.

The _{suffix}_ indicates an alternate file suffix to use for identifying
whether a file is a Jupyter Notebook, where the default is ".ipynb".
In the configuration file, use `jupyter: true` to use the default.
On the command-line, simply omit the value to use the default.

The file pattern arguments (see "Specifying file patterns", above) are still honored,
but if Jupyter notebooks are enabled, the pattern `*{suffix}` will be automatically added
 to the patterns to match. Thus, by default `*.ipynb` will be added to the files to match.

The Jupyter notebook header will be placed in the first 'cell', i.e. the first
item in the notebook, which will be of type "markdown" (i.e., non-code).
The content of the header is the same as for text files, but without the 
delimiters.
Two tags will be added to the cell metadata:
* `header` - Indicates this is the header cell, so it can be modified or removed later.
* `hide-cell` - If you build documentation with Jupyterbook, this will hide the cell in the generated documentation behind a toggle button (see https://jupyterbook.org/interactive/hiding.html).

Just as for text files, Jupyter notebook headers can be updated or removed.

For reference, below is the form of the generated Jupyter notebook cell JSON:

```json
    {
      "id": "ff7680061ca9429081cf6c9791d2a444",
      "cell_type": "markdown",
      "metadata": {
        "tags": [
          "header",
          "hide-cell"
        ]
      },
      "source": [
        "Lines of the input text file",
        "are copied into this array."
      ]
    }
```

### Configuration
To avoid passing commandline arguments every time, you can create a configuration
file that simply lists them as key/value pairs (using the long-option name as
the key). By default, the program will look for a file `addheader.cfg` in the
current directory, but this can also be specified on the command-line with 
`-c/--config`. For example:

```shell
addheader  # looks for addheader.cfg, ok if not present
addheader -c myoptions.conf  # uses myoptions.conf, fails if not present
```

The configuration file is in YAML format. For example:

```yaml
text: myheader.txt
pattern:
   - "*.py"
   - "~__init__.py"
# C/Java style comment block
sep: "-"
comment: "//"
sep-len: 40
# Verbosity as a number instead of -vv
verbose: 2
```

Command-line arguments will override configuration arguments, even if the
configuration file is explicitly provided with `-c/--config`.
The "action" arguments, `-r/--remove` and `-n/--dry-run`, will be
ignored in the configuration file.

### Using in tests

To test your package for files missing headers, use the following formula:
```python
import os
import mypackage
from addheader.add import FileFinder, detect_files

def test_headers():
    root = os.path.dirname(mypackage.__file__)
    # modify patterns to match the files that should have headers
    ff = FileFinder(root, glob_pat=["*.py", "~__init__.py"])
    has_header, missing_header = detect_files(ff)
    assert len(missing_header) == 0
```

## Credits
The _addheader_ program was developed for use in the [IDAES](www.idaes.org) project and is maintained in the
IDAES organization in Github at https://github.com/IDAES/addheader . The primary author
and maintainer is Dan Gunter (dkgunter at lbl dot gov).

## License
Please see the COPYRIGHT.md and LICENSE.md files in the repository for
limitations on use and distribution of this software.