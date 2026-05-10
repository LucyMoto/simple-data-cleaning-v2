# LangGraph Workflow Example

A simple example demonstrating how to build workflows with LangGraph for intelligent data cleaning using agentic decision-making.

## How It Works

The workflow follows these steps:

1. **Load Data** - Reads the CSV file into a pandas DataFrame
2. **Summarize Data** - Generates comprehensive summary including:
   - Statistical description (`.describe()`)
   - Dataset info (`.info()`)
   - Explicit missing value counts
   - Outlier detection using IQR method
3. **LLM Reasoning** - Uses GPT-4o-mini to analyze the data summary and intelligently decide which action is most appropriate:
   - clean_missing - Fill missing values with column means
   - remove_outliers - Remove values outside IQR bounds
   - both - Execute both cleaning steps
   - none - No action needed (data is already clean)
4. **Conditional Routing** - Router function directs flow to the appropriate node based on LLM's decision
5. **Data Cleaning** (if needed) - Can execute:
   - **Handle Missing Values** - Fills numeric missing values with column means
   - **Remove Outliers** - Removes values outside IQR bounds (Q1 - 1.5×IQR, Q3 + 1.5×IQR)
   - **Both** - Executes missing value handling first, then outlier removal
6. **Describe Data** - Generates a statistical summary of the cleaned (or original) data
7. **Output Results** - Prints the action taken and final summary

## Key Features

- Intelligent Decision Making: The LLM analyzes data quality issues and decides whether cleaning is necessary
- Outlier Detection: Automatically detects and counts outliers using the industry-standard IQR method
- Conditional Workflow Routing: Dynamically routes to different processing paths based on LLM decision
- Type-Safe State Management: Uses TypedDict for strongly-typed workflow state
- Graph Visualization: Generates a Mermaid diagram of the workflow


## Setup

### Windows (PowerShell)

1. **Verify Python is installed** (Python 3.9 or higher required):
   ```powershell
   python --version
   ```
   If not installed, download from [python.org](https://www.python.org/downloads/)

2. **Install Poetry**:
   ```powershell
   (Invoke-WebRequest -Uri https://install.python-poetry.org -UseBasicParsing).Content | py -
   ```
   After installation, restart your terminal or IDE. If `poetry` command is not found, add `%APPDATA%\Python\Scripts` to your system PATH.

3. **Install dependencies**:
   ```powershell
   poetry install
   ```

4. **Set up your OpenAI API key**:
   ```powershell
   copy .env.example .env
   ```
   Then edit `.env` and add your OpenAI API key: `OPENAI_API_KEY=sk-your-key-here`

### macOS/Linux

1. **Install Poetry** (if not already installed):
   ```bash
   curl -sSL https://install.python-poetry.org | python3 -
   ```

2. **Install dependencies**:
   ```bash
   poetry install
   ```

3. **Set up your OpenAI API key**:
   ```bash
   cp .env.example .env
   ```
   Then edit `.env` and add your OpenAI API key: `OPENAI_API_KEY=sk-your-key-here`

## Running the Example

From the project root:
```bash
poetry run python workflows/simple_clean_data_workflow.py
```

The workflow will:
1. Save a workflow graph visualization to `outputs/workflow_graph.png`
2. Load data from `data/missing.csv` (you can change this in the script)
3. Analyze the data for missing values and outliers. 
4. Use an LLM to decide which cleaning action to take
5. Execute the appropriate cleaning steps (or skip if data is clean)
6. Display the results

## Project Structure

```
.
├── README.md
├── pyproject.toml
├── .env.example
├── .gitignore
│
├── data/
│   ├── missing.csv              # Data with only missing values
│   ├── outliers.csv             # Data with only outliers
│   └── missing_and_outliers.csv # Data with both issues
│
├── workflows/
│   └── simple_clean_data_workflow.py   # Main workflow implementation
│
└── outputs/
    └── .gitkeep                 # Generated files (graphs, reports)
```

**Folder Organization:**
- `data/` - Sample CSV files with different data quality issues
- `workflows/` - LangGraph workflow scripts
- `outputs/` - Generated outputs (visualizations, reports)

## Workflow Architecture

**State Definition**
python
class DataState(TypedDict):
    csv_path: str          # Path to input CSV
    df: pd.DataFrame       # The data being processed
    action: Literal[...]   # LLM's chosen action (clean_missing, remove_outliers, both, none)
    summary: str           # Text summary of data analysis

**Router Function**
The route_action() function maps LLM decisions to workflow nodes:

- "clean_missing" → handle_missing_values node
- "remove_outliers" → remove_outliers node
- "both" → both node (handles both issues)
- "none" → describe_data node (skip cleaning)

All paths eventually converge to describe_data → output_results → END
