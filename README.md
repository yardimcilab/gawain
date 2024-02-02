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
