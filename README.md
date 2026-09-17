# bert-adversarial-audit

White-box and black-box security evaluation of a DistilBERT sentiment classifier (SST-2).

## Overview

Full adversarial audit of a fine-tuned DistilBERT model, covering both white-box attacks (gradient access) and black-box attacks (API-only access). The goal is to evaluate model robustness, demonstrate model stealing, and test adversarial transferability.

## Attack surface

### White-box

| Attack | Method | Accuracy drop |
|--------|--------|---------------|
| FGSM | Single-step gradient perturbation on embeddings | 91.3% → 25% (ε=0.15) |
| PGD | Iterative gradient perturbation with projection | 91.3% → 5% (ε=0.05) |

### Black-box

| Attack | Method | Result |
|--------|--------|--------|
| Model stealing | Surrogate trained on 2,000 oracle queries | 93.46% agreement, KL ≈ 0.17 |
| Word deletion | Single token removal via surrogate | 26.67% success, 87.5% transfer rate |
| Symbol insertion | Decorative Unicode appended | 0% success (model robust) |
| Contextual substitution | MLM-guided synonym replacement | 13.33% surrogate, 3.33% oracle |
| Membership inference | Logistic regression on confidence features | AUC = 0.57 (weak signal) |

## Key findings

- **PGD is devastating in white-box**: accuracy drops to ~5% with ε=0.05, outperforming FGSM significantly
- **Model stealing works with minimal queries**: 2,000 API calls produce a surrogate that matches 93.46% of the oracle's decisions and closely replicates its confidence distribution
- **Adversarial transfer is real**: 87.5% of successful surrogate attacks also fool the original model
- **Opacity is not security**: a closed API does not prevent reconstruction of the model's decision boundary
- **Robustness is multi-dimensional**: the model resists superficial noise (symbols) but is sensitive to sentiment-pivot tokens

## Stack

- Python, PyTorch, Hugging Face Transformers
- DistilBERT (base-uncased), SST-2 (GLUE)
- scikit-learn (membership inference classifier)
- Google Colab (GPU runtime)

## Project structure

```
├── white_box_attacks.ipynb    # FGSM & PGD on embeddings
├── black_box_attacks.ipynb    # Model stealing, word attacks, MIA
├── audit_report.pdf           # Full 39-page technical report
└── README.md
```


