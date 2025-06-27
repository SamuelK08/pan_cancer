# Pan-Cancer Analysis of miRNA Functional Modules

## Project Overview

This project implements a comprehensive, automated bioinformatics pipeline to investigate the role of microRNAs (miRNAs) across 30 different cancer types from The Cancer Genome Atlas (TCGA). The central challenge in miRNA research is their complex, context-dependent function; a single miRNA can target hundreds of mRNAs, and they often work in cooperative groups.

This pipeline moves beyond single-miRNA analysis by using a network-based approach to identify "functional modules": groups of miRNAs and their target genes that work together as a team. By analyzing these modules, we can uncover the high-level biological processes driven by specific sets of miRNAs in different cancer contexts.

The pipeline culminates in a pan-cancer synthesis that identifies both conserved, universal miRNA functions and unique, cancer-specific regulatory networks.

### Key Features

*   **Automated End-to-End Pipeline:** A single master script executes the entire workflow, from raw network data to final summary tables.
*   **Pan-Cancer Scope:** The analysis is applied to 30 different TCGA cancer types, enabling large-scale comparative analysis.
*   **Bipartite Network Analysis:** Utilizes algorithms specifically designed for both Louvain and Louvain-Bipartite (miRNA-mRNA) networks for methodologically sound module discovery.
*   **Machine Learning for Interpretation:** Employs a novel NLP and clustering pipeline to automatically group thousands of redundant pathway enrichments into clean, high-level functional themes.
*   **Reproducible & Modular:** The entire pipeline is built with modular scripts and a Conda environment to ensure full reproducibility.

## Directory Structure
```.
├── louvain/
|   └── 01_Input_Data/
│       └── pathway_gmt_files/  #contains pathway definitions for enrichment 
│   ├── 02_Module_Discovery/    #scripts & results using standard Louvain
│   ├── 03_Pathway_Enrichment/  #scripts and results for functional enrichment
│   ├── 04_Functional_Analysis/ #scripts for ML grouping and final table generation
│   ├── 05_Pan_Cancer_Analysis/ #final synthesis for the Louvain method
│   └── run_all_cancers.py      #master script for this method
└── louvain-bipartite/
|   └── 01_Input_Data/
│       └── pathway_gmt_files/    
│   ├── 02_Module_Discovery/    #scripts & results using Bipartite-Louvain
│   ├── 03_Pathway_Enrichment/
│   ├── 04_Functional_Analysis/
|   ├── 05_Pan_Cancer_Analysis/ #final synthesis for the Bipartite method
|   └── run_all_cancers.py      
```.

## The Pipeline Workflow

The `run_all_cancers.py` script runs the following core scripts for each of the 30 cancer types:

1.  **`02_Module_Discovery/find_modules.py`**
    *   **Input:** A cancer-specific edge list (e.g., `brca_EdgeList2.txt`).
    *   **Process:** Identifies functional modules using a bipartite-aware Louvain community detection algorithm.
    *   **Output:** A `_Modules.txt` file listing the members of each detected community.

2.  **`03_Pathway_Enrichment/run_enrichment.R`**
    *   **Input:** The `_Modules.txt` file.
    *   **Process:** Performs a functional enrichment analysis (Fisher's Exact Test) for each module against a comprehensive database of gene sets.
    *   **Output:** A folder containing a detailed CSV report of enriched pathways for each module.

3.  **`04_Functional_Analysis/ml_functional_grouping.py`**
    *   **Input:** The folder of enrichment result CSVs.
    *   **Process:** Uses a Sentence Transformer (NLP) model to create semantic embeddings of pathway names, then uses K-Means clustering to group related pathways. Finally, it names these clusters using a TF-IDF analysis.
    *   **Output:** A `_Functions_Summary_ML.xlsx` file summarizing the high-level functions for each module.

4.  **`04_Functional_Analysis/create_final_summary.py`**
    *   **Input:** The ML summary file, the modules file, and the original network file.
    *   **Process:** Synthesizes the results by combining the functional themes with the top 3 most influential miRNAs (ranked by local degree) for each module.
    *   **Output:** A final `_Final_Paper_Table.csv` for the cancer type.

5.  **`synthesize_pan_cancer.py` (Called at the end of the master script)**
    *   **Input:** All 30 `_Final_Paper_Table.csv` files.
    *   **Process:** Aggregates all results to identify conserved and cancer-specific functions.
    *   **Output:** The final `Pan_Cancer_miRNA_Function_Summary.xlsx` file.

## Installation & Setup

This project uses a Conda environment to manage all Python and R dependencies, ensuring full reproducibility.

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/SamuelK08/pan_cancer.git
    cd pan_cancer
    ```

2.  **Create and activate the Conda environment:**
    The following commands will create a new environment named `sam-development` and install all necessary packages.

    ```bash
    # Create the environment with Python
    conda create -n pan_cancer_analysis python=3.9 -y

    # Activate the environment
    conda activate pan_cancer_analysis

    # Install Python packages (using conda-forge for compiled ones)
    conda install -c conda-forge pandas networkx cdlib networkit scikit-learn openpyxl xlsxwriter -y

    # Install the NLP package with pip
    pip install sentence-transformers

    # Install R and R packages (using conda-forge and bioconda)
    conda install -c conda-forge r-base r-tidyverse r-argparse -y
    conda install -c bioconda r-fgsea r-biomart -y
    ```

## Usage

Once the environment is set up and the input data is in the `01_Input_Data` folder, the entire pan-cancer analysis can be run with a single command from the project's root directory:

```bash
python run_all_cancers.py
