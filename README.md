# Marvel-Network

## Overview

This repository provides replication code for the network visualization of Marvel Cinematic Universe (MCU) films reported in **Figure 1** of the accompanying manuscript. The paper examines whether films’ commercial performance within the MCU is associated with **narrative interconnectedness** in a shared cinematic universe.

The repository includes input data in Excel format:
- *mcu_global.xlsx*: Dataset containing information on the Marvel Cinematic Universe (MCU).


<p align="center">
  <img src="README_image/network_image.png" width="100%"><br>
  <em>Figure 1. Network map of MCU films.</em>
</p>

**Figure 1** presents a film-level network of the MCU. Nodes represent individual MCU films. Edges encode pairwise narrative linkages based on character overlap, operationalized using the **Jaccard similarity** between the sets of heroes appearing in two films:
```math
\mathrm{Similarity}(A,B)=\frac{n(A\cap B)}{n(A\cup B)},
```
where $A$ and $B$ denote the sets of heroes appearing in each films.

The visualization provides an interpretable representation of the franchise’s relational structure and serves as descriptive evidence for our empirical analysis of interconnectedness and films’ commercial success.

The code in this repository reproduces Figure 1 (subject to platform-specific rendering differences). Running the provided scripts will (i) construct the MCU film network from the underlying inputs and (ii) generate the corresponding node–edge visualization.


```python
import pandas as pd
import networkx as nx
import matplotlib.pyplot as plt

df = pd.read_excel('mcu_global.xlsx')
df1 = df.iloc[:,[1,5,6,7,8,27,-2,-1]]
df1.columns = ['tt','dir','sw','pd','cst','adjusted_roi','dom_roi','inter_roi']

df1['dir'] = df1['dir'].apply(lambda x: [i.strip() for i in x.split(', ')])
df1['sw'] = df1['sw'].apply(lambda x: [i.strip() for i in x.split(', ')])
df1['pd'] = df1['pd'].apply(lambda x: [i.strip() for i in x.split(', ')])
df1['cst'] = df1['cst'].apply(lambda x: [i.strip() for i in x.split(', ')])

def weight_counter(a,b):
    set1 = set(a)
    set2 = set(b)
    intersection = set1.intersection(set2)
    union = set1.union(set2)
    return len(intersection) / len(union) if union else 0

n = len(df1)
weight_matrix = pd.DataFrame(index = df1['tt'] , columns = df1['tt'], dtype = 'float')

for i in range(n):
    for j in range(n):
        weight_matrix.iloc[i, j] = weight_counter(df1.iloc[i]['cst'], df1.iloc[j]['cst'])

G = nx.Graph()

# Add a node including the inter_roi attribute
for movie in weight_matrix.index:
    roi = df1.loc[df1['tt'] == movie, 'adjusted_roi'].values
    if len(roi) > 0:
        G.add_node(movie, adjusted_roi=roi[0])
    else:
        G.add_node(movie)


# Add an edge if similarity > 0
for i in range(n):
    for j in range(i+1, n):  # 대각선 아래는 생략
        weight = weight_matrix.iloc[i, j]
        
        if weight > 0:
            movie1 = weight_matrix.index[i]
            movie2 = weight_matrix.columns[j]
            G.add_edge(movie1, movie2, weight=weight)

labels = {
    'Iron Man': 'Iron Man',
    'Iron Man 2': 'Iron Man',
    'Iron Man 3': 'Iron Man',

    'The Incredible Hulk': 'The Incredible Hulk',

    'Thor': 'Thor',
    'Thor: The Dark World': 'Thor',
    'Thor: Ragnarok': 'Thor',
    'Thor: Love and Thunder': 'Thor',

    'Captain America: The First Avenger': 'Captain America',
    'Captain America: The Winter Soldier': 'Captain America',
    'Captain America: Civil War': 'Captain America',
    'Captain America: Brave New World': 'Captain America',

    'The Avengers': 'Avengers',
    'Avengers: Age of Ultron': 'Avengers',
    'Avengers: Infinity War': 'Avengers',
    'Avengers: Endgame': 'Avengers',

    'Guardians of the Galaxy': 'Guardians of the Galaxy',
    'Guardians of the Galaxy Vol. 2': 'Guardians of the Galaxy',
    'Guardians of the Galaxy Vol. 3': 'Guardians of the Galaxy',

    'Ant-Man': 'Ant-Man',
    'Ant-Man and the Wasp': 'Ant-Man',
    'Ant-Man and the Wasp: Quantumania': 'Ant-Man',

    'Spider-Man: Homecoming': 'Spider-Man',
    'Spider-Man: Far from Home': 'Spider-Man',
    'Spider-Man: No Way Home': 'Spider-Man',

    'Doctor Strange': 'Doctor Strange',
    'Doctor Strange in the Multiverse of Madness': 'Doctor Strange',

    'Black Panther': 'Black Panther',
    'Black Panther II': 'Black Panther',

    'Captain Marvel': 'Captain Marvel',
    'The Marvels': 'Captain Marvel',

    'Black Widow': 'Black Widow',

    'Shang-Chi and the Legend of the Ten Rings': 'Shang-Chi',
    'Eternals': 'Eternals',

    'Deadpool & Wolverine': 'Deadpool',
    'Thunderbolts*': 'Thunderbolts'
}

# Assign grouped labels to nodes
for node in G.nodes():
    G.nodes[node]['group'] = labels.get(node, 'Other')

nx.write_graphml(G, 'marvle_nw.graphml')

```
