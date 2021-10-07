<img title="a title" alt="turboGSEA" src="https://github.com/MaayanLab/turbogsea/blob/main/icon/bgsea_small.png" width=200>

# turboGSEA Introduction

This Python package provides a computationally performant <b>G</b>ene <b>S</b>et <b>E</b>nrichment <b>A</b>nalysis (GSEA) implementation of the pre-rank algorithm [1]. GSEApy was used as the reference for the running sum and enrichment score calculation [2]. The algorithm estimates the enrichment score (ES) distribution of the null model by fitting data to gamma distibutions instead of calculating permutations for each gene set. turboGSEA calculates p-values with much higher accuracy than other reference implementations available in Python.

Gene set libraries can directly be loaded from Enrichr (<a href="https://maayanlab.cloud/Enrichr" target="_blank">https://maayanlab.cloud/Enrichr</a>). For this use the `turbogsea.enrichr.get_library()` function. All libraries can also be listed with `turbogsea.enrichr.print_libraries()`.

turboGSEA provides plotting functions to generate publication ready figures similar to the original GSEA-P software. `turbogsea.plot.running_sum()` plots an enrichment plot for a single gene set and `turbogsea.plot.top_table()` plots the top `n` gene sets in a compact table. 

# Installation

turboGSEA is currently only available as a Python package in this GitHub repository. You can install the turboGSEA Python package and its dependencies through pip by using the following command:

```
$ pip install git+https://github.com/MaayanLab/turbogsea.git
```

# Run enrichment analysis using turboGSEA

turboGSEA depends on two input files. 1) a gene signature and 2) a gene set library. The gene set library is a dictionary with the name of the gene set as key and a list of gene ids as values. Gene set libraries can be loaded directly from Enrichr. The signature should be a pandas dataframe with two columns [0,1]. The first column should contain the gene ids (matching the gene ids in the gene set library).

| index | 0	| 1 |
|:-----|:-------------:|------:|
| 1	| ADO	| -7.833439 |
| 2	| CHUK	| -7.800920 |
| 3	| GOLGA4	| -7.78722 |
| ... | ... | ... |

The gene set library is a dictionary with the gene set names as key and lists of gene ids as values.

```python
{
'ERBB2 SIGNALING PATHWAY (GO:0038128)': ['CDC37',
                                          'PTPN12',
                                          'PIK3CA',
                                          'SOS1',
                                          'CPNE3',
                                          'EGF',
                                          ...
                                         ],
'HETEROTYPIC CELL-CELL ADHESION (GO:0034113)': ['DSC2',
                                                 'ITGAV',
                                                 'ITGAD',
                                                 'LILRB2',
                                                 ...
                                                ],
...
}
```

### Python example

This short example will download two files (signature and gene set library). The gene set library consists of KEGG pathways and the signature is an example signature of differential gene expression of muscle samples from young and old donors. Differential gene expression was computed with Limma Voom.

```python
import turbogsea as turbo
import urllib.request
import pandas as pd

# download example GMT file from Enrichr
url = "https://github.com/MaayanLab/turbogsea/raw/main/testing/ageing_muscle_gtex.tsv"
urllib.request.urlretrieve(url, "ageing_muscle_gtex.tsv")

# read signature as pandas dataframe
signature = pd.read_csv("ageing_muscle_gtex.tsv")

# list available gene set libraries in Enrichr
turbo.enrichr.print_libraries()

# use enrichr submodule to retrieve gene set library
library = turbo.enrichr.get_library("KEGG_2021_Human")

# run enrichment analysis
result = turbo.gsea(signature, library)
```

### Optional Parameters

The main function of `turbogsea.gsea()` supports several optional parameters. The default parameters should work well for most use cases. 

| parameter name | type | default	| description |
|:-----|:---------|:-------------|:------|
| `permutations`	| int | 2000	| Number of randomized permutations to estimate ES distributions. |
| `min_size` | int | 5 | Minimum number of genes in geneset. |
| `max_size` | int | Inf | Maximal number of genes in gene set. |
| `anchors`	| int | 20 | Number of gene set size distributions calculated. Remaining are interpolated. |
| `processes`	| int | 4	| Number of parallel threads. Not much gain after 4 threads. |
| `symmetric` | bool | False | Use same distribution parameters for negative and positive ES. If `False` estimate them separately. |
| `plotting`| bool | False | Plot estimated anchor parametes. |
| `verbose` | bool | False | Toggle additonal output. |
| `seed` | int | 0 | Random seed. Same seed will result in identical result. If seed equal `-1` generate random seed. |

### Plotting enrichment results

turboGSEA supports several plotting functions. `turbogsea.plot.running_sum()` and `turbogsea.plot.top_table()` can be used after enrichment has been performed. `turbogsea.plot.running_sum()` shows the running sum of an individual gene set. It has a `compact` mode in which the image will be more readable if small. `turbogsea.plot.top_table()` shows the top `n` enriched gene sets and displays the results in a table, with normalized enrichment score (NES) and the distribution of hits relative to the gene ranking of the signature.

### Example
```python

import turbogsea as turbo
import urllib.request
import pandas as pd

# download example GMT file from Enrichr
url = "https://github.com/MaayanLab/turbogsea/raw/main/testing/ageing_muscle_gtex.tsv"
urllib.request.urlretrieve(url, "ageing_muscle_gtex.tsv")

# read signature as pandas dataframe
signature = pd.read_csv("ageing_muscle_gtex.tsv")

# list available gene set libraries in Enrichr
turbo.enrichr.print_libraries()

# use enrichr submodule to retrieve gene set library
library = turbo.enrichr.get_library("KEGG_2021_Human")

# run enrichment analysis
result = turbo.gsea(signature, library)

# plot the enrichment results and save to pdf
fig = turbo.plot.running_sum(signature, "CELL ADHESION MOLECULES", library, result=result, compact=False)
fig.savefig("running_sum.png", bbox_inches='tight')

fig_compact = turbo.plot.running_sum(signature, "CELL ADHESION MOLECULES", library, result=result, compact=True)
fig_compact.savefig("running_sum_compact.png", bbox_inches='tight')

fig_table = turbo.plot.top_table(signature, library, result, n=15)
fig_table.savefig("top_table.png", bbox_inches='tight')

```

The resulting plots will look like the examples below:

#### running_sum.pdf

<div style="bachground-color: white">
<img title="a title" alt="turboGSEA sunning_sum" src="https://github.com/MaayanLab/turbogsea/blob/main/icon/running_sum.png" width=300>
</div>

#### running_sum_compact.pdf
<img title="a title" alt="turboGSEA sunning_sum" src="https://github.com/MaayanLab/turbogsea/blob/main/icon/running_sum_compact.png" width=300>

#### top_table.pdf
<img title="a title" alt="turboGSEA sunning_sum" src="https://github.com/MaayanLab/turbogsea/blob/main/icon/top_table.png" width=300>

# Dependencies
Python 3.6+

# References

[1] Subramanian, Aravind, Heidi Kuehn, Joshua Gould, Pablo Tamayo, and Jill P. Mesirov. "GSEA-P: a desktop application for Gene Set Enrichment Analysis." Bioinformatics 23, no. 23 (2007): 3251-3253.

[2] Fang, Z., A. Wolf, Y. Liao, A. McKay, F. Fröhlich, J. Kimmel, L. Xiaohui, and sorrge. "zqfang/gseapy: gseapy-v0. 10.3." (2021).
