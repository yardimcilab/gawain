**Gawain** is an environment for building, running, organizing and sharing data analysis pipelines.

**Install**
```
curl -O https://raw.githubusercontent.com/yardimcilab/gawain/main/gawain.yaml
mamba env create -f gawain.yaml
mamba activate gawain
```

Gawain's paradigm is the [Unix philosophy](http://www.catb.org/~esr/writings/taoup/html/index.html) of software design.

It is built on an infrastructure of [small](http://www.catb.org/~esr/writings/taoup/html/ch01s06.html#id2878022), [simple](http://www.catb.org/~esr/writings/taoup/html/ch01s06.html#id2877917), [modular](http://www.catb.org/~esr/writings/taoup/html/ch01s06.html#id2877537) [generative tools](http://www.catb.org/~esr/writings/taoup/html/ch01s06.html#id2878742), [composed](http://www.catb.org/~esr/writings/taoup/html/ch01s06.html#id2877684) into [YAML](https://yaml.org/)-based [textual](http://www.catb.org/~esr/writings/taoup/html/ch05s01.html) and [extensible](http://www.catb.org/~esr/writings/taoup/html/ch01s06.html#id2879112) pipelines.

Gawain's is modest. All its parts are orthogonal. You can use and adjust the parts you like:
 - A particular infrastructure tool, like `itertools-cli`
 - A single analysis pipeline built on Gawain infrastructure
 - `sanb` for self-aware Jupyter notebooks

It is fully compatible with standard workflow management tools like [Snakemake](https://snakemake.readthedocs.io/en/stable/) and [Nextflow](https://www.nextflow.io/).
Indeed, its original motivation was to add value to these systems by making it easier to write the shell-based analyses that they orchestrate.

Gawain is ambitious.
 - Infrastructure tools make it easy to orchestrate complex comparisons across data samples.
 - Pipelines run from within self-aware Jupyter notebooks append results to the notebook itself.
 - Figures are produced and tweaked in the notebook's interactive environment.
 - In the future, Gawain will include the ability to produce and store structured descriptions of projects, data and analyses with ontologies,
   such as those hosted at [BioPortal](https://bioportal.bioontology.org/) and the [Ontology Lookup Service (OLS)](https://www.ebi.ac.uk/ols4).

When used to its full potential, Gawain results in a Jupyter notebook containing both the pipeline(s) and its intermediate and final outputs.
This makes the analysis instantly reproducible and easy to share. It is also helpful when you wish to review your own work in the future.

**YAML, stdin and stdout, |, ||, >, >>, &, &&, (), ; and tee**

Not all users are familiar with YAML or the Unix flow-of-control operators.

If you want to use a Gawain pipeline, fortunately, you don't need to. But you most likely will
if you want to build a pipeline on Gawain infrastructure.

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

**stdin (Standard Input)** is a stream from which a program reads its input data. It's typically associated with the keyboard input or another program's output.
**stdout (Standard Output)** is a stream to which a program writes its output data. It's typically displayed on the screen or can be redirected to a file or another program.
**| (Pipe):** Used to pass the output of one command as the input to another command. For example, `ls | grep "txt"` uses the output of ls as the input for grep to filter out lines containing "txt".
**|| (OR):** Used between two commands. The second command is executed only if the first command fails (exits with a non-zero status). For example, `command1 || command2 means command2` is executed only if `command1` fails.
**> (Redirect Output):** This operator is used to redirect the output of a command to a file, overwriting the file if it already exists. For example, `echo "Hello" > file.txt` writes "Hello" to file.txt.
**>> (Append Output):** Similar to `>`, but it appends the output to the file instead of overwriting it. For example, `echo "World" >> file.txt` adds "World" to the end of file.txt.
**& (Background Execution):** When added at the end of a command, it runs the command in the background, allowing the user to continue using the shell without waiting for the command to complete. For example, `long_running_command &` starts the command and returns the prompt immediately.
**&& (AND):** Used between two commands. The second command is executed only if the first command succeeds (exits with a status of zero). For example, `command1 && command2` means command2 is executed only if `command1` is successful.
**() (Subshell):** Commands inside parentheses are executed in a subshell. This means they are executed in a separate process, and any changes to the environment (like changing directories or setting variables) do not affect the current shell. For example, `(cd folder; command)` runs command in folder, but the current shell's directory does not change.
**; (Sequential Execution):** It separates commands to be executed sequentially, regardless of the success or failure of the previous command. For example, `command1; command2` executes `command1` and then `command2`, one after the other.
**tee (Branch Output):** The tee command reads from standard input and writes to both standard output and one or more files, effectively branching the output. For example, `command | tee file.txt` displays the output of command on the screen and also writes it to file.txt.

**Description of tools**

`itertools-cli`: a CLI for parts of the [itertools] library, operating on 
