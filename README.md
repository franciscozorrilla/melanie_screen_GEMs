# ⚗️ Melanie's screen GEMs

## 📕 Description

This repo contains inputs, outputs, and step-by-step details regarding how models were reconstructed for strains used in Melanie's screen. For more details check out the [SymbNET metabolic modeling tutorial](https://github.com/franciscozorrilla/SymbNET).

## 🧱 Contents

* `genomes` DNA fasta files for 40 NCBI strains used in experimental screen
* `models` Genome scale metabolic models gapfilled on various media
* `ec_jaccard` EC-number-based Jaccard distance analysis files
* `media` Growth media formulations used for gapfilling and simulation
* `proteomes` Open reading frame (ORF)-annotated protein fasta files
* `ensembles` Ensemble models for network uncertainty quantification
* `scripts` Code used to run flux balance anlysis (FBA) using models
* `simulations` FBA simulation outputs
* `notebooks` Scripts used to visualize results
* `plots` Figures generated by notebooks  

## 🐪 Software

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

### 2. Create genome scale metabolic models using CarveMe 

Output models with fbc2 format gapfilling on [M3 media](https://www.nature.com/articles/s41564-018-0123-9/figures/1)

```
$ mkdir -p models/M3_gapfilled
$ while read model;do carve -v --mediadb media/media_db.tsv -g M3 --fbc2 -o models/M3_gapfilled/${model%.*}.xml proteomes/$model; done< <(ls proteomes)
```

Output models with fbc2 format without gapfilling

```
$ mkdir -p models/no_gapfill
$ while read model;do carve -v --fbc2 -o models/no_gapfill/${model%.*}.xml proteomes/$model; done< <(ls proteomes/)
```

### 3. Create ensemble models

Create ensembles with 100 versions of each strain

```
$ mkdir -p ensembles
$ while read model; do carve -v --fbc2 -n 100 -o ensembles/${model%.*}.xml proteomes/$model;  done< <(ls proteomes)
```

## 🏌️‍♂️ Results

### 1. Genes, reactions, and metabolites

![](https://github.com/franciscozorrilla/melanie_screen_GEMs/blob/main/plots/model_summary.png?raw=true)

### 2. Ensemble model jaccard distance

![](https://github.com/franciscozorrilla/melanie_screen_GEMs/blob/main/plots/ensemble_dist.png?raw=true)

### 3. Extract EC number information from model sets

The following loop was run on the command line, from within the folder containing M3-gapfilled models to generate a list of EC numbers across models:

```
while read model;do 
  paste $model|grep "EC Number"|sed 's/^.*: //g'|sed 's/<.*$//g'|sort|uniq|sed "s/^/${model%.*}\t/g";
done< <(ls|grep xml) >> M3_ec_models.tsv
```

Alternatively extract reaction IDs:

```
while read model;do    
  paste $model|grep "reaction metaid"|sed 's/^.*reaction metaid="//g'|sed 's/".*$//g'|sort|uniq|sed "s/^/${model%.*}\t/g"; 
done< <(ls|grep xml) >> model_rxns.tsv
```

In R:

```
library(tidyverse)
library(vegan)

# Load list of EC numbers extracted from models

ecnum=read.delim("melanie_screen_GEMs/M3_ec_models.tsv")

# Create presence/absence matrix

ecnum %>% mutate(presence=1) %>% 
          pivot_wider(names_from = ec_number,values_from = presence,values_fill = 0) %>% 
          column_to_rownames(.,var="model") -> ec_mat


vegdist(ec_mat, method="jaccard", binary=TRUE) -> D
as.matrix(D) -> D

as.data.frame(D) %>% rownames_to_column() %>% pivot_longer(cols=NT5001:YK0002,names_to = "model",values_to = "jaccard") -> jaccard_list

ggplot(jaccard_list) + geom_tile(aes(x=rowname,y=model,fill=jaccard)) + theme(axis.text.x = element_text(angle = 45, hjust = 1))

```

![](https://github.com/franciscozorrilla/melanie_screen_GEMs/blob/main/plots/jaccard_dist.png?raw=true)
