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
# METAGENÓMICA EN R
# 1. INSTALAR CRAN

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

# 2. INSTALAR BIOCONDUCTOR

if (!requireNamespace("BiocManager", quietly = TRUE)) {
  install.packages("BiocManager")
}

# 3. INSTALAR PAQUETES BIOC

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
Se extraeran columnas importantes, porque interesa: nombre taxonómico y abundancia estimada  
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
Guardar la matriz de abundacia, ya que reproducible para:
R,
Python,
Colab,
publicaciones.

# NORMALIZACIÓN
Busca convertir counts a abundancia relativa
Permite comparar: 
muestras con distinto número de reads,
composición microbiana real.
```
# NORMALIZACIÓN
# Crear matriz numérica

counts <- abundance_table[, -1]

# Asignar nombres de taxones como rownames

rownames(counts) <- abundance_table$Taxon

counts <- as.matrix(counts)

mode(counts) <- "numeric"

# Calcular abundancia relativa
############################################################

relative_abundance <- sweep(
  counts,
  2,
  colSums(counts),
  FUN = "/"
)

# Convertir a porcentaje
############################################################

relative_abundance <- relative_abundance * 100

############################################################
# Visualizar abundancias relativas
############################################################

head(relative_abundance)

#Guardar

write.csv(
  relative_abundance,
  "relative_abundance.csv"
)
```

# 4. RAREFACCIÓN
Para evaluar profundidad y cobertura de secuenciación
Determina si el esfuerzo de secuenciación fue suficiente.
```
rarecurve(
  t(counts),
  step = 100,
  cex = 0.8,
  col = c("steelblue", "darkred"),
  label = TRUE
)
```
* Si la curva se estabiliza la cobertura es suficiente

# 5. DIVERSIDAD ALFA
Medir diversidad interna.

| Métrica | Significado       |
| ------- | ----------------- |
| Shannon | riqueza + equidad |
| Simpson | dominancia        |
| Chao1   | riqueza estimada  |


```
# Transponer matriz
counts_t <- t(counts)

############################################################
# Shannon

shannon <- diversity(
  counts_t,
  index = "shannon"
)

############################################################
# Simpson

simpson <- diversity(
  counts_t,
  index = "simpson"
)

############################################################
# Chao1


chao1 <- estimateR(counts_t)[2, ]

############################################################
# Crear dataframe

alpha_div <- data.frame(
  Sample = rownames(counts_t),
  Shannon = shannon,
  Simpson = simpson,
  Chao1 = chao1
)

############################################################
# Visualizar diversidad alfa
alpha_div
# Guardar resultados
write.csv(
  alpha_div,
  "alpha_diversity.csv",
  row.names = FALSE
)
```


# 6. VISUALIZACIÓN DIVERSIDAD ALFA
# Boxplot Shannon

```
ggplot(
  alpha_div,
  aes(
    x = Sample,
    y = Shannon,
    fill = Sample
  )
) +
  geom_boxplot() +
  theme_minimal() +
  ggtitle("Diversidad Shannon")
```

### BARPLOT SHANNON

```
ggplot(
  alpha_div,
  aes(
    x = Sample,
    y = Shannon,
    fill = Sample
  )
) +
  geom_bar(
    stat = "identity"
  ) +
  theme_minimal() +
  ggtitle("Diversidad Shannon")
```


# 7. DIVERSIDAD BETA
Para comparar composición entre muestras
Detectar: 
diferencias ecológicas,
impacto del enriquecimiento nutricional.  
## Bray-Curtis
```
bray <- vegdist(
  counts_t,
  method = "bray"
)
```
## Jaccard
```
jaccard <- vegdist(
  counts_t,
  method = "jaccard"
)
```
# Visualizar matrices de distancia
```
bray
jaccard
```

# 8. HEATMAP
Visualizar: taxones dominantes,
diferencias entre muestras.

```
library(pheatmap)
```
```
# TOP TAXONES

top_taxa <- head(
  order(
    rowMeans(counts),
    decreasing = TRUE
  ),
  20
)

############################################################
# HEATMAP

pheatmap(
  counts[top_taxa, ],
  scale = "row",
  color = colorRampPalette(
    c("navy", "white", "firebrick3")
  )(50)
)
```

# TOP 10 TAXONES
```
top_taxa_df <- abundance_table %>%
  mutate(Total = JC1A + JP4D) %>%
  arrange(desc(Total)) %>%
  head(10)
```

### FORMATO LARGO
```
top_long <- top_taxa_df %>%
  pivot_longer(
    cols = c(JC1A, JP4D),
    names_to = "Sample",
    values_to = "Abundance"
  )
```
### GRAFICAR
```
ggplot(
  top_long,
  aes(
    x = Sample,
    y = Abundance,
    fill = Taxon
  )
) +
  geom_bar(
    stat = "identity"
  ) +
  theme_minimal() +
  ggtitle("Top 10 Taxones")
```
