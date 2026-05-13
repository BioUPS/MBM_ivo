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
```
if (!requireNamespace("BiocManager", quietly = TRUE))
    install.packages("BiocManager")

BiocManager::install("microbiome")
```

### Carga de librerias
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

## construir_matriz.R
```
name
new_est_reads
```
```
# Seleccionar columnas relevantes

jc1a_sub <- jc1a %>%
  select(name, new_est_reads)

jp4d_sub <- jp4d %>%
  select(name, new_est_reads)

# Renombrar columnas

colnames(jc1a_sub) <- c("Taxon", "JC1A")
colnames(jp4d_sub) <- c("Taxon", "JP4D")

# Unir tablas

abundance_table <- full_join(
  jc1a_sub,
  jp4d_sub,
  by = "Taxon"
)

# Reemplazar NA por 0

abundance_table[is.na(abundance_table)] <- 0

# Ver tabla

head(abundance_table)

# Guardar

write.csv(
  abundance_table,
  "results/abundance_matrix.csv",
  row.names = FALSE
)
```
