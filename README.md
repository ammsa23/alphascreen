# alphascreen

## Features

* Gets the fasta sequences from uniprot IDs contained in a table of paired proteins.

* Chops up the sequences so that they are of reasonable size for subsequent pairwise predictions.

* Interprets the PAEs only in the region relating to the protein-protein interaction of interest.

* Generates summaries, including a PDF showing the PAE plot next to snapshots of the models.

## Explanation

Use this package to generate fastas for a set of interaction partners to run Alphafold predictions. The input to ```parse``` is a table which includes two columns containing uniprot IDs for the interaction partners (```columnA``` and ```columnB```). These can be generated by BioGRID for example. The sequences are fetched from Uniprot and fragmented before generating fasta files. Fragmenting helps keep the total sequence length short enough so the jobs don't run out of memory (```fragment``` and ```overlap```).

The output is a bash script that allows you to run Alphafold on all the generated fasta files on your machine/cluster. The syntax for job submission will likely not correspond to what you use in your system. You can either edit the output file "colabfoldrun.bsh" or "jobsetup.py" itself so that the alphafold submission commands have the right syntax. If you do change this, make sure the results are output into the "results" directory, which is important for the analysis command ```show_top``` to work. The package has only been tested on Colabfold 1.3.0 and therefore its file naming system. Be careful since you can have many jobs

The default behaviour for the analysis is for each run to look at all the PAE plots in the region corresponding to the protein interaction you are screening for, and choose the best one. Then, all those predictions are compiled and ranked by comparing their respective PAE plots at the interaction.

If you want instead want to rank by iptm score, you can pass ```--rankby iptm```. This relies on your Alphafold/Colabfold implementation outputing the iptm and ptm scores into a "scores.txt" file within the individual results folders. It should have five lines, each containing "iptm:0.09 ptm:0.62" (values are just an example) representing the score for the predictions generated by each of the five models for that run.

The results can be compiled into a pdf showing all PAEs next to snapshots of the predictions ranked by iptm score (```show_top```).

## Installation<a name="installation"></a>

* Set up a fresh conda environment with Python >= 3.8: `conda create -n alphascreen python=3.8`

* Activate the environment: `conda activate alphascreen`.

* Install alphascreen: **`pip install alphascreen`**

* Install pymol dependancies: **`conda install -c schrodinger pymol-bundle`**

## Usage<a name="usage"></a>

### Job setup

```
alphascreen --parse uniprot-id-1/uniprot-id-2 [options]
```

Generate the fasta files and Alphafold commands for the input uniprot IDs.

```
alphascreen --parse filename [options]
```

Generate the fasta files and Alphafold commands for the input table.

**Options**

**```--focus```** *```uniprot-id```*

Uniprot ID to focus on. This means that it will the first chain in any predictions that contain it.

**```--fragment```** *```length```*

Approximate fragment length. Default is 500.

**```--overlap```** *```length```*

Sequence is extended by this amount on either side of slices. Default is 50.

**```--dimerize```** *```uniprot-id```* *or* *```uniprot-ids.txt```*

Dimerize this uniprot ID whenever it is found.

**```--dimerize_all```**

Dimerize all proteins.

**```--dimerize_all_except```** *```uniprot-ids.txt```*

Provide a text file (.txt) with a single column list of uniprot IDs to NOT dimerize. Everything else will be dimerized.

**```--consider```** *```uniprot/start/end```*

Uniprot ID and sequence range to consider. Example: "Q86VS8/1/200" only considers amino acids 1-200 for uniprot ID Q86VS8.

**```--alphafold_exec```** *```path-to-colabfold-executable```*

Path to script that runs Colabfold for writing the commands. Default is "colabfold2".

**```--columnA```** *```columnA-name```*

Name of column heading for uniprot IDs for the first set of interactors. Default is "SWISS-PROT Accessions Interactor A".

**```--columnB```** *```columnB-name```*

Name of column heading for uniprot IDs for the second set of interactors. Default is "SWISS-PROT Accessions Interactor B".

### Check runs

```
alphascreen --check
```

Checks how many runs are finished so far.

```
alphascreen --write_unfinished
```

Checks how many runs are finished so far and writes out a new bash script with the remaining Colabfold commands.

### Analyze results

```
alphascreen --show_top threshold [options]
```

Generate summary files for the runs so far. Only models whereby the parameter specified by ```rankby``` is above the threshold valuewill be considered (e.g. ```alphascreen --show_top 0.3 --rankby iptm``` for iptms above 0.3).

```
alphascreen --write_table [options]
```

Only output all the results into a table ranked by iptm score. This is run automatically in ```--showtop```.

**Options**

**```--rankby```** *```pae```* or *```iptm```* or *```ptm```*

Score by which models are ranked (pae, iptm, or ptm). Default is pae. This is used for both choosing the best model in a prediction as well as ranking those chosen models in the summary files. The option ```pae``` will look for the deepest PAE valleys only in the parts of the plot that are interactions between **different** proteins. The PAE is scaled to be between 0 and 1 where higher values are better predictions (```--show_top 0.8``` is a good starting point) The options ```iptm``` and ```ptm``` rely on a scores.txt file in each results directory (see explanation at the top).

**```--overwrite```**

Overwrite snapshots that have already been generated, otherwise it will skip those to save time. This is only relevant for ``show_top``.