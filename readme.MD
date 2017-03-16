# Koolmogorov

Koolmogorov is a clustering and visualization library in Python based on CompLearn. Koolmogorov uses techniques and data structures from:

- Kolmogorov Complexity (https://www.jstor.org/stable/25049284)
- Normalized Information Distance (https://arxiv.org/abs/0809.2553)
- Normalized Compression Distance (https://arxiv.org/abs/cs/0312044)
- Normalized Web Distance (https://arxiv.org/abs/0905.4039)
- Normalized Count-Min Sketch Distance (Unpublished research)
- Normalized Compression Neighbors (Unpublished research)
- Topological Data Analysis ()
- Count-Min Sketch (http://dl.acm.org/citation.cfm?id=1073718)
- A New Quartet Tree Heuristic for Hierarchical Clustering (https://arxiv.org/abs/cs/0606048)
- Force-Directed Graphs (http://www.cs.cmu.edu/afs/cs/academic/class/15462-s13/www/lec_slides/Jakobsen.pdf)
- K-means Clustering based on Voronoi iteration (http://www.kanungo.com/pubs/tpami02-kmeans.pdf)

Koolmogorov is licensed under MIT.

## Installation

```
pip install koolmogorov
```

### Dependancies on external libraries

*Required*
- matplotlib
- networkx
- numpy

*Optional*
- pylzma
- snappy
- scikit-learn

*Externally loaded*
- D3.js
- Open Sans web font

## Creating distance matrices

The basis for Koolmogorov is a distance matrix. You can create your own distance matrices or use the MakeDistanceMatrix class.

```
from koolmogorov import MakeDistanceMatrix

mdm = MakeDistanceMatrix(verbose=1)
dm = mdm.ncd("data/*.*", compressor="snappy")
```

This makes a distance matrix `dm` with the normalized compression distances (using Snappy compressor) for all files in the `data/` directory.

See MakeDistanceMatrix documentation for more information.

## Creating trees

The distance matrix is used to create a tree (also called: unrooted dendrogram or philogenetic tree).

```
from koolmogorov import MakeTree

mt = MakeTree(verbose=1, n_jobs=8)
T, score = mt.fit_transform(dm)
```

This makes a tree `T` (networkX object) and gives a fitness score `score`, using multi-threading with a Pool-size of 8.

See MakeTree documentation for more information.

## Creating visualizations

Finally we visualize the tree.

```
from koolmogorov import MakeViz

mv = MakeViz(verbose=1, method="d3")
mv.visualize(T, file_name="my_visualization")
```

This creates an HTML5/CSS3 file `my_visualization.html`, using d3.js to model the tree `T` as a force-directed graph.

See MakeViz documentation for more information.

## Examples

### Chain Letters

We download a small collection of 11 chain letters, which are all variations of a common form, called the "Good Luck" or "Saint Jude" chain letter.

Sample:
```
With love all things are possible.

This paper was sent to you for good luck.  The original is in New England.
It has been around the world nine times.  The luck has been sent to you.
You will receive good luck in the mail.  Send no money as faith as no 
price.  Do not keep this letter.  It must leave your hands within 96 hours.
```

We use Koolmogorov to create an evolutionary tree:

```
from koolmogorov import MakeDistanceMatrix, MakeTree, MakeViz

mdm = MakeDistanceMatrix()
dm = mdm.ncd("chain/*.*", compressor="snappy")

mt = MakeTree(n_jobs=80)
G, score = mt.fit_transform(dm)

mv = MakeViz()
mt.visualize(G, method="matplotlib")
```

Result:

visualization.png

### Malware families

We download 2 malware families as used in the Kaggle Microsoft Malware Competition.

```
from koolmogorov import MakeDistanceMatrix, MakeTree, MakeViz

mdm = MakeDistanceMatrix()
dm = mdm.ncd("malware/*.*", compressor="pylzma")

mt = MakeTree(n_jobs=80)
G, score = mt.fit_transform(dm)

mv = MakeViz()
mt.visualize(G, method="d3js")
```

Result:

visualization.png

### Lord of the Rings Character Network

We look at co-occurence of character names inside sentences of the Lord of the Rings trilogy.

```
from koolmogorov import MakeIndex, MakeDistanceMatrix, MakeTree, MakeViz

mi = MakeIndex()
in = mi.index("Lordoftherings.txt", mini_documents=True)

mdm = MakeDistanceMatrix()
dm = mdm.ncmsd(["frodo", "bilbo", "pendergast"], index=in)

mt = MakeTree(n_jobs=80)
G, score = mt.fit_transform(dm)

mv = MakeViz()
mt.visualize(G, method="d3js")
```

Result:

### Word Semantics

We use Bing search engine as a compressor,

```
from koolmogorov import MakeDistanceMatrix, MakeTree, MakeViz

mdm = MakeDistanceMatrix()
dm = mdm.nwd(["red", "white", "blue", "one", "two", "three"], compressor="bing")

mt = MakeTree(n_jobs=80)
G, score = mt.fit_transform(dm)

mv = MakeViz()
mt.visualize(G, method="d3js")
```