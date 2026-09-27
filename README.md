# RecipeNER-FG: Fine-Grained Named Entity Recognition in Recipe Ingredients

**Application of Deep Learning for Named Entity Recognition in Recipes**

Code, data and experiments for **RecipeNER-FG**, a manually annotated corpus and benchmark for
fine-grained NER on recipe ingredient phrases. It uses a new 8-tag schema that captures quantity,
unit, ingredient name, state, form, size and dry/fresh information.

> Ritisha Singh and Shruti Jha, IIIT Delhi. *RecipeNER-FG: A Fine-Grained Annotated Corpus and
> Benchmark for Named Entity Recognition in Recipe Ingredients.* Under review.

---

## Highlights

| | |
|---|---|
| **Corpus** | 10,785 ingredient phrases (85,920 tokens) from RecipeDB, sampled with spherical K-Means over TF-IDF vectors for lexical diversity |
| **Schema** | 8 tags: `NAME`, `QUANTITY`, `UNIT`, `STATE`, `FORM`, `SIZE`, `DF` (dry/fresh), `O` |
| **Annotation quality** | Cohen's κ = 0.90, Krippendorff's α = 0.90 on a 250-phrase doubly-annotated subset (1,954 tokens) |
| **Benchmark** | 10 transformer models; **DeBERTa-v3-large reaches 97.82 macro-F1** |
| **Negative result** | WordNet/entity-swap/span-shuffle augmentation *lowers* F1 for every model (−0.18 to −0.89 pts) |

<p align="center"><i>Pipeline: raw phrase → token-level annotation → sub-word tokenisation → encoder LM → token-classification head.</i></p>

---

## Repository structure

```
.
├── README.md
├── requirements.txt
├── .gitignore
│
├── data/
│   ├── raw/
│   │   └── dataset1.csv                 # column-wise annotation (Quantity, Unit, Name, Form, State, Dry/Fresh, Size, ...)
│   ├── processed/
│   │   ├── 10k_data.csv                 # phrase → token-level tag list (first conversion)
│   │   └── 10k_data_modified.csv        # ★ FINAL RecipeNER-FG corpus used for all experiments
│   ├── augmented/
│   │   ├── aug_train_df.csv             # augmented training split (9,549 phrases)
│   │   └── aug_test_df.csv              # held-out test split (3,235 phrases)
│   └── spacy/
│       ├── train.spacy                  # DocBin files for the spaCy pipeline
│       └── test.spacy
│
├── docs/
│   └── annotation_guidelines.md         # tag definitions, rules and worked examples
│
├── notebooks/
│   ├── 01_data_preparation/
│   │   ├── 01_phrase_sampling.ipynb     # phrase collection, cleaning, sampling
│   │   ├── 02_tags_generation.ipynb     # dataset1.csv → token-level labels (10k_data*.csv)
│   │   └── 03_data_augmentation.ipynb   # WordNet synonyms, entity swap, span shuffle → data/augmented/
│   ├── 02_benchmark_original/           # one notebook per model, trained on the original corpus
│   │   ├── BERT_base.ipynb
│   │   ├── BERT_base_CRF.ipynb
│   │   ├── RoBERTa_base.ipynb
│   │   ├── DistilRoBERTa_base.ipynb
│   │   ├── DistilBERT_base.ipynb
│   │   ├── FoodBaseBERT-NER.ipynb
│   │   ├── BERT_food.ipynb              # ZachBeesley/bert-finetuned-food
│   │   ├── DeBERTa_v3_base.ipynb
│   │   ├── DeBERTa_v3_large.ipynb
│   │   ├── DeBERTa_v3_large_final.ipynb # ★ best model: training, evaluation, per-tag F1, confusion matrix
│   │   └── spacy.ipynb                  # spaCy + DeBERTa-v3-large transformer pipeline
│   ├── 03_benchmark_augmented/          # same models trained on the augmented data (*_Augmentation.ipynb)
│   ├── 04_analysis/
│   │   └── DeBERTa_v3_large_kfold_error_analysis.ipynb  # stratified k-fold, error analysis, HF upload
│   └── archive_early_experiments/       # early BTP experiments (older dataset versions); kept for reference
│
├── configs/
│   └── spacy/
│       ├── config.cfg                   # ★ spaCy training config (microsoft/deberta-v3-large)
│       ├── config_base.cfg              # DeBERTa-v3-base variant
│       ├── config_output.cfg            # config saved with the trained spaCy output
│       └── config_early_experiments.cfg
│
├── results/
│   └── deberta_v3_large_ner_predictions.csv   # test-set predictions of the best model
│
└── models/                              # NOT on GitHub (weights are several GB), see models/README.md
    ├── README.md
    ├── deberta_v3_large/                # deberta_v3_large_ner.pth, test_loader_deberta_v3_large.pth
    ├── checkpoints/                     # best_model.pth (early-stopping checkpoints)
    └── spacy_deberta_v3_large/          # model-best/, model-last/
```

All notebooks use **paths relative to their own folder** (e.g. `../../data/processed/10k_data_modified.csv`),
so run Jupyter from the notebook's directory, or start it at the repository root and open the notebook from there.

---

## Annotation schema

| Tag | Definition | Examples |
|---|---|---|
| `NAME` | Core ingredient name | flour, butter, onion |
| `QUANTITY` | Numerical amount, including fractions and ranges | 2, 1/2, 3–4 |
| `UNIT` | Measurement unit associated with a quantity | cup, tbsp, oz, clove |
| `STATE` | Action-based or condition descriptor applied at time of use | melted, frozen, fat-free, day-old |
| `FORM` | Physical/structural presentation: shape, style, variety, colour-as-type | pieces, finely chopped, fillet, *red* (in red capsicum) |
| `SIZE` | Size descriptor not expressed as a unit–quantity pair | small, large, thick |
| `DF` | Dry vs. fresh variant, affecting flavour and density | dried, fresh |
| `O` | Any token outside all entity spans | and, to taste |

Bracketed content (`-LRB- … -RRB-`) and text after "or" are tagged `O`. The hardest boundary is
**STATE vs. FORM**: *melted* butter, banana *peeled* and *fat-free* milk are `STATE`; onion *finely chopped*,
*red* capsicum and cut into *pieces* are `FORM`. See [`docs/annotation_guidelines.md`](docs/annotation_guidelines.md)
for the full rules and worked examples.

### Token distribution (85,920 tokens)

| Tag | Tokens | | Tag | Tokens |
|---|---:|---|---|---:|
| O | 31,086 | | STATE | 9,860 |
| NAME | 17,179 | | UNIT | 8,530 |
| QUANTITY | 12,446 | | FORM | 3,991 |
| SIZE | 1,801 | | DF | 1,027 |

### Inter-annotator agreement (250 phrases, 1,954 tokens)

| Tag | κ | % agree | n |
|---|---:|---:|---:|
| QUANTITY | 0.957 | 98.98 | 262 |
| UNIT | 0.930 | 98.77 | 180 |
| NAME | 0.908 | 97.19 | 356 |
| STATE | 0.897 | 97.95 | 212 |
| SIZE | 0.911 | 99.59 | 45 |
| DF | 0.945 | 99.85 | 28 |
| FORM | 0.781 | 98.21 | 88 |
| O | 0.878 | 94.22 | 783 |
| **Overall** | **0.90** | | **1,954** |

---

## Data format

`data/processed/10k_data_modified.csv` (and the augmented splits) have two columns:

| Ingredient_Phrases | Labels |
|---|---|
| `5 ounces spinach leaves` | `['QUANTITY', 'UNIT', 'NAME', 'FORM']` |
| `1/4 cup sunflower oil` | `['QUANTITY', 'UNIT', 'NAME', 'NAME']` |

Phrases are whitespace-tokenised and `Labels` is a Python-literal list with one tag per token
(parse it with `ast.literal_eval`).

---

## Setup

```bash
git clone https://github.com/Ritisha-21089/Application-of-DL-in-NER-in-Recipes.git
cd Application-of-DL-in-NER-in-Recipes
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
python -m nltk.downloader wordnet omw-1.4      # only needed for augmentation
```

A CUDA GPU is strongly recommended; DeBERTa-v3-large was trained on a GPU server.

## Reproducing the experiments

1. **Data (optional, the processed CSVs are already included):** run the notebooks in
   `notebooks/01_data_preparation/` in order.
2. **Benchmark on the original corpus:** open any notebook in `notebooks/02_benchmark_original/`.
   For the headline result, use `DeBERTa_v3_large_final.ipynb`.
3. **Augmentation study:** notebooks in `notebooks/03_benchmark_augmented/`.
4. **spaCy pipeline:**
   ```bash
   python -m spacy train configs/spacy/config.cfg \
       --output models/spacy_deberta_v3_large \
       --paths.train data/spacy/train.spacy --paths.dev data/spacy/test.spacy --gpu-id 0
   ```
5. **Error analysis / k-fold:** `notebooks/04_analysis/`.

### Training setup (all transformer models)

| Setting | Value |
|---|---|
| Tokenizer | each model's default |
| Max sequence length | 128 |
| Batch size | 16 |
| Optimizer | AdamW, lr = 2×10⁻⁵ |
| Epochs | up to 8, early stopping with patience 3 (validation loss) |
| Loss | token classification; `O` tokens set to −100 (ignored) |
| Split | 9,549 train / 3,235 test phrases |

## Results

Macro F1 / precision / recall (%) on the original and augmented RecipeNER-FG datasets.

| Model | Orig. F1 | Orig. P | Orig. R | Aug. F1 | Aug. P | Aug. R | Notebook |
|---|---:|---:|---:|---:|---:|---:|---|
| spaCy-DeBERTa-v3-large | 94.00 | 94.00 | 94.00 | 93.66 | 93.00 | 94.00 | `spacy.ipynb` |
| BERT-base + CRF | 96.01 | 95.24 | 96.82 | 95.49 | 94.90 | 96.11 | `BERT_base_CRF.ipynb` |
| BERT-base | 97.52 | 97.44 | 97.61 | 96.63 | 96.08 | 97.24 | `BERT_base.ipynb` |
| RoBERTa-base | 97.55 | 97.32 | 97.79 | 96.75 | 96.27 | 97.26 | `RoBERTa_base.ipynb` |
| DistilRoBERTa-base | 97.37 | 96.54 | 98.25 | 97.02 | 96.99 | 97.06 | `DistilRoBERTa_base.ipynb` |
| DistilBERT-base | 96.61 | 95.69 | 97.73 | 96.45 | 96.71 | 96.31 | `DistilBERT_base.ipynb` |
| FoodBaseBERT-NER | 97.11 | 96.62 | 97.63 | 96.77 | 96.31 | 97.27 | `FoodBaseBERT-NER.ipynb` |
| BERT-finetuned-food | 97.44 | 97.14 | 97.75 | 96.75 | 96.29 | 97.23 | `BERT_food.ipynb` |
| DeBERTa-v3-base | 97.46 | 97.42 | 97.53 | 97.24 | 97.14 | 97.35 | `DeBERTa_v3_base.ipynb` |
| **DeBERTa-v3-large** | **97.82** | **97.69** | **97.96** | **97.27** | **97.07** | **97.48** | `DeBERTa_v3_large_final.ipynb` |

Frequent tags (`NAME`, `QUANTITY`) score highest; sparse or ambiguous tags (`FORM`, `SIZE`) score lower.
The most common errors are STATE→NAME (51) and FORM→NAME (47), where descriptors such as "seeds"
or "ground" get absorbed into the ingredient name, plus FORM↔STATE confusion (19 + 9).

**Augmentation hurt every model.** Replacing `O` tokens with general-vocabulary WordNet synonyms
disrupts the context around entity boundaries and creates a register mismatch with real
ingredient phrases.

## Why fine-grained tags matter

| Tag | What name-only NER misses | What the fine-grained tag enables |
|---|---|---|
| FORM | "1 cup almonds" ≠ "1 cup almond flour" in mass and nutrition | Correct ingredient-database lookup |
| STATE | "softened butter" and "melted butter" behave differently | Preparation-aware substitution |
| DF | "1 tsp dried oregano" ≠ "1 tsp fresh oregano" (≈3:1 flavour) | Quantity rescaling across dry/fresh forms |
| SIZE | "small onion" ≠ "large onion" (≈70 g vs. 150 g) | Mass estimation without an explicit quantity |
| QTY + UNIT | Substitution at parity needs the original amount | Unit conversion and nutrition re-computation |

## Pre-trained models

The weights are several GB each, so they are **not stored on GitHub**. See
[`models/README.md`](models/README.md) for the expected layout and loading code.

## Limitations

- The corpus comes only from RecipeDB. Generalisation to user-generated recipes (AllRecipes, OpenFoodFacts) is untested.
- Rare tags (`FORM`, `SIZE`) remain hard because of class imbalance and overlap with neighbouring tags.
- Structured-NER systems (W2NER, GoLLIE, UniversalNER) are not benchmarked yet. RecipeNER-FG is released as a benchmark for them.

## Citation

```bibtex
@misc{singh2026recipenerfg,
  title  = {RecipeNER-FG: A Fine-Grained Annotated Corpus and Benchmark for Named Entity Recognition in Recipe Ingredients},
  author = {Singh, Ritisha and Jha, Shruti},
  year   = {2026},
  note   = {Under review},
  institution = {IIIT Delhi}
}
```

## Authors

- **Ritisha Singh**, IIIT Delhi ([GitHub](https://github.com/Ritisha-21089), [LinkedIn](https://www.linkedin.com/in/ritishasingh2703))
- **Shruti Jha**, IIIT Delhi

Ingredient phrases come from [RecipeDB](https://cosylab.iiitd.edu.in/recipedb/).
