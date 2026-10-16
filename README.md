# 盐地生金 (Saline-Alkali Land Benchmark)

Welcome to the **Saline-Alkali Land Benchmark**, an open-source evaluation and data processing framework designed to assess the capabilities of Large Language Models (LLMs) in the domain of saline-alkali soil reclamation, agricultural engineering, and ecological restoration.

## Overview

This repository provides comprehensive tools to:
- End-to-end parse and extract structured data from scientific literature (Chinese & English) using MinerU and PDFPlumber.
- Curate, deduplicate, and validate specialized instruction-tuning datasets (e.g., Alpaca and ShareGPT formats).
- Construct targeted Retrieval-Augmented Generation (RAG) evaluation pipelines.
- Generate and automatically evaluate complex multi-objective reclamation plans (e.g., agricultural, chemical, biological, and engineering measures) via automated LLM-as-a-judge comparison (A/B testing).

## Installation and Environment Setup

Follow these steps to set up the environment and install necessary dependencies:

1. **Clone the repository:**
   ```bash
   git clone https://github.com/zuguangh/yan-di-sheng-jin.git
   cd yan-di-sheng-jin
   ```

2. **Create a virtual environment (Recommended):**
   ```bash
   conda create -n saline-benchmark python=3.10 -y
   conda activate saline-benchmark
   ```

3. **Install dependencies:**
   Ensure you have configured your environment for PyTorch if using GPU acceleration, then install the required Python packages:
   ```bash
   pip install -r requirements.txt
   ```

4. **Configure Environment Variables:**
   Create a `.env` file in the project root and add your respective API keys for model clients:
   ```env
   OPENAI_API_KEY="your_openai_api_key"
   DEEPSEEK_API_KEY="your_deepseek_api_key"
   ALI_API_KEY="your_ali_qwen_api_key"
   UIUI_API_KEY="your_uiui_api_key"
   LOCAL_MODEL_BASE_URL="http://localhost:8081/v1"
   # Add other required API keys as needed
   ```

## Datasets and Evaluation Artifacts

Datasets, source documents, and generated evaluation artifacts are intentionally
excluded from this public repository. Add only data that you are authorized to
share, and keep local inputs in directories covered by `.gitignore`.

## Codebase Modules and Function API Analysis

Below is a detailed breakdown of the functional interfaces provided by each Python script under the `code/` directory.

### Data Ingestion and PDF Parsing
- **`mineru_script.py`**
  - `parse_pdfs_with_mineru(...)`: Automates the parsing of raw PDF papers using the MinerU framework to extract structured output.
- **`pdf_plumber.py`**
  - Extracts tables and complex layouts from literature PDFs using `pdfplumber`.

### Text Post-Processing
- **`post_process_script.py`**
  - `remove_references_section(...)`: Strips reference sections from parsed literature.
  - `remove_image_markup(...)` / `remove_empty_spans(...)`: Cleans up redundant HTML/XML tags.
  - `remove_figure_captions(...)`: Deletes figure captions and standalone labels.
  - `html_table_to_markdown(...)` / `extract_and_convert_tables(...)`: Extracts HTML tables from the parsed text and converts them into standardized Markdown formats.
  - `remove_extra_blank_lines(...)` / `clean_text(...)`: High-level text cleaners that orchestrate these filters.

### Data Deduplication
- **`deduplicate_script.py`**
  - `alpaca_to_text(...)`: Serializes Alpaca format entries into plain text arrays for processing.
  - `simhash_filter(...)` / `similarity_filter(...)`: Computes document similarity and removes heavily overlapping text to ensure high-quality dataset curation.

### Fine-Tuning Dataset Generation
- **`fine_tune_data_script.py`**
  - `validate_alpaca_data_format(...)` / `validate_sharegpt_data_format(...)`: Asserts that supervised fine-tuning (SFT) datasets strictly adhere to Alpaca or ShareGPT instruction schemas.
  - `generate_alpaca_data(...)` / `generate_sharegpt_data(...)`: Builds and outputs `.json` datasets for model fine-tuning.
  - `save_to_file(...)`: Utility to safely serialize generated data.

### Dataset Categorization (Taxonomy)
- **`category_dataset.py`**
  - `generate_category(...)`: General dataset partitioning.
  - `generate_ph_category(...)` / `generate_dataset_by_ph(...)`: Subdivides the dataset based on pH severity.
  - `generate_soil_category(...)` / `generate_dataset_by_soil(...)`: Subdivides based on soil salinity levels and types (soda saline, chloride, etc.).
  - `generate_geology_category(...)` / `generate_dataset_by_geology(...)`: Partitions benchmarks by topological and geographic traits (e.g., coastal, inland, secondary).
  - `generate_treatment_category(...)` / `generate_dataset_by_treatment(...)`: Groups benchmarks by specific treatment measures (agronomic, chemical, biological, engineering).

### Utilities & Helpers
- **`utils.py`**
  - `extract_clean_json(...)`: Robustly parses LLM standard output into sanitized JSON mappings.
  - `process_single_paper(...)` / `process_parsed_papers_multithread(...)`: Multithreaded wrappers for batch-processing scientific papers.
  - `generate_single_delecamation_type(...)` / `generate_declamation_type_multithread(...)`: Parallelized metadata extraction.
  - `transform_coordinates(...)` / `transform_coordinates_multithread(...)`: Transforms and projects geospatial markers for map features.
  - `generate_train_test_split(...)`: Performs standard training and evaluation dataset splits.
  - `count_categories(...)`: Aggregates distribution statistics of the data.
  - `calculate_auto_test_accuracy(...)`: Utility to compute quantitative performance.
- **`plot_script.py`**
  - `autolabel(...)`: UI Helper to build professional comparison charts and visual A/B win rates from the markdown evaluation results.
- **`province_city_script.py`**
  - Scripts for resolving and plotting geospatial hierarchies across provinces and cities involved in the study.

### Setup and Model Clients
- **`model_clients.py`**
  - `get_ali_embedding_model(...)` / `get_local_bge_embedding_model(...)`: Instantiates dense retrieval embedding models.
  - `get_local_bge_reranker_model(...)`: Instantiates cross-encoder reranking models.
  - `get_local_deepseek_model_client(...)`, `get_openai_model_client(...)`, `get_ali_model_client(...)`, `get_deepseek_model_client(...)`: Dedicated client factory constructors for corresponding foundation models (OpenAI, DeepSeek, Qwen/Ali).
  - `get_model_clients(...)` / `get_model_tokenizer(...)`: Retrieves instantiated client pools and language tokenizers.

### Prompts & Configuration
- **`prompts.py`** & **`english_prompts.py`**
  - Statically defined LLM instruction templates, system prompts, and few-shot examples for both English and Chinese evaluation pipelines.

### RAG and Logic Flows
- **`rag_script.py`**
  - `metrics_retrieval(...)` / `treatment_metrics_retrieval(...)`: Custom retrievers fetching contextual soil remediation metrics.
  - `include_all_metrics(...)` / `include_all_treatment_metrics(...)`: Augments generative prompts with retrieved context limits.
  - `call_llm(...)` / `treatment_call_llm(...)`: Executes end-to-end RAG-augmented planning queries.
  - `pretty_print_docs(...)`: Formatting utility for logging retrieval contexts.

### Benchmark Evaluation (Choices & Planning)
- **`test_choices_script.py`**
  - `generate_question(...)` / `create_dataset(...)`: Synthesizes objective multiple-choice questions from the corpus facts.
  - `generate_answer(...)` / `test_dataset(...)`: Evaluates multiple-choice answers of benchmarked LLMs to assess domain-specific factual grounding.
- **`test_plan_script.py`**
  - `generate_answer(...)` / `call_llm(...)`: Base executions of plan generation tasks.
  - `generate_cost(...)` / `generate_treatments(...)`: Simulates cost-budget constraints and potential treatment strategies applied to specific geographic field points.
  - `generate_plan(...)` / `generate_direct_plan(...)`: Aggregates the budget, soil profile, and treatments into a unified LLM agricultural remediation plan.
  - `compare_plans(...)`: Automates A/B pairwise testing via an LLM judge (e.g., `gpt-5`) distinguishing the superior plan.
  - `generate_plans(...)` / `count_comparison_results(...)`: Orchestrates the evaluation matrix and tallies LLM win rates over validations.
- **`test_service_client.py`**
  - Interface to ping and test deployed model or processing REST service endpoints.

### Main Workflow & Visualization
- **`main.py`**
  - `process_parsed_papers(...)` / `process_parsed_english_papers(...)`: The central entry points driving the parsing, extraction, and generation pipeline for final dataset curation.

## License
[MIT License](LICENSE)
