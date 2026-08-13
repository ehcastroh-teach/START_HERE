<div align="center">
  <img src="images/thumbnails/ehcastroh_teach_banner_flowers_sparse.png" alt="ehcastroh-teach landing" width="800"/>
</div>

# ehcastroh-teach

**Teaching materials, notebooks, and reference repos for AI, Machine Learning, Applied Data Science and Data Engineering topics.**

---

## Getting started

1. Browse the repository list for the topic you're interested in.
2. Each repo's own README covers prerequisites, setup, and suggested order for working through the material.
3. Notebooks are designed to run top-to-bottom; start from a clean environment using the provided dependency file.

## Index of repositories

<table>
<tr><th align="left">Repository</th><th align="left">Topic</th><th align="left">Level</th><th align="left">Prerequisites</th></tr>
<tr><td><a href="https://github.com/ehcastroh-teach/Herdr_AI_Dev_Environment">Herdr_AI_Dev_Environment</a></td><td>AI-assisted development environment &amp; workflow</td><td>Intermediate</td><td>Linux CLI</td></tr>
<tr><td><a href="https://github.com/ehcastroh-teach/Local_Llama">Local_Llama</a></td><td>Local LLM inference &amp; GPU-accelerated serving</td><td>Intermediate</td><td>Linux CLI</td></tr>
<tr><td><a href="https://github.com/ehcastroh-teach/Forecasting_Using_R">Forecasting_Using_R</a></td><td>Time series forecasting with linear regression - feature selection, VIF, seasonality, and out-of-sample evaluation applied to automotive sales data</td><td>Introductory - Intermediate</td><td>R basics, statistics fundamentals</td></tr>
<tr><td><a href="https://github.com/ehcastroh-teach/Price_Prediction_Using_R">Price_Prediction_Using_R</a></td><td>Applied ML &amp; statistical prediction - cross-sectional models (Ridge, LASSO, Random Forest, Deep Learning) and time series (AR, ARIMA, SARIMA) applied to Ames housing prices</td><td>Intermediate</td><td>R basics, statistics fundamentals</td></tr>
<tr><td><a href="https://github.com/ehcastroh-teach/Visual_Types_and_Data_Visualization">Visual_Types_and_Data_Visualization</a></td><td>Applied data analysis &amp; visualization - Matplotlib, Seaborn, Plotnine, Plotly, and Altair grounded in the Grammar of Graphics and Tufte's design principles</td><td>Beginner - Intermediate</td><td>Python basics</td></tr>
<tr><td><a href="https://github.com/ehcastroh-teach/Visualizations_and_Visual_Storytelling">Visualizations_and_Visual_Storytelling</a></td><td>Data visualization from first principles - encoding theory, Grammar of Graphics, and hands-on notebooks in Matplotlib, Seaborn, Plotly, and Altair, with a Tableau and Dash section for zero-code and hosted dashboards</td><td>Introductory - Intermediate</td><td>Python basics</td></tr>
<tr><td><a href="https://github.com/ehcastroh-teach/Radial-Plots-for-Matplotlib">Radial-Plots-for-Matplotlib</a></td><td>Radar (spider) chart construction in Python - building polar subplots and a custom polygon-framed RadarAxes projection using Matplotlib</td><td>Introductory - Intermediate</td><td>Python basics, Matplotlib basics</td></tr>
<tr><td><a href="https://github.com/ehcastroh-teach/Story_Based_E2E_Interview_Prep">Story_Based_E2E_Interview_Prep</a></td><td>End-to-end data science pipeline - EDA, feature engineering, SQLite storage, and multi-model comparison using the Titanic dataset</td><td>Introductory - Intermediate</td><td>Python basics, pandas basics</td></tr>
<tr><td><em>tbd</em></td><td>Foundations of machine learning</td><td>Beginner</td><td>None</td></tr>
<tr><td><em>tbd</em></td><td>Neural networks &amp; deep learning</td><td>Intermediate</td><td>ML foundations</td></tr>
<tr><td><em>tbd</em></td><td>Transformers &amp; attention</td><td>Intermediate</td><td>Neural networks</td></tr>
</table>

*This table is updated as repositories are added to the organization. Remaining `tbd` entries are placeholders for repos in progress.*

---

## Environment & tooling track

Two repos cover the practical side of working with AI systems - setting up the machine, the models, and the workflow before any modeling work begins. They're designed to be read together: `Herdr_AI_Dev_Environment` sets up where you work, `Local_Llama` sets up what you run.

### [Herdr_AI_Dev_Environment](https://github.com/ehcastroh-teach/Herdr_AI_Dev_Environment)

**A reproducible development environment for AI-assisted engineering work.**

Covers configuring a working setup around [Herdr](https://github.com/ehcastroh-teach/Herdr_AI_Dev_Environment), Neovim, and agentic coding tools - with configuration kept declarative and version-controlled rather than accumulated by hand.

**Prerequisites:** comfort with a Linux command line and a terminal-based editor.

> **TODO - fill in from the repo:** add a short "What's inside" bullet list here mirroring the Local_Llama section below (3-5 items), and a one-line "why it's worth reading even if you don't use this exact stack" note. I've kept this section deliberately thin since I don't have the repo contents.

### [Local_Llama](https://github.com/ehcastroh-teach/Local_Llama)

**Running large language models on your own hardware, declaratively.**

A start-to-finish guide for setting up GPU-accelerated [llama.cpp](https://github.com/ggml-org/llama.cpp) on NixOS, with every dependency declared in version-controlled config rather than installed ad hoc. Covers the full path from a bare driver check to a working OpenAI-compatible inference endpoint.

**What's inside**

- Step-by-step tutorial with an expected-output check after every step, so you can confirm progress instead of guessing
- Drop-in Nix modules for llama.cpp (CUDA), `huggingface-cli`, and an optional systemd service
- Guidance on choosing quantization levels against your available VRAM
- A troubleshooting table mapping real error messages to fixes
- The reasoning behind llama.cpp vs. Ollama vs. vLLM for single-GPU use

**Why it's worth reading even if you don't use NixOS:** the model-selection, quantization, VRAM-budgeting, and GPU-verification material transfers to any local inference setup. The declarative-config discipline is the part that's NixOS-specific.

**Prerequisites:** comfort with a Linux command line. An NVIDIA GPU is assumed for the CUDA path, though the modules note ROCm, Vulkan, and Metal alternatives.

---

## Resources

Reference documents that support the material across repos in this org.

| Document | Description |
|:---|:---|
| [Agents, Bash, and the CLI](Resources/Reference%20%26%20Cheat%20Sheets/agent-bash-cli-reference.md) | A working reference for building and operating AI agents through a shell - command lookup by task, hardened script patterns, output discipline, exit-code semantics, and the safety model. |
| [Prompt Engineering](Resources/Agent%20Best%20Practices/01-prompt-engineering.md) | Personal reference notes on prompt engineering best practices - scope, structure, and patterns drawn from Anthropic and Google documentation. |
| [Tools](Resources/Agent%20Best%20Practices/02-tools.md) | Reference notes on designing and using tools with AI agents - when to build a tool, how to write effective tool descriptions, and function-calling patterns. |

---

## Repository conventions

To keep things predictable across repos in this org:

- **Naming** - repos follow `topic-short-name` (e.g. `neural-nets-intro`, `transformer-attention`).
- **Structure** - most repos include:
  - `notebooks/` - walkthroughs and exercises (Jupyter)
  - `data/` - sample or synthetic datasets used in the material
  - `solutions/` - worked solutions, kept separate from exercises
  - `README.md` - topic overview, prerequisites, and setup instructions
- **Environments** - Python-based repos include a `requirements.txt` or `environment.yml` for reproducibility.
- **Licensing** - teaching content is shared under an open license per-repo (see individual repo `LICENSE` files); check before reuse or redistribution.

Some repos are setup and infrastructure guides rather than notebook collections. These follow the same README conventions but ship reference config files in place of `notebooks/` and `data/` - `Herdr_AI_Dev_Environment` and `Local_Llama` are the current examples.

## Contributing / feedback

These materials are actively maintained. If you spot an error or have a suggestion, open an issue on the relevant repository.

Questions about a specific repo belong on that repo's issue tracker. For anything about the organization itself - new topics, collaboration, or general feedback - reach out via LinkedIn or GitHub below.

---

## Contact

<div align="center">
  <img src="images/thumbnails/ehcastroh_teach_banner_flower.png" alt="ehcastroh" width="90" style="border-radius: 50%;" />

  <sub>ehcastroh</sub>

  <a href="https://github.com/ehcastroh">GitHub</a> · <a href="https://www.linkedin.com/in/ehcastroh/">LinkedIn</a>
</div>
