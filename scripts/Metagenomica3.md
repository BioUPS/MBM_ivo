# Guía práctica 
# Perfilado taxonómico metagenómico (parte 3)
## ECOLOGÍA FUNCIONAL - METAGENÓMICA AVANZADA

## Plataforma: Python (Colab - JupyterLab)

### Archivos necesarios 
| Archivo                | Uso                   |
| ---------------------- | --------------------- |
| abundance_matrix.csv   | abundancias absolutas |
| relative_abundance.csv | abundancias relativas |
| alpha_diversity.csv    | diversidad alfa       |


### Crear manualmente metadata, llamada "metadata.csv" con la siguiente información:
```
Sample,Environment,Treatment
JC1A,Sediment,Control
JP4D,Water,Fertilized
```


# INSTALAR LIBRERÍAS
```
!pip install squarify
!pip install plotly
!pip install scipy
!pip install scikit-bio
```

# IMPORTAR LIBRERÍAS
```
import pandas as pd
import numpy as np

import matplotlib.pyplot as plt
import seaborn as sns

import plotly.express as px
import plotly.graph_objects as go

import squarify

from scipy.cluster.hierarchy import linkage
from scipy.spatial.distance import pdist

from sklearn.preprocessing import StandardScaler

from skbio.diversity import beta_diversity
```

# CONFIGURACIÓN

```
plt.rcParams["figure.figsize"] = (12,8)

sns.set_style("whitegrid")
```

# SUBIR ARCHIVOS:

✅ abundance_matrix.csv      
✅ relative_abundance.csv    
✅ alpha_diversity.csv   
✅ metadata.csv (*) 

* Data en este repositorio (* = excepto)

# LEER ARCHIVOS

```
abundance = pd.read_csv(
    "abundance_matrix.csv"
)

relative = pd.read_csv(
    "relative_abundance.csv"
)

alpha = pd.read_csv(
    "alpha_diversity.csv"
)

metadata = pd.read_csv(
    "metadata.csv"
)
```

# VISUALIZAR
VerificaR que existan columnas: Taxon, JC1A, JP4D
```
abundance.head()
```
# NIVEL 1 — VISUALIZACIÓN 🟢

## TOP 15 TAXONES
############################################################
```
abundance["Total"] = (
    abundance["JC1A"] +
    abundance["JP4D"]
)

top15 = abundance.sort_values(
    by="Total",
    ascending=False
).head(15)


# FORMATO LARGO

top15_long = pd.melt(
    top15,
    id_vars=["Taxon"],
    value_vars=["JC1A", "JP4D"],
    var_name="Sample",
    value_name="Abundance"
)


# ABUNDANCIA RELATIVA


top15_long["Relative"] = (
    top15_long.groupby("Sample")
    ["Abundance"]
    .transform(lambda x: x/x.sum()*100)
)

# GRAFICAR
fig = px.bar(
    top15_long,
    
    x="Sample",
    y="Relative",
    
    color="Taxon",
    
    title="Composición Taxonómica",
    
    labels={
        "Relative":"Abundancia Relativa (%)"
    }
)

fig.show()
```

# HEATMAP AVANZADO
Para visualizar patrones taxonómicos.

### PREPARAR MATRIZ
```
heatmap_data = top15.set_index(
    "Taxon"
)[["JC1A", "JP4D"]]
```

# LOG TRANSFORMACIÓN
```
heatmap_data = np.log10(
    heatmap_data + 1
)
```

# HEATMAP
```
plt.figure(figsize=(10,12))

sns.heatmap(
    heatmap_data,
    
    cmap="viridis",
    
    linewidths=0.5
)

plt.title(
    "Heatmap Taxonómico"
)

plt.show()
```

# BUBBLE PLOT
Para comparar abundancias.

```
fig = px.scatter(
    top15_long,
    
    x="Sample",
    y="Taxon",
    
    size="Relative",
    color="Relative",
    
    title="Distribución Taxonómica"
)

fig.show()
```
# TREEMAP
Para mostrar dominancia taxonómica.
```
plt.figure(figsize=(14,10))

squarify.plot(
    
    sizes=top15["Total"],
    
    label=top15["Taxon"],
    
    alpha=0.8
)

plt.axis("off")

plt.title(
    "Treemap Taxonómico"
)

plt.show()
```

# NIVEL 2 — INTERACTIVO 🔵

## SANKEY DIAGRAM
Para Relacionar: 
muestra,
taxón,
abundancia.

```
# NODOS

labels = (
    list(top15_long["Sample"].unique()) +
    list(top15_long["Taxon"].unique())
)

############################################################
# MAPEO


label_to_index = {
    label:i for i,label in enumerate(labels)
}

############################################################
# LINKS
source = [
    label_to_index[s]
    for s in top15_long["Sample"]
]

target = [
    label_to_index[t]
    for t in top15_long["Taxon"]
]

value = list(
    top15_long["Relative"]
)

############################################################
# SANKEY

fig = go.Figure(
    data=[
        go.Sankey(
            
            node=dict(
                label=labels
            ),
            
            link=dict(
                source=source,
                target=target,
                value=value
            )
        )
    ]
)

fig.update_layout(
    title_text="Sankey Taxonómico",
    font_size=12
)

fig.show()
```

## SUNBURST PLOT
Para visualizar jerarquía ecológica.
```
fig = px.sunburst(
    
    top15_long,
    
    path=["Sample", "Taxon"],
    
    values="Relative",
    
    title="Sunburst Taxonómico"
)

fig.show()
```

# NIVEL 3 — ECOLÓGICO 🟣
## CLUSTERING
Para agrupar taxones similares.
```
############################################################
# MATRIZ

cluster_data = heatmap_data

############################################################
# CLUSTERMAP

sns.clustermap(
    
    cluster_data,
    
    cmap="viridis",
    
    standard_scale=1
)

plt.show()
```
## COMPOSITIONAL ANALYSIS
Para comparar proporciones ecológicas.
```
############################################################
# ABUNDANCIA RELATIVA

relative_data = (
    counts.div(
        counts.sum(axis=1),
        axis=0
    ) * 100
)

############################################################
# STACKED COMPOSITIONAL
relative_data.T.plot(
    
    kind="bar",
    
    stacked=True,
    
    figsize=(14,8),
    
    colormap="Spectral"
)

plt.ylabel(
    "Abundancia Relativa (%)"
)

plt.title(
    "Análisis Composicional"
)

plt.show()
```

