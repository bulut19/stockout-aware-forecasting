## References

1. Wang, Y., Gu, J., Long, L. et al. (2025). *FreshRetailNet-50K: A Stockout-Annotated Censored Demand Dataset for Latent Demand Recovery and Forecasting in Fresh Retail.* [arXiv:2505.16319](https://arxiv.org/abs/2505.16319). Dataset: [Hugging Face](https://huggingface.co/datasets/Dingdong-Inc/FreshRetailNet-50K).
2. Ansari, A. F. et al. (2025). *Chronos-2: From Univariate to Universal Forecasting.* [arXiv:2510.15821](https://arxiv.org/abs/2510.15821). Model: [amazon/chronos-2](https://huggingface.co/amazon/chronos-2).
3. Google Research (2026). *TimesFM-3: a zero-shot foundation model for multivariate forecasting.* [Blog post](https://research.google/blog/timesfm-3-a-zero-shot-foundation-model-for-multivariate-forecasting/). Model: [google/timesfm-3.0-pytorch](https://huggingface.co/google/timesfm-3.0-pytorch).
4. Lim, B., Arık, S. Ö., Loeff, N., Pfister, T. (2021). Temporal Fusion Transformers for interpretable multi-horizon time series forecasting. *International Journal of Forecasting*, 37(4).
5. Ke, G. et al. (2017). LightGBM: A highly efficient gradient boosting decision tree. *Advances in Neural Information Processing Systems 30.*
6. Hu, E. J. et al. (2022). LoRA: Low-rank adaptation of large language models. *International Conference on Learning Representations.*
7. Rohlik Group. *Rohlik Sales Forecasting Challenge V2.* [Kaggle](https://www.kaggle.com/competitions/rohlik-sales-forecasting-challenge-v2).

## Project links

- **Code:** [github.com/bulut19/stockout-aware-forecasting](https://github.com/bulut19/stockout-aware-forecasting). Notebooks for every stream are in `notebooks/`, the shared loader and metrics in `src/`, result tables in `results/tables/`.
- **This blog:** the text lives in `docs/sections/` of the same repository.

## Reused code, pretrained models and AI assistance

As the course policy requires, these components come from outside the team:

| Component | Source | How we used it |
|---|---|---|
| Chronos-2 | Amazon, `chronos-forecasting` package | Zero-shot forecasts; LoRA fine-tuning through the package's `fit` method |
| TimesFM-3 | Google, `timesfm` package | Zero-shot forecasts |
| Temporal Fusion Transformer | `pytorch-forecasting` package | TFT model and training loop |
| LightGBM | `lightgbm` package | Gradient-boosted trees for forecasting and gap filling |
| PyTorch | `torch` | LSTM, GRU, transformer and CNN networks written by the team |
| FreshRetailNet-50K, Rohlik data | Dingdong (CC BY 4.0), Rohlik on Kaggle | Data only |

Everything else, including the data pipeline, missed-demand masking test and fillers, forecasting setup, rolling backtest, fair scoring and error bounds, semi-synthetic test, censored loss and ordering simulation, was written by the team.

**AI assistance.** *(Each stream owner should confirm this paragraph before submission.)* We used AI coding assistants (Anthropic's Claude) to help write and debug parts of the code, notebooks and this write-up. The research question, stream design, modeling decisions and interpretation of results are the team's own, and every result reported here was run and checked by team members in our notebooks.
