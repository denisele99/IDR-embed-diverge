# GO Enrichment from Embedding Neighbours (or BLAST hits)

This script identifies significantly enriched GO terms for protein regions by performing GO enrichment on their nearest neighbours in embedding space.

Neighbours can be selected using:

* `knn`: k nearest neighbours in pLM embedding space (default)
* `BLAST`: top k BLAST hits
* `RANDOM`: random k neighbours as a background/control

Default behaviour uses 50 nearest neighbours (`k = 50`).

## Environment

```bash
conda env create -f envs/env.yml
conda activate diverge_env
pip install -e .
```

[uses env.yml?? No specific env?]

## Command

```bash id="ccjlwm"
python run_go_enrichment.py --config configs/go_config.yaml
```

## Config

```yaml id="ygojyc"
# GO ontology and annotation files
go_obo: ../data/annotations/go_2024.obo
go_annotations: ../data/annotations/uniprot_GO_annotations_2024.tsv

# Number of neighbours used for enrichment
k: 50

# Significance threshold after multiple-testing correction
alpha: 0.01

# Query IDs to analyze.
# Must exactly match IDs in target_embeddings.
query_ids: Q8IZL9_IDR_7_288, P21127_IDR_441_723

# If true, run enrichment for all IDs in target_embeddings
query_all: false

# Neighbour selection method:
# knn = nearest neighbours in embedding space
# BLAST = top BLAST hits
# RANDOM = random neighbours
search_method: knn

# Embedding file used to find neighbours.
# Typically human IDR or domain embeddings (.csv or .h5).
target_embeddings: ../data/embed/human_idrs_pos_updated_esm.csv

# BLAST output table, uses default format
# Required only when search_method = BLAST
target_blast_output: ../data/blast_res/human_idrs_pos_blast_out_names.tsv

# Output CSV containing enriched GO terms
output: ../res/go_enrich/knn_enrichment_goa_idrs.csv
```

## Required Inputs

Specified inside config:

* Embedding file containing the query regions and background dataset
* GO ontology (`.obo`)
* GO annotation table
* Optional BLAST output file if using `search_method: BLAST`

Query IDs must exactly match the identifiers in the embedding file.

## Output

The script writes a CSV table containing significantly enriched GO terms for each query region.
includes dictionaries of enriched GO terms and their adjusted p-values, and dictionary with their term counts in k neighbours / background term count
Example columns:

```text id="8m7rf8"
query_id,term_adj_p,term_counts
```

Example:

```text id="vrq3pw"
Q96ND8_IDR_60_211,{'GO:0006357': 2.23e-14, 'GO:0019219': 1.38e-11, ...}, {'GO:0006355': '20/7717', 'GO:2001141': '20/7774'}
```


## Related Scripts

To evaluate or summarize enrichment results, see:

```text id="f0b7g5"
scripts/run_enrich_eval.py
```

##Notes:
src/idr_diverge/go_enrichment/go_enrichment_.py and go_eval.py will break if configs/go_config.yaml is renamed or moved from configs (depends on go_obo and go_annotation file defined there)
