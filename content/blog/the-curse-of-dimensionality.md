+++
title = 'The Curse of Dimensionality'
date = 2024-05-08T15:15:29+05:30
draft = false
ShowToc = true
cover.image = "images/dimension.jpg"
math = true
+++

As the number of features or dimensions grows in a dataset, the effectiveness of many machine learning algorithms can suffer from an issue called the "curse of dimensionality." This problem is particularly prevalent for instance-based algorithms like K-nearest neighbors (KNN).

The KNN algorithm relies on calculating the distances between data points to determine which points are nearest neighbors.

In low dimensional spaces like 2D or 3D, the concepts of distance and neighborhood are relatively straightforward. However, in high dimensional spaces, some counterintuitive properties emerge.


The KNN classifier makes the assumption that similar points share similar labels. Unfortunately, in high dimensional spaces, points that are drawn from a probability distribution, tend to never be close together. We can illustrate this on a simple example. We will draw points uniformly at random within the unit cube (illustrated in the figure) and we will investigate how much space the k𝑘 nearest neighbors of a test point inside this cube will take up.

Formally, imagine the unit cube $[0,1]^d$. All training data is sampled _uniformly_ within this cube, i.e. $\forall i, x_i \in [0,1]^d$, and we are considering the k=10 nearest neighbors of such a test point.

One might think that one rescue could be to increase the number of training samples, $n$, until the nearest neighbors are truly close to the test point. How many data points would we need such that $\ell$ becomes truly small? Fix $\ell = 1/10 = 0.1$

So as $d \gg 0$ almost the entire space is needed to find the 10-NN. This breaks down the $k$-NN assumptions, because the $k$-NN are not particularly closer (and therefore more similar) than any other data points in the training set. Why would the test point share the label with those $k$-nearest neighbors, if they are not actually similar to it?

$\Rightarrow n = k\ell^{-d} = k \cdot 10^d$, which grows exponentially! For $d > 100$ we would need far more data points than there are electrons in the universe...

As $d \rightarrow \infty$, the ratio of the distances to the 10th and 11th nearest neighbors approaches 1:

$$\lim_{d \rightarrow \infty} \frac{r_{10}}{r_{11}} = 1$$

Where $r_k$ is the distance to the $k^{th}$ nearest neighbor.

Additionally, the expected distance to the 10th nearest neighbor, normalized by $\sqrt{d}$, approaches a constant:

$$\lim_{d \rightarrow \infty} \frac{\mathbb{E}[r_{10}]}{\sqrt{d}} = \sqrt{\frac{8}{\pi}}$$

Let $\ell$ be the edge length of the smallest hyper-cube that contains all $k$-nearest neighbors of a test point. Then $\ell^d \approx kn$ and $\ell \approx (kn)^{1/d}$. If $n=1000$, how big is $\ell$?

So as $d \gg 0$ almost the entire space is needed to find the 10-NN. This breaks down the $k$-NN assumptions, because the KNN are not particularly closer (and therefore more similar) than any other data points in the training set. Why would the test point share the label with those $k$-nearest neighbors, if they are not actually similar to it?

$$\mathrm{As} \quad d \gg 0, \quad \ell \approx (kn)^{1/d} \approx 1$$

This means that when the dimensionality $d$ is very high, the edge length $\ell$ of the smallest hyper-cube containing the $k$-nearest neighbors approaches 1, which is the entire unit cube $[0, 1]^d$.

Therefore, to find the 10-nearest neighbors, almost the entire space is needed. This violates the key assumption of KNN that the nearest neighbors are more similar or closer to the test point than other data points. If the $k$-nearest neighbors are not actually similar or close, then why would we expect the test point to share the same label as those $k$-nearest neighbors?

The breakdown of this core assumption of KNN in high dimensions is a manifestation of the curse of dimensionality, which severely limits the algorithm's effectiveness in such scenarios.


The curse of dimensionality poses a significant challenge for the K-nearest neighbors (KNN) algorithm, as it breaks down one of its fundamental assumptions: the notion of similarity based on proximity. In high-dimensional spaces, the concept of nearness becomes increasingly diluted, and the distances between data points become more uniform, rendering the algorithm less effective.

To mitigate the curse of dimensionality, several techniques can be employed:

1. **Feature selection**: Carefully selecting the most informative features or dimensions can help reduce the overall dimensionality of the data while retaining the essential information. This can be achieved through various methods, such as principal component analysis (PCA), filter methods, or embedded methods like regularization.
2. **Dimensionality reduction**: Techniques like PCA, t-SNE, or autoencoders can be used to project the high-dimensional data into a lower-dimensional subspace, potentially preserving the essential structure and relationships between data points.
3. **Alternative distance metrics**: Instead of relying on the traditional Euclidean distance, other distance metrics that are more robust to high dimensions, such as the cosine similarity or fractional distances, can be explored.
4. **Ensemble methods**: Combining multiple KNN classifiers trained on different subsets of features or using different distance metrics can help mitigate the curse of dimensionality and improve overall performance.
5. **Data transformation**: Applying appropriate data transformations, such as normalization or scaling, can help maintain the relative importance of features and prevent certain dimensions from dominating the distance calculations.
6. **Hybrid approaches**: Integrating KNN with other machine learning techniques, such as decision trees or support vector machines, can leverage the strengths of different algorithms and potentially overcome the limitations of KNN in high dimensions.

While the curse of dimensionality is a fundamental challenge in high-dimensional spaces, these techniques can help alleviate its impact on the KNN algorithm. However, it is essential to carefully evaluate the trade-offs between dimensionality reduction and potential information loss, as well as consider alternative algorithms that may be better suited for high-dimensional data, such as tree-based methods or kernel-based methods.