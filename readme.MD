# Koolmogorov

Koolmogorov is a machine learning, hierarchical clustering, and visualization library in Python based on [CompLearn](http://complearn.org/). Koolmogorov uses approaches, techniques, and data structures from:

- Algorithmic Information Theory
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

### Semantic Word Similarity with Lord of the Rings

We look at co-occurence of word tokens inside sentences of the Lord of the Rings trilogy.

```
from koolmogorov import MakeIndex, MakeDistanceMatrix, MakeTree, MakeViz

tokens = ["two",
          "three",
          "four",
          "red",
          "blue",
          "green",
          "smjagol",
          "gollum",
          "frodo",
          "ring"]

mi = MakeIndex()
ind = mi.index(mi.sentence_splitter("lotr.txt"), 
               ignore_all_but=tokens)

mdm = MakeDistanceMatrix()
dm = mdm.ncmsd(tokens, compressor=ind)

mt = MakeTree(method="native", n_jobs=8, n_iters=450)
G, score = mt.fit_transform(dm)

mv = MakeViz()
mv.visualize(G, 
             method="3Djs", 
             score=score, 
             compressor="Lord of the Rings", 
             title="Semantic Word Analysis on LOTR Trilogy")
```

Result:

![LOTR dendrogram](http://i.imgur.com/jUNd4dO.png)

Live demo: http://mlwave.github.io/tda/Semantic-Word-Analysis-on-LOTR-Trilogy.html

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

### Prime Number Classification

We use Yahoo search engine as a compressor to classify prime numbers.

```
import numpy as np
from sklearn import svm, cross_validation
from koolmogorov import MakeAnchorVector

# We have a train set with 10 prime numbers and 10 non-prime numbers
train = ["6029", "5953", "499", "157", "137", "71", "977", "5323", "7919", "3851",
         "4028", "5334", "501", "1780", "1020", "3173", "4121", "6215", "7023", "7407"]
y = [1] * 10
y += [0] * 10
y = np.array(y)

# For anchors we use 6 prime numbers
anchors = ["433", "919", "1123", "2099", "5233", "7589"]

# We create anchor vectors for the train set with Yahoo search engine
mav = MakeAnchorVector()
X = np.array(mav.nwd(train, anchors, compressor="yahoo"))

# We check leave-one-out performance with a Support Vector Classifier
clf = svm.SVC(probability=True, random_state=42)
kf = cross_validation.KFold(20, n_folds=20, random_state=1, shuffle=False)
for i, (train_fold, test_fold) in enumerate(kf):
	X_train = X[train_fold]
	y_train = y[train_fold]
	X_test = X[test_fold]
	y_test = y[test_fold]
	sample_test = train[test_fold]
	clf.fit(X_train, y_train)
	print(i+1, sample_test, "%f"%clf.predict_proba(X_test)[:,1][0], y_test[0])
```
We get 100% accuracy with 0.5 as a threshold:
```
(1,  '6029', '0.581687', 1)
(2,  '5953', '0.587490', 1)
(3,   '499', '0.563200', 1)
(4,   '157', '0.562242', 1)
(5,   '137', '0.559748', 1)
(6,    '71', '0.559738', 1)
(7,   '977', '0.569535', 1)
(8,  '5323', '0.585396', 1)
(9,  '7919', '0.593069', 1)
(10, '3851', '0.586171', 1)
(11, '4028', '0.447882', 0)
(12, '5334', '0.448476', 0)
(13,  '501', '0.441458', 0)
(14, '1780', '0.449435', 0)
(15, '1020', '0.447405', 0)
(16, '3173', '0.450983', 0)
(17, '4121', '0.449847', 0)
(18, '6215', '0.449977', 0)
(19, '7023', '0.450984', 0)
(20, '7407', '0.451069', 0)
```

Note how non-primes ending with an uneven number get a slightly higher probability of being a prime.

## Documentation

### MakeIndex(verbose=0, w=1000000, d=5)
| Attribute | Value |
| --- | --- |
| verbose | a non-negative integer denoting level of verbosity. Default `0` |
| w | Int. Width of the Count-Min Sketch (number of buckets). Higher leads to more accurate count estimates. Default `1000000`. |
| d | Int. Height of the Count-Min Sketch (number of rows). Higher leads to more accurate count estimates, but increases number of computations. Default `5`. |

| Function | Description |
| --- | --- |
| index(l, ignore_all_but=[]) | Indexes a corpus into a count-min sketch.<br><br>l: list of strings. List of document contents you want to index for co-occurrence.<br><br>ignore_all_but: list of tokens. When non-empty, will only look at these tokens. Default `[]`.<br><br>Returns: ind. Count-min sketch to use as a compressor for MakeDistanceMatrix.ncmsd(). |
| sentence_splitter(loc_corpus) | Helper function to turn a file directly into a list of sentences.<br><br>loc_corpus: String. Location of the file you want to split up into sentences.|
| paragraph_splitter(loc_corpus) | Helper function to turn a file directly into a list of paragraphs.<br><br>loc_corpus: String. Location of the file you want to split up into paragraphs.|

### MakeDistanceMatrix(verbose=0)

| Attribute | Value |
| --- | --- |
| verbose | a non-negative integer denoting level of verbosity. Default `0` |

| Function | Description |
| --- | --- |
| ncd(loc_files, compressor="bz2", compression_level=9) | Computes Normalized Compression Distance between a collection of files.<br><br>loc_files: glob. Location to the files under consideration. For instance: `data/*.*` for all files in the `data`-directory. <br><br> compressor: String. Any of `bz2`, `zlib`, `snappy`, `brotli`, `pylzma`. Using `snappy`, `brotli` or `pylzma` requires installation of those libraries. Default: `bz2`.<br><br>compression_level: Int. The compression level (higher is usually better for approaching Kolmogorov Complexity). When using `snappy`-compressor, the `compression_level` is ignored. Default: `9`.<br><br>Returns: distance_matrix: Dict. A Dictionary with tuples for keys and distances for values.|
| nwd(l, compressor="bing") | Computes Normalized Web Distance between a list of strings.<br><br>l: list of strings.<br><br> compressor: String. Any of `bing`, `google`, `yahoo`, `wikipedia`, `hackernews`. **Note:** Most search engines do not allow automated access in their Terms of Service. Though Koolmogorov does not hammer the search engines (it uses sleep to limit requests per second), search engines may not appreciate that many automated requests from a single IP. Use at your own discretion and calculate risk of a (temporary) ban. Default: `bing`.<br><br>Returns: distance_matrix: Dict. A Dictionary with tuples for keys and distances for values.|
| ncmsd(l, compressor=ind) | Computer Normalized Count-Min Sketch Distance between a list of strings.<br><br>l: list of strings.<br><br> compressor: ind. A Count-Min Sketch indexed on certain corpora using MakeIndex.<br><br>Returns: distance_matrix: Dict. A Dictionary with tuples for keys and distances for values.|

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

## Theory

### Computation
> Algoritmic Information Theory is the result of putting Shannon's information theory and Turing's computability theory into a cocktail shaker and shaking vigorously. The basic idea is to measure the complexity of an object by the size in bits of the smallest program for computing it <sub>— Gregory Chaitin, Centre for Discrete Mathematics and Theoretical Computer Science</sub>

Computation can be done with a Turing machine on a binary input string. Anything that a human can calculate, can be computed with a Turing machine. <sup>[1]</sup> All objects, like DNA, 3D point clouds, prime numbers, documents, agents, populations, and universes have a (possibly course-grained) binary string representation.  <sup>[2]</sup> The counting argument tells us that the majority of strings can not be compressed, yet most objects that we care about have some ordered regularities: They can be compressed.  <sup>[3]</sup>

### Compression

Compression is related to understanding and prediction.  <sup>[4]</sup> Optimal compressions of physical reality yielded short elegant predictive programs like Newton's law of gravity or computer programs calculating the digits of Pi.  <sup>[5]</sup> Compression can be measured by looking at compression ratio's.  <sup>[6]</sup> The more regularities (predictable patterns, laws) the compressor has found in the data, the higher the compression ratio. The absolute shortest program to produce a string gets the highest possible compression ratio.

### Kolmogorov Complexity

The Kolmogorov Complexity of a string is the length (in bits) of the shortest program that produces that string.  <sup>[7]</sup> The more information (unpredictable randomness) a string has, the longer this shortest program has to be.  <sup>[8]</sup> A string is Kolmogorov Random if the shortest program to produce it is not smaller than that string itself: There is no more compression possible, there are no patterns or predictable regularities.  <sup>[9]</sup>

### Information Distance

The Information Distance between two strings is the length (in bits) of the shortest program that transforms one string into another string  <sup>[10]</sup>. This makes it a universal distance measure for objects of all kinds. We apply normalization to take into account the different lengths of the two strings under comparison.  <sup>[11]</sup> The longer the length of the shortest program, the more different two strings are: many computations are needed for the transformation.  <sup>[12]</sup>

### Incomputability

Information distance is based on Kolmogorov Complexity. Unfortunately Kolmogorov Complexity is not computable.  <sup>[13]</sup> Take this Python function:

```
def generate_complexity():
    i = 1
    while True:
      program = "{0:08b}".format(i)
      i += 1
      if kolmogorov_complexity(program) > 900000000:
        return program
```

The function will try every possible binary program, until it finds a program where the shortest description of that program is larger than 900000000 bits. But `generate_complexity` itself is less than 900000000 bits. And so the shortest program length we have found actually has an even shorter description: `generate_complexity()`, which is a contradiction, much like the Berry Paradox: "The<sub>1</sub> smallest<sub>2</sub> positive<sub>3</sub> integer<sub>4</sub> not<sub>5</sub> definable<sub>6</sub> in<sub>7</sub> under<sub>8</sub> twelve<sub>9</sub> words<sub>10</sub>".  <sup>[14]</sup>

Fortunately we can approach Kolmogorov Complexity by using a variaty of compression algorithms and this is exactly what Koolmogorov is doing under the hood.

### Normalized Compression Distance

Using compressors we can apply the Normalized Information Distance with Normalized Compression Distance  <sup>[15]</sup>. To approach Kolmogorov Complexity it is substituted with compression algorithms. Normalized Compression Distance looks at the individual lengths of compressing two strings vs. the length of compressing their concatenation. If the two strings share data patterns, an intelligent compressor can use this repetitive predictable data to compress these concatenated strings better.

### Normalized Web Distance

Normalized Web Distance uses search engines as a compressor <sup>[16]</sup>. Formerly called Normalized Google Distance, this method looks at aggregate page counts of words individually and in concatenation.  <sup>[17]</sup> Words that co-occur more often, are counted as more semantically similar.

### Normalized Count-min Sketch Distance

Normalized Count-min Sketch Distance uses a count-min sketch as a compressor. A Count-min Sketch is a datastructure for approximate counting of events in data streams.  <sup>[18]</sup> We can fit a Count-min Sketch on co-occurence data of strings in a corpus, or collection of corpora.  <sup>[19]</sup> Then the Count-min sketch can be queried just like with Normalized Web Distance.

### Normalized Anchor Distance

With Normalized Anchor Distance we can create vectors for any object by calculating NCD, NWD, or NCMSD between the object and a collection of anchors. These anchors can be anything: random data, different languages, locations, negative sentiment. Vectorized objects can be treated as a semantic knowledge library, dimensionality reduction before clustering, or used for unsupervised and supervised learning.  <sup>[20]</sup>

### Visualization

#### Minimum Quartet Tree Cost

A New Quartet Tree Heuristic for Hierarchical Clustering mutates an initially random unrooted dendrogram until it approaches an optimal weighting of quartet topologies. These trees can be visualized and used for unsupervised data exploration. The basis for a tree is a quartet toplogy:

```
  leaf_a         leaf_c
        \       /
        N1-----N2
        /       \
  leaf_b         leaf_d
```
A tree with 4 leaves is optimal, when the summed distances between leaf_a and leaf_b and leaf_c and leaf_b is the lowest possible (from every possible other configuration). Both methods like Quartet Puzzling and Minimum Quartet Tree Cost, see large dendrograms as composed of these quartet toplogies, sort of like puzzle pieces in a completed puzzle. Quartet Puzzling starts with the best quartet toplogies from all the possible quartet toplogies. It then proceeds to iteravely build a dendrogram. It can not go back to correct an earlier mistake. 

Minimum Quartet Tree Cost uses a form of greedy hillclimbing to mutate a full random dendrogram. Leafs and subtrees are randomly swapped out to create new dendrograms. If the new dendrogram improves on the older dendrogram, then the older dendrogram is discarded. Improvement is measured by weighing the summed distances from the embedded quartet toplogies.

The problem of finding the optimal dendrogram is NP-hard. It may be physically impossible to construct a tree with a fitness of 1. Also Koolmogorov will not always find the best possible tree. But give it some time, and you should get close.

#### Topological Data Analysis

One trick from topological data analysis is to determine "exploratory variables" and the ability to color the graph by a function of interest.

### Lossy Compression

The brain is a lossy compressor. There is not enough brain capacity to infer everything there is to know about the universe. <sup>[21]</sup> By removing noise and slight variations and keeping the most significant features we both save energy and we gain categorization: Even though two lions are not exactly the same, if we look at their significant features, we can group all lions together. <sup>[22]</sup>

### State-Space Compression

Coarse-graining can move the description of a complex system into higher levels of abstraction. We can coarse-grain strings and compare Shanon Entropy between coarse-graining and the original to see how much information we retained. <sup>[23]</sup> Coarse-graining neatly acts a lossy preprocessing step for lossless compression algorithms. Take the following string:

```
inthebeginninggodcreatedtheheavensandtheearth
```

we can coarse-grain (lossy compress) by encoding only vowels vs. non-vowels:

```
100010101001000100011010001011010010000111000
```
Compression here is reduction of the code book (24 possible characters vs. 2 possible characters). We can also coarse-grain by removing stop words:
```
beginninggodcreatedheavensearth
```
Compression here is a literal reduction of the file length.

Using coarse-graining we can study the effect of applying information distance between original strings and coarse-grained strings.

### Shanon Information Theory vs. Autosophy Information Theory

If someone communicates the lossy string `beginninggodcreatedheavensearth`, most people are able to reconstruct the original string without any error. As we share a similar culture, we also share similar code books and conventions about language. <sup>24</sup> Similarly, the string `inthebeginninggodcreatedtheheavensandtheearth` has a shorter program: `open("bible.txt").read()[:45]`.

In the Shannon-Weaver model of communication, we need to communicate all the data. We are not allowed to reference shared resources, without communicating these resources in full. In the same vein: For compression benchmarks we are required to include the size of decoder in calculating the compression ratio. Now the short program `open("bible.txt").read()[:45]` is not so short anymore, since it has to include the length of the entire Bible. This, to ensure that the communication is lossless (the receiver may have a slightly different Bible).

In contrast, Autosophy Information Theory allows communication with references to shared resources. This allows for very high compression rates: A high-density frame from a shared-resource movie can be communicated with just a timestamp and the name of the movie. In this framework, information is only that which creates new knowledge in a receiver. One does not learn what one already knows. Communication is now lossy: Though people have Bibles in different languages, meaning is still being communicated<sup>25</sup>.

The shortest program for an object in Shannon Information Theory is its Kolmogorov Complexity. The shortest program in Autosophy Information Theory is a short reference pointing to the best version of the object. Both the short reference and the best version of the object depend on the Receiver: If the Receiver does not have enough computing power or shared culture then a very difficult to understand/decompress, but elegant, reference may not work. To transfer meaning to a modern English language speaker the best versions of an object are refered with english-language resources. 

Note that in both Shannon and Autosophy Information Theory the conveyed meaning is subjective. The objectively single best Bible, like the roundest circle, does not exist. Regardless of its binary Truth value: A priest and an atheist will have different impression of the first sentence of the Bible.

### Cross-Complexity

In statistical modeling we often use a form of cross-validation. First a train set is split up into *n* parts, then we leave 1 part out and use the remaining parts for training, and we repeat this *n* times with different parts. Performance on the out-of-fold data gives an estimate on how the model would perform on unseen data.<sup>[26]</sup> We can apply a similar principle to string compression: we chunk the string into *n* parts. We calculate Normalized Compression Distance between the parts. Parts that are very dissimilar to other parts, denote a part of the object that is complex. <sup>[27]</sup>

The following string describes the object `a white wall with two windows`. We chunk the string into 12 rows of size 16.

```
1  111111111111111
2  111111111111111
3  111111111111111
4  111000111000111
5  111000111000111
6  111000111000111
7  111111111111111
8  111111111111111
9  111111111111111
10 111111111111111
11 111111111111111
12 111111111111111
```
The very simple strings [1-3] will be closer to strings [7-12]. The majority of this object has a low complexity. Strings [4-6] have larger cross-compression distance to the majority of other strings: They have a higher object-part complexity. Naturally our attention is drawn to the complex parts of an object. Using cross-compression we can tell *where* a `white wall with two windows` differs from a `white wall with no windows`. <sup>[28]</sup> Chunking can be done in multiple dimensions and with overlap: Tokenization, 1-D fixed size chunks/chargrams, 2-D convolutions, or i-D hypercubes.


[1] Turing A., [Systems of Logic Based on Ordinals](https://webspace.princeton.edu/users/jedwards/Turing%20Centennial%202012/Mudd%20Archive%20files/12285_AC100_Turing_1938.pdf) (1938) <br>
[2] Wheeler J. A.: [Information, Physics, Quantum: The Search for Links](http://cqi.inf.usi.ch/qic/wheeler.pdf) (1989)<br>
[3] Cilibrasi R., [Statistical Inference Through Data Compression](https://www.illc.uva.nl/Research/Publications/Dissertations/DS-2007-01.text.pdf) (2007)<br>
[4] Mahoney M., [Data Compression Explained](http://mattmahoney.net/dc/dce.html) (2010) <br>
[5] Schmidhuber J., [On Learning to Think: Algorithmic Information Theory for Novel Combinations of Reinforcement Learning Controllers and Recurrent Neural World Models](https://arxiv.org/abs/1511.09249) (2015) <br>
[6] Bowery J. Mahoney M. Hutter M.: [50'000€ Prize for Compressing Human Knowledge](http://prize.hutter1.net/) (2006)<br>
[7] Li M. Vitanyi P.: [An introduction to Kolmogorov Complexity and its applications](http://www-2.dc.uba.ar/materias/azar/bibliografia/LiVitanyi1997AnIntroductiontoKolmogorov.pdf) (1992)<br>
[8] Shannon C.: [A Mathematical Theory of Communication](http://math.harvard.edu/~ctm/home/text/others/shannon/entropy/entropy.pdf) (1948)<br>
[9] Fortnow L. by Gale, A.: [Kolmogorov Complexity](https://people.cs.uchicago.edu/~fortnow/papers/kaikoura.pdf) (2000) <br>
[10] Bennett C., Gacs P., Li M., Vitanyi P., Zurek W.: [Information Distance](https://arxiv.org/abs/1006.3520) (1993) <br>
[11] Vitanyi P., Balbach F., Cilibrasi R., Li M.: [Normalized Information Distance](http://homepages.cwi.nl/~paulv/papers/chapter08.pdf) (2008)<br>
[12] Levenshtein V.: [Binary codes capable of correcting deletions, insertions, and reversals](https://nymity.ch/sybilhunting/pdf/Levenshtein1966a.pdf) (1963)<br>
[13] Chaitin G., Arslanov A., Calude C.: [Program-size Complexity Computes the Halting Problem](https://researchspace.auckland.ac.nz/bitstream/handle/2292/3517/008HHP.pdf) (1995)<br>
[14] Chaitin G.: [The Berry Paradox](https://www.cs.auckland.ac.nz/~chaitin/unm2.html) (1995) <br>
[15] Cilibrasi R., Vitanyi P.: [Clustering by compression](https://arxiv.org/abs/cs/0312044) (2003)<br>
[16] Cilibrasi R., Vitanyi P.: [Normalized Web Distance and Word Similarity](https://arxiv.org/abs/0905.4039) (2009) <br>
[17] Cilibrasi R., Vitanyi P.: [The Google Similarity Distance](https://arxiv.org/abs/cs/0412098) (2004) <br>
[18] Cormode G., Muthukrishnan S.: [An Improved Data Stream Summary: The Count-Min Sketch and its Applications](http://dimacs.rutgers.edu/~graham/pubs/papers/cm-full.pdf) (2003) <br>
[19] Goyal A. Daume III H. Cormode, G.: [Sketch Algorithms for Estimating Point Queries in NLP](http://www.umiacs.umd.edu/~amit/Papers/goyalPointQueryEMNLP12.pdf) (2012)<br>
[20] Palatucci M., Pomerleau D., Hinton G., T. Mitchell: [Zero-Shot Learning with Semantic Output Codes](https://www.cs.cmu.edu/~fmri/papers/zero-shot-learning.pdf) (2012) <br>
[21] Wolpert D.: [Statistical Limits of Inference](https://arxiv.org/abs/0708.1362) (2007)<br>
[22] Anderson J. A.: [After Digital: Computation as Done by Brains and Machines](https://books.google.com/books?id=KwJLDgAAQBAJ&printsec=frontcover&dq=After+Digital:+Computation+as+Done+by+Brains+and+Machines) (2017)<br>
[23] Wolpert D., Grochow J. A., Libby E., DeDeo S.: [Optimal high-level descriptions of dynamical systems](https://arxiv.org/abs/1409.7403) (2015)<br>
[24] Frege G.: [On Sense and Reference](http://brianrabern.net/sensereference.pdf) (1892)<br>
[25] Holtz K.: [A Primer on Information Theories](http://www.autosophy.net/informth.htm) (1977)<br>
[26] Hastie T., ‎Tibshirani R., ‎Friedman J. [The Elements of Statistical Learning: Data Mining, Inference, and Prediction](https://statweb.stanford.edu/~tibs/ElemStatLearn/printings/ESLII_print10.pdf) (2009)<br>
[27] Leman D., Feelders A., Knobbe A.: [Exceptional Model Mining](https://www.kiminkii.com/publications/PKDD2008.pdf) (2008)<br>
[28] Bengio Y., Courville A., Vincent P.: [Representation Learning: A Review and New Perspectives](http://www.cl.uni-heidelberg.de/courses/ws14/deepl/BengioETAL12.pdf) (2012)