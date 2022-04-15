Single cell RNA-seq Analysis
================
Valentin GabeffThéo ImlerAbigail StrefelerLéo Sumi
March 2019

## Introduction

A heterogenous mix of mouse cells was analysed using single-cell
RNA-seq. After pre-processing and quality control, we have a dataset
containing 73 cells showing expression of 23’351 genes. The number of
read genes per cell, on average, comes out to 9503. This was calculated
by simply counting the number of nonzero entries for each cell, and
performing an average. The number of genes per cell can give information
about quality, since samples epxressing too few genes are more likely
free RNA, and samples expressing too many are likely doublets of cells.
Plotting expression per cell shows a fairly even distribution of gene
counts over the different cells.

![](single-cell-analysis_files/figure-gfm/genes%20per%20cell-1.png)<!-- -->

## Results

### Part 1: Number of cell types

The number of cell types is determined using clustering. The criteria we
use for clustering is the expression of certain genes: cells that
express these genes in a similar manner are more closely related than
those that express them differently.

First we must decide which genes we will use to compare cells. We have
two main options, either we retreive only highly expressed genes or we
use genes with highly variable expression. Here we decided to use genes
with a high variability as the histogram of variability shows an evident
tail. Yet, this choice should not alter our results much. Therefore out
of all of the detected genes, we keep only these ones in our analysis.

![](single-cell-analysis_files/figure-gfm/marker%20genes-1.png)<!-- -->![](single-cell-analysis_files/figure-gfm/marker%20genes-2.png)<!-- -->

To be able to correctly compare expression between cell types, gene
expression must be normalized. This is due to the difference in the
number of reads per cells.

The output of the voom function shows the log2 normalization of number
of reads per gene with respect to the square-root standard deviation.
One can observe that genes with many reads tend to have a low variation
while genes with few reads are quite variable among the cell population.
Our study focuses on the latter case.

![](single-cell-analysis_files/figure-gfm/normalization-1.png)<!-- -->

So far, data are spread on 2335 dimensions. We use Principal Component
Analysis (PCA) to project the data on a 3D space whose basis explain
most of the variance among the data.

Reducing dimensionality allows us to visualize the data, and gives us
some idea about what clusters may be present. The `fviz_eig` function
allows us to determine the percentage of explained variance for each
principal component. For tridimensional visualization, we select the
three first principal components that explain 40% of the data variance,
as shown on the bar plot below. On a 3D interactive plot, we can
visually identify 4 clusters.

To cluster the data, we use a more robust clustering method than k-means
called partitioning around medoids (PAM). Representative objects called
medoids are found, and groups are creating by assigning each point to
the nearest medoid. This method minimizes the sum of dissimilarities of
observations to their closest representative object. The number of
clusters must be specified before execution. Therefore we ran the
algorithm multiple times for a number of clusters varying between 2 and
15, and we evaluated the results each time. The “elbow” point was found
at 4 clusters, so this organization best explains the data.

![](single-cell-analysis_files/figure-gfm/pca%20and%20clustering-1.png)<!-- -->![](single-cell-analysis_files/figure-gfm/pca%20and%20clustering-2.png)<!-- -->

    ## integer(0)

    ## [1] 0.6437378 0.6163140 0.3846867
    ## Creating new device

### Part 2: Marker genes for different cell types

Different cell types will have certain genes that they express in a
manner different from other cell types. These are called differentially
expressed (DE) genes. By choosing one cluster of cells and comparing the
expression of their genes against the other clusters, one can
statistically determine the fold change of expression using the limma
package. The genes were selected as being DE genes when the absolute
value of expression fold change was superior to 2 and the differences
were significant (adjusted p-values \< 0.05).

The 100 differentially expressed genes can be represented using a
heatmap. We indeed see that the 4 different cell types show differential
expression of these.

On the heatmap, we can see that the first two clusters are quite close
to each other. This is also seen on the plot of the DE genes for each
cluster where RGS4 is expressed mainly in one cluster, but also in
another. This means that the segregation between these two groups of
cells is less important than between the other clusters. Heatmaps
effectively retrieved 4 clusters from the 400 DE genes that were
selected, hence confirming the clustering method performed above. (Note
that the name of the cluster displayed on the heatmap may not match the
one of the k-means.)

We found that TPH2 (Tryptophan hydroxylase 2), SPARC (Osteonectin),
FABP4 (Fatty Acid-Binding Protein 4), and ACTC1 (Actin Alpha Cardiac
Muscle 1) were the most differentially expressed genes for each
individual cluster. When we plot their expression on the PCA, they
clearly belong to the different cell types we have identified. It is
interesting to note that SPARC is expressed by all clusters but one,
because of a negative fold change value. The first differentially
expressed gene with a positive fold change in this cluster is RGS4
(Regulator of G protein Signalling 4), and we have represented this as
well on the plot. A red dot means that the gene is more express in the
corresponding cell. Intensity is calculated as log10(expression+1) and
normalized to the maximum expression level.

![pca3d](../data/pca3d.jpg)  
(nb: please run the .rmd file to have access to the 3d interactive plot.
We were not able to embed it in the html output)

![DE genes](../data/DEgenes.gif)

![](single-cell-analysis_files/figure-gfm/heatmap-1.png)<!-- -->

## Determining the cell types

To determine the different cell types present in our sample, we upload
the differentially expressed genes for each cluster into EnrichR, an
integrative web-based gene-list enrichment analysis tool. We found that
we have cells from the heart, adipose tissue, hypothalamus, and cerebral
cortex. These last two categories explain the similarity between two of
our clusters, since they are both neural tissues. For the sake of the
Enrichment analysis, one must only take DE genes that are more expressed
in the cluster, hence genes such as SPARC that are less expressed
specifically in one cluster are not taken into account.

Cluster 1:

![Cell type for cluster 1](../data/C1_type.jpg) ![Cell type for cluster
1](../data/Enrich4.jpeg)

Cluster 2:  
![Cell type for cluster 2](../data/C2_type.jpg) ![Cell type for cluster
2](../data/Enrich3.jpeg)

Cluster 3:  
![Cell type for cluster 3](../data/C3_type.jpg) ![Cell type for cluster
3](../data/Enrich2.jpeg)

Cluster 4:  
![Cell type for cluster 4](../data/C4_type.jpg) ![Cell type for cluster
4](../data/Enrich1.jpeg)

## Conclusion

Overall, this method allowed us to retrieve 4 distinct clusters from the
initial dataset.  
Clustering was performed by reducing dimension using PCA followed by
k-medoids. Those results were then confirmed the heatmap. DE genes from
each clusters were retreived using Limma and the nature of each cell
group was then found using EnrichR.  
We found that Cluster 1 is mainly composed of cells from the cerebral
cortex while Cluster 4 showed evidences of hypothalamus expression
genes. Due to the proximity of these cell lines, this explain why the
two clusters were closely related on the heatmap and on the PCA plot.
The two other clusters account for heart tissues and adipose brown
tissues.
