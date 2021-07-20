# How are descriptors projected in maplab?

## 1. Extract binary descriptors by a SfM pipepline

Binary features and their descriptors (e.g., BRISK, FREAK) can be extracted by a SfM pipeline, e.g., maplab, from some dataset.

## 2. Projecting binary to float descriptors by LRT or PCA

Binary descriptors can be converted to floats by likelihood ratio test (LRT) or principal component analysis
as described in [2]. 
For instance, for FREAK descriptors, this operation transforms 512-dimension 
(512 bits for a FREAK descriptor) descriptors to 10-dimension (40 bytes) float descriptors.
An implementation of LRT is at [here](https://github.com/ethz-asl/maplab/blob/master/algorithms/loopclosure/descriptor-projection/src/build-projection-matrix.cc).

Briefly, this projection can be described by a projection matrix A.
For a descriptor x, the projection result is x' = Ax.

## 3. Variance balancing by Eigenvalue allocation

The space of these float descriptors will be splitted into a few subspaces.
Similar variances of these subspaces are desired to ensure better quantization performance.
This can be done by Eigenvalue allocation as described in Section 3.2.4 of [3].
It involves two steps.
First find the principal directions of all the descriptors by PCA,
second assign the eigenvalues corresponding to these directions into subspaces by a greedy approach.

Variance balancing can be described by a rotation matrix R.
For a descriptor x, the variance balancing and the ealier projection results in 
x' = RAx.

## 4. Projection validation by MCC.

To validate the obtained projection matrix A is better than a random projection matrix to keep the original distribution of binary descriptors,
we can use by the maximum Matthews correlation coefficient (MCC) score as done in [2].

A random projection matrix N can be obtained by methods in [4].

A projection matrix P can be validated with the following steps.
* First, select descriptor matches and random descriptor non-matches,  see [BuildListOfMatchesAndNonMatches](https://github.com/ethz-asl/maplab/blob/master/algorithms/loopclosure/descriptor-projection/src/build-projection-matrix.cc).
* Second, project these descriptors in these pairs with P.
* Third, compute the distance of two projected descriptors in every pair.
* Fourth, select a threshold on distance and predict whether a pair is a match or a non-match. For all pairs, we can construct a confusion matrix and compute the MCC score.
* Fifth, adjust the threshold on distance so as to maximize the MCC score.
In the end, we obtain the maximum MCC score for a projection matrix on a particular dataset.

By showing that the maximum MCC score of P is better than the score of a random projection matrix N, 
we are fairly certain that P is better than N and our hard work in steps 1-3 pays off.

# References
[1] Lynen et al. Get out of my lab.
[2] Lynen et al. Trajectory-based place-recognition.
[3] Ge et al. Optimized product quantization.
[4] Random projection. https://en.wikipedia.org/wiki/Random_projection

