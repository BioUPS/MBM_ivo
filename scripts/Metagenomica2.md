# Guía práctica 
# Perfilado taxonómico metagenómico (parte 2)

## Plataforma: RStudio

Descargar desde Galaxy:  
✅ # Bracken on collection #: Report

:arrow_forward: Cargar los archivos en su proyeto de RStudio
- JP4D.tabular 
- JC1A.tabular

# Instalar paquetes principales
```
install.packages(c(
  "tidyverse",
  "vegan",
  "microbiome",
  "phyloseq",
  "ggplot2",
  "pheatmap",
  "reshape2",
  "RColorBrewer"
))

# Bioconductor
if (!requireNamespace("BiocManager", quietly = TRUE))
    install.packages("BiocManager")

BiocManager::install(c(
  "DESeq2",
  "ALDEx2",
  "ANCOMBC",
  "phyloseq"
))
```

###CArga de librerias
```
library(tidyverse)
library(vegan)
library(microbiome)
library(phyloseq)
library(ggplot2)
library(pheatmap)
library(reshape2)
library(RColorBrewer)

library(DESeq2)
library(ALDEx2)
library(ANCOMBC)
```

# MATRIZ DE ABUNDANCIA
Esta matriz será la base para: diversidad, estadística, ecología microbiana.


```
# Definir ruta
path <- "data/"

# Importar archivos
jc1a <- read.delim(
  paste0(path, "JC1A"),
  header = TRUE,
  sep = "\t"
)

jp4d <- read.delim(
  paste0(path, "JP4D"),
  header = TRUE,
  sep = "\t"
)

# Visualizar estructura
head(jc1a)
head(jp4d)
```
