
# FinRobot: Autogen-Powered Financial Analyst Agent for Annual Report Writing

This repository contains an implementation of a multi-agent system using Microsoft's Autogen framework. The system functions as a Financial Analyst Agent capable of analyzing company 10-K reports, performing comparative analysis against competitors, generating visualizations, and compiling a structured annual report in PDF format.

The primary demonstration (`agent_annual_report.ipynb`) showcases the agent generating an annual report for NextEra Energy (NEE) based on its 2024 10-K filing, comparing it against competitors like DUK, CEG, and AEP.

## Features

*   **Automated 10-K Analysis:** Extracts and analyzes key information from financial statements (Income Statement, Balance Sheet, Cash Flow) within 10-K reports.
*   **Business Highlights Summary:** Summarizes key business developments and segment performance.
*   **Competitive Analysis:** Compares the target company's financial metrics (EBITDA Margin, EV/EBITDA, FCF Conversion, Gross Margin, ROIC, Revenue Growth) against specified competitors over multiple years.
*   **Risk Assessment:** Identifies and summarizes key risks mentioned in the report (Note: This feature is planned in the agent's workflow but might require further implementation or specific prompting).
*   **Data Visualization:** Generates charts for Share Performance and PE & EPS trends.
*   **PDF Report Generation:** Compiles the analysis and visualizations into a formatted PDF annual report using the `ReportLab` toolkit.
*   **Autogen Framework:** Leverages Autogen for defining specialized agents and managing conversational workflows.
*   **Nested Chat:** Implements a nested chat mechanism for handling long-context tasks (like detailed report section analysis) efficiently without cluttering the main conversation history.
*   **Customizable Prompts:** The analysis and report generation process is driven by user prompts, allowing for flexibility in specifying companies, competitors, fiscal years, and specific analysis requirements (like word counts per section).
*   **FinRobot Toolkit:** Utilizes a custom `finrobot` toolkit for financial data fetching (via FMP/SEC APIs), analysis functions, charting, and PDF utilities.

## Demo

The core functionality is demonstrated in the [`agent_annual_report.ipynb`](agent_annual_report.ipynb) Jupyter Notebook. This notebook walks through the setup, agent definition, tool registration, and execution of the annual report generation task.

*(Optional: Consider adding a screenshot or GIF of the agent interaction or the final PDF report here)*

## Architecture & How it Works

The system employs a multi-agent architecture built with Autogen:

1.  **Expert_Investor (AssistantAgent):** The primary agent responsible for financial analysis and report writing. It understands the task, plans the execution steps, and utilizes the provided tools. Its role and objectives are defined by a detailed system message.
2.  **Expert_Investor_Shadow (AssistantAgent):** An "inner monologue" or assistant to the Expert\_Investor. It handles specific, long-context tasks like analyzing large sections of the 10-K report when instructed by the Expert\_Investor. This happens within a *nested chat* which is triggered by specific function call results and runs silently to keep the main chat focused.
3.  **User_Proxy (UserProxyAgent):** Manages the conversation flow, executes code (tool calls) requested by the Expert\_Investor, and relays results back to the agent. In the demo, it's set to fully automated (`human_input_mode="NEVER"`).

**Tools Used (via `finrobot` toolkit):**

*   `FMPUtils`: Fetches SEC report URLs and filing dates (requires FMP & SEC API keys).
*   `ReportAnalysisUtils`: Contains functions for analyzing specific financial statements (Balance Sheet, Income Statement, Cash Flow) and business highlights from 10-K reports.
*   `ReportChartUtils`: Generates financial charts (Share Performance, PE & EPS).
*   `ReportLabUtils`: Builds the final formatted PDF annual report.
*   `TextUtils`: Utility functions like checking text length.
*   `IPythonUtils`: Displays generated images within the notebook environment.

## Setup & Installation

**1. Prerequisites:**

*   Python 3.10+
*   API Keys:
    *   OpenAI API Key
    *   Financial Modeling Prep (FMP) API Key
    *   SEC API Key (Note: The notebook implies this is used, likely within the `finrobot` toolkit)

**2. Clone the Repository:**

```bash
git clone https://github.com/your-username/your-repository-name.git
cd your-repository-name


3. Install Dependencies:

pip install -r requirements.txt
IGNORE_WHEN_COPYING_START
content_copy
download
Use code with caution.
Bash
IGNORE_WHEN_COPYING_END

(Ensure you create a requirements.txt file containing necessary packages like pyautogen, requests, matplotlib, reportlab, pymupdf, pillow, python-dotenv)

4. Configure API Keys:

OpenAI:

Rename OAI_CONFIG_LIST_sample to OAI_CONFIG_LIST.

Edit OAI_CONFIG_LIST (it's a JSON file) and add your OpenAI API key and desired model (the demo uses gpt-4-0125-preview).

Example structure:

[
    {
        "model": "gpt-4-0125-preview",
        "api_key": "sk-YOUR_OPENAI_API_KEY"
    }
]
IGNORE_WHEN_COPYING_START
content_copy
download
Use code with caution.
Json
IGNORE_WHEN_COPYING_END

FMP & SEC APIs:

Rename config_api_keys_sample to config_api_keys.

Edit config_api_keys (it's a JSON file) and add your FMP and SEC API keys.

Example structure (based on notebook):

{
    "FMP_API_KEY": "YOUR_FMP_API_KEY",
    "SEC_API_KEY": "YOUR_SEC_API_KEY"
}
IGNORE_WHEN_COPYING_START
content_copy
download
Use code with caution.
Json
IGNORE_WHEN_COPYING_END

5. Create Working Directory:

The notebook uses a directory to store intermediate files (text analyses, images) and the final PDF report. By default, it's set to /content/FinRobot/report (in Colab).

Ensure the specified work_dir in the notebook exists and is writable, or modify the path to a suitable location (e.g., ./report).

mkdir report # Or the directory name you choose
IGNORE_WHEN_COPYING_START
content_copy
download
Use code with caution.
Bash
IGNORE_WHEN_COPYING_END
Usage

Launch Jupyter Notebook or Jupyter Lab.

Open the agent_annual_report.ipynb notebook.

Verify the paths for configuration files (OAI_CONFIG_LIST, config_api_keys) and the work_dir are correct within the notebook cells.

Run the cells sequentially.

The final code cell initiates the agent chat with the predefined task:

company = "NextEra"
competitors = ["DUK","CEG","AEP"]
fyear = "2024"
work_dir = "./report" # Example relative path

task = dedent(
    f"""
    With the tools you've been provided, write an annual report based on {company}'s and{competitors}'s{fyear} 10-k report, format it into a pdf.
    Pay attention to the followings:
    - Explicitly explain your working plan before you kick off.
    - Use tools one by one for clarity, especially when asking for instructions.
    - All your file operations should be done in "{work_dir}".
    - Display any image in the chat once generated.
    - For competitors analysis, strictly follow my prompt and use data only from the financial metics table, do not use similar sentences in other sections, delete similar setence, classify it into either of the two. The last sentence always talks about the Discuss how {company}’s performance over these years and across these metrics might justify or contradict its current market valuation (as reflected in the EV/EBITDA ratio).
    - Each paragraph in the first page(business overview, market position and operating results) should be between 150 and 160 words, each paragraph in the second page(risk assessment and competitors analysis) should be between 500 and 600 words, don't generate the pdf until this is explicitly fulfilled.
"""
)

with Cache.disk() as cache:
    user_proxy.initiate_chat(
        recipient=expert, message=task, max_turns=50, summary_method="last_msg"
    )
IGNORE_WHEN_COPYING_START
content_copy
download
Use code with caution.
Python
IGNORE_WHEN_COPYING_END

Observe the agent interaction in the notebook output. The final PDF report will be saved in the specified work_dir.

(Optional) Modify the company, competitors, fyear, and task variables to generate reports for different entities or with different requirements.

Dependencies

Python Packages:

pyautogen (>=0.2.0)

requests

matplotlib

reportlab

pymupdf

pillow

python-dotenv (optional, for managing environment variables if preferred over JSON config)

See requirements.txt for a full list.

Local Modules:

finrobot toolkit (contains utils, toolkits, functional modules, data source interfaces) - Included in this repository.

Contributing

Contributions are welcome! Please feel free to submit pull requests or open issues for bugs, feature requests, or improvements.

License

This project is licensed under the MIT License.
