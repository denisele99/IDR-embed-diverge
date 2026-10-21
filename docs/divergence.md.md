# Neighbour Distance and Family Neighbour Divergence (FND) Calculation

Pipeline calculates neighbour distance divergence between paralogous protein regions and compares it to an ortholog-based background to compute Family Neighbour Divergence (FND).

Computes:

1. Paralog neighbour divergence (neighbour distances between paralogous segments) (`neighbour_divergence`)
2. Random/background divergence (neighbours distances within orthologs of randomly sampled segments) (`random_neighbour_divergence`)
3. Family Neighbour Divergence (FND) (`calculate_FND`) -> must first run 1. and 2., then set outputs as arguments for 3. and set calculate_FND to true

FND is the difference between the average paralog neighbour distance and the expected ortholog/background neighbour distance.

---

## Requirements

Install fnd_env environment

```bash id="ccjlwm"
conda env create -f envs/fnd_env.yml
conda activate fnd_env
cd idr_embed_diverge
pip install -e .
```

## Command

```bash id="0kjxzk"
python run_ndivergence.py ../configs/ndist_config.yml
```

---

## Input files

- background embeddings 
- ortholog embeddings (file or directory)
- map of family name -> list of genes to embed (family_to_genes.json)

family_to_genes.json
{"CDK": [GENE1, GENE2, ...]}
Where GENE is the Uniprot ID

## Config

```yaml id="a92d9x"
inputs:
  # Embedding file used as the background space for neighbour-rank calculations.
  # May be .csv or .h5. Typically all human IDR or domain embeddings.
  background_emb: ../data/embed/human_idrs_pos_updated_esm_h_prefix.csv

  # Ortholog embedding file or directory of ortholog embedding files (.h5 or .csv).
  # Used to compute ortholog neighbour distances and/or family divergence.
  ortholog_emb: ../data/embed/orthologs/

  # JSON mapping from family name -> list of genes.
  # Required unless only running random_neighbour_divergence.
  fam_map: ../src/idr_diverge/distances/input_args/family_to_genes.json

outputs:
  # Output file or directory for family neighbour-distance results.
  out_distance: ../res/ndist/fam_ndist

compute:
  # Number of pairwise comparisons processed at once.
  # Increase for speed, decrease if running out of memory.
  chunk_size: 8000

ortholog_filter:
  enabled: true

  # Which Ensembl ortholog types to keep.
  # Options: ortholog_one2one, ortholog_one2many, ortholog_one2one_one2many
  ortholog_types: ortholog_one2one_one2many

  # Minimum fraction of paralog segments that must contain a species.
  min_species_coverage: 0.3

  # Minimum number of species required after filtering.
  min_species: 10

  # Ensembl ortholog annotation file used for filtering.
  homolog_annotation: ../data/annotations/ensembl_homol_reference_10-10.pkl

neighbour_divergence:
  # Distance type:
  # between = pairwise distances between different paralogous segments
  # within = pairwise distances among orthologs of the same segment
  dist_type: between

  # Segment label used only for output naming.
  segment: IDR

  # For within divergence only:
  # per_gene = return distance per gene for each species
  # aggregate = return one distance value per species
  
  return_mode: per_gene

random_neighbour_divergence:
  enabled: false

  # Number of random segments sampled for each background estimate.
  sample_size: 100

  # Optional random seed for reproducibility.
  random_seed: 42

  # Output file for random/background neighbour-distance results.
  out_random: ../res/ndist/rand_ndist/random_idr_diverge

calculate_FND:
  enabled: false

  # Background neighbour-distance file (.pkl) or directory.
  # If omitted, background distances will be recomputed from random samples.
  bg_distance_matrix: ../res/ndist/rand_ndist/random_idr_diverge.pkl

  # Family neighbour-distance file or directory produced by neighbour_divergence.
  fam_distance_matrix: ../res/ndist/fam_ndist

  # Output CSV containing FND values.
  output_results: ../res/FND/fnd_results.csv
```

---



## Output Files

### Neighbour Distance Output

`out_distance` and `out_random` produce `.pkl` files containing neighbour distance dictionaries.

For `dist_type: between`, values are typically stored per family and species:

```text id="d5i9kl"
family -> species -> neighbour distance 
(neighbour distance between gene_1_IDR_1 human vs gene_2_IDR_1 species, neighbour distance between gene_2_IDR_1 human vs gene_1_IDR_1 species)

{'O94921_IDR_1_130|P21127_IDR_1_420': {'ENSABRP': [5712.5, 5714.5], 'ENSACCP': [5607.0, 5716.0], .. }}

```

For `dist_type: within`, values are stored per segment/gene by default.

EXAMPLE OF OUTPUT DICT

{GENE1_IDR_1 | GENE2_IDR_1: {SPP: Distance, SPP2: Distance} }

return_mode: per_gene (default)
{'Q15746_IDR_1767_1803': {'ENSABRP': 1.5, 'ENSACAP': 1.5, 'ENSACCP': 2.0}, ...}


return_mode: aggregate
{'ENSABRP': [1.5, 7.0, ..], 'ENSACCP': [1.5, 1.0, ..], ... }


### FND Output

`output_results` is a CSV containing one row per family (or family/species pair), for example:

```text id="5q2n3s"
Group_id,log_mean Divergence_per_ortholog,FND,1samp_ttest_stat,1samp_ttest_pvalue
CDK,{'ENSTRUP': 1.55, 'ENSAPLP': 2.00,..}, 2.29,50.75,3.31e-108
```

Higher positive FND values indicate paralogous segments are more diverged than expected from the ortholog/background baseline.
