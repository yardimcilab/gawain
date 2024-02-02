**Wistan** is a platform to build, run, organize and share data analysis pipelines.

**Author:** Ben Skubi, M.S. BME (skubi@ohsu.edu)

**Institutional affiliation:** [Yardimci](https://www.ohsu.edu/people/galip-grkan-yardimci-phd) and [Adey](https://adeylab.org/) labs, [CEDAR](https://www.ohsu.edu/knight-cancer-institute/cedar), [OHSU BME](https://www.ohsu.edu/school-of-medicine/biomedical-engineering)



## Table of Contents
- [Installation](#installation)
- [Demo](#demo)
- [Paradigm](#wistans-paradigm)
- [Infrastructure tools](#wistans-infrastructure-tools)
- [Introduction to Unix pipeline control](#introduction-to-unix-pipeline-control)

![Wistan Icon](Wistan.png)

## Installation
```
curl -O https://raw.githubusercontent.com/yardimcilab/wistan/main/wistan.yaml
mamba env create -f wistan.yaml
mamba activate wistan
```

## Demo
After installing Wistan and activating the environment, you can [download](https://raw.githubusercontent.com/yardimcilab/wistan/main/demo-before.ipynb) a carefully-described working demo pipeline. This demo takes you from downloading the data to running the analysis and visualizing the results. Alternatively, you can view the notebook [before](https://nbviewer.org/github/yardimcilab/wistan/blob/main/demo-before.ipynb) and [after](https://nbviewer.org/github/yardimcilab/wistan/blob/main/demo-after.ipynb) the analysis.

## Wistan's paradigm
Wistan embodies the [Unix philosophy](http://www.catb.org/~esr/writings/taoup/html/index.html) of software design.

It is built on an infrastructure of [small](http://www.catb.org/~esr/writings/taoup/html/ch01s06.html#id2878022), [simple](http://www.catb.org/~esr/writings/taoup/html/ch01s06.html#id2877917), [modular](http://www.catb.org/~esr/writings/taoup/html/ch01s06.html#id2877537) [generative tools](http://www.catb.org/~esr/writings/taoup/html/ch01s06.html#id2878742), [composed](http://www.catb.org/~esr/writings/taoup/html/ch01s06.html#id2877684) into [YAML](https://yaml.org/)-based [textual](http://www.catb.org/~esr/writings/taoup/html/ch05s01.html) and [extensible](http://www.catb.org/~esr/writings/taoup/html/ch01s06.html#id2879112) pipelines.

**Wistan is modest.** All its parts are orthogonal. You can use and adjust the parts you like:
 - A particular infrastructure tool, like `itertools-cli`
 - A single analysis pipeline built on Wistan infrastructure
 - `sanb` for self-aware Jupyter notebooks

It is fully compatible with standard workflow management tools like [Snakemake](https://snakemake.readthedocs.io/en/stable/) and [Nextflow](https://www.nextflow.io/).
Indeed, its original motivation was to add value to these systems by making it easier to write the shell-based analyses that they orchestrate.

**Wistan is ambitious.**
 - Infrastructure tools make it easy to orchestrate complex comparisons across data samples.
 - Pipelines run from within self-aware Jupyter notebooks append results to the notebook itself.
 - Figures are produced and tweaked in the notebook's interactive environment.
 - In the future, Wistan will include the ability to produce and store structured descriptions of projects, data and analyses with ontologies,
   such as those hosted at [BioPortal](https://bioportal.bioontology.org/) and the [Ontology Lookup Service (OLS)](https://www.ebi.ac.uk/ols4).

When used to its full potential, Wistan results in a Jupyter notebook containing both the pipeline(s) and its intermediate and final outputs.
This makes the analysis instantly reproducible and easy to share. It is also helpful when you wish to review your own work in the future.

## Wistan's infrastructure tools

 - `itertools-cli`: A CLI for parts of the [itertools](https://docs.python.org/3/library/itertools.html) library. Generate combinations of filenames and other things. Receives data from `stdin` and writes it YAMLized to `stdout`.
 - `pathlib-cli`: A CLI for parts of the [pathlib](https://docs.python.org/3/library/pathlib.html) library. Extract parts of paths, such as the directory, prefix, or suffix.
 - `pandas-cli`: A CLI for constructing a [pandas DataFrames](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.html) named `df` from YAML, executing arbitrary Python commands on it, and (optionally) outputting the YAMLized result. This can be helpful to reshape a dataframe for analysis or display.
 - `curry-batch`: A text filter and command orchestrator. Input is a YAML list of argument lists piped from `stdin` as well as a list of command templates passed as arguments.
   All argument lists are the same length. Member `n` in each argument list replaces the wildcard `{n}` in each command if present. Not all wildcards need to be used or replaced. This produces a set of [curried](https://python-course.eu/advanced-python/currying-in-python.php) functions, which are run batched in parallel. The `stdout` from each curried function is stored in a new list in the order in which the commands were listed. Outputs from the set of curried functions are in the same order as the arguments passed. This can be used:
    - As a simple text filter (`cat args.txt | curry-batch "echo {1}" "echo {2}" "echo {1}{1}{2}{2}"`)
    - To orchestrate execution of the analysis tool
    - To load the contents of output files scattered on the hard drive into a single dataframe for aggregation, analysis and display
    Note that if you are using `curry-batch` in the context of a Jupyter notebook in a `!`-prefixed shell command, its brackets must be double-escaped (i.e. `{{1}}`).
  - `datavis-cli`: A CLI to output boilerplate Python code for loading YAML data to a pandas dataframe and producing a figure based on it. Currently just supports Seaborn's `heatmap` and `clustermap`, but can be straightforwaredly extended to any data visualization package in any language.
  - `nbcell-check-cli`: A tool for running regex on Jupyter notebook cells using Python's [re](https://docs.python.org/3/library/re.html) package. Returns the index of cells matching the regex.
  - `nbformat-cli`: A tool for manipulating Jupyter notebook cells at the command line, based on the [nbformat](https://nbformat.readthedocs.io/en/latest/) package. Currently only adds cells at a specified index, but aims to perform other manipuations: removing, overwriting, deleting, moving, reading and writing notebook and cell metadata and so forth.
  -  `sanb`: Used in conjunction with `nbcell-check-cli` and `nbformat-cli` to make Jupyter notebooks "self-aware." Notebook cells normally have no ability to access their own cell metadata or notebook metadata, making it difficult to make notebooks self-editing. `sanb` allows the user to specify the path of the notebook itself and give cells cell-specific identifiers in code. This lets cells discover their own index in the list of cells the notebook contains. In turn, this allows code in cells to edit themselves and the notebook as a whole. This is what permits a Wistan pipeline run from a Jupyter notebook to append to itself the results of a pipeline, such as code to make and edit a figure.

## Introduction to Unix pipeline control
**YAML, stdin and stdout, !, |, ||, >, >>, &, &&, (), ; and tee**

Not all users are familiar with YAML or the Unix flow-of-control operators.

If you want to use a Wistan pipeline, fortunately, you don't need to. But you most likely will
if you want to build a pipeline on Wistan infrastructure.

Fortunately, these can become second-nature in a day or two.

As its developers say, "YAML is a human-friendly data serialization language for all programming languages."
It can encode nested `lists` and `dicts`. Here is an example of a list of 2-element lists.
```
- - file1.txt
  - file1.txt
- - file1.txt
  - file2.txt
- - file2.txt
  - file1.txt
- - file2.txt
  - file2.txt
```
Popular languages like [Python](https://www.python.org/) have simple ways to load and dump data structures to YAML.
Instead of trying to pass binary datastructures directly between pipeline components, we serialize them to YAML and
pass the text.

- **stdin (Standard Input)** is a stream from which a program reads its input data. It's typically associated with the keyboard input or another program's output.
- **stdout (Standard Output)** is a stream to which a program writes its output data. It's typically displayed on the screen or can be redirected to a file or another program.
- **!:** In Jupyter notebooks, lines prefixed with `!` are interpreted as shell commands. You can store the value of variables from non-`!` using `{}`-enclosed wildcards. Note that if you are using `curry-batch`, its brackets must be double-escaped (i.e. `{{1}}`).
- **| (Pipe):** Used to pass the output of one command as the input to another command. For example, `ls | grep "txt"` uses the output of ls as the input for grep to filter out lines containing "txt".
- **|| (OR):** Used between two commands. The second command is executed only if the first command fails (exits with a non-zero status). For example, `command1 || command2 means command2` is executed only if `command1` fails.
- **> (Redirect Output):** This operator is used to redirect the output of a command to a file, overwriting the file if it already exists. For example, `echo "Hello" > file.txt` writes "Hello" to file.txt.
- **>> (Append Output):** Similar to `>`, but it appends the output to the file instead of overwriting it. For example, `echo "World" >> file.txt` adds "World" to the end of file.txt.
- **& (Background Execution):** When added at the end of a command, it runs the command in the background, allowing the user to continue using the shell without waiting for the command to complete. For example, `long_running_command &` starts the command and returns the prompt immediately.
- **&& (AND):** Used between two commands. The second command is executed only if the first command succeeds (exits with a status of zero). For example, `command1 && command2` means command2 is executed only if `command1` is successful.
- **() (Subshell):** Commands inside parentheses are executed in a subshell. This means they are executed in a separate process, and any changes to the environment (like changing directories or setting variables) do not affect the current shell. For example, `(cd folder; command)` runs command in folder, but the current shell's directory does not change.
- **; (Sequential Execution):** It separates commands to be executed sequentially, regardless of the success or failure of the previous command. For example, `command1; command2` executes `command1` and then `command2`, one after the other.
- **tee (Branch Output):** The tee command reads from standard input and writes to both standard output and one or more files, effectively branching the output. For example, `command | tee file.txt` displays the output of command on the screen and also writes it to file.txt.

## Why "Wistan"?

One of the author's favorite books is [The Buried Giant](https://www.amazon.com/Buried-Giant-Vintage-International/dp/0307455793) by [Kazuo Ishiguro](https://en.wikipedia.org/wiki/Kazuo_Ishiguro). The breath of the dragon Querig spreads amnesia across the land of medieval Briton. Wistan slays the dragon, dispelling the mists of oblivion once and for all.
