# Generative AI Applications

## Generative AI for Inference Cluster Operations using an LLM-Based Operations Advisor

This project implements an **LLM-based operations advisor** that generates natural language operational guidance from inference cluster telemetry. The system transforms raw metrics into actionable recommendations for infrastructure operators.

GitHub repository: `https://github.com/sam-andaluri/generative-ai-project`

## 1. Cloning repo

This repo stores compressed dataset files (`.csv.gz`) with **Git LFS**.

Install `git-lfs` cli using https://git-lfs.com or on MacOS `brew install git-lfs`

Run the following commands to get the datafiles. 
```bash
git lfs install
git clone https://github.com/sam-andaluri/generative-ai-project.git
cd generative-ai-project
git lfs pull
gunzip -k data/azure_multimodal_2025/*.gz
```

## 2. Install `uv`

Install `uv` with the standalone installer:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Or install it with Homebrew on macOS:

```bash
brew install uv
```

Confirm the installation:

```bash
uv --version
```

## 3. Prerequisites

Make sure the following tools are available on your system:

- Python 3.8 or higher
- Jupyter with `nbconvert`
- Pandoc for PDF generation

Quick checks:

```bash
python --version
python -m jupyter nbconvert --version
pandoc --version
```

## 4. Create and Activate a Virtual Environment

From the project folder:

```bash
cd generative-ai-project
uv venv .venv
source .venv/bin/activate
```

## 5. Install Dependencies

Install the pinned dependencies from `requirements.txt`:

```bash
uv pip install -r requirements.txt
```

## 6. Setup dot env for LLM inference

Obtain an OpenAI api key and create `.env` file with following contents.

```
LLM_API_URL=https://api.openai.com/v1/chat/completions
LLM_API_KEY="<openai-api-key>"
LLM_MODEL=gpt-4o-mini
LLM_API_FORMAT=auto
LLM_TIMEOUT_SECONDS=60
LLM_MAX_TOKENS=900
```

## 7. Run the Notebook

Open and run `generative_model.ipynb` in Jupyter:

```bash
jupyter notebook generative_model.ipynb
```

Or using JupyterLab:

```bash
jupyter lab generative_model.ipynb
```

Run all cells from top to bottom (Cell > Run All).

## 8. Generate the Report PDF

If you update the report markdown, regenerate the PDF with:

```bash
pandoc Generative_AI_Analysis_Report.md -o Generative_AI_Analysis_Report.pdf
```

## 9. Generate Final requirements.txt

After running the notebook, generate the exact package versions:

```bash
uv pip freeze > requirements.txt
```

## References

Astral. (n.d.). *Installing uv*. uv documentation. Retrieved April 14, 2026, from https://docs.astral.sh/uv/getting-started/installation/

Astral. (n.d.). *Pip interface*. uv documentation. Retrieved April 14, 2026, from https://docs.astral.sh/uv/pip/

Astral. (n.d.). *Using environments*. uv documentation. Retrieved April 14, 2026, from https://docs.astral.sh/uv/pip/environments/

Zhang, J., Agrawal, A., Gandomi, A., et al. (2025). ModServe: Modality- and stage-aware resource disaggregation for scalable multimodal model serving. *ACM Symposium on Cloud Computing (SoCC '25)*.

Vaswani, A., Shazeer, N., Parmar, N., et al. (2017). Attention is all you need. *Advances in Neural Information Processing Systems (NeurIPS)*.

Arjovsky, M., Chintala, S., & Bottou, L. (2017). Wasserstein generative adversarial networks. *International Conference on Machine Learning (ICML)*.

Floridi, L., Cowls, J., Beltrametti, M., et al. (2018). AI4People—An ethical framework for a good AI society. *Minds and Machines*, 28(4), 689-707.

