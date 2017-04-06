# Koolmogorov

Koolmogorov is a hierarchical clustering and visualization library in Python based on [CompLearn](http://complearn.org/). Koolmogorov uses approaches, techniques, and data structures from:

- Kolmogorov Complexity (https://www.jstor.org/stable/25049284)
- Normalized Information Distance (https://arxiv.org/abs/0809.2553)
- Normalized Compression Distance (https://arxiv.org/abs/cs/0312044)
- Normalized Web Distance (https://arxiv.org/abs/0905.4039)
- Normalized Count-Min Sketch Distance (Unpublished research)
- Normalized Compression Neighbors (Unpublished research)
- Topological Data Analysis (http://brown.edu/Research/kalisnik/mapperPBG.pdf)
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

The distance matrix is used to create a tree (also called: unrooted dendrogram or phylogenetic tree).

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

![Chain Letter Dendrogram](http://i.imgur.com/DobvcIY.png)

### Malware families

We download a few malware families from the theZoo malware database:

- Zeus
- EquationGroup
- Cryptolocker
- Stuxnet

```
from koolmogorov import MakeDistanceMatrix, MakeTree, MakeViz

mdm = MakeDistanceMatrix()
dm = mdm.ncd("malware/*.*", compressor="bz2")

mt = MakeTree(n_iters=200, n_jobs=80)
G, score = mt.fit_transform(dm)

mv = MakeViz()
mv.visualize(G)
```

Result:

![Malware Dendrogram](http://i.imgur.com/vjuFHTB.png)

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

## Documentation

### MakeDistanceMatrix(verbose=0)

| Attribute | Value |
| --- | --- |
| verbose | a non-negative integer denoting level of verbosity. Default `0` |

| Function | Description |
| --- | --- |
| ncd(loc_files, compressor="bz2", compression_level=9) | Computes Normalized Compression Distance between a collection of files.<br><br>loc_files: glob. Location to the files under consideration. For instance: `data/*.*` for all files in the `data`-directory. <br><br> compressor: String. Any of `bz2`, `zlib`, `snappy`, `pylzma`. Using `snappy` or `pylzma` requires installation of those libraries. Default: `bz2`.<br><br>compression_level: Int. The compression level (higher is usually better for approaching Kolmogorov Complexity). When using `snappy`-compressor, the `compression_level` is ignored. Default: `9`.|
| nwd(l, compressor="bing") | Computes Normalized Web Distance between a list of strings.<br><br>l: list of strings.<br><br> compressor: String. Any of `bing`, `google`, `duckduckgo`, `wikipedia`, `hackernews`. **Note:** Most search engines do not allow automated access in their Terms of Service. Though Koolmogorov does not hammer the search engines (it uses sleep to limit requests per second), search engines may not appreciate that many automated requests from a single IP. Use at your own discretion and calculate risk of a (temporary) ban. Default: `bing`.|
| ncmsd(l, compressor=ind) | Computer Normalized Count-Min Sketch Distance between a list of strings.<br><br>l: list of strings.<br><br> compressor: ind. A Count-Min Sketch indexed on certain corpora using MakeIndex.|


### MakeTree(verbose=0, random_state=None, n_jobs=1, n_iters=100, method="native")

| Attribute | Value |
| --- | --- |
| verbose | a non-negative integer denoting level of verbosity. Default `0` |
| random_state | Int or None. If Int set a random seed for replication. Default `None`. |
| n_jobs | Int. Number of multiple processes at once. Default `1`. |
| n_iters | Int. Number of iterations before halting. Default `100`. |
| method | String. Methodology for making the tree. `native` for Python. `MCMC` for original MCMC MQTC. The original method is both faster and more accurate, allowing you to build bigger trees in less time. Set `koolmogorov.LOC_MAKETREE` variable to the location of the MakeTree executable. Original executables can be downloaded and build from the official [CompLearn site](http://complearn.org/download.html). Note: When using `MCMC`-method other attributes like `random_state`, `n_jobs`, and `n_iters` are ignored. Default `native`. |

| Function | Description |
| --- | --- |
| fit_transform(dm) | Builds a tree from a distance matrix. <br><br>Given a dictionary with tuples for keys and distance for values, computes a networkx tree object. <br><br> dm: Dict. A Dictionary with tuples for keys and distances for values. Distances between objects themselves are not stored. Both directions should be stored. For instance, with two objects "bill" and "gates", the distance_matrix may look like: `{("bill", "gates"): 0.15, ("gates", "bill"): 0.15}` <br><br> Returns: G. NetworkX Graph Object. |

