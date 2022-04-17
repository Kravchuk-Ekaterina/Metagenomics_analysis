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

### 1.4. Feature table construction (and more QC)
For the subsequent analysis we will need the metadata table. There is a barcode - a unique sequence in the beginning of each sample, and primer sequence, that was used to amplify the V5 region of the rRNA. We need to strip it out, and filter chimeric sequences. I chose m=21 and n=175. Here we will use the DADA2 pipeline, as follows:<br>
```bash
qiime dada2 denoise-single   --i-demultiplexed-seqs ./data/sequences.qza   --p-trim-left 21 --p-trunc-len 175 --o-representative-sequences ./data/rep-seqs.qza --o-table ./data/table.qza --o-denoising-stats ./data/stats.qza
```
Checking how many reads are passed the filter and were clustered:<br>
```bash
qiime metadata tabulate   --m-input-file ./data/stats.qza   --o-visualization ./data/stats.qzv
```
The visualization at https://view.qiime2.org/ : <br>
![v6](/images/v6.jpg "v6")<br>
### 1.5. FeatureTable and FeatureData summaries
One of the main results of the DADA2 step is a clustering into an amplicon sequence variant (ASV) - a higher-resolution analogue of the traditional OTUs.  Thus, the Feature Table we just obtained is an equivalent of the OTU tables in other metagenomic pipelines.<br>
Let’s create visual summaries of the data - how many sequences are associated with each sample and with each feature, etc.<br>
```bash
qiime feature-table summarize   --i-table ./data/table.qza   --o-visualization ./data/table.qzv   --m-sample-metadata-file ./data/sample-metadata.tsv
```
The vizualization:<br>
![v7](/images/v7.jpg "v7")<br>
![v8](/images/v8.jpg "v8")<br>
![v9](/images/v9.jpg "v9")<br>
![v10](/images/v10.jpg "v10")<br>
![v11](/images/v11.jpg "v11")<br>
![v12](/images/v12.jpg "v12")<br>
Mapping feature IDs to sequences, to use these representative sequences in other applications, e.g. BLAST each sequence against the NCBI nt database.
```bash
qiime feature-table tabulate-seqs  --i-data ./data/rep-seqs.qza   --o-visualization ./data/rep-seqs.qzv
```
The vizualization:<br>
### 1.6. Taxonomic analysis
Database itself is just a fasta file of the 16S representatives, and QIIME2 uses Naive Bayes classifiers trained on this data <br>
https://data.qiime2.org/2022.2/common/silva-138-99-nb-classifier.qza<br>
```bash
qiime feature-classifier classify-sklearn   --i-classifier ./data/silva-138-99-nb-classifier.qza   --i-reads ./data/rep-seqs.qza   --o-classification ./data/taxonomy.qza
```
And visualization:<br>
```bash
qiime metadata tabulate --m-input-file ./data/taxonomy.qza --o-visualization ./data/taxonomy.qzv
qiime taxa barplot \
  --i-table ./data/table.qza \
  --i-taxonomy ./data/taxonomy.qza \
  --m-metadata-file ./data/sample-metadata.tsv \
  --o-visualization ./data/taxa-bar-plots.qzv
```
![v1](/images/v1.png "v1")<br>
![v2](/images/v2.png "v2")<br>
![v3](/images/v3.png "v3")<br>

