# Dead Man’s Teeth. Introduction to metagenomics analysis
All data from the original research are available  in the NCBI Short Read Archive (SRA) under number SRP029257 (BioProject PRJNA216965). In addition, all the data was uploaded to the server MG-RAST, an open source web application server that provides automatic phylogenetic and functional analysis of metagenomes. We are interested in data stored there in Project 365 - “Ancient Oral Metagenome”.
## Part 1. Amplicon sequencing
### 1.1. QIIME2 installation
We will use the package QIIME2 (“Quantitative Insights Into Microbial Ecology”), an open-source bioinformatics pipeline for performing microbiome analysis from raw DNA sequencing data, developed in the laboratory of Dr. Rob Knight at UC San Diego.<br>
Installing QIIME2:<br>
```bash
wget https://data.qiime2.org/distro/core/qiime2-2022.2-py38-linux-conda.yml
conda env create -n qiime2-2022.2 --file qiime2-2022.2-py38-linux-conda.yml
# OPTIONAL CLEANUP
rm qiime2-2022.2-py38-linux-conda.yml
```
Activating the conda environment:
```bash
conda activate qiime2-2022.2
```
Testing the installation:
```bash
qiime --help
```
### 1.2. Importing data
```bash
qiime tools import   --type 'SampleData[SequencesWithQuality]'   --input-path ./data/manifest.tsv   --output-path ./data/sequences.qza   --input-format SingleEndFastqManifestPhred33V2
```
```bash
Imported ./data/manifest.tsv as SingleEndFastqManifestPhred33V2 to ./data/sequences.qza
```
Checking correctness of QIIME artifact with qiime validate:
```bash
qiime tools validate ./data/sequences.qza
```
```bash
Result ./data/sequences.qza appears to be valid at level=max.
```
### 1.3. Demultiplexing and QC
Our reads are already demultiplexed (1 sample per file). Now it is useful to explore how many sequences were obtained per sample, and to get a summary of the distribution of sequence qualities.
```bash
qiime demux summarize --i-data ./data/sequences.qza --o-visualization ./data/sequences.qzv
```
Visualization of .qzv file at https://view.qiime2.org/ <br>
![v1](/images/v1.jpg "v1")<br>
![v2](/images/v2.jpg "v2")<br>
![v3](/images/v3.jpg "v3")<br>
![v4](/images/v4.jpg "v4")<br>
![v5](/images/v5.jpg "v5")<br>


