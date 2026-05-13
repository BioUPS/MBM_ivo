# Guía práctica 
# Perfilado taxonómico metagenómico (parte 1)
---
## Plataforma: Galaxy

---

# Objetivo 

Realizar perfilado taxonómico de muestras metagenómicas shotgun utilizando herramientas bioinformáticas en Galaxy.

El flujo de trabajo incluye:

1. Importación de datos
2. Creación de colecciones paired-end
3. Clasificación taxonómica con Kraken2
4. Estimación de abundancia con Bracken
5. Visualización con Krona y Pavian
6. Perfilado basado en genes marcadores con MetaPhlAn

---

# Conceptos importantes

## Kraken2
Herramienta de clasificación taxonómica basada en k-mers.

### ¿Qué hace?
Compara fragmentos cortos de ADN (k-mers) contra una base de datos taxonómica para identificar microorganismos presentes en la muestra.

### Ventajas
- Muy rápida
- Bajo costo computacional
- Adecuada para datasets grandes

---

## Bracken
Herramienta complementaria de Kraken2.

### ¿Qué hace?
Reestima la abundancia de especies utilizando un modelo bayesiano.

### Importante
Bracken NO utiliza el archivo de clasificación estándar de Kraken2.

Debe usarse:

✅ Kraken REPORT file

NO:

❌ Kraken classification output

---

## MetaPhlAn
Herramienta basada en genes marcadores.

### ¿Qué hace?
Detecta microorganismos usando genes específicos altamente conservados.

### Ventajas
- Menos falsos positivos
- Mayor especificidad taxonómica
- Menor uso de memoria


---

:bookmark: 1 
# Preparación de los datos

---
# Paso 1 — Crear una nueva historia

## Acción

Nombre sugerido:

```text
Metagenomics
```

---

# Paso 2 — Importar datos

## Archivos del tutorial


[JC1A_R1.fastqsanger.gz](https://zenodo.org/record/7871630/files/JC1A_R1.fastqsanger.gz)  
[JC1A_R2.fastqsanger.gz](https://zenodo.org/record/7871630/files/JC1A_R2.fastqsanger.gz)  
[JP4D_R1.fastqsanger.gz](https://zenodo.org/record/7871630/files/JP4D_R1.fastqsanger.gz)  
[JP4D_R2.fastqsanger.gz](https://zenodo.org/record/7871630/files/JP4D_R2.fastqsanger.gz)  

Pegar los links o cargar los archivos.

---

# Paso 3 — Crear colección paired-end

## ¿Por qué? 
Kraken2 y MetaPhlAn trabajan mejor con lecturas paired-end organizadas como colección.

### Procedimiento

1. Seleccionar los 4 datasets
2. Click en:

```text
For all selected → Build List
```

3. Elegir:

```text
List of Paired Datasets
```

4. Verificar emparejamiento automático:

```text
_R1 y _R2
```

5. Asignar nombre a la colección

Ejemplo:

```text
Metagenomic_Pairs
```

---

:bookmark: 2 
# Kraken2

---

# Paso 4 — Ejecutar Kraken2

## Herramienta

```text
Kraken2
```

---

# Configuración recomendada

## Single or paired reads
Los datos corresponden a lecturas paired-end.

```text
Paired
```

## Collection of paired reads
Seleccionar:

```text
Metagenomic_Pairs
```

---

## Confidence
Define el porcentaje mínimo de k-mers que deben coincidir con la base de datos.
```text
0.1
```
---

## Print a report with aggregate counts/clade to file 
:star: Este parámetro genera el archivo REPORT necesario para: Bracken y Krona
```text
YES
```
---

## Select a Kraken2 database

```text
Prebuilt Refseq indexes: PlusPF 
```
---

# Salidas de Kraken2  
## 1. Classification output: Archivo con clasificación por lectura individual.  
NO usar en Bracken.  
## 2. Report output: Archivo agregado taxonómico.  
Debe usarse para: Brackeny Krona
---

:bookmark: 3
# Bracken

---

# Paso 5 — Ejecutar Bracken

## Herramienta

```text
Bracken
```

---

# Configuración

## Kraken report file

Seleccionar:

```text
Report output of Kraken
```
---

## Select a kmer distribution

```text
Prebuilt Refseq indexes: PlusPF 
```
Debe ser EXACTAMENTE la misma base de datos usada en Kraken2.

---

## Level 
Define el nivel taxonómico de reestimación.

```text
Species
```
---

## Produce Kraken-Style Bracken report  
Genera un archivo compatible con herramientas de visualización.
```text
YES
```
---
:bookmark: 4
# Conversión para Krona

---

# Paso 6 — Krakentools

## Herramienta

```text
Krakentools: Convert kraken report file
```

---

# Parámetro

## Kraken report file

Seleccionar:

```text
Report collection of Kraken
```

---

## ¿Qué hace?

Convierte el reporte de Kraken2 al formato requerido por Krona.

---
:bookmark: 5
# Krona

---

# Paso 7 — Generar visualización Krona

## Herramienta

```text
Krona pie chart
```

---

# Configuración

## Type of input data

```text
Tabular
```

---

## Input file

Seleccionar:

```text
Output of Krakentools
```

---

## Resultado

Archivo HTML interactivo, descargue el archivo HTML para visualizarlo mejor. 

---

:bookmark: 6
# MetaPhlAn

---

# Paso 9 — Separar colección paired-end

## Herramienta
MetaPhlAn requiere: colección forward y colección reverse por separado.
```text
Unzip collection
```

---

# Paso 10 — Ejecutar MetaPhlAn

## Herramienta

```text
MetaPhlAn
```

---

# Configuración

## Input(s)

```text
Fasta/FastQ file(s) with microbiota reads
```

---

## Tipo de datos

```text
Paired-end files
```

---

## Forward paired-end files

```text
Forward collection
```

---

## Reverse paired-end files

```text
Reverse collection
```

---

## Output for Krona (final antes de opciones adicionales)  
Genera salida compatible con Krona.

```text
YES
```

---

# Salidas de MetaPhlAn

## Predicted taxon relative abundances
Tabla con: taxones, abundancia relativa, niveles taxonómicos

---

## Krona output
Archivo compatible con Krona.

---
## BIOM file
Formato estándar usado en microbioma.
Compatible con: Qiime, mothur y otros pipelines microbiológicos

---

# Paso 11 — Krona con MetaPhlAn

## Herramienta

```text
Krona pie chart
```

---

# Configuración

## Type of input data

```text
tabular
```

---

## Input file

Seleccionar:

```text
Krona output of MetaPhlAn
```
---

# Herramientas utilizadas

| Herramienta | Función |
|---|---|
| Kraken2 | Clasificación taxonómica k-mer |
| Bracken | Reestimación de abundancia |
| Krakentools | Conversión de reportes |
| Krona | Visualización interactiva |
| MetaPhlAn | Perfilado basado en genes marcadores |

---

# Resultado esperado del pipeline
- Se generó el perfil de la comunidad a partir de datos del microbioma.
- Kraken2 o MetaPhlAn asignan etiquetas taxonómicas a dos conjuntos de datos de muestras del microbioma
- Krona permite visualizar los resultados
- La herramienta de clasificación taxonómica a utilizar depende de los datos


Al finalizar se podrá:

- ejecutar clasificación taxonómica
- interpretar outputs de Kraken2
- usar Bracken correctamente
- visualizar comunidades microbianas
- comparar muestras microbiológicas
- diferenciar enfoques k-mer y genes marcadores

### History compartido en Galaxy [Link](https://usegalaxy.org/u/ivovaca/h/metagenomica-p1)

## Bibliografía
* Sophia Hampe, Bérénice Batut, Paul Zierep, Taxonomic Profiling and Visualization of Metagenomic Data (Galaxy Training Materials). https://training.galaxyproject.org/training-material/topics/microbiome/tutorials/taxonomic-profiling/tutorial.html Online; accessed Wed May 13 2026
* Hiltemann, Saskia, Rasche, Helena et al., 2023 Galaxy Training: A Powerful Framework for Teaching! PLOS Computational Biology 10.1371/journal.pcbi.1010752
* Batut et al., 2018 Community-Driven Data Analysis Training for Biology Cell Systems 10.1016/j.cels.2018.05.012
