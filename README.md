# ⚗️ Melanie screen GEMs

## 📕 Description

This repo contains inputs, outputs, and step-by-step details regarding how models were reconstructed for strains used in Melanie's screen.

## 🧱 Materials

* `genomes` DNA fasta files for 40 NCBI strains used in experimental screen
* `media` Growth media formulations used for gapfilling and simulation
* `proteomes` Open reading frame (ORF)-annotated protein fasta files
* `ensembles` Ensemble models for network uncertainty quantification
* `models` Genome scale metabolic models gapfilled on various media

## 🐪 Software used

* git
* Prodigal
* CarveMe
   * diamond
   * CPLEX

## 🩺 Methods

### 0. Clone repo

```
$ git clone https://github.com/franciscozorrilla/melanie_screen_GEMs.git
```

### 1. Translate genomes to ORF-annotated protein fasta files using prodigal

Move into cloned repository directory and create `proteomes` folder

```
$ cd melanie_screen_GEMs
$ mkdir -p proteomes
```

Run prodigal on each input genome file found in the `genomes/` folder

```
$ while read file; do prodigal -i genomes/$file -a proteomes/${file%.*}.faa;done< <(ls genomes/)
```

### 2 Create genome scale metabolic models using CarveMe 

```

```
