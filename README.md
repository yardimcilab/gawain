**Wistan** is a platform to build, run, organize and share data analysis pipelines.

**Author:** Ben Skubi, M.S. BME (skubi@ohsu.edu)
**Institutional affiliation:** [Yardimci](https://www.ohsu.edu/people/galip-grkan-yardimci-phd) and [Adey](https://adeylab.org/) labs, [CEDAR](https://www.ohsu.edu/knight-cancer-institute/cedar), [OHSU BME](https://www.ohsu.edu/school-of-medicine/biomedical-engineering)

## Table of Contents
- [Installation](#installation)
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

## Anatomy of a Wistan pipeline

For mass adoption, you should probably make a tailored command-line interface for your pipeline. Here, we're going to look at the building blocks
of the first pipeline built with Wistan. It originates in the bioinformatics field of the study of chromatin conformation. The author was using a tool
called [hicrep](https://github.com/dejunlin/hicrep) to compute sample-vs-sample reproducibility scores for a large number of data files. The challenge
was that `hicrep` only allows one to to run a single sample-vs-sample comparison. Its output is a plaintext series of comments and per-chromosome
reproducibility scores in a text file.

The author wanted to batch `hicrep` comparisons on a large number of data files, compute the mean of the per-chromosome scores for each comparison,
and produce a [clustermap](https://seaborn.pydata.org/generated/seaborn.clustermap.html) with row and column captions being the data file prefixes.
Having a whole PhD in this subfield ahead of him, the author wanted a tool to make this conveniently in the future.

The pipeline starts with a single two-cell Jupyter notebook.

**Cell 1: Initializing `sanb`**
```
import sanb
sanb.set_notebook("demo.ipynb")
```

This initializes `sanb` with the name of the notebook, making it possible for the pipeline to append its outputs to the notebook itself.

**Cell 2: Download data**
```
!mkdir -p ~/data
!wget -P ~/data/ https://ftp.ncbi.nlm.nih.gov/geo/samples/GSM4604nnn/GSM4604290/suppl/GSM4604290%5F990.iced.mcool
!wget -P ~/data/ https://ftp.ncbi.nlm.nih.gov/geo/samples/GSM4604nnn/GSM4604276/suppl/GSM4604276%5F868.iced.mcool
!wget -P ~/data/ https://ftp.ncbi.nlm.nih.gov/geo/samples/GSM4604nnn/GSM4604278/suppl/GSM4604278%5F1953.iced.mcool
```
This cell downloads our data (390M).

**Cell 3: Saving plugin script**

```
hicrep_mean=r'''#!/usr/bin/env python3
import sys, statistics, click

hicrep_output = sys.stdin.read().split(\"\\n\")

scc_scores = []
for line in hicrep_output:
    try:
        scc_scores.append(float(line))
    except ValueError:
        continue
click.echo(statistics.mean(scc_scores))'''
!mkdir -p ~/scripts && echo "$hicrep_mean" > ~/scripts/hicrep_mean
!chmod +x ~/scripts/hicrep_mean
```
A single, small, specialized plugin is necessary to transform the raw text output from our `hicrep` analysis tool into the mean value we
are interested in. For easy sharing, we store the plugin in the Jupyter notebook itself and write it to a file for use in the main pipeline.
Note that setting permissions is important!

**Cell 3: Main pipeline**

```
sanb.last = "samb_cell0"; sanb.lidx = sanb.lastidx()
!mkdir -p scc
!ls ~/Desktop/zhang_aml/aml/*.mcool | itertools-cli combinations-with-replacement 2 | curry-batch "echo {{1}}" "echo {{2}}" "pathlib-cli prefix {{1}}" "pathlib-cli prefix {{2}}" > hicrep_inputs.txt
!cat hicrep_inputs.txt | curry-batch "hicrep {{1}} {{2}} scc/{{3}}_{{4}}.txt --h 1 --binSize 1000000 --dBPMax 5000000" > /dev/null && cat hicrep_inputs.txt | curry-batch "echo {{3}}" "echo {{4}}" "cat scc/{{3}}_{{4}}.txt" > hicrep_results.txt
!cat hicrep_results.txt | curry-batch "echo {{1}}" "echo {{2}}" "echo '{{3}}' | ~/scripts/hicrep_mean" | curry-batch "echo {{1}}" "echo {{2}}" "echo {{3}}" "echo {{2}}" "echo {{1}}" "echo {{3}}" | pandas-cli dataframe "df = pd.DataFrame(df.values.reshape(2*df.shape[0], 3)).drop_duplicates().pivot(index=0, columns=1, values=2).apply(pd.to_numeric)" > hicrep_data.yaml
!datavis-cli load-dataframe hicrep_data.yaml df | nbformat-cli cell add {sanb.notebook} {sanb.lidx} --distance 1
!datavis-cli clustermap df | nbformat-cli cell add {sanb.notebook} {sanb.lidx} --distance 2
```

Let's take this line by line.

```
sanb.last = "samb_cell0"; sanb.lidx = sanb.lastidx()
```
This gives the cell a unique ID, stored in the variable `sanb.last`. The `sanb.lastidx()` function looks up in this notebook the index
of a cell containing the phrase `sanb.last = [value of sanb.last]`. This retrieves the index of the current cell.

```
!mkdir -p scc
```
We create a folder to store the 289 intermediateper-comparison output files our analysis tool will generate for a 17-sample cross-comparison.

```
!ls ~/data/*.mcool | itertools-cli combinations-with-replacement 2 | curry-batch "echo {{1}}" "echo {{2}}" "pathlib-cli prefix {{1}}" "pathlib-cli prefix {{2}}" > hicrep_inputs.txt
```
The data to be compared has a file extension `.mcool`. We load all the filenames to be compared, then generate combinations with replacement.
We then extend this to a list of four strings: the two filenames, plus the filename prefixes that we wish to use.
We store this combination in `hicrep_inputs.txt` using a redirect for convenience and as a record of our output so far.

```
!cat hicrep_inputs.txt | curry-batch "hicrep {{1}} {{2}} scc/{{3}}_{{4}}.txt --h 1 --binSize 1000000 --dBPMax 5000000" > /dev/null && cat hicrep_inputs.txt | curry-batch "echo {{3}}" "echo {{4}}" "cat scc/{{3}}_{{4}}.txt" > hicrep_results.txt
```
We load the list of filenames and prefixes. Note that this is not strictly necessary - we could have just piped the output from the previous line into this one.
Next, we call the main `hicrep` command to compute reproducibility scores on all pairs of input files. For each list of `file1 file2 prefix1 prefix2` strings in the input:
 - `{{1}}` and `{{2}}` are replaced by `file1` and `file2`, and represent the input files.
 - `{{3}}` and `{{4}}` are replaced by `prefix1` and `prefix2`. `scc/{{3}}_{{4}}.txt` will be the location of the output file.
We silence the message outputs from hicrep by redirecting them to `/dev/null`. The `hicrep` program saves its outputs to individual text files, so we need to retrieve them.
We do this by once again piping our comparison filenames and prefixes and using `curry-batch` to load the raw output file contents using `cat`.
Again, we save this output to `hicrep_results.txt` as a record of our work so far and for convenience, but this isn't strictly necessary.

```
!cat hicrep_results.txt | curry-batch "echo {{1}}" "echo {{2}}" "echo '{{3}}' | ~/scripts/hicrep_mean" | curry-batch "echo {{1}}" "echo {{2}}" "echo {{3}}" "echo {{2}}" "echo {{1}}" "echo {{3}}" | pandas-cli dataframe "df = pd.DataFrame(df.values.reshape(2*df.shape[0], 3)).drop_duplicates().pivot(index=0, columns=1, values=2).apply(pd.to_numeric)" > hicrep_data.yaml
```
We use `curry-batch` to feed in our previous results into a custom Python script `hicrep_mean` tailor-made for this step, while preserving the filename prefixes for later captioning.
```
#!/usr/bin/env python3
import sys, statistics, click

hicrep_output = sys.stdin.read().split('\n')
scc_scores = []
for line in hicrep_output:
    try:
        scc_scores.append(float(line))
    except ValueError:
        continue
click.echo(statistics.mean(scc_scores))
```
This script merely filters for floating-point numbers in the output for a particular comparison, computes the mean, and prints it to `stdout`. This is the only analysis-specific script in the pipeline.
We then use `curry-batch` to duplicate the result with the order reversed, so that we can conveniently produce a symmetric matrix. This gives us a six-element `col_m row_n value col_n row_m value` list.
We then load this into a pandas dataframe and reshape it to get the desired square, symmetric matrix labeled with our captions. First, we append it to a `2nx3` matrix, with row caption, column caption,
and value columns. As the main diagonal is duplicated, we drop duplicates. Then we create a pivot dataframe to get it into the desired square shape and convert the values to numeric. Finally, we save this
to a data file `hicrep_data.yaml` for convenience.

```
!datavis-cli load-dataframe hicrep_data.yaml df | nbformat-cli cell add {sanb.notebook} {sanb.lidx} --distance 1
```
Our computations are done, and all that remains is to create a figure in the interactive environment of our Jupyter notebook for easy tweaking.
This function starts by leveraging the `sanb` package to add a new cell to the notebook immediately after the pipeline-running cell containing code
to load our output data into a pandas DataFrame called `df`.

```
!datavis-cli clustermap df | nbformat-cli cell add {sanb.notebook} {sanb.lidx} --distance 2
```
Finally, we add another cell immediately after the data-loading function that contains the function call
to produce a Seaborn clustermap from our data. The arguments to that function are comprehensively and explicitly
initialized to default values, and a link to the Seaborn clustermap API is printed for reference. The clustermap
uses a perceptually-accurate, colorblind-friendly colormap from [colorcet](https://colorcet.com/download/index.html) by default. These parameters
can be tweaked in the cell and the clustermap immediately reproduced.

After these cells are produced by the pipeline cell, the notebook will need to be reloaded using `File/Reload Notebook From Disk`.
![image](https://github.com/yardimcilab/wistan/assets/86805107/4a9afd91-d88b-4966-851d-d2d260068ea9)


**Cell 4: Dynamically generated data-loader**
```

import pandas as pd
import yaml

with open("hicrep_data.yaml", 'r') as file:
    df = pd.DataFrame(yaml.safe_load(file))
print(df.head())
print(df.describe())
```
This merely loads our output data from the main pipeline and prints some summary statistics. The cell and its code are auto-generated by the pipeline.

**Cell 5: Dynamically generated clustermap visualization**
```

import seaborn as sns
import matplotlib.pyplot as plt
import colorcet as cc

# Default colormap is perceptually-accurate and colorblind-friendly
# See https://colorcet.com/index.html for details
# Seaborn clustermap documentation
# https://seaborn.pydata.org/generated/seaborn.clustermap.html
sns.clustermap(df, 
               pivot_kws=None, 
               method='average', 
               metric='euclidean', 
               z_score=None, 
               standard_scale=None, 
               figsize=(10, 10), 
               cbar_kws=None, 
               row_cluster=True, 
               col_cluster=True, 
               row_linkage=None, 
               col_linkage=None, 
               row_colors=None, 
               col_colors=None, 
               mask=None, 
               dendrogram_ratio=0.2, 
               colors_ratio=0.03, 
               cbar_pos=(0.02, 0.8, 0.05, 0.18), 
               tree_kws=None, 
               cmap=cc.cm.CET_CBL1)
```
This is pipeline-generated boilerplate code to make a Seaborn clustermap displaying our results, which are now stored in `df`. 

As you can see, outside the tiny analysis-specific `hicrep_mean` plugin script, all components of this pipeline are simple reusable parts.
Although this pipeline is unwieldly in the exposed form presented here, it can be easily wrapped into a more convenient command line interface
exposing just the variables the user needs to set. We provide it here as an example of how to build a useful Wistan pipeline.

## Why "Wistan"?

One of the author's favorite books is [The Buried Giant](https://www.amazon.com/Buried-Giant-Vintage-International/dp/0307455793) by [Kazuo Ishiguro](https://en.wikipedia.org/wiki/Kazuo_Ishiguro). The breath of the dragon Querig spreads amnesia across the land of medieval Briton. Wistan slays the dragon, dispelling the mist.
