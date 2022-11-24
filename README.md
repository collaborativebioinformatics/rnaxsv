# Isocomp: comparing high-quality IsoSeq3 isoforms between samples

![](images/logo.png)

## Contributors
1. Yutong Qiu (Carnegie Mellon)
2. Chia Sin	Liew (University of Nebraska-Lincoln)
3. Chase Mateusiak (Washington University)
4. Rupesh Kesharwani (Baylor College of Medicine)
5. Bida	Gu (University of Southern California)
6. Muhammad Sohail Raza (Beijing Institute of Genomics, Chinese Academy of Sciences/China National Center for Bioinformation)
6. Evan	Biederstedt (HMS)

## Detailed project overview
https://github.com/collaborativebioinformatics/isocomp/blob/main/FinalPresentation_BCM_Hackathon_12Oct2022.pdf 

## Introduction
NGS targeted sequencing and WES have become routine for clinical diagnosis of Mendelian disease [CITATION]. Family sequencing (or "trio sequencing") involves sequencing a patient and parents (trio) or other relatives. This improves the diagnostic potential via the interpretation of germline mutations and enables detection of de novo mutations which underlie most Mendelian disorders. 

Transcriptomic profiling has been gaining used over the past several decades. However, this endeavor has been hampered by short-read sequencing, especially for inferring alternative splicing, allelic imbalance, and isoform variation. 

The promise of long-read sequencing has been to overcome the inherent uncertainties of short-reads. 

Something something Isoseq3: https://www.pacb.com/products-and-services/applications/rna-sequencing/

Provides high-quality, polished, assembled full isoforms. With this, we will be able to identify alternatively-spliced isoforms and detect gene fusions. 

Since the advent of HiFi reads, the error rates have plummeted. 

The goal of this project will be to extend the utility of long-read RNAseq for investigating Mendelian diseases between multiple samples. 

And what about gene fusions? We detect these in the stupidest possible way with short-read sequencing, and we think they're cancer-specific. What about the germline?


## Goals

Given high-quality assembled isoforms from 2-3 samples, we want to algorithmically (definitively) characterize the "unique" (i.e. differing) isoforms between samples.

## Methods

### Methods overview
[Isoform set comparison problem] Given two sets of isoform sam, find two subsets so that each subset of isoform is unique to each sample.

To solve this problem, the naive solution is to perform sequence match between two sets of isoforms. However, this method is time consuming due to the size of the isoform sets in human genome (give an example number (way larger than 10,000, e.g.)). 

We note that it is unnecessary to perform all-against-all alignment between complete sets of isoforms. In fact, we only need to compare the isoforms that are aligned to the same genomic region. We extract region windows from the genome that contain at least one isoform from any sample, then, we divide the set of isoforms into smaller subsets by their origin in the extracted genomic regions.

For each pair of samples that we are comparing, we perform intersection between two subsets of isoforms within each genomic window and identify isoforms that are shared by both samples and unique to each sample. 

For each unique isoform S from sample A, we further investigate the differences between S and other isoforms from sample B within the same genomic window. 

### Aligning isoforms to the reference genome

For every sample, we first prepend sample name to FASTA sequences in final corrected FASTA generated by SQANTI to make sure that all sequence names are unique, then align the renamed FASTA to the human Telomere-to-Telomere genome assembly of the CHM13 cell line (T2T-CHM13v2.0; RefSeq - GCF_009914755.1) using minimap2 (v2.24-r1122). The resulting SAM file is converted to BAM and sorted by samtools (v1.15.1; Danecek et al, 2021).
Dividing isoforms into subsets

We extract regions from the CHM13v2.0 genome that overlap with at least one isoform from any sample. We first obtain the average coverage of isoform per base by using samtools mpileup (citation, version). We next extract the 20,042 annotated protein coding gene regions from the reference genome and take the union of overlapping regions to create windows. Finally, windows were filtered to those which contained a per-base coverage greater than 0.05, which reduced our final set of windows to 11936.


In addition to the annotated gene regions, each sample contains more than 100,000 isoforms (Table 1) of isoforms that aligned to intron region. These isoforms are usually regarded as novel and may be important to the phenotypes of interest. Therefore, in addition to known gene regions, we divide the genome into 100 bp windows and retain the ones that has per-base coverage higher than 0.05. 

After that, we merge the gene windows and 100bp windows to obtain a complete set of windows that overlap with any isoform.

### Intersecting subsets of isoforms

For each isoform S in the subset of sample A, we perform exact string matching with all isoforms in the subset of sample B. If no isoform in sample B in the same genomic window matches S exactly, we say that S is unique.

### Comparing unique isoforms with other isoforms
For each unique isoform U, we perform Needleman-Wunch alignment between U and other isoforms within the same genomic window. We measure similarities between isoforms by the percentage of matched bases in U.
Annotating the differences between unique isoforms and the other sequences

We categorize differences between isoforms into [TODO] SNPs (<5bp), large-scale variants (>5bp), gene fusion, different exon usage and completely novel. Similar categories was used by SQANTI in annotating differences between sample isoforms and reference transcriptome. Note that we extend the categories by SQANTI by adding SNPs and large-scale variants.

### Iso-Seq analysis
 
Isoseq3 (v3.2.2) generated HQ (Full-length high quality) transcripts [Table 1] were mapped to GRCh38 (v33 p13) using Minimap2 long read alignment tools [1] (v2.24-r1122; commands: minimap2 -t 8 -ax splice:hq -uf --secondary=no -C5 -O6,24 -B4 GRCh38.v33p13.primary_assembly.fa sample.polished.hq.fastq.gz). The table 2 shows the basic statistics of the alignment of each sample [NA24385 /HG002, NA24143/HG004 and NA24631/HG005]. Next, we performed cDNA_cupcake [https://github.com/Magdoll/cDNA_Cupcake] workflow to collapse the redundant isoforms from bam, followed by filtering the low counts isoforms by 10 and filter away 5' degraded isoforms that might not be biologically significant. Next, sqanti3 [2] tool was used to generate final corrected fasta [Table 3a] transcripts and gtf [Table 3b] along with the isoform classification reports. The external databases including reference data set of transcription start sites (refTSS), list of polyA motif, tappAS-annotation and genecode hg38 annotation were utilized. Finally, IsoAnnotLite (v2.7.3) analysis was performed to annotate the gtf file from sqanti3.

We categorize differences between isoforms into [TODO] SNPs (<5bp), large-scale variants (>5bp), gene fusion, different exon usage and completely novel. Similar categories was used by SQANTI in annotating differences between sample isoforms and reference transcriptome. Note that we extend the categories by SQANTI by adding SNPs and large-scale variants.

## Description

## Flowchart
![](images/workflow.png)
### To extract sets of unique isoforms
![](images/workflow_part1.png)
### To annotate the unique isoforms
![](images/workflow_part2.png)

## Example Output

For each isoforom that is unique to at least one sample, we output the information of the read and the similarity between that isoform and the most similiar isoform in the same window.

The last column describes the normalized edit distance and the CIGAR string.

```
win_chr win_start       win_end total_isoform   isoform_name    sample_from     sample_compared_to      mapped_start    isoform_sequence        selected_alignments
NC_060925.1     255178  288416  4       PB.6.2  HG004   HG002   255173  GGATTATCCGGAGCCAAGGTCCGCTCGGGTGAGTGCCCTCCGCTTTTT      0.02_HG002_PB.6.2_3=6I1=3I1286=11I
NC_060925.1     255178  288416  4       PB.6.2  HG004   HG005   255173  GGATTATCCGGAGCCAAGGTCCGCTCGGGTGAGTGCCCTCCGCTTTTTG      0.02_HG002_PB.6.2_3=6I1=3I1286=11
```

### Deployment

Eventually, `pip install isocomp`.  But not yet.

## DEPENDENCIES

python >=3.8

If you're working on `ada`, you'll need to update the old, crusty version of 
python to something more modern and exciting. 

__The easy way__ (untested, but should work):

Install miniconda and create a conda env 
with python 3.9

__The manual method ([source](https://askubuntu.com/a/1424179))__ (tested, works):

```
ssh ... # your username login to ada

mkdir /home/${USER}/.local

# use your favorite text editor. no need to be vim
vim /home/${USER}/.bashrc

# add the following to the end (or where ever)
export PATH=/home/$USER/.local/bin:$PATH

# logout of the current session and log back in
exit
ssh ... (your username, etc)

# Download a more current version of python
wget https://www.python.org/ftp/python/3.9.15/Python-3.9.15.tgz

# unpack
tar xfp Python-3.9.15.tgz 
# remove the tarball
rm Python-3.9.15.tgz 

# cd into the Python package dir, configure and make
cd Python-3.9.15/

./configure --prefix=/home/${USER}/.local --exec_prefix=/home/${USER}/.local --enable-optimizations

make # this takes some time

make altinstall

# the following should point at a python in your /home/$USER/.local/bin dir
which python3.9

# optional, but convenient
ln -s /home/$USER/.local/bin/python3.9 /home/$USER/.local/bin/python

# Download the pip installer
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
# install pip
python3.9 get-pip.py

# confirm that pip is where you think it is
which pip # location should be in your .local

# at this point, you can do:
pip install poetry

# and continue with the development install below

```
### Development

Install [poetry](https://python-poetry.org/) and consider setting [the configuration 
such that virtual environments for a given projects are installed in that project 
directory](https://python-poetry.org/docs/configuration/#local-configuration).  

Next, I like working on a fork rather than the actual repository of record. I set my 
[git remotes](https://git-scm.com/book/en/v2/Git-Basics-Working-with-Remotes) so that `origin` points to 
my fork, and `upstream` points to the 'upstream' repository.

```bash
➜  isocomp git:(develop) ✗ git remote -v 
origin  https://github.com/cmatKhan/isocomp.git (fetch)
origin  https://github.com/cmatKhan/isocomp.git (push)
upstream        https://github.com/collaborativebioinformatics/isocomp.git (fetch)
upstream        https://github.com/collaborativebioinformatics/isocomp.git (push)
```

On your machine, `cd` into your local repository, `git checkout` the development 
branch, and make sure it is up-to-date with the upstream (ie the original) repository. 

__NOTE__: if you branch, in general make sure you branch off the `develop` repo, not `main`!  

Then (assuming poetry is installed already), do:

```bash
$ poetry install
```

This will install the virtual environment with the dependencies (and the dependencies' dependencies) 
listed in the [pyproject.toml](./pyproject.toml).
### <u>Adding dependencies</u>

To add a development dependency (eg, `mkdocs` is not something a user needs), 
use `poetry add -D <dependency>` this is equivalent to `pip install`ing into your 
virtual environment with the added benefit that the dependency is tracked in the 
[pyproject.toml](./pyproject.toml).  

To add a deployment dependency, just omit the `-D` flag.  

### <u>Writing code</u>

Do this first!

```bash
$ pip install -e .
```

[This is an 'editable install'](https://stackoverflow.com/questions/35064426/when-would-the-e-editable-option-be-useful-with-pip-install) 
and means that any change you make in your code is immediately available in your environment. 
__NOTE__: If you happen to see a [Logging Error](https://github.com/pypa/pip/issues/11309) when you run the `pip install -e .` 
command, you can ignore it.  

If you use vscode, [this is a useful 
plugin](https://marketplace.visualstudio.com/items?itemName=jshaptic.autodocs-vscode-support) 
which will automatically generate docstrings for you. [Default docstring format is google](https://sphinxcontrib-napoleon.readthedocs.io/en/latest/example_google.html), 
which is what the scripts we currently have use. This is an example of what a google formatted 
docstring looks like:

```python
def get_all_windows(gene_df:pd.DataFrame, bp_df:pd.DataFrame) -> pd.DataFrame:
    """From gene boundaries and 100 bp nonzero coverage windows, produce a merged window df

    Args:
        gene_df (pd.DataFrame): one window per gene, > 0.05 avg coverage
        bp_df (pd.DataFrame): one window per 100 bp, > 0.05 avg coverage

    Returns:
        pd.DataFrame: merged windows df
    """
    ...
```

In the function definition, the [type hints](https://docs.python.org/3/library/typing.html) of the arguments (eg `gene_df:pdDataFrame`)
are *not* required, but if you include them, autoDocs will automatically 
generate the data types in the docstring skeleton, also, which is nice. The `-> <datatype>` 
at the end of the function definition is the return data type.

### <u>Tests</u>

Unit tests can be written into the [src/tests](./src/tests) directory. There is 
an example in [src/tests/test_isocomp.py](./src/tests/test_isocomp.py). There are a couple other 
examples of tests -- ie for logging and error handling -- 
[here, too](https://github.com/cmatKhan/lmdemo/blob/main/tests/test_lmdemo.py).

### <u>Build</u>

It's good to intermittently build the package as you go. To do so, use `poetry build` 
which will create a `.whl` and `.tar.gz` (`dist` is already included 
in the [gitignore](./.gitignore)). You can 'distribute' these files to others -- they 
can be installed with `pip` or `conda` -- or use them to install the software outside of 
your current virtual environment.

### <u>Documentation</u>

If you would like to write documentation (ie not docstrings, but long form letters 
to your adoring users), then this can be done in markdown 
[__or jupyter notebooks__](https://pypi.org/project/mkdocs-jupyter/) (already added as a dev dependency) in the 
[docs](./docs) directory. Add the markdown/notebook document to the `nav` section in 
the [mkdocs.yml](./mkdocs.yml) and it will be added to the menu of the documentation 
site. Use `mkdocs serve` locally to see what the documentation looks like. 
`mkdocs build` will build the site in a directory called `site`, which is in the .gitignore already. 
Like `poetry build` it is a good diea to do `mkdocs build` intermittently as you write documentation. 
Eventually, we'll use `mkdocs gh-deploy` to deploy the site to github pages. Maybe if we get fancy, we'll 
set up the github actions to build the package on mac,windows and linux OSes on every push to develop, and rebuild 
the docs and push the package to pypi on every push to `main`.


## Computational Resources / Operation

## Citations
[1] https://www.pacb.com/products-and-services/applications/rna-sequencing/

