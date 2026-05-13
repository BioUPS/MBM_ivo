# Guía práctica 
# Perfilado taxonómico metagenómico (parte 2)

## Plataforma: RStudio

Descargar desde Galaxy:  
✅ # Bracken on collection #: Report

:arrow_forward: Cargar los archivos en su proyeto de RStudio  :arrow_forward: desde Import Dataset
- JP4D.tabular 
- JC1A.tabular

# Instalar paquetes principales
```
############################################################
# METAGENÓMICA EN R
# Instalación y carga de paquetes
# Versión estable recomendada para análisis ecológico
############################################################

############################
# 1. INSTALAR CRAN
############################

install.packages(c(
  "tidyverse",
  "vegan",
  "ggplot2",
  "pheatmap",
  "reshape2",
  "RColorBrewer",
  "RcppZiggurat",
  "CVXR"
))

############################
# 2. INSTALAR BIOCONDUCTOR
############################

if (!requireNamespace("BiocManager", quietly = TRUE)) {
  install.packages("BiocManager")
}

############################
# 3. INSTALAR PAQUETES BIOC
############################

BiocManager::install(c(
  "phyloseq",
  "DESeq2",
  "ALDEx2",
  "ANCOMBC",
  "microbiome"
))
```

### Carga de librerias
```
library(tidyverse)
library(dplyr)
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
# MATRIZ DE ABUNDANCIA
# Crear copias de trabajo

jc1a <- JC1A
jp4d <- JP4D


# Visualizar estructura

head(jc1a)
head(jp4d)

str(jc1a)
str(jp4d)

# Seleccionar columnas importantes
# Ver nombres de columnas

colnames(jc1a)
colnames(jp4d)

# Normalmente Bracken contiene:
# name
# new_est_reads

jc1a_sub <- jc1a %>%
  select(name, new_est_reads)

jp4d_sub <- jp4d %>%
  select(name, new_est_reads)


# Renombrar columnas


colnames(jc1a_sub) <- c("Taxon", "JC1A")
colnames(jp4d_sub) <- c("Taxon", "JP4D")
colnames(jc1a_sub)
colnames(jp4d_sub)


############################################################
# Construir matriz de abundancia


abundance_table <- full_join(
  jc1a_sub,
  jp4d_sub,
  by = "Taxon"
)

# Reemplazar NA por 0

abundance_table[is.na(abundance_table)] <- 0

# Ver tabla final

head(abundance_table)

# Guardar matriz

write.csv(
  abundance_table,
  "abundance_matrix.csv",
  row.names = FALSE
)
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
