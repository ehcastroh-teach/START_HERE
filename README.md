<div align="center">
  <img src="images/thumbnails/ehcastroh_teach_banner_flowers_sparse.png" alt="ehcastroh-teach landing" width="800"/>
</div>

# Learning Paths for AI, Machine Learning, and Data Science

<sub><strong>ehcastroh-teach</strong> · publish-quality, self-contained tutorials for people learning on their own</sub>

Every repo bundles lesson notebooks, paired homework with hints and solutions, and a runnable reference implementation. Pick a path below, or browse the index by track.

---

## Getting started

1. **Pick a suggested path** or browse the index below for the topic you want.
2. Each repo's README covers prerequisites, setup, and a suggested reading order.
3. Notebooks run top-to-bottom from a clean environment using the provided `requirements.txt` (or `environment.yml`).
4. Homework notebooks are offline-runnable - no API key required - so you can practice before installing anything heavy.

**Level key:** `BEG` = no library prerequisites, `INT` = requires prior library or domain knowledge, `ADV` = requires significant knowledge across multiple areas.

---

## Suggested paths

Three curated sequences for the most common starting points. Each repo builds on the previous.

**New to Python data work**
`NumPy_Introduction` → `Pandas_Data_Analysis` → `Linear_Regression` → `Logistic_Regression` → `Precision_Recall` → `Ensemble_Models`

**Building LLM applications**
`MCP_ArXiv_Chatbot` → `EDD_Agent_Evals` → `Local_Llama` → Resources: [Prompt Engineering](Resources/Agent%20Best%20Practices/01-prompt-engineering.md), [Tools](Resources/Agent%20Best%20Practices/02-tools.md), [MCP](Resources/Agent%20Best%20Practices/06-mcp.md)

**Learning R for data analysis**
`Forecasting_Using_R` → `Price_Prediction_Using_R`

**Practicing for an ML interview**
`Story_Based_E2E_Interview_Prep` → `Hypothesis_Based_E2E_Interview_Prep`

---

## Index

<details open>
<summary><strong>Foundations</strong> &nbsp;·&nbsp; 2 repos</summary>

<br>

<table>
<tr>
  <th align="left">Repository</th>
  <th align="left">Topic</th>
  <th align="left">Level</th>
  <th align="left">Prerequisites</th>
  <th align="left">Keywords</th>
</tr>
<tr>
  <td><a href="https://github.com/ehcastroh-teach/NumPy_Introduction">NumPy_Introduction</a></td>
  <td>NumPy fundamentals - ndarray creation, slicing, boolean masks, vectorized operations, broadcasting, and a linear regression example</td>
  <td>BEG</td>
  <td>Python basics</td>
  <td><kbd>numpy</kbd> <kbd>arrays</kbd> <kbd>vectorization</kbd> <kbd>broadcasting</kbd> <kbd>linear-algebra</kbd></td>
</tr>
<tr>
  <td><a href="https://github.com/ehcastroh-teach/Pandas_Data_Analysis">Pandas_Data_Analysis</a></td>
  <td>Pandas fundamentals - DataFrame creation, data selection and slicing, CSV and Excel I/O, and exploratory analysis with groupby and describe</td>
  <td>BEG</td>
  <td>Python basics, NumPy basics</td>
  <td><kbd>pandas</kbd> <kbd>dataframes</kbd> <kbd>eda</kbd> <kbd>csv</kbd> <kbd>groupby</kbd></td>
</tr>
</table>

</details>

<details open>
<summary><strong>Data Science & Analysis</strong> &nbsp;·&nbsp; 4 repos</summary>

<br>

<table>
<tr>
  <th align="left">Repository</th>
  <th align="left">Topic</th>
  <th align="left">Level</th>
  <th align="left">Prerequisites</th>
  <th align="left">Keywords</th>
</tr>
<tr>
  <td><a href="https://github.com/ehcastroh-teach/Linear_Regression">Linear_Regression</a></td>
  <td>Linear regression from first principles - computing expected value, covariance, and variance by hand, then deriving single-variable and multi-variable predictors using the normal equation on a real estate dataset</td>
  <td>BEG</td>
  <td>Python basics, NumPy basics</td>
  <td><kbd>linear-regression</kbd> <kbd>numpy</kbd> <kbd>statistics</kbd> <kbd>covariance</kbd> <kbd>normal-equation</kbd></td>
</tr>
<tr>
  <td><a href="https://github.com/ehcastroh-teach/Forecasting_Using_R">Forecasting_Using_R</a></td>
  <td>Time series forecasting with linear regression - feature selection, VIF, seasonality, and out-of-sample evaluation applied to automotive sales</td>
  <td>INT</td>
  <td>R basics, statistics fundamentals</td>
  <td><kbd>r</kbd> <kbd>time-series</kbd> <kbd>forecasting</kbd> <kbd>regression</kbd> <kbd>seasonality</kbd></td>
</tr>
<tr>
  <td><a href="https://github.com/ehcastroh-teach/Data_Checks">Data_Checks</a></td>
  <td>Dataset inspection for ML fairness - measuring demographic diversity, counting subgroup representation across gender, race, and age, and scoring image quality using BRISQUE on two facial image datasets</td>
  <td>INT</td>
  <td>Python basics, pandas basics</td>
  <td><kbd>dataset-audit</kbd> <kbd>fairness</kbd> <kbd>brisque</kbd> <kbd>pandas</kbd> <kbd>image-quality</kbd></td>
</tr>
<tr>
  <td><a href="https://github.com/ehcastroh-teach/Signals_Correlation">Signals_Correlation</a></td>
  <td>Introduction to correlation analysis in Python - Pearson, Spearman, and Kendall-tau with NumPy, Pandas, and SciPy.</td>
  <td>INT</td>
  <td>NumPy_Introduction, Pandas_Data_Analysis</td>
  <td><kbd>correlation</kbd> <kbd>pearson</kbd> <kbd>spearman</kbd> <kbd>kendall</kbd> <kbd>statistics</kbd></td>
</tr>
</table>

</details>

<details open>
<summary><strong>Machine Learning & Deep Learning</strong> &nbsp;·&nbsp; 13 repos</summary>

<br>

<table>
<tr>
  <th align="left">Repository</th>
  <th align="left">Topic</th>
  <th align="left">Level</th>
  <th align="left">Prerequisites</th>
  <th align="left">Keywords</th>
</tr>
<tr>
  <td><a href="https://github.com/ehcastroh-teach/Logistic_Regression">Logistic_Regression</a></td>
  <td>Multi-class logistic regression with scikit-learn - One-vs-Rest and Softmax strategies, Min-Max and Standard feature scaling, confusion matrices, precision, and recall applied to building energy load and handwritten digit classification</td>
  <td>BEG</td>
  <td>Python basics, NumPy basics, pandas basics</td>
  <td><kbd>logistic-regression</kbd> <kbd>classification</kbd> <kbd>scikit-learn</kbd> <kbd>confusion-matrix</kbd> <kbd>feature-scaling</kbd></td>
</tr>
<tr>
  <td><a href="https://github.com/ehcastroh-teach/Precision_Recall">Precision_Recall</a></td>
  <td>Precision, recall, and threshold selection for binary classifiers - two case studies with logistic regression and SVM</td>
  <td>INT</td>
  <td>Python basics, pandas, introductory supervised learning</td>
  <td><kbd>precision</kbd> <kbd>recall</kbd> <kbd>threshold</kbd> <kbd>scikit-learn</kbd> <kbd>svm</kbd></td>
</tr>
<tr>
  <td><a href="https://github.com/ehcastroh-teach/Prediction">Prediction</a></td>
  <td>Build regression models to predict housing prices using property features and economic indicators</td>
  <td>INT</td>
  <td>Python basics, pandas, numpy fundamentals</td>
  <td><kbd>regression</kbd> <kbd>machine-learning</kbd> <kbd>time-series</kbd> <kbd>arima</kbd> <kbd>random-forest</kbd></td>
</tr>
<tr>
  <td><a href="https://github.com/ehcastroh-teach/Ensemble_Models">Ensemble_Models</a></td>
  <td>Ensemble learning and AutoML on the Titanic dataset using LightAutoML and FlaML</td>
  <td>INT</td>
  <td>pandas basics, binary classification concepts</td>
  <td><kbd>ensemble-learning</kbd> <kbd>automl</kbd> <kbd>feature-engineering</kbd> <kbd>lightautoml</kbd> <kbd>flaml</kbd></td>
</tr>
<tr>
  <td><a href="https://github.com/ehcastroh-teach/ML_Practices">ML_Practices</a></td>
  <td>Build and tune image classifiers, combine model predictions with ensemble selection, and apply Bayesian optimization</td>
  <td>INT</td>
  <td>supervised learning, scikit-learn</td>
  <td><kbd>cnn</kbd> <kbd>ensemble-methods</kbd> <kbd>hyperparameter-optimization</kbd> <kbd>bayesian-optimization</kbd> <kbd>random-search</kbd></td>
</tr>
<tr>
  <td><a href="https://github.com/ehcastroh-teach/TensorFlow_Introduction">TensorFlow_Introduction</a></td>
  <td>TensorFlow V.2 fundamentals - tensors, operations, automatic differentiation with GradientTape, @tf.function graph compilation, and TensorBoard setup</td>
  <td>INT</td>
  <td>Python basics, NumPy basics, pandas basics, linear algebra</td>
  <td><kbd>tensorflow</kbd> <kbd>deep-learning</kbd> <kbd>gradienttape</kbd> <kbd>gpu</kbd> <kbd>tensorboard</kbd></td>
</tr>
<tr>
  <td><a href="https://github.com/ehcastroh-teach/Ludwig_Regression_and_Classification">Ludwig_Regression_and_Classification</a></td>
  <td>Deep learning regression and classification with Ludwig - sequential models, EarlyStopping, TensorBoard, tabular regression, and image classification</td>
  <td>INT</td>
  <td>Python basics, NumPy basics, TensorFlow basics</td>
  <td><kbd>ludwig</kbd> <kbd>deep-learning</kbd> <kbd>classification</kbd> <kbd>regression</kbd> <kbd>tensorboard</kbd></td>
</tr>
<tr>
  <td><a href="https://github.com/ehcastroh-teach/Ludwig_Regularization_and_Crossvalidation">Ludwig_Regularization_and_Crossvalidation</a></td>
  <td>Cross-validation and regularization for NLP with Ludwig - RNN, CNN, and BERT encoder comparison on the Stanford Sentiment Treebank</td>
  <td>INT</td>
  <td>Python basics, machine learning basics, Ludwig basics</td>
  <td><kbd>ludwig</kbd> <kbd>nlp</kbd> <kbd>cross-validation</kbd> <kbd>regularization</kbd> <kbd>bert</kbd></td>
</tr>
<tr>
  <td><a href="https://github.com/ehcastroh-teach/Price_Prediction_Using_R">Price_Prediction_Using_R</a></td>
  <td>Applied ML and statistical prediction - Ridge, LASSO, Random Forest, and ARIMA models applied to Ames housing prices</td>
  <td>INT</td>
  <td>R basics, statistics fundamentals</td>
  <td><kbd>r</kbd> <kbd>lasso</kbd> <kbd>random-forest</kbd> <kbd>arima</kbd> <kbd>regression</kbd></td>
</tr>
<tr>
  <td><a href="https://github.com/ehcastroh-teach/Story_Based_E2E_Interview_Prep">Story_Based_E2E_Interview_Prep</a></td>
  <td>End-to-end data science pipeline (capstone) - EDA, feature engineering, SQLite storage, and multi-model comparison using the Titanic dataset</td>
  <td>INT</td>
  <td>Python basics, pandas basics</td>
  <td><kbd>capstone</kbd> <kbd>eda</kbd> <kbd>feature-engineering</kbd> <kbd>sqlite</kbd> <kbd>classification</kbd></td>
</tr>
<tr>
  <td><a href="https://github.com/ehcastroh-teach/Hypothesis_Based_E2E_Interview_Prep">Hypothesis_Based_E2E_Interview_Prep</a></td>
  <td>Hypothesis-driven ML pipeline (capstone) - EDA, statistical testing (Mann-Whitney U, Chi-squared), feature engineering, and six-classifier comparison</td>
  <td>INT</td>
  <td>Python basics, pandas basics, statistics fundamentals</td>
  <td><kbd>capstone</kbd> <kbd>hypothesis-testing</kbd> <kbd>eda</kbd> <kbd>classification</kbd> <kbd>credit-risk</kbd></td>
</tr>
<tr>
  <td><a href="https://github.com/ehcastroh-teach/PyTorch_Introduction">PyTorch_Introduction</a></td>
  <td>Hands-on introduction to PyTorch tensors, autograd, and nn.Module for building and training neural networks.</td>
  <td>INT</td>
  <td>Python, NumPy, Linear Algebra</td>
  <td><kbd>pytorch</kbd> <kbd>autograd</kbd> <kbd>nn-module</kbd> <kbd>tensors</kbd> <kbd>deep-learning</kbd></td>
</tr>
<tr>
  <td><a href="https://github.com/ehcastroh-teach/Synthetic_Data_Generation">Synthetic_Data_Generation</a></td>
  <td>Generate synthetic tabular data from seed CSVs and sklearn generators; apply classification and regression to generated datasets.</td>
  <td>INT</td>
  <td>Python, pandas, scikit-learn</td>
  <td><kbd>synthetic data</kbd> <kbd>classification</kbd> <kbd>regression</kbd> <kbd>clustering</kbd> <kbd>pandas</kbd></td>
</tr>
</table>

</details>

<details>
<summary><strong>Data Visualization</strong> &nbsp;·&nbsp; 3 repos</summary>

<br>

<table>
<tr>
  <th align="left">Repository</th>
  <th align="left">Topic</th>
  <th align="left">Level</th>
  <th align="left">Prerequisites</th>
  <th align="left">Keywords</th>
</tr>
<tr>
  <td><a href="https://github.com/ehcastroh-teach/Visual_Types_and_Data_Visualization">Visual_Types_and_Data_Visualization</a></td>
  <td>Data analysis and visualization - Matplotlib, Seaborn, Plotnine, Plotly, and Altair grounded in Grammar of Graphics and Tufte's design principles</td>
  <td>INT</td>
  <td>Python basics</td>
  <td><kbd>matplotlib</kbd> <kbd>seaborn</kbd> <kbd>plotly</kbd> <kbd>grammar-of-graphics</kbd> <kbd>visualization</kbd></td>
</tr>
<tr>
  <td><a href="https://github.com/ehcastroh-teach/Visualizations_and_Visual_Storytelling">Visualizations_and_Visual_Storytelling</a></td>
  <td>Data visualization from first principles - encoding theory, Grammar of Graphics, Matplotlib, Seaborn, Plotly, Altair, Tableau, and Dash</td>
  <td>INT</td>
  <td>Python basics</td>
  <td><kbd>visualization</kbd> <kbd>storytelling</kbd> <kbd>grammar-of-graphics</kbd> <kbd>tableau</kbd> <kbd>dash</kbd></td>
</tr>
<tr>
  <td><a href="https://github.com/ehcastroh-teach/Radial_Plots_for_Matplotlib">Radial_Plots_for_Matplotlib</a></td>
  <td>Build radar (spider) charts in Matplotlib - from a basic polar subplot to a custom polygon-framed RadarAxes projection</td>
  <td>INT</td>
  <td>Matplotlib basics, Pandas DataFrames</td>
  <td><kbd>radar-chart</kbd> <kbd>matplotlib</kbd> <kbd>polar-axes</kbd> <kbd>visualization</kbd> <kbd>spider-chart</kbd></td>
</tr>
</table>

</details>

<details>
<summary><strong>AI Tooling & Environment</strong> &nbsp;·&nbsp; 2 repos</summary>

<br>

<table>
<tr>
  <th align="left">Repository</th>
  <th align="left">Topic</th>
  <th align="left">Level</th>
  <th align="left">Prerequisites</th>
  <th align="left">Keywords</th>
</tr>
<tr>
  <td><a href="https://github.com/ehcastroh-teach/Herdr_AI_Dev_Environment">Herdr_AI_Dev_Environment</a></td>
  <td>AI-assisted development environment and workflow - Neovim, agentic coding tools, and declarative version-controlled configuration</td>
  <td>INT</td>
  <td>Linux CLI</td>
  <td><kbd>ai-dev</kbd> <kbd>neovim</kbd> <kbd>agentic-coding</kbd> <kbd>environment</kbd> <kbd>nixos</kbd></td>
</tr>
<tr>
  <td><a href="https://github.com/ehcastroh-teach/Local_Llama">Local_Llama</a></td>
  <td>Local LLM inference and GPU-accelerated serving - llama.cpp setup, CUDA configuration, quantization, and an OpenAI-compatible endpoint</td>
  <td>INT</td>
  <td>Linux CLI</td>
  <td><kbd>llm</kbd> <kbd>llama-cpp</kbd> <kbd>local-inference</kbd> <kbd>gpu</kbd> <kbd>quantization</kbd></td>
</tr>
</table>

</details>

<details>
<summary><strong>Applied AI & Generative AI</strong> &nbsp;·&nbsp; 2 repos</summary>

<br>

<table>
<tr>
  <th align="left">Repository</th>
  <th align="left">Topic</th>
  <th align="left">Level</th>
  <th align="left">Prerequisites</th>
  <th align="left">Keywords</th>
</tr>
<tr>
  <td><a href="https://github.com/ehcastroh-teach/MCP_ArXiv_Chatbot">MCP_ArXiv_Chatbot</a></td>
  <td>Build LLM tool-calling applications with the Model Context Protocol - arXiv paper-search chatbot end-to-end, with schemas, dispatcher, and async/streaming/FastAPI variants</td>
  <td>BEG</td>
  <td>Python basics, JSON basics</td>
  <td><kbd>mcp</kbd> <kbd>tool-calling</kbd> <kbd>llm</kbd> <kbd>anthropic-api</kbd> <kbd>arxiv</kbd></td>
</tr>
<tr>
  <td><a href="https://github.com/ehcastroh-teach/EDD_Agent_Evals">EDD_Agent_Evals</a></td>
  <td>EDD for LLM agents - trace with Arize Phoenix, score with LLM-as-judge, and make targeted prompt improvements driven by eval results</td>
  <td>INT</td>
  <td>Python basics, OpenAI API basics</td>
  <td><kbd>agent-evals</kbd> <kbd>llm</kbd> <kbd>openai</kbd> <kbd>arize-phoenix</kbd> <kbd>edd</kbd></td>
</tr>
</table>

</details>

<details>
<summary><strong>Web Development</strong> &nbsp;·&nbsp; 1 repo</summary>

<br>

<table>
<tr>
  <th align="left">Repository</th>
  <th align="left">Topic</th>
  <th align="left">Level</th>
  <th align="left">Prerequisites</th>
  <th align="left">Keywords</th>
</tr>
<tr>
  <td><a href="https://github.com/ehcastroh-teach/Flask">Flask</a></td>
  <td>Flask web development - environment setup, routing, Jinja2 templating, SQLAlchemy models, form validation, and cloud deployment</td>
  <td>INT</td>
  <td>Python basics, HTML, CSS</td>
  <td><kbd>flask</kbd> <kbd>web-dev</kbd> <kbd>routing</kbd> <kbd>sqlalchemy</kbd> <kbd>jinja2</kbd></td>
</tr>
</table>

</details>

*This index is updated as repositories are added to the organization.*

---

## Resources

Reference documents that support the material across repos. Grouped by category; the Agent Best Practices series is numbered in suggested reading order.

### Reference and Cheat Sheets

| Document | Description |
|:---|:---|
| [Agents, Bash, and the CLI](Resources/Reference%20%26%20Cheat%20Sheets/agent-bash-cli-reference.md) | Working reference for building and operating AI agents through a shell - command lookup by task, hardened script patterns, output discipline, exit-code semantics, and the safety model. |

### Agent Best Practices - Quick Reference

| Document | Description |
|:---|:---|
| [Agent Best Practices Summary](Resources/Agent%20Best%20Practices/00-agent-best-practices-summary.md) | Condensed best practices for building, combining, and reviewing agent tools, skills, hooks, harnesses, retrieval systems, sandboxes, and evals - two parts covering per-component rules and cross-stack synergies. Paste into a session or reference directly; not a tutorial. |

### Agent Best Practices - Fundamentals

| # | Document | Description |
|:---:|:---|:---|
| 01 | [Prompt Engineering](Resources/Agent%20Best%20Practices/01-prompt-engineering.md) | Personal reference notes on prompt engineering best practices - scope, structure, and patterns drawn from Anthropic and Google documentation. |
| 02 | [Agent Tools](Resources/Agent%20Best%20Practices/02-tools.md) | Reference notes on designing and using tools with AI agents - when to build a tool, how to write effective tool descriptions, and function-calling patterns. |
| 03 | [Agent Skills](Resources/Agent%20Best%20Practices/03-skills.md) | Reference notes on agent skills - what a skill is, how to author and package them, security considerations, and the open agentskills standard. |
| 04 | [Context Engineering](Resources/Agent%20Best%20Practices/04-context-engineering.md) | Reference notes on context engineering for AI agents - managing what goes into the context window, effective structuring, and the distinction from prompt engineering. |
| 05 | [RAG](Resources/Agent%20Best%20Practices/05-rag.md) | Reference notes on retrieval-augmented generation - contextual retrieval, chunking strategies, and when RAG fits versus other context-injection approaches. |
| 06 | [MCP (Model Context Protocol)](Resources/Agent%20Best%20Practices/06-mcp.md) | Reference notes on the Model Context Protocol - server features, tool and resource exposure, authorization, and practical integration patterns. |

### Agent Best Practices - Systems

| # | Document | Description |
|:---:|:---|:---|
| 07 | [Harness Engineering](Resources/Agent%20Best%20Practices/07-harness-engineering.md) | Reference notes on agent harness design - the definition of harness, three architectural patterns, and lessons from long-running agent deployments. |
| 09 | [Agent Sandboxing](Resources/Agent%20Best%20Practices/09-agent-sandboxing.md) | Reference notes on containing agent execution - sandboxing layers, isolation primitives (namespaces, microVMs), and the distinction from filesystem permissions. |
| 10 | [Loop Engineering](Resources/Agent%20Best%20Practices/10-loop-engineering.md) | Reference notes on agentic loop design - ReAct and Reflexion patterns, loop termination, and how the loop sits inside a larger harness. |
| 11 | [Hooks](Resources/Agent%20Best%20Practices/11-hooks.md) | Reference notes on Claude Code hooks - what a hook is, event types, and how hooks integrate into the agent execution cycle. |
| 12 | [Sandboxing](Resources/Agent%20Best%20Practices/12-sandboxing.md) | Reference notes on OS-level sandboxing primitives - Linux namespaces, Nix sandbox configuration, and comparison of gVisor and Firecracker. |
| 14 | [Agent Orchestration](Resources/Agent%20Best%20Practices/14-agent-orchestration.md) | Reference notes on multi-agent orchestration - task decomposition, parallel execution, subagent coordination, and lessons from Anthropic's multi-agent research system. |

### Agent Best Practices - Applied

| # | Document | Description |
|:---:|:---|:---|
| 08 | [Running LLMs Locally](Resources/Agent%20Best%20Practices/08-running-llms-locally.md) | Reference notes on local LLM inference - llama.cpp with CUDA on NixOS, quantization, function-calling, and serving options including vLLM and Ray Serve. |
| 13 | [Evals](Resources/Agent%20Best%20Practices/13-evals.md) | Reference notes on AI evaluation - the distinction between tests and evals, layered evaluation strategies, and RAGAS metrics for RAG pipelines. |

---

## Repository conventions

Every repo in this org follows the same shape so that a learner who has completed one can navigate the next without hunting.

- **Naming** - repos use `Topic_Snake_Case` with capitalized words separated by underscores (e.g. `NumPy_Introduction`, `MCP_ArXiv_Chatbot`, `Ludwig_Regression_and_Classification`). No hyphens in repo names or filenames.
- **File naming** - notebooks, scripts, and datasets inside a repo use `lowercase_snake_case` (e.g. `01_intro_numerical_analysis_using_numpy.ipynb`, `arxiv_chatbot.py`). Sequential material carries a numeric prefix (`01_`, `02_`, `03_`) matching the README's suggested reading order.
- **Structure** - most notebook-based repos include:
  - Numbered `.ipynb` files at the root - the lesson notebooks
  - `homework/` - paired homework notebooks with hints and solutions, offline-runnable
  - `examples/` (where applicable) - production-shaped variants of the reference code
  - `assets/` or `images/` - diagrams, thumbnails, and figures used inside the notebooks
  - `README.md` - overview, learning objectives, file dictionary, walkthrough, run instructions, glossary, credits, contact
- **Environments** - Python repos ship a `requirements.txt`; R repos ship an `renv.lock` or an equivalent setup note.
- **Licensing** - teaching content is shared under an open license per-repo (see individual `LICENSE` files); check before reuse or redistribution.

Some repos are setup and infrastructure guides rather than notebook collections. They follow the same README conventions but ship reference config files in place of notebooks and homework. `Herdr_AI_Dev_Environment` and `Local_Llama` are the current examples.

---

## Contact and feedback

These materials are actively maintained. Bug reports and suggestions belong on the relevant repository's issue tracker. For anything about the organization itself - new topics, collaboration, general feedback - reach out via LinkedIn or GitHub below.

<div align="center">
  <img src="images/thumbnails/ehcastroh_teach_banner_flower.png" alt="ehcastroh" width="90" style="border-radius: 50%;" />

  <sub>ehcastroh</sub>

  <a href="https://github.com/ehcastroh">GitHub</a> · <a href="https://www.linkedin.com/in/ehcastroh/">LinkedIn</a>
</div>

<br>

<sub>Last updated: 2026-09-02</sub>
