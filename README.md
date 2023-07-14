scDiagnosis
================
Smriti Chawla
2023-07-08

<h2>
scDiagnosis: The package implements set of diagnostic functions to check
appropriateness of cell assignments in single cell gene expression
profiles
</h2>
<h3>
Getting started
</h3>
<h2>
Prerequisites
</h2>

R version tested: 4.2.3 (2023-03-15)

To use the scDiagnosis package, you need the following R packages
installed:

- SingleCellExperiment
- scRNAseq
- scater
- scran
- ggplot2
- AUCell
- SingleR
- Matrix
- corrplot
- RColorBrewer
- gridExtra

## Usage

To explore the capabilities of the scDiagnosis package, you can load
your own data or use the provided example with publicly available data
of Marrow tissue single cell gene expression profiles from He S et
al. (2020). Single-cell transcriptome profiling of an adult human cell
atlas of 15 major organs. Genome Biol 21, 1:294. This dataset is
obtained from the scRNAseq R package and is used for analyzing and
visualizing diagnostic plots to inspect query/test and reference
datasets and check the appropriateness of cell type assignments. In the
provided example, the dataset is divided into 70% reference and 30%
query data.

``` r
## Loading libraries
library(scDiagnosis)
library(scater)
library(scran)
library(scRNAseq)
library(RColorBrewer)
library(SingleR)
library(AUCell)
library(corrplot)
library(gridExtra)
```

## Scatter Plot: Total UMI Counts vs. Annotation Scores

Let’s load the Marrow dataset for demonstration purposes. The dataset
represents single-cell gene expression profiles of Marrow tissue. In
this example, we will analyze the relationship between total UMI (Unique
Molecular Identifier) counts and SingleR scores for a specific cell
type.

To perform this analysis, we first divide the dataset into reference and
query datasets. The reference dataset serves as a reference for cell
type annotation, while the query dataset contains cells that we want to
assign cell types to. We then log-transform the expression values in
both datasets for further analysis.

Next, we use the SingleR package to obtain cell type scores for the
query dataset. These scores provide information about the likelihood of
each cell being a certain cell type.

For the scatter plot, we focus on a specific cell type, in this case,
“CD4”. We extract the corresponding SingleR scores for this cell type
from the SingleR results.

To overlay the total UMI counts and SingleR scores, we assign the
SingleR scores to the colData of the query dataset. This allows us to
access the scores for each cell when generating the scatter plot. The
scatter plot displays the log-transformed total UMI counts on the x-axis
and the SingleR scores on the y-axis.

It’s important to note that the use of SingleR for cell type annotation
is just one example. Users can use any other cell type annotation method
to obtain cell type scores. The scatter plot provides a visual
representation of this relationship, allowing for further examination
and interpretation of the cell type assignments in the dataset.

``` r

   # Load data
   sce <- HeOrganAtlasData(tissue = c("Marrow"), ensembl = FALSE)

   # Divide the data into reference and query datasets
   indices <- sample(ncol(assay(sce)), size = floor(0.7 * ncol(assay(sce))), replace = FALSE)
   ref_data <- sce[, indices]
   query_data <- sce[, -indices]

   # Log-transform datasets
   ref_data <- logNormCounts(ref_data)
   query_data <- logNormCounts(query_data)

   # Run PCA
   ref_data <- runPCA(ref_data)
   query_data <- runPCA(query_data)

   # Get cell type scores using SingleR
   pred <- SingleR(query_data, ref_data, labels = ref_data$reclustered.broad)

   # Extract scores and labels for a specific cell type
   cell_type <- "CD4"
   score <- pred$scores[pred$labels == cell_type, cell_type]
  
   # Assign labels to query data
   colData(query_data)$labels <- pred$labels

   # Generate scatter plot for Total UMIs vs. Annotation Scores
   plotUMIsAnnotationScatter(query_data, score, "labels", cell_type)
```

<img src="man/figures/Scatter plot Total UMIs vs annotation scores-1.png" width="100%" />

## Examining Distribution of UMI Counts and Annotation Scores

In addition to the scatter plot, we can gain further insights into the
gene expression profiles by visualizing the distribution of total UMI
(Unique Molecular Identifier) counts and annotation scores for a
specific cell type. This allows us to examine the variation and patterns
in expression levels and scores across cells assigned to the cell type
of interest.

To accomplish this, we create two separate histograms. The first
histogram displays the distribution of the annotation scores. The x-axis
represents the scores, while the y-axis represents the frequency of
cells with a given score.

The second histogram visualizes the distribution of total
log-transformed UMI counts per cell. This provides insights into the
overall gene expression levels for the specific cell type. The x-axis
represents the total logcounts, while the y-axis represents the
frequency of cells within each count range.

By examining the histograms, we can observe the range, shape, and
potential outliers in the distribution of both annotation scores and
total UMI counts. This allows us to assess the appropriateness of the
cell type assignments and identify any potential discrepancies or
patterns in the gene expression profiles for the specific cell type.

``` r
# Generate histogram
plotCellTypeDistribution(score, query_data, cell_type)
```

<img src="man/figures/Distribution of UMIs and Annotation Score-1.png" width="100%" />

The example code provided demonstrates how to utilize the
plotCellTypeDistribution function with the necessary data and packages.
Here we are visualizing the distribution of counts and scores for the
“CD4” cell type.

## Exploring Gene Expression Distribution

To gain insights into the gene expression values for a specific gene and
its distribution across cells, we can use the
plotGeneExpressionDistribution function. This function allows us to
visualize the distribution of expression values for a particular gene of
interest, both overall and within specific cell types.

``` r
# Generate histogram
plotGeneExpressionDistribution(query_data, "labels", "B_and_plasma", "VPREB3")
```

<img src="man/figures/histogram gene expression-1.png" width="100%" />

In the provided example, we are examining the distribution of expression
values for the gene “VPREB3” in the dataset. The function generates a
histogram to display the distribution of expression values across all
cells, with the x-axis representing the expression values and the y-axis
representing the frequency of cells within each expression range.
Additionally, the expression values can be further stratified by
specific cell types, providing insights into the gene expression
patterns within different cell populations.

## Visualize Gene Expression on Dimensional Reduction Plot

To gain insights into the gene expression patterns and their
representation in a dimensional reduction space, we can utilize the
plotGeneExpressionDimred function. This function allows us to plot the
gene expression values of a specific gene on a dimensional reduction
plot generated using methods like t-SNE, UMAP, or PCA. Each single cell
is color-coded based on its expression level of the gene of interest.

In the provided example, we are visualizing the gene expression values
of the gene “VPREB3” on a PCA plot. The PCA plot represents the cells in
a lower-dimensional space, where the x-axis corresponds to the first
principal component (Dimension 1) and the y-axis corresponds to the
second principal component (Dimension 2). Each cell is represented as a
point on the plot, and its color reflects the expression level of the
gene “VPREB3,” ranging from low (lighter color) to high (darker color).

``` r
# Generate dimension reduction plot color code by gene expression
plotGeneExpressionDimred(query_data, "PCA", c(1, 2), "VPREB3")
```

<img src="man/figures/scatter plot gene expression-1.png" width="100%" />

The dimensional reduction plot allows us to observe how the gene
expression of “VPREB3” is distributed across the cells and whether any
clusters or patterns emerge in the data.

## Visualize Gene Sets or Pathway Scores on Dimensional Reduction Plot

In addition to examining individual gene expression patterns, it is
often useful to assess the collective activity of gene sets or pathways
within single cells. This can provide insights into the functional
states or biological processes associated with specific cell types or
conditions. To facilitate this analysis, the scDiagnosis package
includes a function called plotGeneSetScores that enables the
visualization of gene set or pathway scores on a dimensional reduction
plot.

The plotGeneSetScores function allows you to plot gene set or pathway
scores on a dimensional reduction plot generated using methods such as
PCA, t-SNE, or UMAP. Each single cell is color-coded based on its scores
for specific gene sets or pathways. This visualization helps identify
the heterogeneity and patterns of gene set or pathway activity within
the dataset, potentially revealing subpopulations with distinct
functional characteristics.

``` r

# Compute scores using AUCell
expression_matrix <- assay(query_data, "logcounts")
cells_rankings <- AUCell_buildRankings(expression_matrix, plotStats = F)

# Generate gene sets
gene_set1 <- sample(rownames(expression_matrix), 10)
gene_set2 <- sample(rownames(expression_matrix), 20)

gene_sets <- list(geneSet1 = gene_set1,
                  geneSet2 = gene_set2)

# Calculate AUC scores for gene sets
cells_AUC <- AUCell_calcAUC(gene_sets, cells_rankings)

# Assign scores to colData
colData(query_data)$geneSetScores <- assay(cells_AUC)["geneSet1", ]

# Plot gene set scores on PCA
plotGeneSetScores(query_data, method = "PCA", feature = "geneSetScores")
```

<img src="man/figures/Visualize gene set or pathway scores on dimensional reduction scatter plot -1.png" width="100%" />

In the provided example, we demonstrate the usage of the
plotGeneSetScores function using the AUCell package to compute gene set
or pathway scores. Custom gene sets are generated for demonstration
purposes, but users can provide their own gene set scores using any
method of their choice. It is important to ensure that the scores are
assigned to the colData of the query_data object and specify the correct
feature name for visualization.

By visualizing gene set or pathway scores on a dimensional reduction
plot, you can gain a comprehensive understanding of the functional
landscape within your single-cell gene expression dataset and explore
the relationships between gene set activities and cellular phenotypes.

## Visualizing Reference and Query Cell Types using Multidimensional Scaling (MDS)

Here, we perform Multidimensional Scaling (MDS) analysis on the query
and reference datasets to examine their similarity. The dissimilarity
matrix is calculated based on the correlation between the datasets, and
MDS is used to obtain low-dimensional coordinates for each cell.
Subsequently, a scatter plot is generated, where each data point
represents a cell, and the cell types are color-coded using custom
colors provided by the user. This visualization enables the comparison
of cell type distributions between the query and reference datasets in a
reduced-dimensional space.

``` r
## Selcting highly variable genes
ref_var <- getTopHVGs(ref_data, n=2000)
query_var <- getTopHVGs(query_data, n=2000)

# Intersect the gene symbols to obtain common genes
common_genes <- intersect(ref_var, query_var)

# Select desired cell types
selected_cell_types <- c("CD4", "CD8", "B_and_plasma")
ref_data_subset <- ref_data[common_genes, ref_data$reclustered.broad %in% selected_cell_types]
query_data_subset <- query_data[common_genes, query_data$reclustered.broad %in% selected_cell_types]

# Extract cell types for visualization
ref_labels <- ref_data_subset$reclustered.broad
query_labels <- query_data_subset$reclustered.broad

# Combine the cell type labels from both datasets
mdata <- c(paste("Query", query_labels), paste("Reference", ref_labels))

## Define the cell types and legend order
cell_types <- c("Query CD8", "Reference CD8", "Query CD4", "Reference CD4", "Query B_and_plasma", "Reference B_and_plasma")
legend_order <- cell_types

## Define the colors for cell types
color_palette <- brewer.pal(length(cell_types), "Paired")
color_mapping <- setNames(color_palette, cell_types)
cell_type_colors <- color_mapping[cell_types]

## Generate the MDS scatter plot with cell type coloring
visualizeCellTypeMDS(query_data_subset, ref_data_subset, mdata, cell_type_colors, legend_order)
```

<img src="man/figures/CMD scatter plot-1.png" width="100%" />

Upon examining the MDS scatter plot, we observe that the “CD4” and “CD8”
cell types overlap to some extent.By observing the proximity or overlap
of different cell types, we can gain insights into their potential
relationships or shared characteristics.

The selection of custom genes and desired cell types depends on the
user’s research interests and goals. It allows for flexibility in
focusing on specific genes and examining particular cell types of
interest in the visualization.

## Cell Type-specific Pairwise Correlation Analysis and Visualization

This analysis aims to explore the correlation patterns between different
cell types in a single-cell gene expression dataset. It involves
comparing the gene expression profiles of cells from a reference dataset
and a query dataset.

To perform the analysis, we start by computing the pairwise correlations
between the query and reference cells for selected cell types (“CD4”,
“CD8”, “B_and_plasma”). The Spearman correlation method is used, user
can also use Pearsons correlation coeefficient.

This will return average correlation matrix which can be visulaized by
user’s method of choice. Here, the results are visualized as a
correlation plot using the corrplot package.

``` r
selected_cell_types <- c("CD4", "CD8", "B_and_plasma")
cor_matrix_avg <- computeAveragePairwiseCorrelation(query_data_subset, ref_data_subset, "labels", "reclustered.broad", selected_cell_types, "spearman")

# Plot the pairwise average correlations using corrplot
corrplot(cor_matrix_avg, method = "number", tl.col = "black")
```

<img src="man/figures/Cell Type-specific Pairwise Correlation Analysis and Visualization -1.png" width="100%" />

This analysis allows us to examine the correlation patterns between
different cell types in the single-cell gene expression dataset. By
visualizing the average pairwise correlations, we can gain insights into
the relationships and similarities between cell types based on their
gene expression profiles.

In this case, users have the flexibility to extract the gene expression
profiles of specific cell types from the reference and query datasets
and provide these profiles as input to the function. Additionally, they
can select their own set of genes that they consider relevant for
computing the pairwise correlations. For demonstartion we have used
common highly variable genes from reference and query dataset.

By providing their own gene expression profiles and choosing specific
genes, users can focus the analysis on the cell types and genes of
interest to their research question.

## Pairwise Distance Analysis and Density Visualization

his function allows for the calculation of pairwise distances between
query and reference cells of a specific cell type in a single-cell gene
expression dataset. The pairwise distances are calculated using a
specified distance metric, such as “euclidean”, “manhattan”, etc.
Subsequently, the function generates density plots to visualize the
distribution of distances for different pairwise comparisons.

The analysis focuses on the specific cell type “CD8”, and the pairwise
distances are calculated using the “euclidean” distance metric.

``` r
calculatePairwiseDistancesAndPlotDensity(query_data_subset, ref_data_subset, "labels", "reclustered.broad", "CD8", "euclidean")
```

<img src="man/figures/Pairwise Distance Analysis and Density Visualization-1.png" width="100%" />

Further, user can also use correlation for calculation of pairwise
distances between query and reference cells of a specific cell type.

The analysis focuses on the specific cell type “CD8”, and the pairwise
distances are calculated using the correlation distance metric. User can
use spearman or pearson correlation coefficient as a method of choice.

``` r
calculatePairwiseDistancesAndPlotDensity(query_data_subset, ref_data_subset, "labels", "reclustered.broad", "CD8", "correlation" ,"spearman")
```

<img src="man/figures/Pairwise Distance Analysis and Density Visualization correlation based-1.png" width="100%" />

By utilizing this function, users can explore the pairwise distances
between query and reference cells of a specific cell type and gain
insights into the distribution of distances through density plots. This
analysis aids in understanding the similarities and differences in gene
expression profiles for the selected cell type within the query and
reference datasets.

## Linear regression analysis

Performing linear regression analysis on a SingleCellExperiment object
enables users to examine the relationship between a principal component
(PC) from the dimension reduction slot and an independent variable of
interest. By specifying the desired dependent variable as one of the
principal components (e.g., “PC1”, “PC2”, etc.) and providing the
corresponding independent variable from the colData slot, users can
explore the associations between these variables within the single-cell
gene expression dataset (reference and query).

``` r
summary <- performLinearRegression(query_data, "PC1", "labels")
#> 
#> Call:
#> lm(formula = Dependent ~ Independent, data = df)
#> 
#> Residuals:
#>     Min      1Q  Median      3Q     Max 
#> -8.1992 -2.5498 -0.5047  2.2330 14.3817 
#> 
#> Coefficients:
#>                    Estimate Std. Error t value Pr(>|t|)    
#> (Intercept)         -8.0845     0.2954  -27.37   <2e-16 ***
#> IndependentCD4       4.4826     0.3520   12.74   <2e-16 ***
#> IndependentCD8      13.8634     0.3430   40.42   <2e-16 ***
#> IndependentMyeloid   8.2913     0.5863   14.14   <2e-16 ***
#> ---
#> Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
#> 
#> Residual standard error: 3.581 on 965 degrees of freedom
#> Multiple R-squared:  0.6953, Adjusted R-squared:  0.6944 
#> F-statistic: 734.2 on 3 and 965 DF,  p-value: < 2.2e-16
```

By conducting linear regression, one can assess whether the PC values
are significantly associated with the cell types. This analysis helps
uncover whether there is a systematic variation in PC values across
different cell types. Understanding the relationship between PC values
and cell types can provide valuable insights into the biological or
technical factors driving cellular heterogeneity. It can help identify
PC dimensions that capture variation specific to certain cell types or
distinguish different cellular states.

## Conclusion

In this analysis, we have demonstrated the capabilities of the
scDiagnosis package for assessing the appropriateness of cell
assignments in single-cell gene expression profiles. By utilizing
various diagnostic functions and visualization techniques, we have
explored different aspects of the data, including total UMI counts,
annotation scores, gene expression distributions, dimensional reduction
plots, gene set scores, pairwise correlations, pairwise distances, and
linear regression analysis.

Through the scatter plots, histograms, and dimensional reduction plots,
we were able to gain insights into the relationships between gene
expression patterns, cell types, and the distribution of cells in a
reduced-dimensional space. The examination of gene expression
distributions, gene sets, and pathways allowed us to explore the
functional landscape and identify subpopulations with distinct
characteristics within the dataset. Additionally, the pairwise
correlation and distance analyses provided a deeper understanding of the
similarities and differences between cell types, highlighting potential
relationships and patterns.
