---
title: Linear Algebra Notes
layout: notes
---

# Linear Algebra Notes

__Warning__: These notes are a work in progress and probably contain some errors. Please let me know at [keithshep@gmail.com](mailto:keithshep@gmail.com) know if you spot any.

## Terminology & Notation

__diagonal matrix__: a (usually square) matrix where all entries off the main diagonal are 0. Eg:
\[ \begin{bmatrix}
	4 & 0 & 0 \\
	0 & 7.7 & 0 \\
	0 & 0 & 0.3
\end{bmatrix} \]

__identity matrix__ or __unit matrix__: can be written as $\mathbf{I_n}$ where $n$ is the number of rows and columns. The Identity Matrix is a diagonal matrix where all values on the diagonal are $1$. Eg:
\[ \mathbf{I_3} = \begin{bmatrix}
	1 & 0 & 0 \\
	0 & 1 & 0 \\
	0 & 0 & 1
\end{bmatrix} \]
A property of identity matrices is that for some matrix $\mathbf{A}$: \[ \mathbf{A_{m \times n}} = \mathbf{A_{m \times n} \mathbf{I_n}} =  \mathbf{\mathbf{I_m} A_{m \times n}} \]

__unit vector__: a vector whose length is $1$.

__vector norm__: the norm of vector $\mathbf{v}$ is written as $\|\mathbf{v}\|$. The norm is the length of a vector. A vector has been *normalized* when it's direction is unchanged but it's length is scaled to $1$. This is sometimes written as the vector name with a ``hat''. Eg: \[ \hat{v} = {\mathbf{v} \over \|\mathbf{v}\|} \]

__dot product__: the dot product of vectors $\mathbf{u}$ and $\mathbf{v}$ can be written as $\mathbf{u} \cdot \mathbf{v}$ and the __inner product__ which is a generalization of the dot product can be written as $\langle \mathbf{u}, \mathbf{v} \rangle$. In order to calculate the dot product you multiply all corresponding elements of the two vectors and then sum the resulting values. This implies that the vectors must be the same length. In Euclidean space, the inner product of two vectors (expressed in terms of coordinate vectors on an orthonormal basis) is the same as their dot product. However, in more general contexts (e.g., vector spaces that use Complex numbers as scalars) the inner and the dot products are typically defined in ways that do not coincide.

__orthogonal__: vectors are orthogonal when they are perpendicular with each other. So, if we think about this in 3D space the x, y and z axes are all mutually perpendicular. We can also rotate the axes vectors together and the resulting vectors will be perpendicular. Also note that the __dot product__ of two orthogonal vectors with be zero: \[u \cdot v = 0 \]

__basis__: In linear algebra, a basis which we'll call $\mathbf{B}$ is a set of linearly independent vectors that, in a linear combination, can represent every vector in a given vector space which we'll call $\mathbf{V}$. In order to simplify this let's say that for our purposes when we talk about vector spaces we will be referring to one of the $\mathbf{R^n}$ vector spaces (that is, all possible vectors of some length $n$ containing real values). A basis must satisfy the following properties:

* Every vector in $\mathbf{V}$ can be expressed as a \underline{linear combination} of vectors in $\mathbf{B}$ in a unique way.
* $\mathbf{B}$ is a minimal generating set of $\mathbf{V}$, i.e., it is a generating set and no proper subset of $\mathbf{B}$ is also a generating set.
* $\mathbf{B}$ is a maximal set of linearly independent vectors, i.e., it is a linearly independent set but no other linearly independent set contains it as a proper subset.

Note that there is also the idea of a __standard basis__ which is composed of __unit vectors__ along each axis (if we think in terms of cartesian coordinates). So the standard basis for $\mathbf{R^n}$ is made up of the columns of the identity matrix $\mathbf{I_n}$

__orthonormal basis__: a basis where every vector in the basis is a __unit vector__ and orthogonal. All __standard bases__ are also orthonormal.

__rank__: (wikipedia) The column rank of a matrix $\mathbf{A}$ is the maximum number of linearly independent column vectors of $\mathbf{A}$. The row rank of a matrix $\mathbf{A}$ is the maximum number of linearly independent row vectors of $\mathbf{A}$. The rank of an $ m \times n $ matrix cannot be greater than $m$ nor $n$. A matrix that has a rank as large as possible is said to have __full rank__; otherwise, the matrix is __rank deficient__.

__matrix transpose__: the transpose operation involves taking all matrix rows in sequence and making them matrix columns (or vice versa). So, if $\mathbf{A=B^T}$ then it is also true that $a_{i,j} = b_{j, i}$. Sometimes an apostrophe is used to denote the transpose as in $\mathbf{A=B'}$

__symmetric matrix__: a square matrix that is equal to its transpose: $\mathbf{C} = \mathbf{C}^T$

__martix row/column notation__: we can refer to the $i^{th}$ row of matrix $\mathbf{A}$ like $\mathbf{A_{(i)} = (a_{i,0}, \dots, a_{i,n})}$ and the $j^{th}$ column like $\mathbf{A^{(j)} = (a_{0,j}, \dots, a_{m,j})}$


## Matrix Multiplication

Take as an example the matrix multiplication $\mathbf{AB}=\mathbf{C}$:

![graphical illustration of matrix multiplication](https://upload.wikimedia.org/wikipedia/commons/e/eb/Matrix_multiplication_diagram_2.svg "Image from http://en.wikipedia.org/wiki/Matrix_multiplication")

$c_{i,j}$ is calculated by taking the dot product of the $i^{th}$ row vector from $\mathbf{A}$ and the $j^{th}$ column from $\mathbf{B}$. Also note that because these two vectors must be the same length:

* The column count of $\mathbf{A}$ must equal the row count of $\mathbf{B}$
* The row count of $\mathbf{C}$ will equal the row count of $\mathbf{A}$
* The column count of $\mathbf{C}$ will equal the column count of $\mathbf{B}$

## Singular Value Decomposition

Applications of SVD include:

* SVD is a building block for some calculations
	* calculating matrix rank
	* calculating a least squares solution (as an alternative to the QR decomposition)
	* principal component analysis
	* efficient computation of eigenvalue decomposition
* image compression
* edge detection
* facial (and I assume other feature) recognition

### Intuition for SVD

Three mutually compatible points of view:

* a method for transforming correlated variables into a set of uncorrelated ones that better expose the various relationships among the original data items
* a method for identifying and ordering the dimensions along which data points exhibit the most variation
* once we have identified where the most variation is, it's possible to find the best approximation of the original data points using fewer dimensions. Hence, SVD can be seen as a method for data reduction.

"These are the basic ideas behind SVD: taking a high dimensional, highly variable set of data points and reducing it to a lower dimensional space that exposes the substructure of the original data more clearly and orders it from most variation to the least. What makes SVD practical for NLP applications is that you can simply ignore variation below a particular threshhold to massively reduce your data but be assured that the main relationships of interest have been preserved."

### The Components of SVD

We can write the decomposition as:

* $\mathbf{A_{m \times n}} = \mathbf{U_{m \times p} \Sigma_{p \times p} V_{p \times n}^T}$
* ... *or* $\mathbf{AV} = \mathbf{U\Sigma}$
* ... *or* $\mathbf{Av_j} = \mathbf{u_j \sigma_j}$: "This formula says that the matrix A takes any basis vector $\mathbf{v_j}$ and maps it to a direction $\mathbf{u_j}$ with length $\mathbf{\sigma_j}$".

... where ...
* $\mathbf{A_{m \times n}}$ is some real-valued matrix to be decomposed
* $p$ is $min(M, N)$
* $\mathbf{V}$: the columns of $\mathbf{V}$ are the orthonormal eigenvectors of $\mathbf{A^TA}$. Geometrically speaking this is one of the two "rotations"
* $\mathbf{\Sigma}$ is a __diagonal matrix__ with with the __singular values__ along the diagonal. $diag(\sigma_1, \dots, \sigma_p)$. These values are the square roots of __eigenvalues__ from $\mathbf{U}$ or $\mathbf{V}$. By convention these rows are ordered such that $\sigma_i \geq \sigma_{i + 1}$. The geometric interpretation of $\mathbf{\Sigma}$ is that it acts as a scaling function.
* $\mathbf{U}$: the columns of $\mathbf{U}$ are orthonormal eigenvectors of $\mathbf{AA^T}$. Geometrically speaking this is the other rotation matrix

## Eigenvalue Decomposition

For symmetric matrix $\mathbf{C}$, $\mathbf{C} = \mathbf{U\Lambda U^T}$

"PCA can be done by eigenvalue decomposition of a data covariance (or correlation) matrix or singular value decomposition of a data matrix, usually after mean centering (and normalizing or using Z-scores) the data matrix for each attribute."

"When $\mathbf{A}$ is interpreted as a covariance matrix and its eigenvalue decomposition is performed, each of the $\mathbf{u_j}$ axes denote a principal direction (component) and each $\mathbf{\sigma_j}$ denotes one standard deviation along that direction."

In this case the eigenvalue decomposition is a __principal components analysis__ with eigenvectors ($\mathbf{U}$) giving the direction of the major effects and eigenvalues ($\mathbf{\Lambda}$) giving the magnitude of those effects.

We also see a relationship between SVD and eigenvalue decomposition:

Given $\mathbf{A} = \mathbf{U \Sigma V^T}$, we get:

$\mathbf{C} = \mathbf{AA^T} = \mathbf{U \Sigma V^T}\mathbf{V \Sigma U^T} = \mathbf{U \Lambda U^T}$

Note: appendix A of the Szeliski book tells us how this relationship can be used, for example, to efficiently perform eigenvalue decomposition for facial recognition.

## QR Factorization

A building block for SVD and Eigenvalue Decomposition. We have (a very inefficient) example implementation in F#.

## Cholesky Factorization

A more stable, more efficient alternative to using gaussian elimination: (example of gaussian elimination is in F#).

## Linear Least Squares Regression

Applications include:

* image alignment
* large scale structure from motion
* image restoration
