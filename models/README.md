# Model weights

Trained checkpoints are **not stored in the GitHub repository** — they are far larger than
GitHub's 100 MB per-file limit. Keep (or download) them locally with the layout below so that
the notebooks' relative paths resolve.

```
models/
├── deberta_v3_large/
│   ├── deberta_v3_large_ner.pth            # ~5.2 GB  best model (DeBERTa-v3-large, 97.82 macro-F1)
│   │                                        #          torch checkpoint: {'epoch','model_state_dict','optimizer_state_dict'}
│   └── test_loader_deberta_v3_large.pth     # ~5 MB    serialized held-out test dataset (3,235 phrases)
├── checkpoints/
│   ├── best_model.pth                       # ~1.7 GB  early-stopping checkpoint (lowest val loss) from the HF training loop
│   └── best_model_spacy_output.pth          # ~1.7 GB  same kind of checkpoint, found in the old spaCy output folder
└── spacy_deberta_v3_large/
    ├── model-best/                          # spaCy pipeline (transformer + tagger + parser + ner), ents_f = 0.970
    └── model-last/
```

## Loading the DeBERTa-v3-large checkpoint

```python
import torch
from transformers import AutoTokenizer, AutoModelForTokenClassification

# Same map as the training notebooks: "O" tokens are set to -100 and ignored in the loss,
# so the classifier has 7 entity classes.
label_map = {"QUANTITY": 0, "UNIT": 1, "NAME": 2, "FORM": 3, "STATE": 4, "DF": 5, "SIZE": 6}
id2label = {v: k for k, v in label_map.items()}

tokenizer = AutoTokenizer.from_pretrained("microsoft/deberta-v3-large")
model = AutoModelForTokenClassification.from_pretrained(
    "microsoft/deberta-v3-large", num_labels=len(label_map), id2label=id2label, label2id=label_map
)
ckpt = torch.load("models/deberta_v3_large/deberta_v3_large_ner.pth", map_location="cpu")
model.load_state_dict(ckpt["model_state_dict"])
model.eval()
```

> If `load_state_dict` complains about the classifier shape, check how the model was built in
> `notebooks/02_benchmark_original/DeBERTa_v3_large_final.ipynb` and mirror it exactly.

## Loading the spaCy pipeline

```python
import spacy
nlp = spacy.load("models/spacy_deberta_v3_large/model-best")
doc = nlp("2 cups finely chopped fresh parsley")
print([(ent.text, ent.label_) for ent in doc.ents])
```

## Getting the weights

If you do not have the weights locally, contact the authors (see the main README) or check the
project's Hugging Face page once they are released.
